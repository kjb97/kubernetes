# Jenkins Plugin AirGap Install
## 개요 
-  Jenkins Plugin들을 폐쇄망 환경에서 설치하는 방법
- 폐쇄망에서는 플러그인을 자동 설치할 수 없어서 파일을 들고 들어가야 한다.
- 플러그인들 간에 의존성을 맞추려면 외부에서 Jenkins 설치 최초에 설치하는 게 가장 이슈가 적다.
## 준비
- 외부 환경에서 Jenkins를 설치하고 기본 Plugin들을 설치한다.
- 여기서는 docker run으로 설치

## 1. Plugin .jpi 파일 획득
- Jenkins 컨테이너 안에 있는 Plugin 파일들을 꺼낸다.
```
docker cp jenkins:/var/jenkins_home/plugins .
```
## 2. .hpi로 변경
- script 사용
```
#!/bin/sh
for f in *.jpi; do 
 mv -- "$f" "${f%.jpi}.hpi"; 
done
```


## 3. 폐쇄망 이동후 업로드 
- UI에서 Jenkins 관리 -> Plugin managed -> 고급 -> 업로드를 통해 하나씩 올리거나

- Jenkins 컨테이너 내 /var/jenkins_home/plugins 에 직접 
