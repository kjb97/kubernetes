# Install Keycloak Using Helm

## 1. Get helm chart
```
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo update
helm pull codecentric/keycloak --untar
```

## 2. values 수정
- keycloak 콘솔에 로그인 하기 위한 id, password 설정
- admin 설정을 위해 https 통신 필수

### values-set.yaml
```
extraEnv: |
  - name: KEYCLOAK_USER
    value: admin	# ID
  - name: KEYCLOAK_PASSWORD
    value: P@88w0rd		#PASSWORD
  - name: PROXY_ADDRESS_FORWARDING
    value: "true"
```

## 3. Install
```
 helm upgrade --install keycloak . -n keycloak -f values.yaml,values-set.yaml
```


## 4. Ingress 설정 (예시)

```
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: keycloak
  namespace: keycloak
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    ingress.kubernetes.io/ssl-redirect: 'true'
    kubernetes.io/tls-acme: 'true'
    kubesphere.io/creator: admin
    nginx.ingress.kubernetes.io/proxy-body-size: '0'
    nginx.ingress.kubernetes.io/ssl-redirect: 'true'
spec:
  tls:
    - hosts:
        - keycloak.keycloak.gantry.ai
      secretName: tls-keycloak
  rules:
    - host: keycloak.keycloak.gantry.ai
      http:
        paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
              service:
                name: keycloak-http
                port:
                  number: 80
```
