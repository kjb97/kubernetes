# kubernetes gitlab
## 
```
helm repo add gitlab https://charts.gitlab.io
helm pull gitlab/gitlab-runner --untar
```
###
- 
```
helm upgrade --install gitlab . \
--namespace=gitlab \
--set gitlab-runner.install=true \
--set certmanager.install=false \
--set nginx-ingress.enabled=false \
--set global.ingress.configureCertmanager=false \
--set global.ingress.tls.secretName=gitlab-tls \
--set gitlab.gitlab-runner.certsSecretName="gitlab-runner-certs" \
--set gitlab-runner.certsSecretName="gitlab-runner-certs" \
--set gitlab-runner.runners.cache.cacheShared=true \
--set gitlab-runner.runners.cache.secretName=gitlab-minio-secret \
--set gitlab-runner.runners.cache.s3CachePath=runner-cache \
--set gitlab.gitlab-runner.certsSecretName="gitlab-runner-cert" \
--set global.certificates.customCAs[0].secret=custom-ca \
--set minio.persistence.storageClass=ceph-filesystem \
--set minio.persistence.accessMode=ReadWriteMany \
--set prometheus.server.persistentVolume.storageClass=ceph-filesystem \
-f values.yaml
```
