GNUU Infrastructure Documentation
=================================

Architecture Overview
---------------------

![GNUU-K8S.draw.svg](GNUU-K8S.draw.svg)

K3S
---

```
curl -sfL https://get.k3s.io | sh -

```
#extend node port range in /etc/systemd/system/k3s.service
#change

ExecStart=/usr/local/bin/k3s server --kube-apiserver-arg service-node-port-range=1-65535

```
systemctl daemon-reload
systemctl restart k3s.service

```
Verify config, enabled addons.


```
k3s check-config

Verifying binaries in /var/lib/rancher/k3s/data/3a8d3d90c0ac3531edbdbde77ce4a85062f4af8865b98cedc30ea730715d9d48/bin:
- sha256sum: good
- links: good

System:
- /sbin iptables v1.6.1: older than v1.8
- swap: disabled
- routes: ok

Limits:
- /proc/sys/kernel/keys/root_maxkeys: 1000000

modprobe: module configs not found in modules.dep
info: reading kernel config from /boot/config-4.15.0-106-generic ...

Generally Necessary:
- cgroup hierarchy: properly mounted [/sys/fs/cgroup]
- /sbin/apparmor_parser
apparmor: enabled and tools installed
- CONFIG_NAMESPACES: enabled
- CONFIG_NET_NS: enabled
- CONFIG_PID_NS: enabled
- CONFIG_IPC_NS: enabled
- CONFIG_UTS_NS: enabled
- CONFIG_CGROUPS: enabled
- CONFIG_CGROUP_CPUACCT: enabled
- CONFIG_CGROUP_DEVICE: enabled
- CONFIG_CGROUP_FREEZER: enabled
- CONFIG_CGROUP_SCHED: enabled
- CONFIG_CPUSETS: enabled
- CONFIG_MEMCG: enabled
- CONFIG_KEYS: enabled
- CONFIG_VETH: enabled (as module)
- CONFIG_BRIDGE: enabled (as module)
- CONFIG_BRIDGE_NETFILTER: enabled (as module)
- CONFIG_NF_NAT_IPV4: enabled (as module)
- CONFIG_IP_NF_FILTER: enabled (as module)
- CONFIG_IP_NF_TARGET_MASQUERADE: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_ADDRTYPE: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_CONNTRACK: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_IPVS: enabled (as module)
- CONFIG_IP_NF_NAT: enabled (as module)
- CONFIG_NF_NAT: enabled (as module)
- CONFIG_NF_NAT_NEEDED: enabled
- CONFIG_POSIX_MQUEUE: enabled

Optional Features:
- CONFIG_USER_NS: enabled
- CONFIG_SECCOMP: enabled
- CONFIG_CGROUP_PIDS: enabled
- CONFIG_BLK_CGROUP: enabled
- CONFIG_BLK_DEV_THROTTLING: enabled
- CONFIG_CGROUP_PERF: enabled
- CONFIG_CGROUP_HUGETLB: enabled
- CONFIG_NET_CLS_CGROUP: enabled (as module)
- CONFIG_CGROUP_NET_PRIO: enabled
- CONFIG_CFS_BANDWIDTH: enabled
- CONFIG_FAIR_GROUP_SCHED: enabled
- CONFIG_RT_GROUP_SCHED: missing
- CONFIG_IP_NF_TARGET_REDIRECT: enabled (as module)
- CONFIG_IP_SET: enabled (as module)
- CONFIG_IP_VS: enabled (as module)
- CONFIG_IP_VS_NFCT: enabled
- CONFIG_IP_VS_PROTO_TCP: enabled
- CONFIG_IP_VS_PROTO_UDP: enabled
- CONFIG_IP_VS_RR: enabled (as module)
- CONFIG_EXT4_FS: enabled
- CONFIG_EXT4_FS_POSIX_ACL: enabled
- CONFIG_EXT4_FS_SECURITY: enabled
- Network Drivers:
  - "overlay":
    - CONFIG_VXLAN: enabled (as module)
      Optional (for encrypted networks):
      - CONFIG_CRYPTO: enabled
      - CONFIG_CRYPTO_AEAD: enabled
      - CONFIG_CRYPTO_GCM: enabled
      - CONFIG_CRYPTO_SEQIV: enabled
      - CONFIG_CRYPTO_GHASH: enabled
      - CONFIG_XFRM: enabled
      - CONFIG_XFRM_USER: enabled (as module)
      - CONFIG_XFRM_ALGO: enabled (as module)
      - CONFIG_INET_ESP: enabled (as module)
      - CONFIG_INET_XFRM_MODE_TRANSPORT: enabled (as module)
- Storage Drivers:
  - "overlay":
    - CONFIG_OVERLAY_FS: enabled (as module)

STATUS: pass

```

Admin Clients
-------------

```
snap install kubectl --classic
snap install helm --classic
kubectl completion bash  >> .bashrc
helm completion bash  >> .bashrc
bash

```

Traefic/Ingress
---------------

```
helm -n kube-system list

NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
traefik kube-system     5               2020-06-29 16:44:29.673154471 +0000 UTC deployed        traefik-1.81.0  1.7.19
```

in `/var/lib/rancher/k3s/server/manifests/traefik.yaml` enable Let's Encrypt:

```
    acme:
      enabled: true

```


Cert-Manager
------------


```
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager --namespace cert-manager jetstack/cert-manager
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.crds.yaml
kubectl apply -f https://raw.githubusercontent.com/gnuu-de/k8s/master/clusterissuer.yaml
```

OpenEBS
-------


```
kubectl apply -f https://raw.githubusercontent.com/openebs/openebs/master/k8s/openebs-operator.yaml
kubectl apply -f https://raw.githubusercontent.com/gnuu-de/k8s/master/storageclass.yaml
```

MySQL
-----

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm -n gnuu upgrade -i mysql --set persistence.storageClass=openebs-standalone stable/mysql
kubectl get secret --namespace gnuu mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
```


MySQL User
----------

```
CREATE USER 'gnuuweb'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON gnuu.* TO 'gnuuweb'@'%';
flush privileges;
```


GNUU Application
----------------

### Container developement

 
All GNUU application used their [own Docker files](https://github.com/gnuu-de/dockerfiles).
Docker images are auto-build and published in [Docker Hub](https://hub.docker.com/orgs/gnuu).
Image names are common, like inn, postfix, but there are special adjustments for GNUU like
bsmtp backend for e-mail.

### GNUU Kubernetes deployment

All deployment manifests are in the [K8S repo](https://github.com/gnuu-de/k8s). There are core
deployments like storage or the default namespace and then in each folder app specific
deployments. A HELM chart is planned for the future, currently it's easier to deploy the
manifest per app. To let the ip-addresses of clients to the apps, the main PODs are running
with host-network and host-port. That's required for access limitations in mail & news.

TODO: set security context, pod security policy, more images as none-root

### GNUU web services

Static web content is delivered on behalf of nginx, Let's Encrypt and Ingress service. 
Source of the static content is the [www repo](https://github.com/gnuu-de/www).
The logic of the web services like user management or site configuration are in 
the [apps repo](https://github.com/gnuu-de/apps), There are Python Flask services
as replacement of Perl CGI and a Job API to update configuration files or configmaps
in Kubernetes. For that reason are service accounts created via RBAC.


### TODO's

* Backup concept/S3

* Monitoring

* Functional tests UUCP packer

* CI/CD pipeline

* Firewall/Server hardening

^

