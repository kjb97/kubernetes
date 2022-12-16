# Air-rGap install using helm ( guide )
- 폐쇄망 환경에서 helm 차트를 이용한 설치 방법입니다.
- 기본적으로 설치에 필요한 모든 파일은 외부 연결이 가능한 곳에서 받은 다음 폐쇄망으로 옮겨야 합니다.

## 1. helm 설치 ( binary ) 
###  Release 확인 및 다운
- [Release](https://github.com/helm/helm/releases)

```
wget https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
```

### 압축 해제

-   폐쇄망으로 helm-v3.8.2-linux-amd64.tar.gz 옮기고 압축해제

```
tar -zxvf helm-v3.8.2-linux-amd64.tar.gz
```

###  helm binary 파일 이동
```
mv linux-amd64/helm /usr/local/bin/
```
- 설치 끝


## 2. helm 차트 다운
- helm이 설치된 external 서버에서 설치하고자 하는 애플리케이션의 helm 차트를 받아야 합니다. ( 예시로 nginx 사용하겠습니다. )
- helm 차트 정보는 보통 https://artifacthub.io/ 를 이용합니다.
> helm chart 버전과 app version은 다르기 때문에 잘 확인해야 합니다.  
> 예를 들어 nginx:1.23.2을 설치하려면 해당하는 helm chart 버전은 13.2.11 입니다.

```
# 차트 repository 추가
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

...
Update Complete. ⎈Happy Helming!⎈
...

# repository에서 nginx 차트 검색
helm search repo bitnami | grep nginx
...
bitnami/nginx                                   13.2.11         1.23.2          NGINX Open Source is a web server that can be a...
bitnami/nginx-ingress-controller                9.3.18          1.4.0           NGINX Ingress Controller is an Ingress controll...
bitnami/nginx-intel                             2.1.9           0.4.8           NGINX Open Source for Intel is a lightweight se...
...

# helm 차트 다운로드
# --untar 옵션이 없으면 일반 tar 압축파일로 받습니다.
helm pull bitnami/nginx --untar
```

## 3. values customize
- helm 차트는 기본적으로 아래의 구성을 가집니다.
```
charts  Chart.yaml  README.md  templates  values.yaml
```
- 실제 설치되는 기본 yaml 파일들은 templates에 있고 해당 동적 값들의 설정은 변수로 values.yaml을 통해 전달됩니다. 
- values.yaml을 직접 수정하면 history 파악이 어렵기 때문에 보통은 values-setting.yaml 같이 yaml 파일을 따로 만들어서 사용합니다.
- 간단한 예시로 image registry를 설정하겠습니다.
- 세부적인 변수들에 대해서는 공식 문서 또는 https://artifacthub.io/ 를 이용합니다.

values-setting.yaml 
```
#values.yaml을 참고합니다.
vi values-setting.yaml 
...
global:
  imageRegistry: "example.com"
...
```

## 4. 배포
- 커스텀이 끝나면 해당 yaml 파일을 이용해서 apply합니다.
- values.yaml, values-setting.yaml을 모두 이용합니다.
  > 더 오른쪽에 명시한 파일 값을 우선합니다.
```
helm upgade --install example-nginx . -n default -f values.yaml,values-setting.yaml

...
NAME: example-nginx
LAST DEPLOYED: Mon Oct 24 13:11:47 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 13.2.11
APP VERSION: 1.23.2
...
```
## 5. 확인

```
# helm list
helm list -A
...
NAME                    NAMESPACE                       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
example-nginx           default                         1               2022-10-24 13:11:47.208256553 +0900 KST deployed        nginx-13.2.11                   1.23.2
...


# k8s resources
kubectl get all -n default
...
NAME                                READY   STATUS    RESTARTS   AGE
pod/example-nginx-95f9c6b8b-hw7lz   1/1     Running   0          9m30s

NAME                    TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
service/example-nginx   LoadBalancer   10.43.33.3   <pending>     80:31577/TCP   9m30s
service/kubernetes      ClusterIP      10.43.0.1    <none>        443/TCP        2d20h

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/example-nginx   1/1     1            1           9m30s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/example-nginx-95f9c6b8b   1         1         1       9m30s

```
