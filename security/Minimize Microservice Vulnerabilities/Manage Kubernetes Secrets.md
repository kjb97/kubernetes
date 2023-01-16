 # **Managing Secrets**
만약 데이터베이스에 연결되는웹 애플리케이션이 있다면  
호스트 이름 사용자 이름과 암호가 하드 코딩되는 경우가 있을 수 있다.  
보안상 매우 취약하다.  
configmap으로 옮기는 방법도 있겠지만, password를 저장하기는 이것도 취약점이 있다.
그래서 secrets을 사용.    

민감 데이터를 pod에 연결하는 방법은 3가지.  

- **secret으로 생성한 후 환경변수로 secret 참조.**
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: mysecret
```    

- **각 데이터를 직접 참조**
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
            optional: false
```    
- **볼륨 마운트 (컨테이너 내부에 지정 경로로 파일 생성)**  
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```    
| **etcd에도 secret 데이터는 저장되고**  
| **secret은 절대 암호화된 값이 아닌 인코딩 값이다.**  
| **따라서 secret 암호화 설정이 필요하고**  
| **rbac 설정을 통해 secret에 아무나 접근하지 못하도록 해야 함.**  

