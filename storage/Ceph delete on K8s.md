# Ceph delete on K8s

## 제거
- 1. ceph 클러스터의 cleanupPolicy를 yes-really-destroy-data로 설정한다
```
kubectl -n rook-ceph patch cephcluster rook-ceph --type merge -p '{"spec":{"cleanupPolicy":{"confirmation":"yes-really-destroy-data"}}}'
```
- 2. 이 후, rook-ceph 네임스페이스에 cleanup 이라는 job이 생성되고 ceph의 volume 데이터를 삭제한다.  
- 3. job이 complete 상태가 되면 crd를 삭제한다.
```
kubectl -n rook-ceph delete cephcluster rook-ceph
```
- 4. 배포한 파일을 이용해 삭제한다.
```
kubectl delete -f operator.yaml kubectl delete -f common.yaml kubectl delete -f crds.yaml
```
