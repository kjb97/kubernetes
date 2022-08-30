# KE CICD 작업내용

## CI 등록 순서
### 1. 작업대상 workspace 이동
### 2. DevOps Projects 생성
- 생성시 작업 대상 cluster 선택
### 3. 생성한 Devops Project로 이동
### 4. Credentials 설정
- 필요한 인증 정보는 총 두 가지 입니다.
1. gitlab repository access token
2. gitlab login infomation
- create 버튼을 클릭해 Access token과 Username and password type 두 가지를 생성합니다.
### 5. Code Repository 등록
- 소스코드가 들어 있는 Repository를 등록합니다.
- 좌측탭 Code Repositories를 클릭하여 이동합니다.
- import 버튼을 통해 Code Repository를 등록합니다.
- 최 하단 Code Repository칸을 클릭하면 , Code Repository를 선택하는 화면으로 이동합니다.
- gitlab을 이용하기에 Git 버튼을 클릭합니다.
- Code Repository URL에 해당 repository의 clone url 을 등록합니다 . ( http ~ )
- Credential에는 4번에서 생성해두었던 Access token을 선택하고 체크박스를 클릭하여 구성을 완료합니다.
### 6. Pipeline 등록
- 좌측 Pipelines 탭으로 이동합니다.
- Create 버튼을 클릭하여 Pipelines를 생성합니다.
- Name과 Code Repository를 생성합니다.
- Code Repository URL은 이전에 Code Repository를 등록할 때 사용했던 clone url을 넣어줍니다.
- Credential에는 Username and password type으로 생성한 Credential을 넣어줍니다.
- next 버튼을 클릭하여 pipeline 세부 설정을 진행합니다.
	- Path에는 jenkinsfile의 경로가 들어갑니다.
	- default값으로 Jenkinsfile 만 들어가 있는데 , 해당 값은 Jenkinsfile이 루트 경로에 위치한다는것을 의미 합니다.
	- create 버튼으로 Pipeline을 등록합니다.
	- Jenkinsfile 경로만 맞다면 , pipeline이 등록되고 스크립트가 실행됩니다.


##  [corona application](https://gitlab.kuberix.co.kr/kuberix/corona-tracker-backend-cicdtestapp)
- 샘플 중 코로나 애플리케이션( Maven )의 back-end 부분을 CI 파이프라인으로 생성 시도했습니다.
- gitlab 레파지토리와 연결은 잘 되었고 git clone까지 문제 없었습니다. 
### Jenkinsfile
```
pipeline {

    environment {
        DOCKER_REGISTRY = '/'
    }

    agent {
        label 'maven'
    }

    stages {
        stage('clone'){
            steps {
                sh 'git clone https://github.com/amrityam/spring-boot-reactjs-kubernetes.git'
            }
        }
        stage('build') {
            steps {
                container ('maven'){
                    sh 'mvn --version'
                    sh 'mvn clean install'
                }
            }
        }        
        stage('build docker image'){
            steps{
                container ('maven'){
                    script{
                      echo 'make docker image'
                      docker_image_name = "${DOCKER_REGISTRY}/demo-corona-backend-spring-app"
                      builded_dockerimage = docker.build("${docker_image_name}")
                    }
                }       
            }
        }
        stage("push dockerimage"){
            steps{
                container ('maven'){
                  script{
                      docker.withRegistry('/msa_demo_app', 'harbor'){
                          builded_dockerimage.push()
                      }
                      sh "docker rmi ${docker_image_name}"
                  }
                }
            }
        }
    }
}
```

- 허나, stage('build') 부분에서 해당 에러가 발생합니다.

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.0:compile (default-compile) on project demo: Fatal error compiling: invalid target release: 11 -> [Help 1]
```
- build를 수행하는 container의 자바 버전이 1.8로 올라오기 때문에 자바 버전 에러라고 생각되어 container의 자바 버전을 11로 업그레이드 하는 작업을 추가해봤습니다.
```
        stage('build') {
            steps {
                container ('maven'){
                    sh 'mvn --version'
                    sh 'yum install java-11-openjdk-devel.x86_64 -y'
                    sh 'update-alternatives --config java <<< 3'
                    sh 'update-alternatives --config javac <<< 3'
                    sh 'java -version && javac -version'
                    sh 'mvn --version'
                    sh 'mvn clean install'
                }
            }
        }    
```
- 자바 버전은 성공적으로 업그레이드 되었으나 여전히 build에서 같은 에러가 발생했습니다.
- KE 초기화 이후 [Callicoder Application](https://gitlab.kuberix.co.kr/kuberix/callicoder)을 이용해서 새로 파이프라인 테스트를 진행하려 했으나 KE 초기화 이후 파이프라인을 run 하는 순간 running 상태이던 KE의 Jenkins가 CrashLoofBack 상태가 되어 테스트가 불가한 상태입니다.
```
#jenkkins pod error message
Readiness probe failed: Get "http://10.233.96.89:8080/login": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

## CD 등록 순서
### 1. DevOps Project 선택
### 2. 좌측 메뉴에서 Continuous Deployment 선택
### 3. Create를 통해 미리 Code Repositories에 등록한 Repository 선택 (이미지 첨부했습니다. )
