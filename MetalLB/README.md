# MetalLB: Bare Metal Load Balancer  

MetalLB takes one or more unique IP addresses and assigns them to services of type `LoadBalancer`. This provides a central access point into the cluster for external services, while also providing automatic failover. In other words, MetalLB provides the same functionality for the ingress-controller as Keepalived provides for the API endpoint.

In this section, we install and configure MetalLB in [Layer 2 mode](https://metallb.universe.tf/concepts/layer2/). In the next section, we create an ingress controller and mark it as type LoadBalancer. MetalLB then assigns the ingress controller a `LoadBalancer Ingress` address from the pool.

## Preparation
If you’re using kube-proxy in IPVS mode (iptables is the default), you have to enable strict ARP mode.   
`kubectl logs kube-proxy-xxxxx -n kube-system | grep "proxy mode"`

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
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
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
