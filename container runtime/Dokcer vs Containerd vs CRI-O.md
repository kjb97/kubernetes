## Docker 
Docker Engine 하나에 API, CLI, 네트워크 등 여러기능이 합쳐진 구조.  
Kubernetes 1.24 버전부터 런타임으로 지원하지 않음.  
장점 : 도커 하나로 컨테이너 관련 작업 모두 수행 가능.  
단점 : 모놀리식 구조로 인해 무겁고, 버전이 바뀌면 Kubernetes에도 영향을 준다.  

## Containerd
Docker에서 만든 OCI (Open Container Initiative) 표준으로 만들어진 런타임.
Docker에서 컨테이너 런타임을 분리시킨 