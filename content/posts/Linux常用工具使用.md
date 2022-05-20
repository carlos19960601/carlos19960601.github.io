---
title: "Linux常用工具使用"
date: 2022-04-21T15:18:31+08:00
draft: false
original: true
categories: 
  - 经验
tags: 
  - 工具
---


### SSH 无密码登陆

首先在自己的本地机器生成ssh key

```shell
ssh-keygen -t rsa
```

SSH密钥会保存在home目录下的**.ssh/id_rsa**文件中．SSH公钥保存在**.ssh/id_rsa.pub**文件中．

然后将SSH公钥上传到Linux服务器

```shell
ssh-copy-id username@remote-server
```

输入远程用户的密码后，SSH公钥就会自动上传了．SSH公钥保存在远程Linux服务器的**.ssh/authorized_keys**文件中。

<!--more-->

### 服务启动

ubuntu中，服务可以通过systemctl来管理

比如安装完mysql之后就会有mysql.service这个服务

**启动服务**

```
systemctl start mysql.service
```

**关闭服务**

```
systemctl stop mysql.service
```

**重启服务**

```
systemctl restart mysql.service
```

**显示服务的状态**

```
systemctl status mysql.service
```

**在开机时启用服务**

```
systemctl enable mysql.service
```

**在开机时禁用服务**

```
systemctl disable mysql.service
```

**查看服务是否开机启动**

```
systemctl is-enabled mysql.service
```

**查看已启动的服务列表**

```
systemctl list-unit-files|grep enabled
```

### 判断终端是否走了代理服务器

一行命令搞定：

已翻墙

```
$ curl cip.cc
IP	: 149.129.75.xxx
地址	: 中国  香港  阿里云

数据二	: 香港 | 阿里云

数据三	: 中国香港 | 阿里云
```

未翻墙

```
$ curl cip.cc
IP	: 222.209.33.xxx
地址	: 中国  四川  成都
运营商	: 电信

数据二	: 四川省成都市 | 电信

数据三	: 中国四川省成都市 | 电信
```

### 合并视频和音频

最近下载了Youtube上面的视频

```
format code  extension  resolution note
249          webm       audio only tiny   54k , opus @ 50k (48000Hz), 12.73MiB
250          webm       audio only tiny   69k , opus @ 70k (48000Hz), 16.60MiB
140          m4a        audio only tiny  129k , m4a_dash container, mp4a.40.2@128k (44100Hz), 33.56MiB
251          webm       audio only tiny  132k , opus @160k (48000Hz), 32.78MiB
160          mp4        256x144    144p   64k , avc1.4d400c, 24fps, video only, 11.00MiB
278          webm       256x144    144p   96k , webm container, vp9, 24fps, video only, 23.29MiB
133          mp4        426x240    240p  124k , avc1.4d4015, 24fps, video only, 19.61MiB
242          webm       426x240    240p  188k , vp9, 24fps, video only, 35.61MiB
243          webm       640x360    360p  373k , vp9, 24fps, video only, 63.51MiB
134          mp4        640x360    360p  503k , avc1.4d401e, 24fps, video only, 47.47MiB
244          webm       854x480    480p  758k , vp9, 24fps, video only, 105.52MiB
135          mp4        854x480    480p 1185k , avc1.4d401e, 24fps, video only, 92.02MiB
247          webm       1280x720   720p 1751k , vp9, 24fps, video only, 207.76MiB
136          mp4        1280x720   720p 2872k , avc1.4d401f, 24fps, video only, 174.77MiB
248          webm       1920x1080  1080p 3076k , vp9, 24fps, video only, 394.94MiB
137          mp4        1920x1080  1080p 6553k , avc1.640028, 24fps, video only, 349.89MiB
18           mp4        640x360    360p  411k , avc1.42001E, 24fps, mp4a.40.2@ 96k (44100Hz), 108.63MiB
22           mp4        1280x720   720p  788k , avc1.64001F, 24fps, mp4a.40.2@192k (44100Hz) (best)
```
可以看到，youtube的一个视频提供了很多格式，我当然是下载质量最好的是吧，于是把format是137的视频下载下来，然而发现没有声音，这才发现137是`video only`，没办法再把音频文件140下载现在。然后要做的就是合并这2个文件。
我首先想到的就是pr，于是pr走起，导入视频、音频素材，然后开始导出，发现导出的最终视频2G，原视频才350MB，音频33MB，可能是pr导出时参数设置的问题，无奈我不是很懂，于是想到之前搞过ffmpeg，于是尝试了以下，果然真香

```
ffmpeg -i audio.mp4 -i video.mp4 -acodec copy -vcodec copy output.mp4
```

最终导出的视频400MB，而且速度贼开，2s不到吧

### youtube-dl下载视频

下载所有的字幕和最好的视频

```
youtube-dl --proxy 'socks5://127.0.0.1:1080' --all-subs -f best
```

* --proxy 指定代理
* --all-subs 下载所有的字幕
* -f 指定下载视频的格式，可以通过-F来查看所有的格式

查看视频格式

```
youtube-dl --proxy 'socks5://127.0.0.1:1080' -F 
```

下载播放列表

```
youtube-dl --proxy 'socks5://127.0.0.1:1080' --yes-playlist -f best
```

下载超过100个视频的播放列表

由于目前youtube-dl对于超过100个视频的播放列表会报错[issues 25768](https://github.com/ytdl-org/youtube-dl/issues/25768) [issues 14076](https://github.com/ytdl-org/youtube-dl/issues/14076)，只能通过指定下载列表的[start,end]进行下载

```
youtube-dl --proxy 'socks5://127.0.0.1:1080' --yes-playlist --playlist-start 1 --playlist-end 100 -f best
```

选择最好的格式下载

```
$ youtube-dl --proxy 'socks5://127.0.0.1:1080' -f bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio --merge-output-format mp4 
```

### brew

#### 执行brew命令出现一堆 Warning: Cask 'xxx' is unreadable: undefined method `method_missing_message' for Utils:Module

Firstly, update the local formula repositories to the latest state. Cause reading issues on GitHub repo Hombrew-cask, the error may be introduced by typos in formula definitions.

```sh
# enable --verbose to get more info
brew update --verbose
```

Then try `brew search metabase` again.

If the above command doesn't fix your problem, go into local repository of Homebrew-cask and reset it.

```sh
cd "$(brew --repo homebrew/cask)"
git clean -dfx

git reset --hard origin/master
git pull origin master
```

参考[原回答](https://apple.stackexchange.com/questions/371785/warning-cask-xxx-is-unreadable-undefined-method-method-missing-message-for)

### apt源中有一些google的源，导致 apt-get update无法连接

可以给apt-get设置代理

```
sudo apt-get -o Acquire::http::proxy="http://127.0.0.1:8086" update
```

### mac无线鼠标移动慢

- 鼠标双击阈值：defaults read -g com.apple.mouse.doubleClickThreshold
- 鼠标加速度：defaults read -g com.apple.mouse.scaling
- 滚动速度：defaults read -g com.apple.scrollwheel.scaling

如果鼠标使用有异常，可以再终端中读以上三个参数，并根据自己的需要适当调高调低

- 鼠标双击阈值：defaults write -g com.apple.mouse.doubleClickThreshold 0.75
- 鼠标加速度：defaults write -g com.apple.mouse.scaling 5
- 滚动速度：defaults write -g com.apple.scrollwheel.scaling 0.75

### 制作U盘启动盘

1. 查看U盘设备号，本例使用了8G的U盘，

```
Disk /dev/sdc: 7.5 GiB, 8054112256 bytes, 15730688 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x40993ab6
```

2. 准备好一个iso文件，使用dd命令将这个iso写入u盘

```
sudo dd if=./ubuntu-20.04.4-live-server-arm64.iso of=/dev/sdc
```


### Ubuntu 开启 ssh-server

参考 https://ubuntu.com/server/docs/service-openssh

```
sudo apt install openssh-server
```

执行后查看ssh是否已经运行

```
sudo systemctl status sshd
```

### sudo curl 的时候 http_proxy 不生效

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

在国内直接执行上述命令是会报错的，google被屏蔽了，但是尽管使用了

```
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

还是不能执行成功，原因是sudo执行的时候会clean所有的环境变量，所以是看不到http_proxy的

所以只能使用curl命令自带的proxy

```
sudo curl --socks5 proxy-server:port  -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

### apt-get通过proxy进行

```
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890

sudo apt-get update
```

你会发现有些package还是不能更新，但是通过proxy能直接访问对应的package地址

这时可以使用

```
sudo apt-get -o Acquire::http::proxy="http://proxy-server:port/" update
```

### sudo使用http_proxy失败

```
# The following should be in `/etc/sudoers`. To edit `/etc/suoders` use the command `sudo visudo` in a terminal.
# *Do NOT attempt to modify the file directly*
Defaults  env_reset
Defaults	mail_badpass
Defaults	env_keep+="http_proxy ftp_proxy all_proxy https_proxy no_proxy" # Add this line
```

### Ubuntu设置root密码

```
$ sudo passwd root
```

切换到root

```
su -
```


### Ubuntu安装k8s

#### 前置准备

**disable swap**

临时关闭swap

```
swapoff -a     
```

永久关闭

1. Comment the correct mounting point

```
$ vim /etc/fstab

#UUID=xxxx  none  swap  sw  0  0
```

在Ubuntu 20中reboot后swap任然会mount上


2. the swap is controlled by systemctl. To find the responsible, use systemctl --type swap.

```
systemctl --type swap

UNIT           LOAD   ACTIVE SUB                DESCRIPTION                                                                                                                                                                     
dev-sdb2.swap loaded active active /dev/sdb2 
```

3. Disable by masking it with sysctl:

```
 systemctl mask dev-sdb2.swap
```

**关闭firewall**

```
$systemctl disable ufw.service
```

#### 安装Container Runtime

参考 https://kubernetes.io/docs/setup/production-environment/container-runtimes/

这里选择安装containerd

1. 安装和配置前置条件(仅适用于linux)

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

2. 安装containerd

说明：The containerd.io packages in DEB and RPM formats are distributed by Docker (not by the containerd project). 所以需要[参考Docker的文档](https://docs.docker.com/engine/install/ubuntu/)安装containerd

**Set up the repository**

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:

```
$ sudo apt-get update

$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

2. Add Docker’s official GPG key:

```
$  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

3. Use the following command to set up the stable repository. To add the nightly or test repository, add the word nightly or test (or both) after the word stable in the commands below. Learn about nightly and test channels.

```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Install Docker Engine**

1. Update the apt package index, and install the latest version of Docker Engine and containerd, or go to the next step to install a specific version:

```
$ sudo apt-get update
$ sudo apt-get install containerd.io
```

由于我们不安装Docker，所以就到此为止。

完成后你可以看到一个有效的配置文件 linux: /etc/containerd/config.toml

For containerd, the CRI socket is /run/containerd/containerd.sock by default.

**Configuring the systemd cgroup driver**

To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

If you apply this change, make sure to restart containerd:

```
sudo systemctl restart containerd
```

注意 When using kubeadm, manually configure the cgroup driver for kubelet.

> 按照上述文档中的描述进行操作，在最后执行kubeadm init的时候会报错，解决方案是删除 /etc/containerd/config.toml，这样就不会报错了。可以`containerd config default > /etc/containerd/config.toml`,然后修改配置文件中的`SystemdCgroup = true`

3.  使用kubeadm安装k8s

**安装kubuadm**

在  https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ 中有一些安装之前的要求

**Letting iptables see bridged traffic**

Make sure that the `br_netfilter` module is loaded. This can be done by running `lsmod | grep br_netfilter`. To load it explicitly call sudo modprobe br_netfilter.

As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config, e.g.

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

**Installing kubeadm, kubelet and kubectl**


1. Update the apt package index and install packages needed to use the Kubernetes apt repository:

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

2. Download the Google Cloud public signing key:

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

3. Add the Kubernetes apt repository:

```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

4. Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

**Configuring the kubelet cgroup driver**

kubeadm allows you to pass a KubeletConfiguration structure during kubeadm init. This KubeletConfiguration can include the cgroupDriver field which controls the cgroup driver of the kubelet.

A minimal example of configuring the field explicitly:

```
# kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.23.6
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```

kubeadm init --config kubeadm-config.yaml

**使用kubeadm创建cluster**

参考 https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

```
kubeadm init --config kubeadm-config.yaml --control-plane-endpoint 192.168.0.112
```

运行会报错

```
can not mix '--config' with arguments [control-plane-endpoint]
```

参考 https://www.frakkingsweet.com/specify-control-plane-endpoint-in-kubeadm-init-file

需要在kubeadm-config.yaml中添加

```
# kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.23.0
controlPlaneEndpoint: 192.168.0.112
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```


执行`kubeadm init --config kubeadm-config.yaml`还是报错，删除 `rm /etc/containerd/config.toml`   就可以了

```
rm /etc/containerd/config.toml
systemctl restart containerd
```

由于`kubeadm init`需要从外网下载镜像，而且镜像仓库一般都是被屏蔽了的，所以我们需要通过其他方式提前 下载好需要的 镜像

```
$ kubuadm config images list
k8s.gcr.io/kube-apiserver:v1.23.6
k8s.gcr.io/kube-controller-manager:v1.23.6
k8s.gcr.io/kube-scheduler:v1.23.6
k8s.gcr.io/kube-proxy:v1.23.6
k8s.gcr.io/pause:3.6
k8s.gcr.io/etcd:3.5.1-0
k8s.gcr.io/coredns/coredns:v1.8.6
```

所以我们需要提前下载好这些镜像

最后执行`kubeadm init --config kubeadm-config.yaml`

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.112:6443 --token 3iv8b3.z87w3ezyl2uj0pbr \
  --discovery-token-ca-cert-hash sha256:456751003ac134a1f505fa343b8f74dbbafa81055eee951e17c43284e5aab848
```

看到这样的结果就算成功了

**安装CNI**

参考 https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart

1. Install the Tigera Calico operator and custom resource definitions.

```
kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
```

2. Install Calico by creating the necessary custom resource. For more information on configuration options available in this manifest, see the installation reference.

```
kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml
```

这里的 custom-resources.yaml 需要自定义。我的配置是

```
# This section includes base Calico installation configuration.
# For more information, see: https://projectcalico.docs.tigera.io/v3.22/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
    nodeAddressAutodetectionV4:
      interface: wlo.*        
---

# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/v3.22/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer 
metadata: 
  name: default 
spec: {}
```

**我在安装过程中遇到的坑**

1. 因为我使用了代理服务器，下载的是k8s.gcr.io仓库的镜像，`kubeadmin init`命令的时候会启动api-server的static pod并且回调用api-server的健康检查接口，这个时候curl api-server的健康检查接口的时候可能回走到代理服务器，导致api-server启动失败，进而 `kubeadmin init`失败

2. kubernetes官网给的 kubeadmn-config.yaml不完整。可以使用命令`kubeadm config print init-defaults --component-configs KubeletConfiguration`和`kubeadm config print init-defaults --component-configs InitConfiguration`打印出完整的配置来。然后根据自己的需求来修改。下面是我的kubbeadm-config.yaml文件。

```
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.0.112  # 
  bindPort: 6443
nodeRegistration:
  criSocket: /run/containerd/containerd.sock # 默认用的 docker，所以需要改成containerd
  imagePullPolicy: IfNotPresent
  taints: null
---
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.23.6
networking:
  podSubnet: 10.244.0.0/16
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd # 需要改成 systemd 和 容器运行时一致，我使用的containerd，containerd的 cgroupDriver配置成systemd的
```

主要修改的地方时`cgroupDriver: systemd`，`criSocket`和`advertiseAddress`


### kubectl set default namespace

```
kubectl config set-context --current --namespace=NAMESPACE
```

### kubectl for port-forward

```
kubectl -n infra port-forward dm-mysql-primary-0 3306:3306
```

```
mysql -u username -p -h 127.0.0.1
```

### clash api

Proxies
  
  * /proxies
    * Method: GET
      * Full Path: GET /proxies
      * Description: Get proxies information
  * /proxies/:name
    * Method: GET
      * Full Path: GET /proxies/:name
      * Description: Get specific proxy information
    * Method: PUT
      * Full Path: PUT /proxies/:name
      * Description: Select specific proxy
  * /proxies/:name/delay
    * Method: GET
      * Full Path: GET /proxies/:name/delay
      * Description: Get specific proxy delay test information