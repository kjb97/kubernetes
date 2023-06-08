# Error: rendered manifests contain a resource that already exists.

- helm으로 애플리케이션 설치 시에 만날 수 있는 에러
- 같은 애플리케이션을 이전에 설치한 적 있는 경우에 발생할 수 있다.
- 과거에 helm uninstall 또는 helm delete를 했음에도 관련 리소스가 전부 삭제되지 않은 경우에 발생하는 리소스 충돌.
- crd 관련 리소스에서 자주 보이는 것 같다.


## Chart를 이용한 관련 리소스 삭제
```
helm template <NAME> <CHART> --namespace <NAMESPACE> | kubectl delete -f - 

#ex
helm template koo koo/test --namespace koo | kubectl delete -f -
```
- 이걸로 안되면 관련 crd 수동 삭제
