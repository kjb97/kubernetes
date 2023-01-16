# **Set up a High Availability Cluster with Kubeadm**
- stacked ETCD : ETCD가 컨트롤플레인에 존재
- external ETCD : ETCD가 컨트롤플레인과 분리되어 별도로 존재
- external ETCD를 기준으로 작성.

### ***전제 조건***
- 외부 ETCD 클러스터
- LoadBalancer ( 클라우드 환경이라면 LB 서비스를 사용하면 쉽지만 그게 아니라면 HAProxy를 이용 )
- master 노드 3대 이상, worker 노드 3대 이상. ( master는 홀수 )

