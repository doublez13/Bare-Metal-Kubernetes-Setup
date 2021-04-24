# Bare Metal Provision with High Availability 

 0. Disable swap
 1. Disable IPv6 if you're not using it. Just makes things easier to toubleshoot in my opinion.
 2. Disable firewalld
 3. install/enable iptables
 4. Create iptables rules, and set them to load on boot (the the `iptables.up.rules` file)
 5. Install container runtime
     1. Add Docker repo
     2. Install containerd
 8. As of 1.21, Kubernetes uses the `systemd` cgroup driver by default, but containerd still needs to be set to use it.
     ```
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true
     ```
 8. Make sure the required modules load on boot
     1. `overlay`
     2. `br_netfilter`
 9. Set sysctl parameters
     1. `net.bridge.bridge-nf-call-iptables  = 1`
     2. `net.ipv4.ip_forward                 = 1`
     3. `net.bridge.bridge-nf-call-ip6tables = 1`
 10. Add Kubernetes Repo
 11. Install Kubernetes packages
     1. `kubelet` `kubeadm` `kubectl`
 12. Clone the VM
     1. Remove SSH keys.
         1. `rm /etc/ssh/ssh_host*`
     2. Shutdown system and clone
     3. On Debian distros, you'll need to regenerate SSH keys manually
         1. `dpkg-reconfigure openssh-server`
     4. Change the IPs and hostnames of the clones
     5. Verify `/sys/class/dmi/id/product_uuid` is uniqe on every host
 13. Initialize Kubernetes cluster
     1. `kubeadm init --config provision.yml --upload-certs` on a to-be master
     2. Copy the kubeconfig to the correct user account
     3. Install network addon. I used Flannel:
         1. `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`
     4. `kubectl get nodes` should now show `Ready`
     5. Join the other master nodes
         1. On the already-running master
            1. `kubeadm init phase upload-certs --upload-certs`
            2. `kubeadm token create --print-join-command`
         3. Paste the join command with `--control-plane --certificate-key xxxx` appended, on each to-be master
         4. Approve the CSRs for the new nodes `kubectl get csr` and then `kubectl certificate approve <name>`
     6. Join the other worker nodes
         1. `kubeadm token create --print-join-command`
         2. Approve the CSRs for the new nodes `kubectl get csr` and then `kubectl certificate approve <name>`
     7. `kubectl get nodes` should now show all nodes as `Ready`
        ```
        NAME      STATUS   ROLES                  AGE   VERSION
        node-01   Ready    control-plane,master   23h   v1.20.5
        node-02   Ready    control-plane,master   23h   v1.20.5
        node-03   Ready    control-plane,master   23h   v1.20.5
        node-04   Ready    <none>                 23h   v1.20.5
        node-05   Ready    <none>                 23h   v1.20.5
        node-06   Ready    <none>                 23h   v1.20.5
        ```
     8. `kubectl get pods --all-namespaces` should show all pods as `Running`
     9. Reboot all nodes for good measure.
 14. Run the Sonobuoy conformance test
     1. Start the tests. They take awhile:`sonobuoy run --wait`
     2. Watch the logs in another window: `kubectl logs sonobuoy --namespace sonobuoy -f`
     3. Get the results: `results=$(sonobuoy retrieve)`
     4. View the results: `sonobuoy results $results`
     5. Delete the tests: `sonobuoy delete --wait`
 15. Run the kube-bench security conformance tests
     1. `kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml`
     2. Wait until `kubectl get pods | grep kube-bench` shows `Completed`
     3. `kubectl logs kube-bench-xxxxx | less`
     4. `kubectl delete pod kube-bench-xxxxx`
