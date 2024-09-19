# Bare Metal Provision with High Availability 

This section provisions a fresh Kubernetes cluster using [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/). These steps should work fine on both Debian and RedHat based distros. This configuration assumes you already have an HA-API configured, as discussed in the [previous section](../HA-API/).

## Optional
1. Enable NTP: `systemctl enable --now systemd-timesyncd`
2. Disable IPv6 if you're not using it. It just makes things easier to toubleshoot in my opinion.  
   Append the following lines to `/etc/sysctl.conf`
   ```
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
   ```
3. Enable Secure Boot
    1. Verify secure boot is enabled: `mokutil --sb-state`
    2. Verify lockdown is integrity: `cat /sys/kernel/security/lockdown`
4. Set journald max size with `sed -i 's/#SystemMaxUse=/SystemMaxUse=1G/' /etc/systemd/journald.conf`
5. Network redundancy (note that bond-mode 4 is LACP and requires switch config as well)
   ```
   #apt install ifenslave
   #cat /etc/network/interfaces
   
   auto eno1
   iface eno1 inet manual
        bond-master bond0
        bond-mode 4
   auto eno2
   iface eno2 inet manual
        bond-master bond0
        bond-mode 4

   auto bond0
   iface bond0 inet static
        bond-slaves eno1 eno2
        bond-mode 4
        address ADDRESS/MASK
        gateway GATEWAY
        dns-nameservers DNS_SERVERS
        dns-search DNS_SEARCH_DOMAINS
   ``` 

## Prerequisites
1. Disable swap
    1. Remove swap references from /etc/fstab 
    2. Reboot, or deactive the active swap with `swapoff -a` 
2. Install iptables/nftables and enable it to start on boot
    1. `systemctl enable nftables --now`

## Install a container runtime
As the [Dockershim CRI is now deprecated](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/), containerd is a good choice to use.
1. Add the [Docker repo](https://docs.docker.com/engine/install/) (provides the containerd packages).
2. Install `containerd.io` and enable it to start on boot.
3. Cgroups Config:
    1. **Kubernetes Cgroup Driver:** As of 1.21, Kubernetes [uses the `systemd` cgroup driver by default](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md#no-really-you-must-read-this-before-you-upgrade), but we'll specify it in `provision.yaml` as well.
    2. **Systemd Cgroup Version:** As of Debian 11, systemd [defaults to using control groups v2.](https://www.debian.org/releases/bullseye/amd64/release-notes/ch-whats-new.en.html#cgroupv2)
    2. **Containerd Cgroup Version:** The default value of `runtime type` is `io.containerd.runc.v2`, which means cgroups v2.
    3. **Containerd Cgroup Driver:** [Set containerd to use the `SystemdCgroup` driver.](https://github.com/containerd/containerd/issues/4203#issuecomment-651532765) 
        ```
        containerd config default > /etc/containerd/config.toml
        ```
        ```
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
          SystemdCgroup = true
        ```
        Starting with containerd 1.5, the cgroup driver and version can be verified as follows. A bug in versions < 1.5 produces the wrong output. The `crictl` command will be available after installing the Kubernetes packages.
        ```
        # crictl -r unix:///run/containerd/containerd.sock info | grep runtimes -A 29
        "runtimes": {
          "runc": {
            "runtimeType": "io.containerd.runc.v2",
            "runtimePath": "",
            "runtimeEngine": "",
            "PodAnnotations": [],
            "ContainerAnnotations": [],
            "runtimeRoot": "",
            "options": {
              "BinaryName": "",
              "CriuImagePath": "",
              "CriuPath": "",
              "CriuWorkPath": "",
              "IoGid": 0,
              "IoUid": 0,
              "NoNewKeyring": false,
              "NoPivotRoot": false,
              "Root": "",
              "ShimCgroup": "",
              "SystemdCgroup": true
            },
            "privileged_without_host_devices": false,
            "privileged_without_host_devices_all_devices_allowed": false,
            "baseRuntimeSpec": "",
            "cniConfDir": "",
            "cniMaxConfNum": 0,
            "snapshotter": "",
            "sandboxMode": "podsandbox"
          }
        },
        ```

## Make sure the required modules load on boot
Add the following modules to a conf file in `/etc/modules-load.d`. Ex: `/etc/modules-load.d/k8.conf`
```
overlay
br_netfilter
```

## Set sysctl parameters
Add the following parameters to a conf file in `/etc/sysctl.d`. Ex: `/etc/sysctl.d/99-k8s.conf`
```
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
```

Load the new paramters with `sysctl --system`

## Install Kubernetes packages
Add the [Kubernetes repo](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) and install the following packages.
1. `kubelet`: The component that runs on all of the nodes in the cluster and does things like starting pods and containers.
2. `kubeadm`: The command to bootstrap the cluster. 
3. `kubectl`: The command line util to talk to your cluster.

## Clone the VM
1. Remove SSH keys.
    1. `rm /etc/ssh/ssh_host*`
2. Shutdown system and clone. In vSphere environments, this VM could be converted into a template. This same image can be used for both the controllers and workers.
3. On Debian distros, you'll need to regenerate SSH keys manually
    1. `dpkg-reconfigure openssh-server`
4. Change the IPs and hostnames of the clones
5. Verify `/sys/class/dmi/id/product_uuid` is uniqe on every host

## Initialize Kubernetes cluster
1. Provision the cluster on a to-be master
    1. `kubeadm init --config provision.yaml --upload-certs`
    2. Copy the kubeconfig to the correct user account
    3. Install a network addon, paying attention to Network Policy support. Calico is a good option:
       1. Install the Operator: `kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml`
       2. Download the custom resources: `curl https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml -O`
       3. Customize if necessary
       4. Create the manifest: `kubectl create -f custom-resources.yaml` 
       5. [Install calicoctl](https://docs.projectcalico.org/getting-started/clis/calicoctl/install)
       6. When it comes time to upgrade Calico, instructions can be found [here.](https://docs.tigera.io/calico/3.25/operations/upgrading/kubernetes-upgrade#upgrading-an-installation-that-uses-the-operator)
    4. Approve the kubelet CSRs for the new nodes.
       1. `kubectl get csr`
       2. `kubectl certificate approve <name>`
    5. `kubectl get nodes` should now show the new master node as `Ready`
       ```
       NAME      STATUS   ROLES                  AGE   VERSION
       node-01   Ready    control-plane,master   1h   v1.20.5
       ```
2. Join the other master nodes
    1. On the already-running master
       1. Reupload control plane certs and print the decryption key to retrieve them on the other master nodes.  
          `kubeadm init phase upload-certs --upload-certs`
       3. Print the join command to use on the other master nodes.  
          `kubeadm token create --print-join-command`
    3. Paste the join command with `--control-plane --certificate-key xxxx` appended, on each to-be master
    4. Approve the CSRs for the new master nodes.
3. Join the other worker nodes
    1. `kubeadm token create --print-join-command`
    2. Approve the CSRs for the new worker nodes.
4. Verify
    1. `kubectl get nodes` should now show all nodes as `Ready`
       ```
       NAME      STATUS   ROLES                  AGE   VERSION
       node-01   Ready    control-plane,master   1h   v1.20.5
       node-02   Ready    control-plane,master   1h   v1.20.5
       node-03   Ready    control-plane,master   1h   v1.20.5
       node-04   Ready    <none>                 1h   v1.20.5
       node-05   Ready    <none>                 1h   v1.20.5
       node-06   Ready    <none>                 1h   v1.20.5
       ```
    2. `kubectl get pods --all-namespaces` should show all pods as `Running`

## Run the [Sonobuoy](https://github.com/vmware-tanzu/sonobuoy) conformance test
1. NOTE: If this exits within a couple minutes, it most likely timed out connecting to the API or looking up a name in CoreDNS. 
2. Start the tests. They take awhile: `sonobuoy run --wait`
3. Watch the logs in another window: `kubectl logs sonobuoy --namespace sonobuoy -f`
4. Get the results: `results=$(sonobuoy retrieve)`
5. View the results: `sonobuoy results $results`
6. Delete the tests: `sonobuoy delete --wait`

## Run the [kube-bench](https://github.com/aquasecurity/kube-bench) security conformance tests
1. `kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml`
2. Wait until `kubectl get pods | grep kube-bench` shows `Completed`
3. `kubectl logs kube-bench-xxxxx | less`
4. `kubectl delete job kube-bench`
