# RKE Install
## 사전 준비
- docker가 모든 노드에 설치되어야 하고 계정 권한 설정이 필요함
```
#centos 기준
yum install -y yum-utils yum-config-manager \ --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo

yum install docker-ce docker-ce-cli containerd.io

systemctl start docker 
systemctl enable docker


sudo groupadd docker
sudo usermod -aG docker $USER
sudo service docker restart
사용자 다시 로그인

```
- swap off, network bridge 설정
```
swapoff -a
sudo sed -i.bak '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

cat <<EOF | tee /etc/sysctl.d/k8s.conf \ 
net.bridge.bridge-nf-call-ip6tables = 1 \ 
net.bridge.bridge-nf-call-iptables = 1 \
net.ipv4.ip_forward = 1 EOF

sysctl --system
```
- 방화벽 disable 또는 port 열기
```
# disable
systemctl stop firewalld
systemctl disable firewalld

# port 개방
for i in 22 80 443 2376 2379 2380 6443 8472 9099 10250 10251 10252 10254 30000-32767; do firewall-cmd --add-port=${i}/tcp --permanent done 
firewall-cmd --reload
```
-  SSH key가 필요
```
# 온프레미스인 경우 직접 생성
ssh-keygen
```
## RKE Install
### binary download
- 필요한 릴리즈 다운로드 [releases](https://github.com/rancher/rke/releases)
```
# binary download 및 설정
wget https://github.com/rancher/rke/releases/download/v1.2.9/rke_linux-amd64
mv rke_linux-amd64 rke
chmod +x rke
mv rke /usr/bin/rke
```
### cluster.yml 생성
```
# rke cluster.yml 설정 ( [] 안의 값이 현재 값이고 하나씩 원하는 데로 설정 )
rke config

[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]:

[+] Number of Hosts [1]:

[+] SSH Address of host (1) [none]: 192.168.253.140

[+] SSH Port of host (1) [22]:

[+] SSH Private Key Path of host (192.168.253.140) [none]: ~/.ssh/id_rsa

[+] SSH User of host (192.168.253.140) [ubuntu]: centos

[+] Is host (192.168.253.140) a Control Plane host (y/n)? [y]:

[+] Is host (192.168.253.140) a Worker host (y/n)? [n]: y

[+] Is host (192.168.253.140) an etcd host (y/n)? [n]: y

[+] Override Hostname of host (192.168.253.140) [none]:

[+] Internal IP of host (192.168.253.140) [none]: 192.168.253.140

[+] Docker socket path on host (192.168.253.140) [/var/run/docker.sock]:

[+] Network Plugin Type (flannel, calico, weave, canal) [canal]: calico

[+] Authentication Strategy [x509]:

[+] Authorization Mode (rbac, none) [rbac]:

[+] Kubernetes Docker image [rancher/hyperkube:v1.18.3-rancher2]:

[+] Cluster domain [cluster.local]:

[+] Service Cluster IP Range [10.43.0.0/16]:

[+] Enable PodSecurityPolicy [n]:

[+] Cluster Network CIDR [10.42.0.0/16]:

[+] Cluster DNS Service IP [10.43.0.10]:

[+] Add addon manifest URLs or YAML files [no]:
```
### RKE install
- 위에서 생성된 cluster.yml을 이용해서 install 된다.
```
rke up
```

### kubectl 설치
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl 

# 실행권한 추가 
chmod +x kubectl 

# binary 추가
mv kubectl /usr/bin/kubectl

sudo mkdir -p $HOME/.kube 
sudo cp kube_config_cluster.yml $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# 확인
kubectl get nodes
```
