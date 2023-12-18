# Upgrading your Kubernetes Cluster
Kubernetes versions take the format Major.Minor.Patch (1.24.2)

1. Read the CHANGELOG file! Seriously
2. Upgrade each control plane node fully before moving to the next
    1. Upgrade kubeadm
       ```
       #Change the kubernetes version in /etc/apt/sources.list.d/kubernetes.list
       apt-mark unhold kubeadm
       apt-get update
       apt-get upgrade -y kubeadm
       apt-mark hold kubeadm
       ```
    2. Upgrade node with kubeadm 
        1. On first control plan node upgrade only
           ```
           kubeadm upgrade plan
           kubeadm upgrade apply v1.27.8
           ```
        2. On other control plan nodes only
           ```
           kubeadm upgrade node
           ```
    5. Upgrade kubelet and kubectl
       ```
       kubectl drain <node-to-drain> --ignore-daemonsets
       apt-mark unhold kubelet kubectl
       apt-get update
       apt-get install -y kubelet kubectl
       apt-mark hold kubelet kubectl
       ```
    4. Restart kubelet service
       ```
       systemctl daemon-reload
       systemctl restart kubelet
       ```
    5. Upgrade Containerd
       ```
       apt-mark unhold containerd.io
       apt upgrade containerd.io
       apt-mark hold containerd.io
       ``` 
    6. Uncordon the node
       ```
       kubectl uncordon <node-to-drain>
       ```
3. Upgrade each worker node fully before moving to the next
    1. Upgrade kubeadm
       ```
       #Change the kubernetes version in /etc/apt/sources.list.d/kubernetes.list
       apt-mark unhold kubeadm
       apt-get update
       apt-get upgrade -y kubeadm
       apt-mark hold kubeadm
       ```
    2. Upgrade node with kubeadm
       ```
       kubeadm upgrade node
       ```
    3. Upgrade kubelet and kubectl
       ```
       kubectl drain <node-to-drain> --ignore-daemonsets
       apt-mark unhold kubelet kubectl
       apt-get update
       apt-get install -y kubelet kubectl
       apt-mark hold kubelet kubectl
       ```
    4. Restart kubelet service
       ```
       systemctl daemon-reload
       systemctl restart kubelet
       ```
    5. Upgrade Containerd
       ```
       apt-mark unhold containerd.io
       apt upgrade containerd.io
       apt-mark hold containerd.io
       ``` 
    5. Uncordon the node
       ```
       kubectl uncordon <node-to-drain>
       ```
