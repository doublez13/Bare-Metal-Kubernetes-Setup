# Calico CNI

### Installing (Operator based)
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml

curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/custom-resources.yaml -O
#Optionally customize

kubectl create -f custom-resources.yaml
```
A default manifest will produce the following config:
```
spec:
  calicoNetwork:
    bgp: Enabled
    hostPorts: Enabled
    ipPools:
    - blockSize: 26
      cidr: 192.168.0.0/16
      disableBGPExport: false
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
    linuxDataplane: Iptables
    multiInterfaceMode: None
    nodeAddressAutodetectionV4:
      firstFound: true
  cni:
    ipam:
      type: Calico
    type: Calico
```


### Upgrading (Operator based)
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml -O
kubectl replace -f tigera-operator.yaml
```
