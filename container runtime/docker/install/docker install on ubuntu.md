
# docker install on ubuntu ( apt-get )

### 이전 버전 삭제
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 1. 필수 package 설치
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

## 2. GPG key 등록
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

## 3. stable repo 등록
```
 echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 4. update 후 docker 설치
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

