---
title: "Nerdctl Build Image"
date: 2022-05-06T19:10:04+08:00
draft: false
original: true
categories: 
  - Kubernetes
tags: 
  - Kubernetes
---

download from https://github.com/moby/buildkit/releases

```
$ sudo mkdir /usr/local/buildkit
$ sudo tar zxvf buildkit-v0.10.2.linux-amd64.tar.gz -C /usr/local/buildkit/
$ tree /usr/local/buildkit/
/usr/local/buildkit/
└── bin
    ├── buildctl
    ├── buildkitd
    ├── buildkit-qemu-aarch64
    ├── buildkit-qemu-arm
    ├── buildkit-qemu-i386
    ├── buildkit-qemu-mips64
    ├── buildkit-qemu-mips64el
    ├── buildkit-qemu-ppc64le
    ├── buildkit-qemu-riscv64
    ├── buildkit-qemu-s390x
    └── buildkit-runc
```

<!--more-->

```
ln -s /usr/local/buildkit/bin/buildkitd /usr/local/bin/buildkitd
ln -s /usr/local/buildkit/bin/buildctl /usr/local/bin/buildctl
```

create systemd file

/etc/systemd/system/buildkit.socket:

```
[Unit]
Description=BuildKit
Documentation=https://github.com/moby/buildkit

[Socket]
ListenStream=%t/buildkit/buildkitd.sock
SocketMode=0660

[Install]
WantedBy=sockets.target
```

/etc/systemd/system/buildkit.service:

```
[Unit]
Description=BuildKit
Requires=buildkit.socket
After=buildkit.socket
Documentation=https://github.com/moby/buildkit

[Service]
Type=notify
ExecStart=/usr/local/bin/buildkitd --addr fd://

[Install]
WantedBy=multi-user.target
```

```
$ sudo systemctl enable --now buildkit
```

/etc/buildkit/buildkitd.toml:

```
[worker.oci]
  enabled = false

[worker.containerd]
  enabled = true
  # namespace should be "k8s.io" for Kubernetes (including Rancher Desktop)
  namespace = "default"
```

```
nerdctl build -t image-name .
```