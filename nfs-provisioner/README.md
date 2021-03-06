# NFS Subdirectroy Provisioner #

[The NFS Subdirectroy Provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner) automatically creates PVs on an existing NFS server whenever a PVC with the appropriate storageClass is created.  

NOTE: Make sure the nfs client libs are installed on all the nodes. Ex: `nfs-common`

## Helm based install
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -n nfs-subdir-external-provisioner --create-namespace -f values.yaml
```
