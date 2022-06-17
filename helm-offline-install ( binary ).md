# helm-offline-install
### 1. Release 확인 및 다운
- 릴리즈 [enter link description here](https://github.com/helm/helm/releases)
```
wget https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
```
### 2. 압축 해제
- 폐쇄망에 받은 파일을 옮기고 압축해제
```
tar -zxvf helm-v3.8.2-linux-amd64.tar.gz
```
### 3.  helm binary 파일 이동
```
mv linux-amd64/helm /usr/local/bin/
```
### 4.  helm 확인
