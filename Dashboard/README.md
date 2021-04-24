# Web UI (Dashboard)

## Installation
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml`

We first create a Service Account called admin, and grant that Service Account the permissions in the `cluster-admin` role.
```
kubectl apply -f admin-user.yaml
```


We then configure an ingress for the dashboard, and a certificate issuer.
```
kubectl apply -f production-issuer.yaml
kubectl apply -f dashboard-ingress.yaml
```

Using cert-manager.io, a separate certificate issuer is required for each namespace. The dashboard is under the `kubernetes-dashboard` namespace. The issuer file has been updated accordingly.

## Optional Metrics Server
`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

### Error message after deploy ###
`unable to fully scrape metrics from node because it doesn't contain any IP SANs`  
`unable to fully scrape metrics from node. x509: certificate signed by unknown authority`  
This occurs when when the metrics-server tries to connect to the kubelet server on each node. By default, the kubeletes are running with self signed certs. The metrics server will not trust these by default. You can either start the metrics server with the `--kubelet-insecure-tls` or TLS bootstrap the nodes. The [Bare-Metal-Provision](../Bare-Metal-Provision) section has instructions for TLS bootstrapping. If you've already set up a cluster, you can just set the `serverTLSBootstrap: true` option in the kubelet config.yaml file, restart the kubeletes, and then approve the CSRs as instructed in the Bare-Metal-Provision section.
