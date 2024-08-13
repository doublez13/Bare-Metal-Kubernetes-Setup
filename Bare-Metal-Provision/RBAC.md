# Kubernetes Role Based Access Control
This example shows how to generate a kubeconfig file with access to only one namespace. We use the wordpress namespaces as an example.
Docs found [here](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user)

## Adding a New User
### Generate Certificate Signing Request
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
cat << EOF > $USER-csr.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: $USER
spec:
  request: $(cat $USER.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000  # one year
  usages:
  - digital signature
  - key encipherment
  - client auth

EOF
```

```
kubectl apply -f $USER-csr.yaml
kubectl get csr
kubectl certificate approve $USER
kubectl get csr $USER -o jsonpath='{.status.certificate}'| base64 -d > $USER.crt
rm $USER.csr $USER-csr.yaml
```

### Create A Role (if a proper role doesn't already exist)
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
  name: $USER-wordpress-admin-role-binding
  namespace: wordpress
subjects:
  - kind: User
    name: $USER
    namespace: wordpress
roleRef:
  kind: Role
  name: wordpress-admin
  apiGroup: rbac.authorization.k8s.io
```
### Export the Config
```
kubectl config set-cluster kubernetes --server=$APISERVER --kubeconfig=$USER.kubeconfig
kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}" >> ca.crt
kubectl config set-cluster kubernetes --embed-certs --certificate-authority=ca.crt --kubeconfig=$USER.kubeconfig
kubectl config set-context $USER --cluster=kubernetes --user=$USER --namespace=wordpress --kubeconfig=$USER.kubeconfig
kubectl config set-credentials $USER --client-certificate=$USER.crt --client-key=$USER.key --embed-certs=true --kubeconfig=$USER.kubeconfig
#Distribute kube config file to user
kubectl config use-context $USER
```


## Renew a Certificate for an Existing User
### Generate Certificate Signing Request
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
cat << EOF > $USER-csr.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: $USER
spec:
  request: $(cat $USER.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000  # one year
  usages:
  - digital signature
  - key encipherment
  - client auth

EOF
```

```
kubectl apply -f $USER-csr.yaml
kubectl get csr
kubectl certificate approve $USER
kubectl get csr $USER -o jsonpath='{.status.certificate}'| base64 -d > $USER.crt
rm $USER.csr $USER-csr.yaml
```

```
kubectl config set-credentials $USER --client-certificate=$USER.crt --client-key=$USER.key --embed-certs=true --kubeconfig=$USER.kubeconfig
```
