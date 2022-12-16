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
    repository: harbor.xxx.xxx.xyz/argo/quay.io/argoproj/argocd
    tag: v2.4.0

dex:
  image:
    repository: harbor.xxx.xxx.xyz/argo/ghcr.io/dexidp/dex
    tag: v2.30.2


redis:
  image:
    repository: harbor.xxx.xxx.xyz/argo/redis
    tag: 7.0.0-alpine

```
- 배포
```
helm upgrade --install argocd . --namespace argo --set controller.service.type=NodePort -f values.yaml,setting-images.yaml
