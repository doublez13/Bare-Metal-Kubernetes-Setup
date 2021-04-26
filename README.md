# Bare Metal Kubernetes Cluster Setup

This is a very basic setup (with example configs) for setting up a bare metal Kubernetes cluster. This is really just an organized collection of my notes and configs, but others can use these as a starting point. 
![alt text](src/Bare-Metal-Kube.png)

## Configuration:
### [Configuring API High Availability](HA-API)
### [Provisioning a Bare Metal Cluster](Bare-Metal-Provision)
### [Configuring a Bare Metal Load Balancer](MetalLB)
### [Configuring an Ingress Controller with SSL](NGINX-ingress)
### [Configuring the Kubernetes Dashboard with Metrics](Dashboard)

## Components
1. [Keepalived](https://www.keepalived.org/): VRRP Software for high-availability
2. [HAProxy](https://www.haproxy.org/): Load-balancer for Kubernetes API
3. [MetalLB](https://metallb.universe.tf/): Provides a highly-available IP address for all external traffic entering the cluster
4. [Ingress-NGINX](https://kubernetes.github.io/ingress-nginx/): Ingress controller that acts as a reverse proxy, and handles SSL termination
5. [cert-manager](https://cert-manager.io/): Automates TLS certificates
6. [Dashboard](https://github.com/kubernetes/dashboard): Official web UI for Kubernetes clusters
7. [Metrics Server](https://github.com/kubernetes-sigs/metrics-server): Provides resource usage data


