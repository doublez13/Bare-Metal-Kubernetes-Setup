Add the helm repo for Wordpress.  
Note creating a separate PVC should not be required. This can be handled in helm.

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

```
helm install my-site bitnami/wordpress -f values.yaml
```
