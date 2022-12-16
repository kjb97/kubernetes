# Air-Gap gitlab install (helm).md
## 1. Install 
### 1.1 gitlab을 설치할  namespace 생성
```
kubectl create namespace gitlab
```

### 1.2 helm repo에 gitlab repo추가 및 pull
```
helm repo add gitlab https://charts.gitlab.io
helm pull gitlab/gitlab --untar
```

### 1.3 tls 인증서 생성
```
kubectl create -n gitlab secret tls custom-tls --key custom.key --cert custom.crt
```

### 1.4 values.yaml 수정
#### persistence 설정 ( persistent-volume.yaml )
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
#### image 설정 (setting-images.yaml)
- 폐쇄망은 prive registry를 사용하기 때문에 일일이 이미지를 설정해줘야 함
```
cert-manager:
  image:
    repository: quay.io/jetstack/cert-manager-controller
  webhook:
    image:
      repository: quay.io/jetstack/cert-manager-webhook

gitlab:
  prometheus:
    server:
      image:
        repository: harbor.xxxx/gitlab/quay.io/prometheus/prometheus
        tag: v2.31.1
        pullPolicy: IfNotPresent
  kas:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-kas
      tag: v15.1.0
      pullPolicy: IfNotPresent
  gitlab-shell:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-shell
      pullPolicy: IfNotPresent
      tag: v14.7.4
  gitaly:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitaly
      tag: v15.1.0
      pullPolicy: IfNotPresent
    init:
      image:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/alpine-certificates
        tag: 20191127-r2
  gitlab-exporter:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-exporter
      tag: 11.16.0
      pullPolicy: IfNotPresent
  migrations:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-toolbox-ce
      tag: v15.0.3
      pullPolicy: IfNotPresent
    init:
      image:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/alpine-certificates
        tag: 20191127-r2
  sidekiq:
    init:
      image:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-sidekiq-ce
        tag: v15.0.3
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-sidekiq-ce
      tag: v15.0.3
  webservice:
    init:
      dependencies:
        image:
          repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-webservice-ce
          tag: v15.0.3
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-webservice-ce
      tag: v15.0.3
    workhorse:
      image: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-workhorse-ce

  global:
    communityImages:
      migrations:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-toolbox-ce
      sidekiq:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-sidekiq-ce
      toolbox:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-toolbox-ce
      webservice:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-webservice-ce
      workhorse:
        repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-workhorse-ce



gitlab-runner:
  gitlabUrl: http://gitlab.jinseong.leedh.xyz
  image: harbor.xxxx/gitlab/gitlab/gitlab-runner:alpine-v15.0.0

grafana:
  image:
    repository: grafana/grafana
    tag: 7.5.5

minio:
  image: harbor.xxxx/gitlab/minio/minio
  imageTag: RELEASE.2017-12-28T01-21-00Z
  minioMc:
    image: harbor.xxxx/gitlab/minio/mc
    tag: RELEASE.2018-07-13T00-53-22Z

postgresql:
  image:
    registry: harbor.xxxx
    repository: gitlab/bitnami/postgresql
    tag: 12.11.0-debian-11-r13
  metrics:
    image:
      registry: harbor.xxxx
      repository: gitlab/bitnami/postgres-exporter
      tag: 0.8.0-debian-10-r99
      pullPolicy: IfNotPresent

redis:
  image:
    registry: harbor.xxxx
    repository: gitlab/bitnami/redis
    tag: 6.0.9-debian-10-r0
  metrics:
    image:
      registry: harbor.xxxx
      repository: gitlab/bitnami/redis-exporter
      tag: 1.12.1-debian-10-r11
      pullPolicy: IfNotPresent

registry:
  image:
    repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/gitlab-container-registry
    tag: v3.48.0-gitlab

prometheus:
  server:
    image:
      repository: harbor.xxxx/gitlab/quay.io/prometheus/prometheus
      tag: v2.31.1
      pullPolicy: IfNotPresent
  configmapReload:
    prometheus:
      image:
        repository: harbor.xxxx/gitlab/jimmidyson/configmap-reload
        tag: v0.5.0
        pullPolicy: IfNotPresent

global:
  edition: ce
  hosts:
    domain: xxxx
    gitlab:
      name: gitlab.xxxx
      https: true
    registry:
      name: registry.xxxx
      https: true
    minio:
      name: minio.xxxx
      https: true
  ingress:
    configureCertmanager: false
    class: "nginx"
    tls:
      enable: false
  certificates:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/alpine-certificates
      tag: 20191127-r2@sha256:56d3c0dbd1d425f24b21f38cb8d68864ca2dd1a3acc28b65d0be2c2197819a6a
      pullPolicy: IfNotPresent
  kubectl:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/build/cng/kubectl
      tag: 1.18.20@sha256:aebdfcf7bde7b80ad5eef7a472d1128542c977dc99b50c3e471fc98afbb9f52c
      pullPolicy: IfNotPresent
  busybox:
    image:
      repository: harbor.xxxx/gitlab/registry.gitlab.com/gitlab-org/cloud-native/mirror/images/busybox
      tag: latest
      pullPolicy: IfNotPresent

upgradeCheck:
  enabled: true
  image:
    repository:
    tag:
    pullPolicy: IfNotPresent

shared-secrets:
  selfsign:
    image:
      pullPolicy: IfNotPresent
      repository:
      tag:

certmanager:
  install: false

nginx-ingress:
  enabled: false

```

### 1.5 배포 
- helm upgrade
```
helm upgrade --install gitlab . --namespace gitlab  \
--set gitlab-runner.install=true \
--set certmanager.install=false \
--set global.ingress.configureCertmanager=false \
--set nginx-ingress.enabled=false \
--set global.ingress.tls.secretName=custom-tls \
--set global.certificates.customCAs[0].secret=custom-tls \
-f values.yaml,setting-values.yaml
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
