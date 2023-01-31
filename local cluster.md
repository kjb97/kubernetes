# Local Cluster 구축

## Network 설정

###  공유기 포트포워딩
공유기 public IP:<?> -> vm private IP:<?>  
IPTIME DDNS 설정 <dns 이름>.iptime.org    

### SSH
**IPTIME**  
Advanced Setup -> NAT/Routing -> Port Fowarding -> VM IP + 외부 포트 xxx22, 내부 포트 22로 포워딩 (Bastion으로)    

**VM**
Bastion 서버 
