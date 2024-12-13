+++
title = 'WebRTC入门'
date = 2024-12-10T10:18:59+08:00
draft = true
+++

**信令：peer 如何在 WebRTC 中找到彼此**

当 WebRTC Agent 启动时，它不知道与谁通信。需要首先获取一些信息，因此需要SDP（会话描述协议）。

SDP包含

* Agent 可供外部访问的（候选的）IP 和端口。
* Agent 希望发送多少路音频和视频流。
* Agent 支持哪些音频和视频编解码器。
* 连接时需要用到的值（uFrag/uPwd）。
* 加密传输时需要用到的值（证书指纹）。

这些信息可以使用任何基础设施来交换，比如http，websocket等。


### 参考资料

* [P2P 打洞原理](https://webrtc.mthli.com/basic/p2p-hole-punching/)
* [ICE 交互流程介绍](https://webrtc.mthli.com/basic/ice-stun-turn/)