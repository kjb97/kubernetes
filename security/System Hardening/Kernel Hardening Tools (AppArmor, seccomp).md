# **Linux Syscall**
커널은 하드웨어와 프로세스 사이의 핵심 인터페이스.
둘 사이에서 통신하여 리소스를 효율적으로 관리.
커널은 커널 공간과 사용자 공간 두 개의 메모리 영역을 가짐.
예를 들어 C, 자바, 등으로 만들어진 애플리케이션, 사용자가 실행하는 프로세스와 같은 애플리케이션은 사용자 공간 내에서 실행.
커널 자체는 커널 공간 내에서 실행되며 커널 코드, 커널 확장, 장치 드라이버 포함

예를 들어, 응용 프로그램이 메모리 또는 디스크에 저장된 파일을 열고 데이터를 쓰기가 필요한 경우,
사용자 공간에서 실행되는 응용 프로그램은 syscall 커널에 특별한 요청을 함으로써 장치 데이터에 액세스.
빈 파일 하나 생성에도 여러 syscall이 호출.
대표적으로 touch 바이너리를 실행하는 Execve

# **프로세스에서 사용하는 syscall 추적**
## 1. strace
```
# touch 명령으로 파일을 만들고 strace를 사용하면 
touch /tmp/error.log
strace touch /tmp/error.log

~~~

execve("/usr/bin/touch", ["touch", "/tmp/error.log", "-c"], 0x7fff87e5e4e0 /* 19 vars */) = 0
brk(NULL)                               = 0x558ec9417000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffc80b52170) = -1 EINVAL (Invalid argument)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3

~~~
```
| 첫줄을 보면 execve라는 syscall을 호출했고 뒤에 넘긴 파라미터들이 출력.
| 19 vars 는 19개 변수는 사용자 쉘에 정의된 ENV 변수로 시스템 환경변수. 

- 실행중인 프로세스는 프로세스의 PID를 이용해 확인할 수 있다.
```
# etcd 프로세스 PID
pidof etcd
2405220

# PID를 이용한 strace (etcd에 의해 수행된 모든 이후 시스템 호출을 반환)
strace -p 2405220
~~~
```
| strace는 요약이 가능
```
# -c 옵션으로 요약
strace -c -p 2405220
strace: Process 2405220 attached

% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 93.83    0.050754          36      1387       203 futex
  3.86    0.002088          49        42           epoll_pwait
  1.43    0.000771          30        25           write
  0.57    0.000311          19        16         8 read
  0.31    0.000167          55         3           nanosleep
  0.00    0.000000           0         2           lseek
  0.00    0.000000           0         2           fdatasync
------ ----------- ----------- --------- --------- ----------------
100.00    0.054091                  1477       211 total

```

## 2. AquaSec Tracee
Tracee는 아쿠아 시큐리티가 개발한 오픈 소스 도구로, eBPF를 사용하여 런타임에 시스템을 추적.  
eBPF는 커널 소스 코드를 간섭하거나 커널 모듈을 로드하지 않고 커널 공간에서 직접 프로그램을 실행할 수 있는 확장 버클리 패킷 필터.  
eBPF 프로그램은 OS를 모니터링하고 의심스러운 동작을 탐지할 수 있는 아쿠아 보안 추적기와 같은 도구를 만드는 데 사용.  
Tracee는 eBPF 기술을 사용하며 도커 컨테이너로 실행될 때 eBPF 프로그램을 빌드하여 기본적으로 /temp/Tracee 디렉토리에 저장.  
/temp/Tracee 디렉토리를 호스트에서 컨테이너로 바인딩 필요.  
추가로, eBPF 프로그램을 컴파일하기 위해서 트레이시는 /lip/modules 아래의 ubuntu와 /usr/src 아래의 종속성인 커널 헤더에 대한 액세스가 필요.    
마지막으로 system call을 추적하려면 권한이 필요.


```
# ls 명령에 대한 추적
docker run --name tracee --rm --privileged --pid=host  \
-v /lib/modules/:/lib/modules/:ro  \
-v /usr/src:/usr/src:ro  \
-v /tmp/tracee:/tmp/tracee aquasec/tracee:0.4.0  \
--trace comm=ls

# 모든 새로운 프로세스에 대한 추적
docker run --name tracee --rm --privileged --pid=host  \
-v /lib/modules/:/lib/modules/:ro  \
-v /usr/src:/usr/src:ro  \
-v /tmp/tracee:/tmp/tracee aquasec/tracee:0.4.0  \
--trace comm=ls
```

# **Restrict syscalls using seccomp (Secure Computing Mode)**
현재 리눅스에는 약 435개의 SYSCALLS가 있으며 모두 사용자 공간에서 실행 중인 애플리케이션이 하드웨어 및 네트워킹 작업을 수행하는 데 사용할 수 있다.  
실제로는 어떤 응용 프로그램도 이렇게 많은 SYSCALL을 사용하지 않고,  
모든 SYSCALL에 대해서 프로그램이 액세스 권한을 갖는 것은 공격 표면을 증가시킬 수 있다.
SECCOMP는 리눅스 커널 레벨의 기능으로 컨테이너가 실행 가능한 syscall을 제한하는 역할.  

```
# 해당 명령의 출력으로 CONFIG_SECOMP=yes 가 있다면 seccomp를 사용 가능하다는 것.
grep -i seccomp /boot/config-$(uname -r)

```

- syscall 제한의 예시
```
# 컨테이너 sh 연결
docker run --rm -it docker/whalesay sh

# 시스템 date 변경 시도
date -s '19 APR 2012 22:00:01'

~~~
date: cannot set date: Operation not permitted  # 권한 오류 발생
~~~

```    

| date 변경이 실패하는 이유는  
| 현재 동작 중인 sh의 PID를 보면, 1로 나올 것이다.  
| 프로세스 상태에서 seccomp 값을 확인해보면  
```
grep Seccomp /proc/1/status

~~~
Seccomp:        2
Seccomp_filters:        1
~~~
``` 
| 이런 식의 출력이 나온다.  
| seccomp의 상태는 3가지이고 0은 비활성,  
| 1은 read, write, exit 등을 제외한 거의 대부분의 syscall 차단,  
| 2는 선택적으로 syscall 필터링  

# **Docker seccomp**
도커는 기본적으로 컨테이너를 만들 때마다 기본적으로 사용하는 SECCOMP 필터를 가지고 있다.
JSON 문서로 설명된 이 필터는 리눅스에서 300개 이상의 SYSCALLS 중 약 60개를 제한
default.json -> https://github.com/docker/labs/tree/master/security/seccomp/seccomp-profiles

```
{
	"defaultAction": "SCMP_ACT_ERRNO",
	"architectures": [
		"SCMP_ARCH_X86_64",
		"SCMP_ARCH_X86",
		"SCMP_ARCH_X32"
	],
	"syscalls": [
		{
			"name": "accept",
			"action": "SCMP_ACT_ALLOW",
			"args": []
		},
		{
			"name": "accept4",
			"action": "SCMP_ACT_ALLOW",
			"args": []
		},
    ~~~
```
| 우선, x86 64비트와 32비트와 같은 아키텍처는 프로파일을 사용할 수 있는 시스템을 정의.  
| 그리고 SYSCALLS 배열을 사용하여 SYSCALL 이름 배열과 이를 허용할지 거부할지 여부에 대한 관련 작업을 정의.  
| 이 특별한 경우에는 SCMP_ACT_ALLOW의 작업이 포함된 세 개의 SYSCALL이 있습니다.  
| 응용프로그램에서 이 배열에 정의된 SYSCALLS만   사용할 수 있음.  
| "defaultAction"이 "SCMP_ACT_ERRNO"이면 whitelist 방식이고 "SCMP_ACT_ALLOW"이면 blacklist 방식. 
| 프로그램이 필요한 SYSCALL이 추가되어 있지 않으면 프로그램이 동작하지 않을 수 있다.  
| docker의 기본 설정은  X86 아키텍처에서 300개 이상의 SYSCALLS 중  reboot, setting and manipulating the system clock 같은 약 60개를 차단.  

# **Kubernetes Seccomp**
/var/lib/kubelet/seccomp에 default Seccomp profile이 위치.  
kubernetes 1.20부터 기본적으로 seccomp는 off 상태.  
securityContext를 사용해서 pod에 적용.
https://kubernetes.io/docs/tutorials/security/seccomp/

- pod 적용 예시
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: audit-nginx
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json
  containers:
  - image: nginx
    name: nginx
```

# **AppArmor**
seccomp를 사용하여 컨테이너가 사용할 수 있거나 사용할 수 없는 syscall을 효과적으로 제한할 수 있지만  
파일이나 디렉터리와 같은 특정 개체에 대한 프로그램의 액세스를 제한하는 데 사용할 수는 없다.  
사용자 정의 profile을 사용하여 mkdir sys 호출을 제한함으로써  
컨테이너가 내부에 디렉토리를 생성하는 것을 막을 수 있다.
그러나 파일 시스템이나 특정 디렉토리에 쓰지 못하도록 제한하는 보다 섬세한 컨트롤은 할 수 없다.

AppArmor는 프로그램을 제한된 리소스 집합으로 제한하는 데 사용되는 Linux 보안 모듈.
AppArmor는 대부분의 Linux 배포판에 기본적으로 설치.  
```
systemctl status AppArmor 
```
AppArmor를 사용하려면 컨테이너가 실행될 모든 노드에 AppArmor 커널 모듈을 로드.
이제 /sys/module/apparmor/parameters 디렉토리에서 활성화된 파일을 확인하여 확인  
Y 또는 yes 값은 AppArmor 커널 모듈이 로드되었음을 의미.  
seccomp와 마찬가지로 AppArmor는 profile을 통해 응용 프로그램에 적용.  
 /sys/kernel/security/AppArmor/profiles 파일을 확인하여 확인.  



