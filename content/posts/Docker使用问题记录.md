---
title: 'Docker使用问题记录'
date: 2024-07-13T07:02:21+08:00
draft: false
---

# 安装Docker

参考[Install using the apt repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```


其中在这步中，即使设置了Proxy

```shell
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

好像并没有生效，我当时是直接下载`gpg`文件，然后手动copy到`/etc/apt/keyrings/docker.asc`

当然也可以使用

```shell
sudo curl -x http://127.0.0.1:7890 -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

后续安装就很顺利了。

# 关于Docker使用Proxy

参考[Configure Docker to use a proxy server](https://docs.docker.com/network/proxy/)


这里说明了2中Proxy。一种是Docker自身使用Proxy。一种是Docker Container容器使用Proxy。

一般我们pull image需要使用Proxy应该配置Docker deamon `/etc/docker/daemon.json`


```json
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:7890",
    "https-proxy": "http://127.0.0.1:7890",
    "no-proxy": "127.0.0.0/8"
  }
}
```

最后别忘了

```
sudo systemctl restart docker
```

我犯错就是配置成`~/.docker/config.json`，导致容器中使用`http-proxy`请求`127.0.0.1`请求不同，容器之间通信失败。

```json
{
 "proxies": {
   "default": {
      "http-proxy": "http://127.0.0.1:7890",
      "https-proxy": "http://127.0.0.1:7890",
      "no-proxy": "127.0.0.0/8"
   }
 }
}
```

