# docker install on CentOs
```
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io

systemctl start docker 
systemctl enable docker


sudo groupadd docker
sudo usermod -aG docker $USER
sudo service docker restart
사용자 다시 로그인
```
