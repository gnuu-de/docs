GNUU Infrastructure Documentation
=================================

K3S
---


Cert-Manager
------------

```
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager --namespace cert-manager jetstack/cert-manager
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.crds.yaml
kubectl apply -f https://raw.githubusercontent.com/gnuu-de/k8s/master/clusterissuer.yaml
```

