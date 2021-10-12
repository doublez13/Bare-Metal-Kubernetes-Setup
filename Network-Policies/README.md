# Network Policies

### The Key Points
- You must run a network plugin that supports network policies.
- By default, no network policies are applied to any pods, and all communication between pods is permitted.
- Network policies only permit traffic.
- Once at least one ingress/egress network policy targets a pod, all respective ingress/egress traffic to that pod is blocked unless permitted by a network policy.

### Watch Out For DNS
When defining an egress network policy for a container, make sure that you allow DNS access to the kube-dns service, and pods may need this to look up the IPs of other pods/services in the cluster.
