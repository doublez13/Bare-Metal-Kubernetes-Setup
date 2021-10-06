# Dynamic Volume Provisioning

NOTE: Make sure the nfs client libs are installed on all the nodes. Ex: `nfs-common`

This example uses an existing NFS server to provision volumes.
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -f values.yaml
```
