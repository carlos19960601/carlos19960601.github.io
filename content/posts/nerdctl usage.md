---
title: "Nerdctl Usage"
date: 2022-05-07T21:58:32+08:00
draft: true
original: true
categories: 
  - tool
tags: 
  - nerdctl
---


tag image
```
sudo nerdctl -n k8s.io tag k8s.gcr.io/ingress-nginx/controller:v1.2.0 harbor.dmtech.com/ingress-nginx/controller:v1.2.0
```


push image to private registry
```
$ sudo nerdctl -n k8s.io push harbor.dmtech.com/ingress-nginx/controller:v1.2.0

INFO[0000] pushing as a reduced-platform image (application/vnd.docker.distribution.manifest.list.v2+json, sha256:28fd2b8e6c8f74893a743ee7c92f99b229726a33a9060ed2cd9a99e161a5347c) 
index-sha256:28fd2b8e6c8f74893a743ee7c92f99b229726a33a9060ed2cd9a99e161a5347c:    waiting        |--------------------------------------| 
manifest-sha256:2746adf7d60c782b83f6fa6e1ceb938878b9f5b16c217e530ba8895f94221a04: waiting        |--------------------------------------| 
config-sha256:04fcc70194086eb9118c8a015dc455c0f7f0249b10346f8b03f97d86ae99fb0c:   waiting        |--------------------------------------| 
elapsed: 0.1 s                                                                    total:   0.0 B (0.0 B/s)                                         
FATA[0000] failed to do request: Head "https://harbor.dmtech.com/v2/ingress-nginx/controller/blobs/sha256:d5603763507267abab41c628516298bad883b788b992cb3cd68fef9c689100a3": x509: certificate signed by unknown authority 
carlos@carlos-k8s-master:/etc/containerd/certs.d/harbor.dmtech.com$ 
```

<!--more-->


login fail
```
$ sudo nerdctl login harbor.dmtech.com
ERRO[0000] failed to call tryLoginWithRegHost            error="failed to call rh.Client.Do: Get \"https://harbor.dmtech.com/v2/\": x509: certificate signed by unknown authority" i=0
```

```
sudo vim /etc/containerd/config.toml
```
```
[plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/containerd/certs.d"
```

```
sudo mkdir -p /etc/containerd/certs.d/harbor.dmtech.com
```

```
$ sudo vim /etc/containerd/certs.d/harbor.dmtech.com/hosts.toml
```

```
server = "https://harbor.dmtech.com"

[host."https://harbor.dmtech.com"]
  ca = "/home/carlos/Documents/harbor.crt"
```


push image success
```
$ sudo nerdctl -n k8s.io push harbor.dmtech.com/ingress-nginx/controller:v1.2.0

INFO[0000] pushing as a reduced-platform image (application/vnd.docker.distribution.manifest.list.v2+json, sha256:28fd2b8e6c8f74893a743ee7c92f99b229726a33a9060ed2cd9a99e161a5347c) 
index-sha256:28fd2b8e6c8f74893a743ee7c92f99b229726a33a9060ed2cd9a99e161a5347c:    done           |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:2746adf7d60c782b83f6fa6e1ceb938878b9f5b16c217e530ba8895f94221a04: done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:04fcc70194086eb9118c8a015dc455c0f7f0249b10346f8b03f97d86ae99fb0c:   done           |++++++++++++++++++++++++++++++++++++++| 
elapsed: 0.8 s                                                                    total:  14.1 K (17.6 KiB/s)   
```




