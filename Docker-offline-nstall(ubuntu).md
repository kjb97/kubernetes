# Docker Offline Install (ubuntu)
### 1. Release 확인 및 다운
[deb packages](https://download.docker.com/linux/ubuntu/dists/)

```
wget http://security.ubuntu.com/ubuntu/pool/main//libs/libseccomp/libseccomp2_2.5.4-1ubuntu1_amd64.deb
wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/containerd.io_1.6.4-1_amd64.deb
wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-cli_20.10.16~3-0~ubuntu-focal_amd64.deb
wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce_20.10.16~3-0~ubuntu-focal_amd64.deb
```
- libsecomp의 버전이 낮으면 설치할 때 dependency problems이 발생할 수 있다.

### 2. 폐쇄망으로 파일을 옮기고 dpkg로 설치
- 순서대로 설치하지 않으면 dependency problems이 발생할 수 있다.
```
sudo dpkg -i libseccomp2_2.5.4-1ubuntu1_amd64.deb
sudo dpkg -i containerd.io_1.6.4-1_amd64.deb
sudo dpkg -i docker-ce-cli_20.10.16~3-0~ubuntu-focal_amd64.deb
sudo dpkg -i docker-ce_20.10.16~3-0~ubuntu-focal_amd64.deb
```

### 3.  설치 확인
```
sudo docker ps
sudo docker --help
등등
```
