---
title: 本地RTSP协议视频流实现公网播放
categories: 技术
comments: true
excerpt: RTSP WebRTC mediamtx 公网
---

# 介绍
RTSP是一种基于客户端-服务器模型的协议，允许客户端请求和控制流媒体的实时传输。主要应用场景包括视频会议，在线直播和监控系统等，本文详细写一下最近遇到的需求和解决办法。

甲方部署了视频监控，提供了视频流地址，让我们在web上加载显示，地址是`rtsp://192.168.1.154:554/channel=0&stream=1`，显而易见这是一个本地地址，想要实现播放需要实现以下步骤:
## 一、RTSP协议转换
由于RTSP协议是不支持直接在Web上播放的，所以需要先转换为Web支持的格式，比如HLS或WebRTC。这里我选择转成WebRTC，转码是需要转换服务的，网上有很多方式，比如SRS(Simple Realtime Server)，Janus Gateway等等，这里我选择[mediamtx](https://github.com/bluenviron/mediamtx)，mediamtx原生支持 RTSP、RTMP、HLS、WebRTC、SRT 等，并且WebRTC 传输延迟可控制在 0.3-1 秒，配置简单，只需要通过 YAML 文件快速定义流路由规则就可以实现。如下：

在mediamtx的GitHub的[Releases](https://github.com/bluenviron/mediamtx/releases)页面下载系统对应的版本，由于我是使用windows服务器，所以选择如下图所示，下载解压到服务器任意目录。
![release](/img/image.png)
解压后内部有3个文件，一份授权，一份exe文件和一份yml配置文件，在运行mediamtx的exe文件前需要先修改yml配置。
![mediamtx](/img/mediamtx.png)
yml文件主要修改以下几个属性，默认`webrtc`是开启的，不用修改，`webrtcAddress`表示对外开放的TCP端口，`webrtcLocalUDPAddress`表示对外开放的UDP端口，`webrtcIPsFromInterfaces`表示是否自动获取本地IP，对于多网卡或虚拟网络需要关闭，并指定`webrtcAdditionalHosts`为公网IP，以下xx仅为示例，`webrtcICEServers2`用于用于解决网络穿透问题，确保端到端连接的建立，`Paths`下添加`my_camera`表示名称，可任意填写，`source`填写rtsp地址。
```
webrtc:yes
webrtcAddress: :8889
webrtcLocalUDPAddress: :8189
webrtcIPsFromInterfaces: no
webrtcAdditionalHosts: [xxx.xx.xx.xx]
webrtcICEServers2: 
  - url: stun:stun.l.google.com:19302

paths:
  my_camera:
     source: rtsp://192.168.1.154:554/channel=0&stream=1

```
配置修改完成后，双击`mediamtx.exe`运行，显示日志控制台可查看运行情况。
## 二、公网访问
实现公网访问主要需要将公网的8889和8189端口映射到内网上即可，此处省略。
### 三、前端播放
完成以上步骤后，打开浏览器输入`http://xxx.xx.xx.xx:8889/my_camera`即可看到播放视频，只需要将此地址使用`iframe`标签加载到Web页面上即可。


