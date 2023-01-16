# containerd registry config
### config.toml
- containerd의 설정 파일은 /etc/containerd/config.toml

### registry 설정
- 기본적으로 docker.io가 설정되어 있다.
```
/.../

[plugins]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    runtime_type = "io.containerd.runc.v2"
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "harbor.test.co.kr/test/test:3.6"
    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      max_conf_num = 1
      conf_template = ""
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://registry-1.docker.io"]

```

- private registy 정보와 필요한 경우 auth와 tls를 설정할 수 있다.
```
       /.../
       
       [plugins."io.containerd.grpc.v1.cri".registry.configs]
          [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.test.co.kr".auth]
            username = "test"
            password = "test"
            [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.test.co.kr".tls]
              ca_file = ""
              cert_file = ""
              key_file = ""
              insecure_skip_verify = false

```
