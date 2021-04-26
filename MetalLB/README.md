# MetalLB: Bare Metal Load Balancer  

https://metallb.universe.tf/concepts/layer2/  

MetalLB takes one or more unique IP addresses and assigns them to services of type `LoadBalancer`. In this section, we install and configure MetalLB. In the next section, we create an ingress controller and mark it as type LoadBalancer. MetalLB then assigns the ingress controller a `LoadBalancer Ingress` address from the pool. 

## Preparation
If you’re using kube-proxy in IPVS mode (iptables is the default), since Kubernetes v1.14.2 you have to enable strict ARP mode.

`kubectl edit configmap -n kube-system kube-proxy`

```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

## Installation
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

## Configuration
```
#config.yml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - EXETERNAL_INGRESS_IP(S)_GO_HERE
```
`kubectl apply -f config.yaml`
