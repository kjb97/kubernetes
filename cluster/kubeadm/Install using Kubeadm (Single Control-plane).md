# **Kubernetes Cluster Construction Using Kubeadm (Single Control-Plane)**
 - ubuntu 20.04
 - 단일 컨트롤플레인 클러스터

 
# **Master Node**
 ## 1. Swap memory off
 ```
 sudo swapoff -a && sudo sed -i '/swap/s/^/#/' /etc/fstab
 ```

 ## 2. Container Runtime 설치
 - 1.24부터 docker를 사용하지 않음.
 - containerd 설치

```
# Configure persistent loading of modules
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

# Load at runtime
sudo modprobe overlay
sudo modprobe br_netfilter

# Ensure sysctl params are set
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload configs
sudo sysctl --system

# Install required packages
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

# Add Docker repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install containerd
sudo apt update
sudo apt install -y containerd.io

# Configure containerd and start service
sudo su -
mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml

# restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status  containerd
```    
    
| systemmd cgroud driver 설정
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```    
    
| 추가로 endpoint 설정을 하지 않으면 docker 같이 다른 socket을 찾으려 할 수 있다.
```
crictl ps

WARN[0000] runtime connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
ERRO[0000] unable to determine runtime API version: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"
...
```

# crictl enpoint 설정
```
vi /etc/crictl.yaml

~~~

runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 5
debug: true

~~~
```



## 3. kubeadm, kubectl, kubelet 설치
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl


# google cloud public 키 등록 및 kubeadm, kubectl, kubelet 설치
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg     # 에러날 때가 있음
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
또는
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt -y install vim git curl wget kubelet kubeadm kubectl


```

## 4. kube-system 이미지 pull
- containerd를 사용하기 때문에 --cri-socket 옵션 추가

```
sudo kubeadm config images pull --cri-socket unix:///run/containerd/containerd.sock
```


## 5. kubeadm init
- 컨트롤플레인 노드 초기화
- --pod-network-cidr은 cni plugin 설치를 위해 설정. calico를 설치할 것이기 때문에 192.168.0.0/16으로 지정.
- --cri-socket 으로 containerd.sock 지정
- --control-plane-endpoint 를 이용한 클러스터 endpoint 설정. 추후에 HA 설정을 위한 LB를 추가할 것이라면 반드시 해당 옵션을 지정해야 한다.
```
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --cri-socket unix:///run/containerd/containerd.sock \
  --upload-certs \
  --control-plane-endpoint=192.168.xx.xx

~~~

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.28.166:6443 --token u80qnm.e8e9mextvduwz71l \
        --discovery-token-ca-cert-hash sha256:ab65f9166521ab82db9a6bee05316bfd179a319f88bc86fb8c463d0726eb8101 \
        --control-plane --certificate-key e306a05027a17c291ec57a4bbb91670db0469614bff41f9a13b72e75eda4ddc7

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.28.166:6443 --token u80qnm.e8e9mextvduwz71l \
        --discovery-token-ca-cert-hash sha256:ab65f9166521ab82db9a6bee05316bfd179a319f88bc86fb8c463d0726eb8101

~~~  
```
## 6. kubeconfig 지정
```
# 일반 user
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

# root user
  export KUBECONFIG=/etc/kubernetes/admin.conf

```

## 7. CNI 설치
- calico

```
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml

# 마스터 노드 taint 제거
kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-
```

# **Worker 노드 추가**
- kubeadm, kubelet, cri (containerd)를 설치.
- master에서 kubeadm init 하고 나온 worker node join 명령을 실행한다.
- kubeadm init 당시에 생성된 토큰은 2시간의 유효기간을 가지고 이후에는 따로 생성해야 한다.
```
kubeadm join 192.168.28.166:6443 --token u80qnm.e8e9mextvduwz71l \
        --discovery-token-ca-cert-hash sha256:ab65f9166521ab82db9a6bee05316bfd179a319f88bc86fb8c463d0726eb8101
```

