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
Which can also be verified using `calicoctl`
```
calicoctl get ipPool -o wide
NAME                  CIDR             NAT    IPIPMODE   VXLANMODE     DISABLED   DISABLEBGPEXPORT   SELECTOR   
default-ipv4-ippool   192.168.0.0/16   true   Never      CrossSubnet   false      false              all()      
```

It should be noted that although BGP is enabled, it is not being used. If all IpPools are using VXLan, BGP in calico can actually be disabled.


### Upgrading (Operator based)
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml -O
kubectl replace -f tigera-operator.yaml
```
