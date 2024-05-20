# NGINX Ingress Controller

## Redundancy 
If the node running the nginx ingress controller crashes, it would take five minutes (default pod-eviction-timeout) for the ingress controller to be rescheduled on another node. This would result in a five minute downtime to all containers behind the ingress controller.  

To work around this, we scale up the ingress-nginx-controller deployment to two replicas. This allows MetalLB to move the IP address to the node running the other replica.  

To ensure the pods are scheduled on separate nodes, an anti-affinity rule can be added under the template section of the spec.  
```
affinity: 
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app.kubernetes.io/name
            operator: In
            values:
            - ingress-nginx
          - key: app.kubernetes.io/instance
            operator: In
            values:
            - ingress-nginx
          - key: app.kubernetes.io/component
            operator: In
            values:
            - controller
        topologyKey: kubernetes.io/hostname
```
## Helm based install
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace -f values.yaml

#To upgrade to the latest chart
helm repo update
helm upgrade ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx

#To revert to a prior release
helm history ingress-nginx -n ingress-nginx
helm rollback helm history ingress-nginx -n ingress-nginx 0
```

## Pod Security Standard
NOT CURRENTLY WORKING: THE ADMISSION WEBHOOK WON'T RUN. CURRENTLY RUNNING IN BASELINE.

After verifying the ingress controller works as expected, you can apply the restricted Pod Security Standard to the namespace.
```
kubectl label --dry-run=server --overwrite ns ingress-nginx pod-security.kubernetes.io/enforce=restricted
```
If there are no errors on the dry run, apply the policy.
```
kubectl label --overwrite ns ingress-nginx pod-security.kubernetes.io/enforce=restricted
kubectl label --overwrite ns ingress-nginx pod-security.kubernetes.io/enforce-version=v1.24
```

# Cert-Manager
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.12.2/cert-manager.yaml
wget https://cert-manager.io/docs/tutorials/acme/example/production-issuer.yaml
wget https://cert-manager.io/docs/tutorials/acme/example/staging-issuer.yaml
```
Edit the email address used for Let's Encrypt.  
```
kubectl apply -f production-issuer.yaml
kubectl apply -f staging-issuer.yaml
```
Check the issuer status.  
```
kubectl get issuer
NAME                  READY   AGE
letsencrypt-prod      True    29m
letsencrypt-staging   True    21m
```

## Troubleshooting
```
kubectl get issuer
kubectl get certificates
kubectl get certificaterequest
kubectl get order
kubectk get challenge
```

## Upgrading
Check out the [upgrade notes](https://cert-manager.io/docs/installation/upgrading/) for a particular target release.

Apply the target manifest file.
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/<version>/cert-manager.yaml
```

# Notes
## Session affinity:
https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/
