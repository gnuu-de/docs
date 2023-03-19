## 2.0

19.03.2023

* System upgrade [5.4.0-144-generic](https://www.ubuntuupdates.org/package/core/focal/main/proposed/linux-modules-5.4.0-144-generic)

* Upgrade Kubernetes [1.24.8](https://github.com/k3s-io/k3s/releases/tag/v1.24.8%2Bk3s1)

* Move and rebuild all Docker images from docker.io registry to ghcr.io [v2.0.0](https://github.com/gnuu-de/dockerfiles/releases/tag/v2.0.0). Docker Free Team "gnuu" will removed soon, see [Docker Blog](https://www.docker.com/blog/we-apologize-we-did-a-terrible-job-announcing-the-end-of-docker-free-teams/)

* Install [Cosignwebhook](https://github.com/eumel8/cosignwebhook)

* Redeploy all apps from ghcr.io and verify container images ([#3](https://github.com/gnuu-de/k8s/pull/3))

* Upgrade Matrix [1.11.25](https://github.com/vector-im/element-web/releases/tag/v1.11.25)

* Upgrade Rancher [2.7.1](https://github.com/rancher/rancher/releases/tag/v2.7.1)
