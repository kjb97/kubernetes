# Kubesphere 배포 방안

-   전체 설치 방안은 아래의 공식 문서를 참조
-  [설치 방안 공식 문서](https://kubesphere.io/docs/v3.3/installing-on-linux/introduction/multioverview/)

## 1. System Requirements

###  node 요구 사항

-   모든 노드는 ssh 통신이 가능해야 함. ex ) ssh ~
-   모든 노드의 시간은 동기화 되어야 함
-   sudo , curl 등 모든 노드에서 사용해야 함

### container runtime 요구사항

-   클러스터에는 사용 가능한 container runtime이 존재
-   kubekey를 사용할 경우 , default container runtime은 docker
-   클러스터 설치 시 , 원하는 container runtime을 직접 설치 가능
     > docker, CRI-O, cotainer-d


### dependency requirements
- 필요한 패키지 설치
```
sudo apt-get install -y socat conntrack ebtables ipset
```

### network requirements

-   DNS 주소를 /etc/resolv.conf 에서 사용할 수 있는지 확인해야 함
-   cluster config 파일을 작성할 때 설정해주는 dns 정보를 resolv.conf가 해석 가능해야 함

## 2. install KubeKey

### 2.1 get install package

-   github에서 kubekey releases를 직접 선택하여 설치 가능 
-  [kubekey github](https://github.com/kubesphere/kubekey/releases)

```
$ curl -sfL https://get-kk.kubesphere.io | VERSION=v2.2.1 sh -
```

가지고온 파일을 x권한을 부여하여 실행 파일로 변환합니다.
```
$ chmod +x kk
```

### 2.2 kubernetes cluster 구성 파일 설정

-   클러스터 설정 파일 생성
```
$ ./kk create config [--with-kubernetes version] [--with-kubesphere version] [(-f | --file) path]

```

-   이 단계에서 명령에 플래그를 추가하지 않으면 구성 파일의 필드를 사용하여 설치하거나 나중에 사용할 때 이 플래그를 다시 추가  `--with-kubesphere`하지 않는 한 KubeSphere는 배포되지 않음
```
$ ./kk create config --with-kubernetes version v1.23.7  --with-kubesphere v3.3.0
```

```
$ cat config-sample.yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: node1, address: 192.168.208.145, internalAddress: 192.168.208.145, user: jinseong, password: "1234"} # node의 접근 정보를 입력합니다. 
  - {name: node2, address: 192.168.208.146, internalAddress: 192.168.208.146, user: koo, password: "1234"}
  - {name: node3, address: 192.168.208.147, internalAddress: 192.168.208.147, user: koo, password: "1234"}
  roleGroups:
    etcd:
    - node1 # etcd 노드 선택
    control-plane:
    - node1 # master 노드 선택
    worker:
    - node2 # worker 노드 선택
    - node3
  controlPlaneEndpoint:
    ## Internal loadbalancer for apiservers
    # internalLoadbalancer: haproxy # 앞단 LB 설정또한 가능합니다.

    domain: lb.kubesphere.local
    address: ""
    port: 6443 # port 지정 ( default 권장 )
  kubernetes:
    version: v1.23.7 # k8s version
    clusterName: cluster.local # cluster의 AAA 방식 dns name 선택
    autoRenewCerts: true
    containerManager: docker # container runtime 설정
  etcd:
    type: kubekey
  network: 
    plugin: calico # cni 선택
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    multusCNI:
      enabled: false
  registry:
    privateRegistry: ""
    namespaceOverride: ""
    registryMirrors: []
    insecureRegistries: []
  addons: []


```

-   만약 ssh를 사용한 암호 없는 로그인인 경우에는 아래와 같이 노드 접근 정보를 기입합니다.

```
hosts: - {name: master, address: 192.168.0.2, internalAddress: 192.168.0.2, privateKeyPath: "~/.ssh/id_rsa"}

```


### 3.3 cluster 구성 파일을 사용하여 클러스터 생성
-   클러스터를 생성
-   전체 설치 프로세스는 10 ~ 20분 정도 소요
```
$ ./kk create cluster -f config-sample.yaml

```

## 4. console 확인

-   기본적으로 30880 포트를 사용하게 되며 , 웹 콘솔에 엑세스하여 정상 설치 여부를 확인

```
# default
id : admin
pwd : P@88w0rd
```
