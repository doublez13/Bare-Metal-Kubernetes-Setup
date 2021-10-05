# NGINX Ingress Controller

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
```

# Cert-Manager
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml
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

# Notes
## Session affinity:
https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/
