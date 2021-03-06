# AirGap RKE2 설치 (tarball)
## 사전 준비 ( 모든 노드 준비 사항 )
##### 설치에 필요한 파일을 미리 준비

- rke2-images.linux-amd64.tar.zst
- rke2.linux-amd64.tar.gz
- sha256sum-amd64.txt

RKE2 Release (https://github.com/rancher/rke2/releases)
```
curl -sfL https://get.rke2.io --output install.sh
 ```

- Rancher Server 설치 대상 모든 Node 환경에 접속하여 Swap 비활성화, Network 브릿지 설정
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
- 설치 진행할 디렉토리로 설치 파일들 복사
```
sudo mkdir /root/rke2-artifacts
sudo cp rke2-images.linux-amd64.tar.zst /root/rke2-artifacts
sudo cp rke2.linux-amd64.tar.gz /root/rke2-artifacts
sudo cp sha256sum-amd64.txt /root/rke2-artifacts
```
- 준비한 파일 디렉토리에 스크립트 실행
```
INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
```

## 1. Master Node

- config.yaml 작성
```
mkdir -p /etc/rancher/rke2
cat << EOF >>  /etc/rancher/rke2/config.yaml
write-kubeconfig-mode: "0644"
profile: "cis-1.5"  #설정하게 되면 파드가 루트 권한을 가질 수 없게 설정 ( 파드 동작함에 있어서 권한 문제를 겪게 될 수 있다. )
selinux: false
EOF
```
- CIS mode enable selinux 설정
- 설정하게 되면 파드가 루트 권한을 가질 수 없게 설정 ( 파드 동작함에 있어서 권한 문제를 겪게 될 수 있다. )
```
$ sudo cp -f /usr/local/share/rke2/rke2-cis-sysctl.conf /etc/sysctl.d/60-rke2-cis.conf
$ sysctl -p /etc/sysctl.d/60-rke2-cis.conf
$ useradd -r -c "etcd user" -s /sbin/nologin -M etcd
```

- 서비스 활성화
```
systemctl enable rke2-server.service
systemctl start rke2-server.service
```
- setting
```
export  PATH=$PATH:/var/lib/rancher/rke2/bin/
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export CONTAINERD_ADDRESS=/run/k3s/containerd/containerd.sock
```

## 2. Agent Node
- config.yaml 작성
```
$ mkdir -p /etc/rancher/rke2
$ sudo cat << EOF >>  /etc/rancher/rke2/config.yaml
write-kubeconfig-mode: "0644"
server:  https://master 노드 ip:9345
# Token 값은 첫번 째 Node의 /var/lib/rancher/rke2/server/node-token 디렉토리 참조
token:  K106b9afcb136aa3a088e508882ad4fa1f94b9d814f36cd7c85b8a5c87643510d16::server:ca24fd2bebf3d41ec7c180cceb3b2768 
profile: "cis-1.5"
selinux: false
EOF
```

- 서비스 활성화
```
systemctl enable rke2-agent.service
systemctl start rke2-agent.service
```
