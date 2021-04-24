# NGINX Ingress Controller
We use the AWS service provider files, as that configures the service as LoadBalancer.  
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.45.0/deploy/static/provider/aws/deploy.yaml`

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
