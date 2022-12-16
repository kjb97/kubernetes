
# Air-Gap ingress-nginx install(helm)
## 1. 외부에서 차트 받기
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm search repo ingress-nginx/ingress-nginx --versions
helm pull ingress-nginx/ingress-nginx --version=4.1.4 --untar
```
## 2. 이미지 받기
```
docker pull registry.k8s.io/ingress-nginx/controller:v1.2.1
docker save -o nginx-controller.tar registry.k8s.io/ingress-nginx/controller
```
- 내부망으로 1,2번 파일들 옮기고 private registry에 올림
## 3. 설치할 내부망에 설치하기
```
helm upgrade --install ingress-nginx . --namespace nginx-ingress --set controller.service.type=NodePort -f values.yaml
```




## 오류 PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.capabilities.add: Invalid value: "NET_BIND_SERVICE": capability may not be added spec.containers[0].securityContext.allowPrivilegeEscalation: Invalid value: true: Allowing privilege escalation for containers is not allowed]
### ClusterRole
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: ingress-psp-clusterrole
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - privileged
  verbs:
  - use
 ```
 ###  RoleBinding 
 ```
kubectl -n nginx-ingress create rolebinding ingress-psp-clusterrole-rolebinding --clusterrole=ingress-psp-clusterrole --group=system:serviceaccounts:nginx-ingress
 ```
### values.yaml
```
~
podSecurityPolicy:
  enabled: true
~
```
