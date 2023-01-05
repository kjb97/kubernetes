# Minimize external access to the network ( 네트워크 외부 액세스 최소화 )
- 기본적으로 netstat -nltp를 해보면 포트가 전부 0.0.0.0에서 수신 중이라 네트워크 내의 모든 장치가 접근 가능.
- 다양한 서비스와 포트에 대한 액세스를 허용하거나 제한하도록 네트워크 보안을 구현.
- 외부 방화벽을 이용해 네트워크 전체에 적용하여 네트워크를 오가는 트래픽을 제어하거나
 서버의 방화벽이나 IP 테이블 등을 사용해 규칙 적용.


## UFW Firewall ( 방화벽 )
- 서버의 in/out 트래픽 규칙을 설정
- 기본적으로 outbound 트래픽을 오픈하고 inbound 트래픽에 대해 세밀한 조정 필요.

1. outbound 오픈
```
ufw default allow outgoing
```

2. inbound 차단
```
sudo ufw default deny incoming
```

3. 특정 IP에 대한 규칙 추가
```
# 172.1.1.1에서 서버에서 사용 중인 IP 주소의 TCP 22번 port로 인바운드 연결 허용
ufw allow ufw allow from 172.1.1.1 to any port 22 proto tcp

# 172.10.10.0/28 범위의 클라이언트 서버에서 사용 중인 IP 주소의 TCP 22번 port로 인바운드 연결 허용
ufw allow ufw allow from 172.10.10.0/28 to any port 22 proto tcp

# 모든 IP에 대한 서버의 8080 port 차단
UFW deny 8080

# 정의된 규칙 삭제
UFW delete deny 8080

or

UFW delete <숫자>   # ufw status로 출력된 규칙의 라인 번호로 삭제
```

