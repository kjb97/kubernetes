# kubernetes gitlab 설치 ( helm chart )
## 1. Install 
### 1.1 gitlab을 설치할  namespace 생성
```
kubectl create namespace gitlab
```

### 1.2 helm repo에 gitlab repo추가 및 pull
```
helm repo add gitlab https://charts.gitlab.io
helm pull gitlab/gitlab-runner --untar
```

### 1.3 tls 인증서 생성
```
kubectl create -n gitlab secret tls custom-ca --key eks.xxx.xyz.key --cert eks.xxx.xyz.crt
```

### 1.4 values.yaml 수정 ( values 파일 생성 )
- persistence 설정 ( persistent-volume.yaml )
- accessMode, storageClass 설정
```
grafana:
  persistence:
    storageClassName: ceph-filesystem
    accessModes:
      - ReadWriteMany

minio:
  persistence:
    accessMode: ReadWriteMany
    storageClass: ceph-filesystem

postgresql:
  persistence:
    accessModes:
      - ReadWriteMany
    storageClass: ceph-filesystem

prometheus:
  server:
    persistentVolume:
      storageClass: ceph-filesystem
      accessModes:
        - ReadWriteMany

pushgateway:
  persistentVolume:
    storageClassName: ceph-filesystem
    accessModes:
      - ReadWriteMany

alertmanager:
  persistentVolume:
    storageClassName: ceph-filesystem
    accessModes:
      - ReadWriteMany

redis:
  master:
    persistence:
      accessModes:
        - ReadWriteMany
      storageClass: ceph-filesystem
```

### 1.5 배포 
- helm upgrade
```
helm upgrade --install gitlab gitlab/gitlab \
--namespace=gitlab \
--set gitlab-runner.install=true \
--set certmanager.install=false \
--set nginx-ingress.enabled=false \
--set global.ingress.configureCertmanager=false \
--set global.ingress.tls.secretName=custom-ca \
--set gitlab.gitlab-runner.certsSecretName="gitlab-runner-certs" \
--set gitlab-runner.certsSecretName="gitlab-runner-certs" \
--set gitlab-runner.runners.cache.cacheShared=true \
--set gitlab-runner.runners.cache.secretName=gitlab-minio-secret \
--set gitlab-runner.runners.cache.s3CachePath=runner-cache \
--set gitlab.gitlab-runner.certsSecretName="gitlab-runner-cert" \
--set gitlab.gitaly.persistence.storageClass=ceph-filesystem \
--set gitlab.gitaly.persistence.accessMode="ReadWriteMany" \
--set global.certificates.customCAs[0].secret=custom-ca \
--set prometheus.server.persistentVolume.storageClass=ceph-filesystem \
-f values.yaml,persistent-volume.yaml
```
- gitlab runner에서 gitlab-runner-certs 못찾는다는ㅇ ㅔ러 발생시 :
  ca 파일을 이용해 아래 명령어로 secret 생성
```
kubectl create secret generic gitlab-runner-certs -n gitlab \
--from-file=ca.crt
```
### 1.6 gitlab 외부 접근을 위한 Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gitlab-ingress
  namespace: gitlab
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: "gitlab.xxx.xyz"
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: gitlab-webservice-default
            port:
              number: 8181

  tls:
  - hosts:
    - gitlab.xxx.xyz
    secretName: custom-ca

```

- gitlab install command
[enter link description here](https://docs.gitlab.com/charts/installation/command-line-options.html#rbac-settings)


## 2. 확인
### 2.1 gitlab 상태 확인
```
kubectl get all -n gitlab
```
### 2.2 gitlab 접속을 위한 초기 비밀번호 생성
```
kubectl get secret gitlab-gitlab-initial-root-password -n gitlab -ojsonpath='{.data.password}' | base64 --decode ; echo
```
### 2.3 ui 접속 확인
- gitlab.xxx.xyz로 접근
- id: root, password:  생성한 비밀번호 
- 
### 2.4 Gitlab Runner 설정 변경
- Gitlab Runner 설정 변경, Cert Manager로 생성 된 인증서가 아닌 Self Sign 인증서를 사용 할 경우 Gitlab Runner에서 아래와 같은 Error Meassge가 발생
```
Couldn't execute POST against https://xxx.com/api/v4/jobs/request: Post https://hostname.tld/api/v4/jobs/request: x509: certificate signed by unknown authority
```
- 위와 같은 에러 발생 시 Runner 아래 설정을 추가
- CA 파일 Secret 생성 (이전 Harbor 설치 시 사용한 CA)
```
$ kubectl create secret generic gitlab-runner-certs \
--from-file=ca.crt
```
- Runner Deployment Yaml File에 gitlab-runner-certs 인증서 Mount와 환경 변수 설정 후 Redeploy
```
volumeMounts:
- mountPath: /etc/gitlab-runner/certs
  name: gitlab-runner-certs
volumes:
- name: gitlab-runner-certs
   secret:
     defaultMode: 438
     secretName: gitlab-runner-certs
env:
- name: CI_SERVER_TLS_CA_FILE
  value: /etc/gitlab-runner/certs/xxx.xxx.leedh.cloud
```
- 위 설정 완료 후 Runner 실행 시 /etc/gitlab-runner/certs/xxx.xxx.leedh.cloud 파일을 읽어 오다 Permission Denie 에러가 발생 할 경우 아래 설정을 0, 0으로 변경하여 Pod Security 변경
```
securityContext:
  fsGroup: 0
  runAsUser: 0
```
