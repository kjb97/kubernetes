# Docker Offline Install (ubuntu)
### 1. Release 확인 및 다운
[docker deb packages](https://download.docker.com/linux/ubuntu/dists/)

[libseccomp deb packages](http://security.ubuntu.com/ubuntu/pool/main//libs/libseccomp/)


```
wget http://security.ubuntu.com/ubuntu/pool/main//libs/libseccomp/libseccomp2_2.5.4-1ubuntu1_amd64.deb
wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/containerd.io_1.6.4-1_amd64.deb
wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-cli_20.10.16~3-0~ubuntu-focal_amd64.deb
wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce_20.10.16~3-0~ubuntu-focal_amd64.deb
```
- libsecomp의 버전이 낮으면 설치할 때 dependency problems가 발생할 수 있다.

### 2. 폐쇄망으로 파일을 옮기고 dpkg로 설치
- 순서대로 설치하지 않으면 dependency problems가 발생할 수 있다.
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

### 4. 사용자 권한 설정
```
sudo groupadd docker
sudo usermod -aG docker $USER
sudo service docker restart
사용자 다시 로그인
```
# Private Registry
### 1.이미지 준비
- 외부 접속이 가능한 곳에서 공식 registry docker image를 가져와 save한다.
```
docker pull registry
docker save -o registry.tar registry
```
- 폐쇄망 서버에 옮기고 load해서 이미지 압축해제
```
docker load -i registry.tar
```
- registry build
```
docker run -dit --name registry -p 5000:5000 registry
```

### 2. private registry로 push
- 먼저 push할 이미지의 태그를 지정
```
# 이미 존재하는 이미지일 경우
docker image tag nginx:latest 192.168.xx.xx:5000/test:1.0

#새로 빌드하는 경우
docker build --tag 192.168.xx.xx:5000/test:1.0 nginx
```

- push
```
docker push 192.168.xx.xx:5000/test:1.0
```
- pull
```
docker pull 192.168.xx.xx:5000/test:1.0
```

### 오류:  Get "https://192.168.xx.xx:5000/v2/": http: server gave HTTP response to HTTPS client 

- /etc/docker 디렉터리에 daemon.json 파일을 추가

```
vi /etc/docker/daemon.json

{

"insecure-registries": ["192.xx.xx.xx:5000"]

}
