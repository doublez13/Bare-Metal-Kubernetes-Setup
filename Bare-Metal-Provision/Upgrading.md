# Upgrading your Kubernetes Cluster

1. Read the CHANGELOG file! Seriously
2. Upgrade control plane nodes
    1. Upgrade kubeadm 
    2. `kubeadm upgrade plan`
    3. `kubeadm upgrade apply v1.21.0`
    4. Check if your CNI has any special upgrade instructions
    5. `kubeadm upgrade node` on any additional control plane nodes
    6. Upgrade kubelet and kubectl
    7. Restart kubelet service
3. Upgrade worker nodes 
    1. Upgrade kubeadm on the worker node
    2. Drain the node `kubectl drain <node-to-drain> --ignore-daemonsets`
    3. Upgrade kubelet and kubectl on the worker node
    4. Restart kubelet service
    5. Uncordon the node `kubectl uncordon <node-to-drain>`
