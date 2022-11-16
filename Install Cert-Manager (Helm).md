# Install Cert-Manager Using Helm
## 1. install using helm
- helm chart를 이용해 k8s 클러스터에 배포
```
helm repo add jetstack https://charts.jetstack.io

helm repo update

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.0/cert-manager.crds.yaml

helm pull jetstack/cert-manager --version v1.10.0 --untar

helm upgrage --install  \
 cert-manager . \
 --namespace cert-manager \
 --create-namespace \
-f values.yaml
```

## 2. create clusterIssuer 
- staging과 prod 중 선택
- cert-manager api를 이용해 clusterissuer 커스텀 리소스 생성
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: kkk@kkk.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging
    # Enable the HTTP-01 challenge provider
    solvers:
    # An empty 'selector' means that this solver matches all domains
    - selector: {}
      http01:
        ingress:
          class: nginx

---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: kkk@kkk.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
```

## 3. ingress 적용
- 생성한 clusterissuer를 이용한 인증서 적용
- tls를 정의하면 자동으로 secret이 생성
```
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: keycloak
  namespace: keycloak
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    ingress.kubernetes.io/ssl-redirect: 'true'
    kubernetes.io/tls-acme: 'true'
    kubesphere.io/creator: admin
    nginx.ingress.kubernetes.io/proxy-body-size: '0'
    nginx.ingress.kubernetes.io/ssl-redirect: 'true'
spec:
  tls:
    - hosts:
        - keycloak.keycloak.gantry.ai
      secretName: tls-keycloak
  rules:
    - host: keycloak.keycloak.gantry.ai
      http:
        paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
              service:
                name: keycloak-http
                port:
                  number: 80
```
