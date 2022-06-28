# Argo-CD offline install 
## 1. 외부 서버에서 필요 이미지 및 helm Chart 받기
```
 docker pull quay.io/argoproj/argocd:v2.4.0
 docker pull ghcr.io/dexidp/dex:v2.30.2
 docker pull redis:7.0.0-alpine
```
```
helm repo add argocd https://argoproj.github.io/argo-helm
helm repo update
helm search repo argocd
helm pull argocd/argo-cd
```
- 파일들 폐쇄망으로 옮김

## 2. values.yaml 
- private resistry로 이미지 load, tag, push한 다음 values.yaml에 이미지들 명시 
### setting-images.yaml 
```
global:
  image:
    repository: 10.250.xxx:5000
    tag: ""

dex:
  initImage:
    repository: 10.250.xxx:5000/quay.io/argoproj/argocd
    tag: v2.4.0
  image:
    repository: 10.250.xxx:5000/ghcr.io/dexidp/dex
    tag: v2.30.2

controller:
  image:
    repository: 10.250.xxx:5000/quay.io/argoproj/argocd
    tag: v2.4.0

applicationSet:
  image:
    repository: 10.250.xxx:5000/quay.io/argoproj/argocd
    tag: v2.4.0

repoServer:
  image:
    repository: 10.250.xxx:5000/quay.io/argoproj/argocd
    tag: v2.4.0

server:
  image:
    repository: 10.250.xxx:5000/quay.io/argoproj/argocd
    tag: v2.4.0

redis:
  image:
    repository: 10.250.xxx:5000/redis
    tag: 7.0.0-alpine

notifications:
  image:
    repository: 10.250.xxx:5000/quay.io/argoproj/argocd
    tag: v2.4.0
```
- 배포
```
helm upgrade --install argocd . --namespace argo --set controller.service.type=NodePort -f values.yaml,setting-images.yaml
