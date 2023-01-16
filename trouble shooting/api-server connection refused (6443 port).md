# Kubernetes api-server connection was refused
- 기본적인 해결법    

이런 문구와 함께 6443포트로 연결할 수 없을 때 발생하는 에러  
```
The connection to the server 192.168.28.166:6443 was refused - did you specify the right host or port?
```    

여러가지 원인이 있을 수 있지만 핵심은 kube-apiserver와 연결할 수 없다는 뜻.  
가장 많이 본 원인은  
- kubelet inactive  
- kube-apiserver 컨테이너 not running  
- kubeconfig 파일 없거나 잘못됨    

## kubelet inactive
```
systemctl status kubelet
systemctl restart kubelet
```  
- restart 자체가 안된다면 로그를 따라 트러블슈팅이 필요 ( swap을 해제하지 않았을 경우가 많았음 )    

## kube-apiserver 컨테이너 not running  
- cri에 맞게 확인
```
crictl ps | grep apiserver
```    
- apiserver나 다른 kube-system의 pods들은 보통 static pods이기 때문에  
 그냥 동작하지 않는 건 노드 자체 문제거나 다른 이유가 있을 수 있기 때문에  
 에러를 보며 최대한 짐작해야 함.  
 ( 최후의 방법은 클러스터 초기화 후, 다시 생성 )

## kubeconfig 파일 없거나 잘못됨

```
# 일반 user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# root user
export KUBECONFIG=/etc/kubernetes/admin.conf
```    


### 클러스터 초기화 방법
- kubeadm의 예시  
- rke, kubespray 등 다른 클러스터들은 방법이 다를 수 있다.

```
$ kubeadm reset

~~~

[reset] Deleting contents of directories: [/etc/kubernetes/manifests /var/lib/kubelet /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
~~~

# cni, 클러스터관련 설정파일 삭제 
$ sudo rm -r /etc/cni/net.d/*
$ sudo rm -r /etc/kubernetes/*
```