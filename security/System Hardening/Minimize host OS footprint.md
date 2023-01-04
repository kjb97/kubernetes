# 1. Least Privilege Principle
최소 권한 원칙

- Kubernetes 인프라를 보호하기 위해 노드, 노드에서 실행되는 소프트웨어, 
  Kubernetes 구성 요소 및 워크로드를 포함한 시스템이 
  클러스터의 노드에 액세스할 수 있는 
  사용자 및 계정 제한과 같은 최소 액세스 권한만 갖도록 
  동일한 최소 권한 원칙을 사용

- 역할 기반 액세스 제어를 사용하여 
  클러스터의 어떤 부분에 액세스하고 
  어떤 작업을 수행할 수 있는지 명확하게 정의

- 필요한 소프트웨어만 호스트에 설치하는 등 시스템의 다른 구성 요소에도 이 원리를 사용

- 원하지 않는 서비스가 노드에 노출되지 않도록 하고, 
  특정 커널 모듈이 노드에서 제공하는 포트에 로드되지 않도록 하며, 
  시스템에서 열려 있는 포트를 식별하고 수정

## 2. Reducing the Attack Surface
공격 가능 범위 줄이기

보안과 관련하여 여러 보안 취약점이 있다.
위협을 제한하고 공격 표면을 줄이는 한 가지 방법은 클러스터의 모든 시스템을 일관되고 단순한 상태로 유지하는 것.
제대로 구성되지 않은 노드가 있거나 수정되지 않은 심각한 취약성이 있는 노드가 있으면 전체 클러스터가 손상될 가능성이 높다.
시스템의 복잡성을 줄이고, 공격 표면을 줄이기 위해, 최소 특권의 원칙을 활용할 수 있다.

 ### Limit Node Access (노드 접근 제한)
- 우선, 컨트롤플레인과 워커노드의 인터넷 노출을 제한해야 한다.
 클라우드의 Kubernetes 서비스의 경우 아예 컨트롤 플레인에 접근을 못할 수도 있다.
 그러나 온프레미스이거나 자체 k8s 클러스터의 경우, 
 전용 네트워크에서 노드를 프로비저닝하여 VPN 솔루션을 통해 클러스터에 접근하는 방법이 있다.
 VPN 솔루션을 구현할 수 없는 경우, 
 인프라 방화벽 내에서 인증된 네트워크를 사용할 수 있도록 하는 것이 또다른 대안.

- 그리고 접근 가능한 사용자를 명확히 정의해야 한다.
 예를 들어, 시스템 관리자는 노드를 관리할 때 액세스 권한이 필요하며, 
 노드가 작동 중인 상태인지, 업그레이드된 상태인지, 전달된 상태인지 확인하고, 노드의 문제를 해결해야 한다.
 개발자는 기본적으로 접근할 수 없어야 하는 것이 맞다.
 하지만 prod 환경이 아닌 dev 환경의 경우는 또 예외일 수 있다.

# Managing Linux Accounts ( 리눅스 계정 관리 )
- 리눅스 계정을 상황에 맞게 필요한 최소한의 권한으로 사용자 상태를 유지해야 한다.

# SSH Hardening ( SSH 강화 )
- 패스워드 로그인보단 key pair를 사용하도록 설정.

- root로 SSH를 사용하지 않도록 설정
 이렇게 하면 아무도 루트 계정을 사용하여 원격으로 로그인할 수 없고
 사용자 계정만으로 시스템에 연결 가능.
 SSH를 통한 root 로그인을 사용하지 않도록 설정하려면 
 /etc/ssh/sshd_config  SSH 구성 파일 업데이트. 
 추가로 key pair를 사용한다면 password 로그인을 비활성화.
 ```
 PermitRootLogin no
 PasswordAuthentication no
 ```
 sshd를 restart하면 적용된다.
 ```
 systemctl restart sshd
 ```

# Privilege Escalation in Linux ( 권한 상승 )
- root 계정을 막았다면 일반 사용자 계정만을 사용하기 때문에
 root 권한이 필요하다면 sudo 명령어를 통해 권한을 빌려와서 사용.
 sudo의 구성파일은 /etc/sudoers
 
```
# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
~
```
| 필드는 순서대로 User 또는 group(앞에 %가 붙음), host, user, command 순


# Remove Obsolete Packages and Services ( 사용하지 않는 패키지와 서비스 삭제 )
- 필요한 소프트웨어만 설치. 
- 설치된 소프트웨어는 지속적으로 업데이트되므로 시스템을 최대한 간소화
- ex ) kubernetes 노드에서 apache 같은 web 서버는 필요하지 않음.
- 불필요하게 자원이 낭비되고 시스템 복잡도가 증가하면 클러스터의 취약점이 증가.
```
systemctl list-units --type service
```
| service 리스트를 확인해서 불필요한 소프트웨어 확인


# Restrict Kernel Modules 
- 소스에서 Kubernetes 워크로드를 실행하는 경우 포드에서 실행되는 권한 없는 프로세스도 
 네트워크 소켓을 생성하여 특정 네트워크 프로토콜 관련 모듈을 커널에 로드할 수 있다.

 이 잠재적 취약점을 해결하기 위해 클러스터의 모든 노드에서 불필요한 모듈을 블랙리스트에 추가.

 예를 들어, SCTP 커널 모듈은 Kubernetes 클러스터 내에서 일반적으로 사용되지 않으며, 
 블랙리스트 구성 파일에 다음 항목을 추가하여 노트에 블랙리스트에 추가.
 ```
 vi /etc/modprobe.d/blacklist.conf

 ~

 # blacklist 추가
 blacklist sctp

 ~

 # 이후 reboot 하면 적용되고 lsmod로 모듈 listㄹ르 보면 없어진 것을 확인.

 ```

# Identify and Disable Open Ports
- 당연하게도 사용하지 않는 포트는 닫아야 한다.
- kubernetes에서 기본적으로 사용되는 포트는 docs를 확인할 수 있다. https://kubernetes.io/ko/docs/reference/ports-and-protocols/
- 또, 특정 포트의 용도를 알아내려면 /etc/services 파일로 확인.
```
# 53번 port 정보
cat /etc/services | grep -w 53

~

domain          53/tcp                          # Domain Name Server
domain          53/udp
```


