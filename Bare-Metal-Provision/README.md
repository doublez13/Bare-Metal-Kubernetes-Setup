# Bare Metal Provision with High Availability 

This section provisions a fresh Kubernetes cluster using [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/). These steps should work fine on both Debian and RedHat based distros. This configuration assumes you already have an HA-API configured, as discussed in the [previous section](../HA-API/).

## Prerequisites
1. Disable swap
2. Disable IPv6 if you're not using it. It just makes things easier to toubleshoot in my opinion.
3. Firewall
    1. IPTables works great, and is what I use. I've read Firewalld works okay as well.
    2. Create iptables rules, and set them to load on boot (the `iptables.up.rules` file)

## Install a container runtime
As the [Dockershim CRI is now deprecated](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/), containerd is a good choice to use.
1. Add the [Docker repo](https://docs.docker.com/engine/install/) (provides the containerd packages)
2. Install containerd
3. As of 1.21, Kubernetes uses the `systemd` cgroup driver by default, but containerd still needs to be set to use it.
    ```
     [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
       SystemdCgroup = true
    ```

## Make sure the required modules load on boot
Add the following modules to a conf file in `/etc/modules-load.d`. Ex: `/etc/modules-load.d/k8.conf`
1. `overlay`
2. `br_netfilter`

## Set sysctl parameters
Add the following parameters to a conf file in `/etc/sysctl.d`. Ex: `/etc/sysctl.d/99-k8.conf`
1. `net.bridge.bridge-nf-call-iptables  = 1`
2. `net.ipv4.ip_forward                 = 1`
3. `net.bridge.bridge-nf-call-ip6tables = 1`

Load the new paramters with `sysctl --system`

## Install Kubernetes packages
Add the [Kubernetes repo](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) repo and install the following packages.
1. `kubelet`: The command to bootstrap the cluster.
2. `kubeadm`: The component that runs on all of the machines in your cluster and does things like starting pods and containers.
3. `kubectl`: The command line util to talk to your cluster

## Clone the VM
1. Remove SSH keys.
    1. `rm /etc/ssh/ssh_host*`
2. Shutdown system and clone
3. On Debian distros, you'll need to regenerate SSH keys manually
    1. `dpkg-reconfigure openssh-server`
4. Change the IPs and hostnames of the clones
5. Verify `/sys/class/dmi/id/product_uuid` is uniqe on every host

## Initialize Kubernetes cluster
1. `kubeadm init --config provision.yaml --upload-certs` on a to-be master
2. Copy the kubeconfig to the correct user account
3. Install a network addon. [Flannel](https://github.com/flannel-io/flannel) generally works out of the box:
    1. `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`
4. `kubectl get nodes` should now show the new master node as `Ready`
    ```
    NAME      STATUS   ROLES                  AGE   VERSION
    node-01   Ready    control-plane,master   1h   v1.20.5
    ```
6. Join the other master nodes
    1. On the already-running master
       1. `kubeadm init phase upload-certs --upload-certs`
       2. `kubeadm token create --print-join-command`
    3. Paste the join command with `--control-plane --certificate-key xxxx` appended, on each to-be master
    4. Approve the CSRs for the new nodes.
       1. `kubectl get csr`
       2. `kubectl certificate approve <name>`
7. Join the other worker nodes
    1. `kubeadm token create --print-join-command`
    2. Approve the CSRs for the new nodes `kubectl get csr` and then `kubectl certificate approve <name>`
8. `kubectl get nodes` should now show all nodes as `Ready`
   ```
   NAME      STATUS   ROLES                  AGE   VERSION
   node-01   Ready    control-plane,master   1h   v1.20.5
   node-02   Ready    control-plane,master   1h   v1.20.5
   node-03   Ready    control-plane,master   1h   v1.20.5
   node-04   Ready    <none>                 1h   v1.20.5
   node-05   Ready    <none>                 1h   v1.20.5
   node-06   Ready    <none>                 1h   v1.20.5
   ```
8. `kubectl get pods --all-namespaces` should show all pods as `Running`
9. Reboot all nodes for good measure.

## Run the [Sonobuoy](https://github.com/vmware-tanzu/sonobuoy) conformance test
1. NOTE: If this exits within a couple minutes, it most likely timed out connecting to the API or looking up a name in CoreDNS. 
2. Start the tests. They take awhile:`sonobuoy run --wait`
3. Watch the logs in another window: `kubectl logs sonobuoy --namespace sonobuoy -f`
4. Get the results: `results=$(sonobuoy retrieve)`
5. View the results: `sonobuoy results $results`
6. Delete the tests: `sonobuoy delete --wait`

## Run the [kube-bench](https://github.com/aquasecurity/kube-bench) security conformance tests
1. `kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml`
2. Wait until `kubectl get pods | grep kube-bench` shows `Completed`
3. `kubectl logs kube-bench-xxxxx | less`
4. `kubectl delete pod kube-bench-xxxxx`
