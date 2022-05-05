---
title: "Helm Install Ingress Nginx"
date: 2022-05-04T17:19:35+08:00
draft: true
original: true
categories: 
  - Kubernetes
tags: 
  - Kubernetes
  - ingress-nginx
---

**添加ingress-nginx仓库**

```
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

**修改如下参数**

```
$ helm show values ingress-nginx/ingress-nginx > values.yaml
```

```yaml
controller:
  ## Use host ports 80 and 443
  ## Disabled by default
  hostPort:
    # -- Enable 'hostPort' or not
    enabled: true # 设置使用主机网络
    ports:
      # -- 'hostPort' http port
      http: 80
      # -- 'hostPort' https port
      https: 443

  ## This section refers to the creation of the IngressClass resource
  ## IngressClass resources are supported since k8s >= 1.18 and required since k8s >= 1.19
  ingressClassResource:
    # -- Name of the ingressClass
    name: nginx
    # -- Is this ingressClass enabled or not
    enabled: true
    # -- Is this the default ingressClass for the cluster
    default: true # 设置ingress-nginx为默认ingressClass控制器，否则使用ingress时需要指定使用nginx
```

<!--more-->


**创建命名空间**

```
kubectl create namespace ingress-nginx
```

**安装ingress-nginx**


```
helm install ingress-nginx ingress-nginx -n ingress-nginx

NAME: ingress-nginx
LAST DEPLOYED: Thu May  5 22:38:34 2022
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

**查看pod**

```
$ kubectl get pod -n ingress-nginx -o wide

NAME                                       READY   STATUS    RESTARTS   AGE   IP               NODE                NOMINATED NODE   READINESS GATES
ingress-nginx-controller-7dcb8777f-x8z76   1/1     Running   0          37m   10.244.103.130   carlos-k8s-master   <none>           <none>
```

**遇到的问题**

之前在安装MySQL的时候没有遇到 pull image 失败的情况，在安装ingress-nginx的时候需要gcr的镜像，所以需要配置一下containerd，下载镜像的时候可以走代理
```
$ cat <<EOF >/etc/systemd/system/containerd.service.d/http-proxy.conf    
[Service]    
Environment="HTTP_PROXY=${HTTP_PROXY:-}"    
Environment="HTTPS_PROXY=${HTTPS_PROXY:-}"    
Environment="NO_PROXY=${NO_PROXY:-localhost},${LOCAL_NETWORK}"    
EOF

$ systemctl daemon-reload
$ systemctl restart containerd
```

### refrence

* https://blog.accepted.fun/2021/11/22/Kubernetes%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2ingress-nginx/
