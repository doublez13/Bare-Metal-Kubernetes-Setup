Add the helm repo for Wordpress
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

```
kubectl apply -f pvc.yaml
helm install my-site bitnami/wordpress -f values.yaml
```
