# NFS server 및 nfs provisoner 설치

## 1. server
- storage로 사용될 서버

```
sudo apt-get update
```
-   NFS 서버를 위한 패키지를 설치
```
$ sudo apt-get install nfs-common nfs-kernel-server portmap

```
-   NFS에서 사용될 디렉토리 생성 및 권한 부여

```
$ sudo mkdir -p /home/share/nfs
$ sudo chmod 777 /home/share/nfs

```

-   공유 디렉토리 설정

```
$ sudo vi /etc/exports
## 형식 : [/공유디렉토리] [접근IP] [옵션]
/home/share/nfs *(rw,no_root_squash,async)
```

> rw - 읽기쓰기  
> no_root_squash - 클라이언트가 root 권한 획득 가능, 파일생성 시 클라이언트 권한으로 생성  
> async - 요청에 의해 변경되기 전에 요청에 응답, 성능 향상용

-   nfs 서버 재시작
```
$ sudo /etc/init.d/nfs-kernel-server restart
$ sudo systemctl restart portmap

```

2.2. 동작 확인

-   설정 확인

```
$ sudo exportfs -v

```

-   정상 결과

```
/home/share/nfs
                <world>(rw,async,wdelay,no_root_squash,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```


## 2. client
- 만든 nfs server를 k8s 클러스터에서 사용하는 방법
- kubernetes에서 helm을 이용해 nfs-provisioner 설치

1. helm chart 가져오기
```
helm repo add kvaps https://kvaps.github.io/charts
helm repo update
helm pull kvaps/nfs-server-provisioner --untar
```
2. helm 설치
```
helm upgrade --install --kubeconfig=$KUBE_CONFIG  nfs-provisioner . --set nfs.server=10.xxx.xxx.xxx --set nfs.path=/home/share/nfs -n nfs -f values.yaml
```
3. default storageClass 설정
```
kubectl patch storageclass nfs -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
