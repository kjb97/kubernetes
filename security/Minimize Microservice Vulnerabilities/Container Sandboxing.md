## 1. Multi Tenant 환경
RAM 스토리지와 여러 CPU 코어를 자유롭게 사용할 수 있는 물리적 서버가 아래에 있고,  
이 하드웨어 위에 운영 체제를 설치.</br>
그리고 하이퍼바이저가 운영 체제에 설치.</br>
마지막으로 여러 가상 시스템을 분할하여 필요에 따라 RAM 및 CPU 코어를 할당.</br>
따라서 각 가상 시스템은 서로 다른 운영 체제를 실행 가능. </br>
중요한 점은 모든 가상 시스템이 고유한 운영 체제를 가지고 있고, 더 중요한 것은 전용 커널을 가지고 있다는 것.</br></br>

클라우드의 가상 시스템에도 동일한 사항이 적용.  
AWS, Azure, GCP 또는 다른 클라우드 프로바이더에 관계없이 클라우드에서 생성하는 모든 VM은 자체 전용 커널을 사용.  
그래서 동일한 물리적 인프라에서 실행되는 가상 시스템 간에 강력한 격리가 가능.  
가상 머신이 제공하는 강력한 격리 기능으로 인해 가상 머신을 사용하여 다양한 환경을 여러 사용자에게 안전하게 호스팅할 수 있다.    

이러한 설정을 MultiTenant 환경이라고 한다.    

## 2. VM과 Container의 차이
주요 차이점 중 하나는 가상 시스템에서 실행 중이든 물리적 서버에서 실행 중이든 서버의 모든 컨테이너가 동일한 기본 커널을 공유한다는 것.  
호스트의 관점에서 보면 호스트 및 호스트에서 실행 중인 다른 컨테이너와 분리된 또 다른 프로세스일 뿐이다.  
예를 들어, 1,000초 동안 절전 모드로 전환되는 BusyBox 컨테이너가 있을 때,  
이 컨테이너는 PID가 1인 컨테이너 내에서 루트 사용자로서 sleep 명령을 실행.  
컨테이너를 실행한 호스트에서 PS 명령을 실행하는 경우 호스트에서 동일한 프로세스가 실행되고 있지만 PID가 다른 것을 확인할 수 있다.  
컨테이너 내부 process와 호스트의 process가 각각 하나씩 실행 중.    

이를 Process ID Namespace라고 하며, 컨테이너가 프로세스를 서로 격리하는 방법이다.  
호스트에서 모든 컨테이너가 실행 중인 것.  
OS에 직접 배포하든 컨테이너로 배포하든 애플리케이션은 호스트에서 실행된다.  
그래서 결국은 둘 다 호스트의 하드웨어 자원을 사용한다는 점은 동일하다.    

그러나 전용 커널이 있는 VM은 자체 커널을 가지고 있지만 컨테이너는 동일한 커널(호스트의 커널)을 공유한다.  
호스트의 모든 컨테이너는 동일한 커널에 syscall을 수행하여 동작하기 때문에  
Dirty COW와 같은 공격을 사용하여 컨테이너에서 취약한 커널을 실행하는 호스트로 침입하는 등의 취약점이 생긴다.  
손상된 컨테이너는 호스트에 대한 백도어 항목을 생성할 수 있기 때문에 호스트에서 실행 중인 다른 모든 컨테이너를 손상시킬 수 있다.      

이 문제를 해결하기 위해서 sandbox를 이용한다.    

## 3. Sandbox
IT 분야에서 sandbox는 여러 의미가 있을 수 있지만, 보안에서 sandbox는 격리 기술을 의미.    

Docker는 기본 Seccomp 프로필을 사용하여 컨테이너가 잠재적으로 위험한 시스템 호출을 수행할 수 없도록 한다.  
Seccomp를 Kubernetes에서 사용하고 컨테이너를 응용 프로그램 실행에 필요한 최소한의 권한으로 제한한다.  
또한 AppArmor profile을 사용하여 컨테이너가 액세스할 수 있는 리소스를 세부적으로 제어하는 방법도 있다.    

**문제는 사용하는 애플리케이션의 수가 많아지면 이것들을 일일히 제어하긴 어렵다.**
 

## 3. gVisor
- 컨테이너와 운영 체제의 커널 사이에 추가적인 보호 계층    

컨테이너 격리에서 이슈는 결국 컨테이너끼리 같은 커널과 직접 연결된다는 것.  
gVisor는 컨테이너와 커널 사이의 추가적인 격리 계층을 허용하는 tool.  
컨테이너에서 커널로 직접 syscall하지 않고 gVisor에 호출하도록 한다.    

즉, 각각의 컨테이너의 syscall이 호스트의 커널이 아닌 gVisor가 제공하는 각각의 가상 커널을 사용하도록 하는 것.    

### 예시
**install**

```
# 필수 패키지
sudo apt-get update && \
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

(
  set -e
  ARCH=$(uname -m)
  URL=https://storage.googleapis.com/gvisor/releases/release/latest/${ARCH}
  wget ${URL}/runsc ${URL}/runsc.sha512 \
    ${URL}/containerd-shim-runsc-v1 ${URL}/containerd-shim-runsc-v1.sha512
  sha512sum -c runsc.sha512 \
    -c containerd-shim-runsc-v1.sha512
  rm -f *.sha512
  chmod a+rx runsc containerd-shim-runsc-v1
  sudo mv runsc containerd-shim-runsc-v1 /usr/local/bin
)
```    

containerd의 경우
/etc/containerd/config.toml 파일에 cri를 추가.
gVisor 전용 cri는 runsc
```
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
          runtime_type = "io.containerd.runsc.v1"

```    

**gVisor를 이용한 pod 생성**
- runtimeclass.yaml
```
# RuntimeClass is defined in the node.k8s.io API group
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: test 
# 사용할 cri를 지정 
handler: runsc
```    

- pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  runtimeClassName: gvisor
  containers:
  - name: nginx
    image: nginx:1.14.2
```    

- 확인
```
kubectl exec nginx -- dmesg | grep -i gvisor

# 호스트와 컨테이너 내부에서 uname -r 을 해보면 호스트와 커널 격리가 된 것을 볼 수 있다.
```    
## 4. Kata Containers
초경량 VM  
가상 커널을 컨테이너와 돌리는 gVisor랑 달리 각 컨테이너가 VM처럼 동작하는 방식.    

초경량이라고는 하지만 각 컨테이너에 약간 더 많은 메모리 및 계산 리소스가 필요하므로 기존 컨테이너에 비해 성능 저하는 필연적.  
더 큰 문제는 이미 가상화된 일반적인 클라우드 서비스에서는 가상 자원 내에서의 가상화를 지원하지 않음.
Google 클라우드 같은 곳은 지원을 하기는 하지만 확실히 성능이 떨어짐.  
베어메탈 같은 환경에 적합하다고 할 수 있다.  

