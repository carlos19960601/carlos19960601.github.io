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

```yaml
expose:
  type: ingress
  tls:
    # Enable TLS or not.
    # Delete the "ssl-redirect" annotations in "expose.ingress.annotations" when TLS is disabled and "expose.type" is "ingress"
    # Note: if the "expose.type" is "ingress" and TLS is disabled,
    # the port must be included in the command when pulling/pushing images.
    # Refer to https://github.com/goharbor/harbor/issues/5291 for details.
    enabled: true
    # The source of the tls certificate. Set as "auto", "secret"
    # or "none" and fill the information in the corresponding section
    # 1) auto: generate the tls certificate automatically
    # 2) secret: read the tls certificate from the specified secret.
    # The tls certificate can be generated manually or by cert manager
    # 3) none: configure no tls certificate for the ingress. If the default
    # tls certificate is configured in the ingress controller, choose this option
    certSource: secret
    auto:
      # The common name used to generate the certificate, it's necessary
      # when the type isn't "ingress"
      commonName: ""
    secret:
      # The name of secret which contains keys named:
      # "tls.crt" - the certificate
      # "tls.key" - the private key
      secretName: "tls-harbor"
      # The name of secret which contains keys named:
      # "tls.crt" - the certificate
      # "tls.key" - the private key
      # Only needed when the "expose.type" is "ingress".
      notarySecretName: "tls-harbor"
  ingress:
    hosts:
      core: harbor.dmtech.com
      notary: notary.dmtech.com

# The external URL for Harbor core service. It is used to
# 1) populate the docker/helm commands showed on portal
# 2) populate the token service URL returned to docker/notary client
#
# Format: protocol://domain[:port]. Usually:
# 1) if "expose.type" is "ingress", the "domain" should be
# the value of "expose.ingress.hosts.core"
# 2) if "expose.type" is "clusterIP", the "domain" should be
# the value of "expose.clusterIP.name"
# 3) if "expose.type" is "nodePort", the "domain" should be
# the IP address of k8s node
#
# If Harbor is deployed behind the proxy, set it as the URL of proxy
externalURL: https://harbor.dmtech.com

persistence:
  enabled: true
  # Setting it to "keep" to avoid removing PVCs during a helm delete
  # operation. Leaving it empty will delete PVCs after the chart deleted
  # (this does not apply for PVCs that are created for internal database
  # and redis components, i.e. they are never deleted automatically)
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      # Use the existing PVC which must be created manually before bound,
      # and specify the "subPath" if the PVC is shared with other components
      existingClaim: "harbor-registry-pvc-local"
      # Specify the "storageClass" used to provision the volume. Or the default
      # StorageClass will be used (the default).
      # Set it to "-" to disable dynamic provisioning
      storageClass: ""
      subPath: ""
      accessMode: ReadWriteOnce
      size: 100Gi
      annotations: {}
    chartmuseum:
      existingClaim: "harbor-chartmuseum-pvc-local"
      storageClass: ""
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi
      annotations: {}
    jobservice:
      existingClaim: "harbor-jobservice-pvc-local"
      storageClass: ""
      subPath: ""
      accessMode: ReadWriteOnce
      size: 1Gi
      annotations: {}
    # If external database is used, the following settings for database will
    # be ignored
    database:
      existingClaim: "harbor-database-pvc-local"
      storageClass: ""
      subPath: ""
      accessMode: ReadWriteOnce
      size: 1Gi
      annotations: {}
    # If external Redis is used, the following settings for Redis will
    # be ignored
    redis:
      existingClaim: "harbor-redis-pvc-local"
      storageClass: ""
      subPath: ""
      accessMode: ReadWriteOnce
      size: 1Gi
      annotations: {}
    trivy:
      existingClaim: "harbor-trivy-pvc-local"
      storageClass: ""
      subPath: ""
      accessMode: ReadWriteOnce
      size: 5Gi
      annotations: {}
```

<!--more-->

harbor-pv-local-claim.yaml:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: harbor-registry-pvc-local
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: harbor-chartmuseum-pvc-local
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: harbor-jobservice-pvc-local
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: harbor-database-pvc-local
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: harbor-redis-pvc-local
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: harbor-trivy-pvc-local
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage

```

harbor-pv-local.yaml:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-registry-pv-local
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/s4/harbor/registry
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - carlos-k8s-master
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-chartmuseum-pv-local
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/s4/harbor/chartmuseum
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - carlos-k8s-master
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-jobservice-pv-local
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/s4/harbor/jobservice
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - carlos-k8s-master
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-database-pv-local
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/s4/harbor/database
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - carlos-k8s-master
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-redis-pv-local
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/s4/harbor/redis
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - carlos-k8s-master
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-trivy-pv-local
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /opt/s4/harbor/trivy
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - carlos-k8s-master
```

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