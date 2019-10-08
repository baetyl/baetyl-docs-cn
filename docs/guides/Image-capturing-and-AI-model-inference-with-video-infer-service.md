# 利用 Video infer 服务实现摄像头图像采集与 AI 模型推断

**声明**：

- 本文测试所用设备系统为 Ubuntu18.04
- 本文测试所用摄像头为海康威视 IP 网络摄像头，其型号为 DS-IPC-B12-1
- 本文测试 video-infer 服务基于 CPU 和 AI 模型对摄像头采集照片进行推断
- 本文测试采用的 AI 模型为 [ssd_mobilenet_v1_coco_2017_11_17](https://baetyl.cdn.bcebos.com/mqtt-example/guide/video-infer/ssd_mobilenet_v1_coco_2017_11_17.tar.gz)
- 模拟 MQTT client 向本地 Hub 服务发送消息的客户端为 [MQTTBox](../Resources.html#mqttbox)
- 本文所应用的各服务与本地 Hub 服务间连接采用 TCP 连接方式

## 操作流程

- 步骤 1：在 Ubuntu18.04 上安装 Baetyl，相关内容可参考 [快速安装 Baetyl](../install/Quick-Install.md)
- 步骤 2：依据本文测试需求，撰写各服务的配置文件，然后执行 `sudo systemctl start baetyl` 命令启动 Baetyl 服务，可以通过 `sudo systemctl status baetyl` 命令查看 Baetyl 服务的运行情况，关于各服务的配置情况请参考下文 [各服务配置](#id2)
- 步骤 3：根据 Hub 服务的配置信息对 MQTTBox 进行连接配置，确保 MQTTBox 与 Hub 服务建立连接，更多可参考 [与 Hub 服务建立连接](Device-connect-to-hub-service.md)
- 步骤 4：MQTTBox 订阅主题 `video/infer/result`，观察其是否能够正常接收到模型的推断结果消息。

上述步骤的操作流程示意图如下图所示。

![利用 video-infer 服务进行视频采集和 AI 模型推断](../images/guides/inference/video-infer-flow.png)

## 各服务配置

Baetyl 主程序的配置文件位置 `var/db/baetyl/application.yml`，配置信息如下：

```yaml
version: V0
services:
  - name: localhub
    image: 'hub.baidubce.com/baetyl/baetyl-hub:latest'
    replica: 1
    ports:
      - '1883:1883'
    mounts:
      - name: localhub-conf
        path: etc/baetyl
        readonly: true
      - name: localhub-persist-data
        path: var/db/baetyl/data
      - name: demo-log
        path: var/log/baetyl
  - name: function-manager
    image: 'hub.baidubce.com/baetyl/baetyl-function-manager:latest'
    replica: 1
    mounts:
      - name: function-manager-conf
        path: etc/baetyl
        readonly: true
      - name: demo-log
        path: var/log/baetyl
  - name: function-python
    image: 'hub.baidubce.com/baetyl/baetyl-function-python36:0.1.6-opencv41'
    replica: 0
    mounts:
      - name: function-python-conf
        path: etc/baetyl
        readonly: true
      - name: function-python-code
        path: var/db/baetyl/code
        readonly: true
      - name: image-data
        path: var/db/baetyl/image
  - name: video-infer
    image: 'hub.baidubce.com/baetyl-beta/baetyl-video-infer:latest'
    replica: 1
    mounts:
      - name: infer-person-model
        path: var/db/baetyl/model
        readonly: true
      - name: image-data
        path: var/db/baetyl/image
      - name: demo-log
        path: var/log/baetyl
      - name: video-infer-conf
        path: etc/baetyl
        readonly: true
volumes:
  - name: localhub-conf
    path: var/db/baetyl/localhub-conf
  - name: localhub-persist-data
    path: var/db/baetyl/localhub-persist-data
  - name: demo-log
    path: var/db/baetyl/demo-log
  - name: function-manager-conf
    path: var/db/baetyl/function-manager-conf
  - name: function-python-conf
    path: var/db/baetyl/function-python-conf
  - name: function-python-code
    path: var/db/baetyl/function-python-code
  - name: image-data
    path: var/db/baetyl/image-data
  - name: remote-mqtt-conf
    path: var/db/baetyl/remote-mqtt-conf
  - name: infer-person-model
    path: var/db/baetyl/infer-person-model
  - name: video-infer-conf
    path: var/db/baetyl/video-infer-conf
```

Hub 服务配置文件位置 `var/db/baetyl/localhub-conf/service.yml`，配置信息如下：

```yml
listen:
  - tcp://0.0.0.0:1883
principals:
  - username: test
    password: hahaha
    permissions:
      - action: 'pub'
        permit: ['#']
      - action: 'sub'
        permit: ['#']
logger:
  path: var/log/baetyl/localhub-service.log
  level: "debug"
```

Function manager 服务配置文件位置 `var/db/baetyl/function-manager-conf/service.yml`，配置信息如下：

```yml
server:
  address: 0.0.0.0:50051
hub:
  address: tcp://localhub:1883
  username: test
  password: hahaha
functions:
  - name: analyse
    service: function-python
    instance:
      max: 10
logger:
  path: var/log/baetyl/func-service.log
  level: "debug"
```

Function python 服务配置文件位置 `var/db/baetyl/function-python-conf/service.yml`，配置信息如下：

```yml
functions:
  - name: 'analyse'
    handler: 'analyse.handler'
    codedir: 'var/db/baetyl/code'
logger:
  path: "var/log/baetyl/python-service.log"
  level: "debug"
```

Video infer 服务配置文件位置 `var/db/baetyl/video-infer-conf/service.yml`，配置信息如下：

```yml
hub:
  address: tcp://localhub:1883
  username: test
  password: hahaha
video:
  uri: "rtsp://admin:admin@192.168.1.2:554/Streaming/channels/1/"
  limit:
    fps: 1
infer:
  model: var/db/baetyl/model/frozen_inference_graph.pb
  config: var/db/baetyl/model/ssd_mobilenet_v1_coco_2017_11_17.pbtxt
process:
  before:
    swaprb: true
    width: 300
    hight: 300
  after:
    function:
      name: analyse
functions:
  - name: analyse
    address: function-manager:50051
logger:
  path: var/log/baetyl/infer-service.log
  level: "debug"
```

需要注意的是，这里 `uri` 配置代表的是 IP 网络摄像头的地址，其通用格式为 `rtsp://<username>:<password>@<ip>:<port>/Streaming/channels/<stream_number>`，其中，`<username>` 和 `<password>` 为激活成功后的 IP 摄像头的登录认证口令，`<ip>` 为该摄像头的IP地址，`<port>` 为 RTSP 协议的端口号，默认为 554，后面内容为信道，其中 `<stream_number>` 为 1 代表抓取主码流，为 2 代表抓取次码流。

此外，Video infer 服务除支持抓取 IP 网络摄像头图片信息外，还支持抓取 USB 摄像头采集图像信息和读取视频文件进行抽帧。相应地，若为 USB 摄像头，则 `uri` 配置为设备编号，通用性配置为 "0"，同时需要将设备地址 `/dev/video0` 映射进容器；若配置读取视频文件进行抽帧，则直接配置 `uri` 内容为视频文件的地址，同时将该视频文件以（自定义）存储卷形式挂载到 Video infer 服务即可，更多关于存储卷的内容可以参考 [如何正确地引入存储卷](https://cloud.baidu.com/doc/BIE/s/Cjzdn8xig)。

Video infer 服务捕获 USB 摄像头采集图像配置可参考：

```yaml
video:
  uri: "0"
  limit:
    fps: 1
```

同时在 `application.yml` 配置中将 `/dev/video0` 设备地址映射到容器中，相关配置可参考：

```yaml
version: V0
services:
  - name: video-infer
    image: 'hub.baidubce.com/baetyl-beta/baetyl-video-infer:latest'
    replica: 1
    devices:
      - /dev/video0 # 将 USB 设备映射进容器
    mounts:
      - name: infer-person-model
        path: var/db/baetyl/model
        readonly: true
      - name: image-data
        path: var/db/baetyl/image
      - name: demo-log
        path: var/log/baetyl
      - name: video-infer-conf
        path: etc/baetyl
        readonly: true
```

不难发现，从容器内读取宿主机 的USB 、串口等设备，均可采用上述设备映射的方式配置。

## 测试及验证

如本文开头所述，本次测试我们采用 `ssd_mobilenet_v1_coco_2017_11_17` 模型，该模型可用于检测人、水果等多达 90 项物品。这里，我们给出用于检测人的 Python 脚本仅供参考，具体如下：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
"""
function to analyse video infer result in python
"""
import numpy as np

location = "var/db/baetyl/image/{}.jpg"
classes = {
    1: 'person'
}

def handler(event, context):
    """
    function handler
    """
    data = np.fromstring(event, np.float32)
    mat = np.reshape(data, (-1, 7))
    objects = []
    scores = {}
    for obj in mat:
        clazz = int(obj[1])
        if clazz in classes:
            score = float(obj[2])
            if classes[clazz] not in scores or scores[classes[clazz]] < score:
                scores[classes[clazz]] = score
            if score < 0.6:
                continue
            objects.append({
                'class': classes[clazz],
                'score': score,
                'left': float(obj[3]),
                'top': float(obj[4]),
                'right': float(obj[5]),
                'bottom': float(obj[6])
            })

    res = {}
    res["imageDiscard"] = len(objects) == 0
    res["imageObjects"] = objects
    res["imageScores"] = scores
    res["publishTopic"] = "video/infer/result"
    res["messageTimestamp"] = int(context["messageTimestamp"]/1000000)
    if len(objects) != 0:
        res["imageLocation"] = location.format(context["messageTimestamp"])

    return res
```

若检测其他物品，直接在这里添加或修改即可，可支持检测物品列表参考 [mscoco_label_map](https://github.com/tensorflow/models/blob/master/research/object_detection/data/mscoco_label_map.pbtxt)。

一切就绪后，启动 Baetyl 服务，然后通过 `sudo systemctl status baetyl` 或 `docker ps` 命令查看 Baetyl 服务的运行状态及正在运行的容器列表。

![ Baetyl 服务运行情况](../images/guides/inference/running-status-of-baetyl.png)

![ 正在运行的容器列表](../images/guides/inference/running-container-view.png)

可以发现，Baetyl 服务处于 `active (running)` 状态，系统现在正在运行了 `baetyl-hub`、`baetyl-video-infer`、`baetyl-function-manager` 及 `baetyl-function-python36` 四个容器。

然后，我们启动 MQTTBox，根据 Hub 服务配置完连接信息后，可以发现 MQTTBox 已经与 Hub 服务建立连接。

![ MQTTBox 与 Hub 服务建立连接](../images/guides/inference/MQTTBox-connect-to-hub.png)

然后，通过 MQTTBox 订阅主题 `video/infer/result`，若服务运行正常，则 MQTTBox 会收到消息。

![ MQTTBox 接收消息，未检测到人](../images/guides/inference/model-infer-result-none-person.png)

![ MQTTBox 接收消息，检测到人](../images/guides/inference/model-infer-result-person.png)

如上所示，MQTTBox 通过订阅主题 `video/infer/result` 正确收到了消息。对比上述两条消息，可以发现一个检测到了人，一个未检测到人，如果模型检测到了人，则会在接收到的信息中给出类别 `class` 类别信息，同时给出检测到的人在捕获图片中的位置信息，可以用于后续的画框标记操作，感兴趣的可以参考 [代码](https://github.com/baetyl/baetyl/blob/master/baetyl-video-infer/example/docker/var/db/baetyl/function-python-code/mark.py)。

至此，我们已经基于 Video infer 服务实现了 IP 网络摄像头图像的采集和 AI 模型的推断。