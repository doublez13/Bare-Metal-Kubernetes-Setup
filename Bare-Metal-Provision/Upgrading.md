# Upgrading your Kubernetes Cluster
Kubernetes versions take the format Major.Minor.Patch (1.24.2)

1. Read the CHANGELOG file! Seriously
2. Upgrade first control plane nodes
    1. Upgrade kubeadm on all master nodes
       ```
       apt-mark unhold kubeadm
       apt-get update
       apt-get install -y kubeadm=1.24.x-00
       apt-mark hold kubeadm
       ```
    3. `kubeadm upgrade plan`
    4. `kubeadm upgrade apply v1.24.x`
    5. Upgrade kubelet and kubectl
       ```
       kubectl drain <node-to-drain> --ignore-daemonsets
       apt-mark unhold kubelet kubectl
       apt-get update
       apt-get install -y kubelet=1.24.x-00 kubectl=1.24.x-00
       apt-mark hold kubelet kubectl
       ```
    7. Restart kubelet service
    8. Check if your CNI has any special upgrade instructions
    9. `kubeadm upgrade node` on any additional control plane nodes
    10. Upgrade kubelet and kubectl
    11. Restart kubelet service
3. Upgrade worker nodes 
    1. Upgrade kubeadm on the worker nodes
    2. `kubeadm upgrade node`
    3. Drain the node `kubectl drain <node-to-drain> --ignore-daemonsets`
    4. Upgrade kubelet and kubectl on the worker node
    5. Restart kubelet service
    6. Uncordon the node `kubectl uncordon <node-to-drain>`
