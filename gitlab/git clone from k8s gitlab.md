# 배포된 GitLab에서 프로젝트 Clone
## 1. ui 접속 후 프로젝트 생성
## 2. Clone할 url 복사
- Clone with HTTPS의 경로 X  // https://gitlab.example.com/test.git
- 쿠버네티스가 배포한 경로 O // https://gitlab.xxx.xyz/test.git
- 
## 3. SSL 인증 X
```
git config --global http.sslVerify` `false`
```
### 4. Clone
```
git clone http://gitlab.xxx.xyz/gitlab-instance-c730ebd7/test.git
```
- 이후 gitlab에 로그인한 정보로  로그인 진행
- 
``
Cloning into 'test'...
Username for 'http://gitlab.xxx.xyz': root
Password for 'http://root@gitlab.xxx.xyz':
```
