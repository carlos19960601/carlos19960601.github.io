---
title: "Helm Install Harbor"
date: 2022-05-06T21:32:28+08:00
draft: false
original: true
categories: 
  - Kubernetes
tags: 
  - Kubernetes
  - Harbor
---

添加harbor仓库

```
helm repo add harbor https://helm.goharbor.io
```

打开values配置文件

```
$ helm show values harbor/harbor > values.yaml
```

<!--more-->

创建命名空间

```
kubectl create namespace harbor
```

添加证书到集群secret中

```
kubectl create secret tls tls-harbor --cert=harbor.crt --key=harbor.key -n harbor
```

安装harbor

```
helm install -f values.yaml --namespace harbor harbor harbor/harbor
```

查看pod

```
kubectl get pod -n harbor

harbor-chartmuseum-5586bf49d8-6lj9c     1/1     Running   0          19h
harbor-core-897494845-gqk5t             1/1     Running   0          19h
harbor-database-0                       1/1     Running   0          19h
harbor-jobservice-69987bf585-n6p75      1/1     Running   0          19h
harbor-notary-server-776fd5c5fc-tpgfr   1/1     Running   0          19h
harbor-notary-signer-78fd79ccff-qncwx   1/1     Running   0          19h
harbor-portal-97fcbbd96-6mf2b           1/1     Running   0          19h
harbor-redis-0                          1/1     Running   0          19h
harbor-registry-58ddfdfcbd-5hp7z        2/2     Running   0          19h
harbor-trivy-0                          1/1     Running   0          19h
```