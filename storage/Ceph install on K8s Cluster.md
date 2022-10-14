# Ceph install on K8s Cluster
- disk를 서버에 추가한 후에 진행
- disk가 추가된 worker 노드가 3대 이상 필요
- 디스크에 파티션이 없는 상태여야 함
- kubernetes에 배포가 용이한 rook-ceph 사용
- 보통 ceph RBD와 ceph FileSystem 중 하나를 사용
  >둘의 차이는 크게 블록 저장소냐 파일시스템이냐의 차이  
  >보통은  가상 머신과 함께 사용하기 위해 디스크 이미지를 저장하려는 경우  RBD를 사용하고   여러 컴퓨터 간에 많은 파일을 공유하려는 경우,  FileSystem이 가장 적합 ( nfs와 유사 ).

## 1. rook git clone
```
git clone --single-branch --branch release-1.7 https://github.com/rook/rook.git 
```

## 2. yaml 파일을 이용한 배포
- 1. 먼저 ceph operator를 배포
```
cd rook/cluster/examples/kubernetes/ceph 
kubectl create -f crds.yaml -f common.yaml -f operator.yaml 
```
- 2. operator pod가 running 상태가 되면 ceph cluster를 배포
> 모든 파드가 올라오기까지 10분 내외의 시간 소요
```
# RBD 설치
kubectl create -f cluster.yaml

# FS 설치
kubectl create -f filesystem.yaml
```

- 3. ceph의 모니터링 툴과 storageclass를 생성
```
kubectl create -f toolbox.yaml 

# RBD storageClass
kubectl create -f csi/rbd/storageclass.yaml

# FS storageClass
kubectl create -f csi/cephfs/storageclass.yaml
```

- 4. 상태 확인
- HEALTH_OK로 표시되면 정상 설치 완료
```
kubectl get cephcluster -A
```


## 기타 
- ceph tool 사용 커맨드 ( toolbox 컨테이너 진입 )
- ceph status, ceph osd pool stats 등 사용
```
kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools"  -o jsonpath='{.items[0].metadata.name}')  -- bash
```
