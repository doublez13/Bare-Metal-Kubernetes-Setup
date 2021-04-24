# API Endpoint High Availability

There is an excellent guide available for this in the [kubeadm repo](https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md).

To sum it up, we'll be using HAProxy as a load-balancer across the masters. However, having a single load balancer would again introduce a single point of failue in the control plane. To get around this, we configure a second HAProxy server that acts as a standby, and then use Keepalived (VRRP) to tie the active and standby load-balancers together with one IP address that we'll use as the API endpoint.

# Affinity
In this configuration, we have three groups of nodes: `Masters`, `Workers`, and `Master APIs`. Somee thought should be put into keeping members of the same group apart from each other. For example, if hosting on a vSphere cluster, affinity rules can be created that force the VMs to run on different hosts.
