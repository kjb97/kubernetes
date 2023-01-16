## Unable to connect to the server: x509
이런 메시지와 함께 kubectl을 사용할 수 없는 경우  
```
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```    
| 만약 클러스터를 다시 구성하거나 여러 클러스터를 사용하는 경우에 자주 볼 수 있다.  
| kubeconfig 파일 내 인증서가 클러스터의 인증서와 다르거나 인증서가 잘못된 경우.
| 보통은 config 파일을 재생성하면 된다.    

```
# 일반 user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# root user
export KUBECONFIG=/etc/kubernetes/admin.conf
```