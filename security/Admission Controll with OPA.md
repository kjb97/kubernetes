- 참조 : OPA Docs https://www.openpolicyagent.org/docs/latest/kubernetes-tutorial/


# Admission Controll=======

인가(Authentication)된 사용자에 대해서 관리자가 추가적인 제한(validate) 혹은 변경(mutate)하는 작업
Admission Controller는 Admission Controll을 수행하는 주체이고 예를 들면,
LimitRange로 관리자가 정한 리소스를 초과하면(관리자의 정책) pod 생성을 막는다(제한).

Admission Controller는 WebHook으로 API가 열려있고 custom이 가능하다.
이를 Dynamic Admission이라 하고  MutatingWebhook과 ValidatingWebhook이 있다.
 - MutatingWebhook
 사용자 요청과 무관하게 관리자가 임의로 값을 변경하는 작업.
 특정 리소스에 대한 사용자 요청을 관리자가 정한 리소스로 강제할 수 있다.
 
 - ValidatingWebhook
 사용자 요청에 대해 관리자가 제한하는 작업
 예를 들어 사용자가 외부 이미지 registry를 사용하면 해당 요청을 반려 할 수 있다.


# OPA =====================

application에서 인증(Authentication) 이후에 인가(Authorization)에 관한 내용을 코드로 정의하여 보안, 운영, 자동화를 가능하게 한다.
boolean 타입으로 반환하며 권한이 있는가? 만을 판단한다.
즉, 실제로 제한 같은 동작을 수행하는 메커니즘은 없고 단지 판단만 수행하며 실제 동작은 OPA의 판단을 기반으로 플랫폼의 메커니즘을 따른다.
서비스에서 Json으로 질의하고 Rego 언어의 Policy 파일과 Json 데이터로 구성되어 있다.


# kubernetes OPA 
- 흐름
| 사용자가 쿠버네티스로 요청을 보내면
| input이라는 객체를 통해 kubernetes의 AdmissionReview라는 Json 객체가 OPA로 전달된다.
| 해당 Json 객체는 사용자가 요청한 전체 스펙이 정의되어 있다.
| OPA는 관리자가 Rego로 정의한 정책을 이용해 사용자의 요청이 이에 부합하는 지 판단하여 결과를 반환한다.

## 1. OPA install

### 1. namespace 생성
```
kubectl create namespace opa
```

### 2. TLS 구성을 위한 인증서 생성
```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -sha256 -key ca.key -days 100000 -out ca.crt -subj "/CN=admission_ca"  # Certificate Authority의 CN

# Server Certificate을 만들기 위한 설정 파일을 생성. CN이 opa.opa.svc인 것을 확인
cat >server.conf <<EOF
[ req ]
prompt = no
req_extensions = v3_ext
distinguished_name = dn

[ dn ]
CN = opa.opa.svc

[ v3_ext ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, serverAuth
subjectAltName = DNS:opa.opa.svc,DNS:opa.opa.svc.cluster,DNS:opa.opa.svc.cluster.local
EOF

openssl genrsa -out server.key 2048
openssl req -new -key server.key -sha256 -out server.csr -extensions v3_ext -config server.conf
openssl x509 -req -in server.csr -sha256 -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 100000 -extensions v3_ext -extfile server.conf

```

### 3. secret 생성
```
kubectl create secret tls opa-server --cert=server.crt --key=server.key -n opa
```

### 4. OPA 배포

```
# admission-controller.yaml
# Grant OPA/kube-mgmt read-only access to resources. This lets kube-mgmt
# replicate resources into OPA so they can be used in policies.
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: opa-viewer
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:serviceaccounts:opa
  apiGroup: rbac.authorization.k8s.io
---
# Define role for OPA/kube-mgmt to update configmaps with policy status.
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: opa
  name: configmap-modifier
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["update", "patch"]
---
# Grant OPA/kube-mgmt role defined above.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: opa
  name: opa-configmap-modifier
roleRef:
  kind: Role
  name: configmap-modifier
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:serviceaccounts:opa
  apiGroup: rbac.authorization.k8s.io
---
kind: Service
apiVersion: v1
metadata:
  name: opa
  namespace: opa
spec:
  selector:
    app: opa
  ports:
  - name: https
    protocol: TCP
    port: 443
    targetPort: 8443
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: opa
  namespace: opa
  name: opa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: opa
  template:
    metadata:
      labels:
        app: opa
      name: opa
    spec:
      containers:
        # WARNING: OPA is NOT running with an authorization policy configured. This
        # means that clients can read and write policies in OPA. If you are
        # deploying OPA in an insecure environment, be sure to configure
        # authentication and authorization on the daemon. See the Security page for
        # details: https://www.openpolicyagent.org/docs/security.html.
        - name: opa
          image: openpolicyagent/opa:0.47.4-rootless
          args:
            - "run"
            - "--server"
            - "--tls-cert-file=/certs/tls.crt"
            - "--tls-private-key-file=/certs/tls.key"
            - "--addr=0.0.0.0:8443"
            - "--addr=http://127.0.0.1:8181"
            - "--set=bundles.default.resource=bundle.tar.gz"
            - "--log-format=json-pretty"
            - "--set=status.console=true"
            - "--set=decision_logs.console=true"
          volumeMounts:
            - readOnly: true
              mountPath: /certs
              name: opa-server
          readinessProbe:
            httpGet:
              path: /health?plugins&bundle
              scheme: HTTPS
              port: 8443
            initialDelaySeconds: 3
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              scheme: HTTPS
              port: 8443
            initialDelaySeconds: 3
            periodSeconds: 5
        - name: kube-mgmt
          image: openpolicyagent/kube-mgmt:2.0.1
          args:
            - "--replicate-cluster=v1/namespaces"
            - "--replicate=networking.k8s.io/v1/ingresses"
      volumes:
        - name: opa-server
          secret:
            secretName: opa-server
```

```
kubectl apply -f admission-controller.yaml
```


### 5. OPA 제어에서 제외하기
- namespace에 openpolicyagent.org/webhook=ignore을 label한다.

```
kubectl label ns kube-system openpolicyagent.org/webhook=ignore
kubectl label ns opa openpolicyagent.org/webhook=ignore
```


### 6. OPA를 admission controller로 등록
- ValidatingWebhook

```
cat > webhook-configuration.yaml <<EOF
kind: ValidatingWebhookConfiguration
apiVersion: admissionregistration.k8s.io/v1
metadata:
  name: opa-validating-webhook
webhooks:
  - name: validating-webhook.openpolicyagent.org
    namespaceSelector:
      matchExpressions:
      - key: openpolicyagent.org/webhook
        operator: NotIn
        values:
        - ignore
    rules:
      - operations: ["CREATE", "UPDATE"]
        apiGroups: ["*"]
        apiVersions: ["*"]
        resources: ["*"]
    clientConfig:
      caBundle: $(cat ca.crt | base64 | tr -d '\n')
      service:
        namespace: opa
        name: opa
    admissionReviewVersions: ["v1"]
    sideEffects: None
EOF
```


```
kubectl apply -f webhook-configuration.yaml
```

- 여기까지 k8s에서 OPA의 구성이고 이제 정책을 만들어 적용한다.

## 2. OPA 정책 정의
- Rego를 사용 ( 선언형 )
- OPA Docs 예시를 참고하기
- Rego로 정의된 정책을 configmap으로 쿠버네티스의 요청을 OPA가 판단한다.

예) Image 제한 정책
- 이미지 이름이 "test.com/"으로 시작하지 않으면 생성 불가

image-validate.rego 
```
package kubernetes.validating.images

deny[msg] {
    some i
    input.request.kind.kind == "Pod"
    image := input.request.object.spec.containers[i].image
    not startswith(image, "test.com/")
    msg := sprintf("Image '%v' comes from untrusted registry", [image])
}
```
