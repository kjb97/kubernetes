# Linux Syscall
커널은 하드웨어와 프로세스 사이의 핵심 인터페이스.
둘 사이에서 통신하여 리소스를 효율적으로 관리.
커널은 커널 공간과 사용자 공간 두 개의 메모리 영역을 가짐.
예를 들어 C, 자바, 등으로 만들어진 애플리케이션, 사용자가 실행하는 프로세스와 같은 애플리케이션은 사용자 공간 내에서 실행.
커널 자체는 커널 공간 내에서 실행되며 커널 코드, 커널 확장, 장치 드라이버 포함

예를 들어, 응용 프로그램이 메모리 또는 디스크에 저장된 파일을 열고 데이터를 쓰기가 필요한 경우,
사용자 공간에서 실행되는 응용 프로그램은 syscall 커널에 특별한 요청을 함으로써 장치 데이터에 액세스.
빈 파일 하나 생성에도 여러 syscall이 호출.
대표적으로 touch 바이너리를 실행하는 Execve

# 프로세스에서 사용하는 syscall 추적
- strace
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
| 첫줄을 보면 execve라는 syscall을 호출했고 뒤에 넘긴 파라미터들이 출력. </br>
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

# AquaSec Tracee
