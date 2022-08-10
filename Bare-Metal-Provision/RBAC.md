# Kubernetes Role Based Access Control
This example shows how to generate a kubeconfig file with access to only one namespace. We use the wordpress namespaces as an example.
Docs found [here](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user)

```
openssl genrsa -out myuser.key 4096
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser/O=group1/O=group2"
```
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: $(cat user.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000  # one year
  usages:
  - digital signature
  - key encipherment
  - client auth
```
```
kubectl get csr
kubectl certificate approve myuser
kubectl get csr myuser -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
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
### Create a Role Binding for the User Account
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: wordpress-admin
  namespace: wordpress
subjects:
  - kind: User
    name: wordpress-admin
    namespace: wordpress
roleRef:
  kind: Role
  name: wordpress-admin
  apiGroup: rbac.authorization.k8s.io
```
### Export the Config
```
kubectl config set-cluster kubernetes --server=$APISERVER --kubeconfig=myuser.kubeconfig
kubectl config set-cluster kubernetes --embed-certs --certificate-authority=$PATH_TO_CA_FILE --kubeconfig=myuser.kubeconfig     #/etc/kubernetes/pki/ca.crt
kubectl config set-credentials myser --client-certificate=user.crt --client-key=user.key --embed-certs=true --kubeconfig=myuser.kubeconfig
kubectl config set-context myuser --cluster=kubernetes --user=myser --namespace=wordpress --kubeconfig=myuser.kubeconfig
#Distribute kube config file to user
kubectl config use-context myuser
```
