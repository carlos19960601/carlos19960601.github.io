---
title: "Helm Install Gitea"
date: 2022-05-08T10:06:35+08:00
draft: true
original: true
categories: 
  - Kubernetes
tags: 
  - gitea
---


```
helm repo add gitea https://dl.gitea.io/charts
```


```
helm show values gitea-charts/gitea > values.yaml
```

<!--more-->

```yaml
ingress:
  enabled: true
  className: nginx
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: gitea.dmtech.com
      paths:
        - path: /
          pathType: Prefix
  tls: 
   - secretName: tls-gitea
     hosts:
       - gitea.dmtech.com
persistence:
  enabled: true
  existingClaim: "gitea-pvc-local"
  size: 10Gi

postgresql:
  enabled: true
  global:
    postgresql:
      postgresqlDatabase: gitea
      postgresqlUsername: gitea
      postgresqlPassword: gitea
      servicePort: 5432
  persistence:
    size: 10Gi
    enabled: true
    existingClaim: gitea-pvc-local
```

gitea-pv-local-claim.yaml:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitea-pvc-local
  namespace: gitea
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-storage
```

gitea-pv-local.yaml:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitea-pv-local
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/s4/gitea
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - carlos-k8s-master
```

```
kubectl create namespace gitea
```

```
kubectl create secret tls tls-gitea --cert=gitea.crt --key=gitea.key -n gitea
```

```
helm install -f values.yaml --namespace gitea gitea gitea-charts/gitea
```