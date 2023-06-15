# Calico CNI

### Installing (Operator based)
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml

curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/custom-resources.yaml -O
#Optionally customize

kubectl create -f custom-resources.yaml
```
