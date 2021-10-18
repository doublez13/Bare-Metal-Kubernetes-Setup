# Network Policies

### The Key Points
- You must run a network plugin that supports network policies.
- By default, no network policies are applied to any pods, and all communication between pods is permitted.
- Network policies only permit traffic.
- Once at least one ingress/egress network policy targets a pod, all respective ingress/egress traffic to that pod is blocked unless permitted by a network policy.

### Examples ###
`default-deny-egress.yaml`: Set a default deny egress policy for all pods in the default namespace.  
`default-deny-ingress.yaml`: Set a default deny ingress policy for all pods in the default namespace.  
`dns-egress.yaml`: Allow egress dns traffic for all pods in the default namespace.  
`example-web-stack.yaml`: Allow all communication between pods with a particular label.  
`nfs-provisioner-egress.yaml`: Allow the nfs-provisioner to reach the API.  
`nginx-ingress-controller.yaml` Allow the nginx-ingress-controller to reach web ports in the default namespace.  

### Watch Out For DNS
When defining an egress network policy for a container, make sure that you allow DNS access to the kube-dns service, and pods may need this to look up the IPs of other pods/services in the cluster.
