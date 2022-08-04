# Kubernetes Role Based Access Control
This example shows how to generate a kubeconfig file with access to only one namespace. We use the wordpress namespaces as an example.

## User Account Method
Instructions found here: https://cloudhero.io/creating-users-for-your-kubernetes-cluster

## Service Account Method (Not Recommended)
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
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: wordpress-admin
  namespace: wordpress
subjects:
  - kind: ServiceAccount
    name: wordpress-admin
    namespace: wordpress
roleRef:
  kind: Role
  name: wordpress-admin
  apiGroup: rbac.authorization.k8s.io
```
### Generate Kubeconfig File
