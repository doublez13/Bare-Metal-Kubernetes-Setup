# Kubernetes Role Based Access Control
This example shows how to generate a kubeconfig file with access to only one namespace. We use the wordpress namespaces as an example.
Docs found [here](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user)

```
USER=myuser
SUBJ="/CN=$USER/O=group1/O=group2"

mkdir /tmp/RBAC
chmod 700 /tmp/RBAC
cd /tmp/RBAC
openssl genrsa -out $USER.key 4096
openssl req -new -key $USER.key -out $USER.csr -subj "$SUBJ"
```
NOTE: CN is the name of the user and O is the group that this user will belong to. Users or groups are then referenced in the role binding.


```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: $(cat myuser.csr | base64 | tr -d '\n')
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
  name: myuser-wordpress-admin-role-binding
  namespace: wordpress
subjects:
  - kind: User
    name: myuser
    namespace: wordpress
roleRef:
  kind: Role
  name: wordpress-admin
  apiGroup: rbac.authorization.k8s.io
```
### Export the Config
```
kubectl config set-cluster kubernetes --server=$APISERVER --kubeconfig=myuser.kubeconfig
kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}" >> ca.crt
kubectl config set-cluster kubernetes --embed-certs --certificate-authority=ca.crt --kubeconfig=myuser.kubeconfig
kubectl config set-credentials myser --client-certificate=myuser.crt --client-key=user.key --embed-certs=true --kubeconfig=myuser.kubeconfig
kubectl config set-context myuser --cluster=kubernetes --user=myser --namespace=wordpress --kubeconfig=myuser.kubeconfig
#Distribute kube config file to user
kubectl config use-context myuser
```
