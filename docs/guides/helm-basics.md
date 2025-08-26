# Helm Basics

Helm simplifies Kubernetes deployments by packaging manifests into charts.

## Install
```bash
choco install kubernetes-helm -y   # Windows

helm create myapp
helm upgrade --install myapp ./chart -n dev
helm rollback myapp 1 -n dev
Typical values
image repo & tag

replicas

service type (ClusterIP, NodePort, LoadBalancer)

env vars & secrets