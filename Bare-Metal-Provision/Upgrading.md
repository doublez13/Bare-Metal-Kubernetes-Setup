# Upgrading your Kubernetes Cluster
Kubernetes versions take the format Major.Minor.Patch (1.24.2)

1. Read the CHANGELOG file! Seriously
2. Upgrade each control plane node fully before moving to the next
    1. Upgrade kubeadm
       ```
       apt-mark unhold kubeadm
       apt-get update
       apt-get install -y kubeadm=1.24.x-00
       apt-mark hold kubeadm
       ```
    2. On first control plan node upgrade only
       ```
       kubeadm upgrade plan`
       kubeadm upgrade apply v1.24.x`
       ```
    3. On other control plan nodes only
       ```
       kubeadm upgrade node
       ```
    5. Upgrade kubelet and kubectl
       ```
       kubectl drain <node-to-drain> --ignore-daemonsets
       apt-mark unhold kubelet kubectl
       apt-get update
       apt-get install -y kubelet=1.24.x-00 kubectl=1.24.x-00
       apt-mark hold kubelet kubectl
       ```
    4. Restart kubelet service
       ```
       systemctl daemon-reload
       systemctl restart kubelet
       ```
    5. Uncordon the node
       ```
       kubectl uncordon <node-to-drain>
       ```
3. Upgrade worker nodes 
    1. Upgrade kubeadm on the worker nodes
    2. `kubeadm upgrade node`
    3. Drain the node `kubectl drain <node-to-drain> --ignore-daemonsets`
    4. Upgrade kubelet and kubectl on the worker node
    5. Restart kubelet service
    6. Uncordon the node `kubectl uncordon <node-to-drain>`
