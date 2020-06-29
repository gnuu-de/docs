GNUU Infrastructure Documentation
=================================

K3S
---

```
curl -sfL https://get.k3s.io | sh -

```
extend node port range in /etc/systemd/system/k3s.service
change

ExecStart=/usr/local/bin/k3s server --kube-apiserver-arg service-node-port-range=1-65535

```
systemctl daemon-reload
systemctl restart k3s.service

```


Cert-Manager
------------

```
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager --namespace cert-manager jetstack/cert-manager
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.crds.yaml
kubectl apply -f https://raw.githubusercontent.com/gnuu-de/k8s/master/clusterissuer.yaml
```

