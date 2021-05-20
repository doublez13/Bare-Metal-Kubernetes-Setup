# Upgrading your Kubernetes Cluster
Kubernetes version arin the format Major.Minor.Patch (1.21.1)

1. Read the CHANGELOG file! Seriously
2. Upgrade control plane nodes
    1. Upgrade kubeadm on all master nodes
    2. `kubeadm upgrade plan`
    3. `kubeadm upgrade apply v1.21.0`
    4. Upgrade kubelet and kubectl
    5. Restart kubelet service
    6. Check if your CNI has any special upgrade instructions
    7. `kubeadm upgrade node` on any additional control plane nodes
    8. Upgrade kubelet and kubectl
    9. Restart kubelet service
3. Upgrade worker nodes 
    1. Upgrade kubeadm on the worker nodes
    2. `kubeadm upgrade node`
    3. Drain the node `kubectl drain <node-to-drain> --ignore-daemonsets`
    4. Upgrade kubelet and kubectl on the worker node
    5. Restart kubelet service
    6. Uncordon the node `kubectl uncordon <node-to-drain>`
