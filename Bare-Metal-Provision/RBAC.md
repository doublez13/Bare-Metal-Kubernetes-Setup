# Kubernetes Role Based Access Control
This example shows how to generate a kubeconfig file with access to only one namespace. We use the wordpress namespaces as an example.

### Create A Service Account
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: wordpress-admin
  namespace: wordpress
```

### Create A Role
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: wordpress-admin
  namespace: wordpress
rules:
  - apiGroups: ['*']
    resources: ['*']
    verbs: ['*']
```
### Create a Role Binding for the Service Account
### Generate Kubeconfig File
