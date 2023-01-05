# kaniko를 이용한 pipeline ( Jenkinsfile )
- docker가 없는 환경에서 docker image를 만들기 위해 kaniko를 사용할 수 있다.
- 예시는 node.js 코드
- 먼저 npm 빌드 후에 kaniko를 이용해서 docker image를 생성하고 registry로 push

```
pipeline {
  agent {
    node {
      label 'node'
    }

  }
  stages {
    stage('Clone') {
      agent none
      steps {
        git(url: 'http://gitlab.test.co.kr/test/test-platform-ui.git', credentialsId: 'test', branch: 'develop', changelog: true, poll: false)
   
      }
    }

    stage('Build') {
      agent none
      steps {
        container('node') {
          sh '''apk add --no-cache --virtual .build-deps ca-certificates python2 python3 py3-pip make openssl g++ bash
npm install yarn@1.22.4
yarn install && yarn upgrade @kube-design/components && yarn rebuild-kube-design && yarn build
mkdir -p out/server
mv dist/ out/
mv server/locales \\
       server/public \\
       server/views \\
       server/sample \\
       server/config.yaml out/server/
mv package.json out
'''
        }

      }
    }

    stage('kaniko build image') {
      agent none
      steps {
        container('kaniko') {
          withCredentials([string(credentialsId : 'harbor' ,variable : 'HARBOR' ,)]) {
            sh '''echo $HARBOR > /kaniko/.docker/config.json
/kaniko/executor --context $WORKSPACE --dockerfile build/Dockerfile.Kaniko --destination harbor.test.co.kr/test/test:v3.3.0'''
          }

        }
      }
    }
  }
}

```
