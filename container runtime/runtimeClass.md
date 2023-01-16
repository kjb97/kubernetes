# kubernetes runtimeclass
- pod로 container를 생성 시 어떤 container runtime tool을 사용할건지 지정이 가능한 쿠버네티스 리소스.    

- runtimeclass.yaml
```
# RuntimeClass is defined in the node.k8s.io API group
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: test 
# 사용할 cri를 지정 
handler: test
```    

- pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  runtimeClassName: test
  containers:
  - name: nginx
    image: nginx:1.14.2
```
이대로 배포해보면 test라는 cri가 없기 때문에 runtime error가 발생한다.  
```
Warning  FailedCreatePodSandBox  10s (x2 over 21s)  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to get sandbox runtime: no runtime for "test" is configured
```    

### containerd 지정 예시
/etc/containerd/config.toml을 확인하여 containerd의 cri 확인
```
~~~
    [plugins."io.containerd.grpc.v1.cri".containerd]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
~~~
```
runc임을 할 수 있다.    

- runtimeclass.yaml
```
# RuntimeClass is defined in the node.k8s.io API group
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: test 
# 사용할 cri를 지정 
handler: runc
```    

- pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  runtimeClassName: test
  containers:
  - name: nginx
    image: nginx:1.14.2
```
