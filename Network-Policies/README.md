# Network Policies

### The Key Points
- You must run a network plugin that supports network policies.
- By default, no network policies are applied to any pods, and all communication between pods is permitted.
- Network policies only permit traffic.
- Once at least one ingress/egress network policy targets a pod, all respective ingress/egress traffic to that pod is blocked unless permitted by a network policy.
