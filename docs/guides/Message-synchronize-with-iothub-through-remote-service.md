# 利用 Remote 远程服务实现 Baetyl 与百度 IoTHub 消息同步

**声明**：

- 本文测试所用设备系统为 Ubuntu 18.04
- 本文测试前先安装 Baetyl，并导入示例配置包，可参考 [快速安装 Baetyl](../install/Quick-Install.md)
- 模拟 MQTT client 向百度云 IoTHub 订阅消息的客户端为 [MQTT.fx](../Resources.html#mqtt-fx)
- 模拟 MQTT client 向本地 Hub 服务发送消息的客户端为 [MQTTBox](../Resources.html#mqttbox)
- 远程 Hub 接入平台选用 [Baidu IoTHub](https://cloud.baidu.com/product/iot.html)

Remote 远程服务模块是为了满足物联网场景下另外一种用户需求而研发，能够实现本地 Hub 与远程 Hub 服务（如[Baidu IoTHub](https://cloud.baidu.com/product/iot.html)等）的数据同步。即通过 Remote 远程服务模块我们既可以从远程 Hub 订阅消息到本地 Hub，也可以将本地 Hub 的消息发送给远程 Hub，完整的配置可参考 [Remote 模块配置](Config-interpretation.html#baetyl-remote-mqtt)。

## 操作流程

- Step 1：依据 Baidu IoTHub 的操作规章，在 Baidu IoTHub 创建测试所用的 endpoint、user、principal（身份）、policy（主题权限策略）等信息；
- Step 2：依据步骤 `Step 1` 中创建的连接信息，选择 MQTT.fx 作为测试用 MQTT 客户端，配置相关连接信息，并将之与 Baidu IoTHub 建立连接，并订阅既定主题；
  - 若成功建立连接，则继续下一步操作；
  - 若未成功建立连接，则重复上述步骤，直至看到 MQTT.fx 与 Baidu IoTHub 成功[建立连接](https://cloud.baidu.com/doc/IOT/s/Sjwvy72aq)。
- Step 3：打开终端，执行 `sudo systemctl start baetyl` 以容器模式启动 Baetyl 可执行程序（要求 Baetyl 已事先在设备上部署完毕，相关内容可参考 [快速安装 Baetyl](../install/Quick-Install.md)），然后执行 `sudo systemctl status baetyl` 来查看 Baetyl 是否正常运行，并观察 Hub 模块、Remote 模块启动状态；
  - 若 Hub、Remote 模块成功启动，则继续下一步操作；
  - 若 Hub、Remote 模块未成功启动，则重复 `Step 3`，直至看到 Hub、Remote 模块成功启动。
- Step 4：选择 MQTTBox 作为测试用 MQTT 客户端，与 Hub 模块[建立连接](Device-connect-to-hub-service.md)，并订阅既定主题；
  - 若成功与 Hub 模块建立连接，则继续下一步操作；
  - 若与 Hub 建立连接失败，则重复 `Step 4` 操作，直至 MQTTBox 与本地 Hub 模块成功建立连接。
- Step 5：依据 Remote 模块的相关配置信息，从 MQTTBox 向既定主题发布消息，观察 MQTT.fx 的消息接收情况；同理，从 MQTT.fx 向既定主题发布消息，观察 MQTTBox 的消息接收情况。
- Step 6：若 `Step 5` 中双方均能接收到对方发布的消息内容，则表明功能测试顺利通过。

上述操作流程相关的流程示意图具体如下图示。

![使用 Remote 模块进行消息同步](../images/guides/remote/remote-flow.png)

## Remote 模块消息远程同步

Baetyl 位于 `var/db/baetyl/application.yml` 的应用配置如下：

```yaml
version: v0
services:
  - name: localhub
    image: hub.baidubce.com/baetyl/baetyl-hub:latest
    replica: 1
    ports:
      - 1883:1883
    mounts:
      - name: localhub-conf
        path: etc/baetyl
        readonly: true
      - name: localhub-data
        path: var/db/baetyl/data
      - name: localhub-log
        path: var/log/baetyl
   - name: remote-iothub
    image: hub.baidubce.com/baetyl/baetyl-remote-mqtt:latest
    replica: 1
    mounts:
      - name: remote-iothub-conf
        path: etc/baetyl
        readonly: true
      - name: remote-iothub-cert
        path: var/db/baetyl/cert
        readonly: true
      - name: remote-iothub-log
        path: var/log/baetyl
volumes:
  # hub
  - name: localhub-conf
    path: var/db/baetyl/localhub-conf
  - name: localhub-data
    path: var/db/baetyl/localhub-data
  - name: localhub-log
    path: var/db/baetyl/localhub-log
  # remote mqtt
  - name: remote-iothub-conf
    path: var/db/baetyl/remote-iothub-conf
  - name: remote-iothub-cert
    path: var/db/baetyl/remote-iothub-cert
  - name: remote-iothub-log
    path: var/db/baetyl/remote-iothub-log
```

Hub 模块的配置文件位于 `var/db/baetyl/localhub-conf/service.yml`，具体配置信息如下：
```yaml
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

Remote 模块的配置文件位置 `var/db/baetyl/remote-iothub-conf/service.yml`，配置信息如下：

```yaml
name: remote-iothub
hub:
  address: tcp://localhub:1883
  username: test
  password: hahaha
remotes:
  - name: iothub
    address: '<iothub_endpoint>' # 从物接入的项目列表中复制 ssl 地址替换 <iothub_endpoint>，比如：ssl://xxxxxx.mqtt.iot.gz.baidubce.com:1884，xxxxxx 为 endpoint
    clientid: remote-iothub-1
    username: '<username>' # 从上面选定（address）的物接入项目下创建的用户名列表中复制支持 ssl 连接的用户名替换 <username>，比如：xxxxxx/test，xxxxxx 为 endpoint
    ca: var/db/baetyl/cert/ca.pem
    cert: var/db/baetyl/cert/client.pem
    key: var/db/baetyl/cert/client.key
rules:
  - hub:
      subscriptions:
        - topic: t1
    remote:
      name: iothub
      subscriptions:
        - topic: t2
          qos: 1
logger:
  path: var/log/baetyl/remote-service.log
  level: 'debug'
```

依据上述 Remote 模块的配置信息，意即 Remote 模块向本地 Hub 模块订阅主题 `t1` 的消息，向 Baidu IoTHub 订 阅主题 `t2` 的消息；当 MQTTBox 向主题 `t1` 发布消息时，Hub 模块接收到主题 `t1` 的消息后，将其转发给 Remote 模块，再由 Remote 模块将之转发给 Baidu IoTHub，这样如果 MQTT.fx 订阅了主题 `t1`，即会收到该条从 MQTTBox 发布的消息；同理，当 MQTT.fx 向主题 `t2` 发布消息时，Baidu IoTHub 会将消息转发给 Remote 模块，由 Remote 模块将之转发给本地 Hub 模块，这样如果 MQTTBox 订阅了主题 `t2`，即会收到该消息。

简单来说，由 MQTT.fx 发布的消息，到 MQTTBox 接收到该消息，流经的路径信息为：

> **MQTT.fx -> Remote Hub -> Remote Module -> Local Hub Module -> MQTTBox**

同样，由 MQTTBox 发布的消息，到 MQTT.fx 接收到该消息，流经的路径信息为：

> **MQTTBox -> Local Hub Module -> Remote Module -> Remote Hub -> MQTT.fx**

### 通过 MQTT.fx 与 Baidu IoTHub 建立连接

如 `Step 1, Step 2` 所述，通过 MQTT.fx 与 Baidu IoTHub 建立连接，涉及的通过云端 Baidu IoTHub 场景的 `endpoint` 等相关信息，及 MQTT.fx 连接配置信息分别如下图示。

![基于 Baidu IoTHub 创建的 endpoint](../images/guides/remote/cloud-iothub-config.png)

![用于连接 Baidu IoTHub 的 MQTT.fx 配置信息](../images/guides/remote/mqttfx-connect-hub-config.png)

完成连接信息的相关配置工作后，点击 `OK` 或 `Apply` 按钮使配置信息生效，然后在 MQTT.fx 连接操作页面点击 `Connect` 按钮，通过按钮的 **颜色** （成功建立连接后，右上方指示灯变为 **绿色**）即可判断 MQTT.fx 是否已与 Baidu IoTHub 建立连接，在建立连接后，切换至 `Subscribe` 页面，依据既定配置，订阅相应主题 `t1`，相关示意图如下图示。

![MQTT.fx 成功与 Baidu IoTHub 建立连接](../images/guides/remote/mqttfx-connect-success.png)

### 通过 MQTTBox 与本地 Hub 模块建立连接

依据步骤 `Step 3` 所述，调整 Baetyl 主程序启动加载配置项，执行 `sudo systemctl start baetyl` 以容器模式启动 Baetyl，这里，要求 Baetyl 启动后加载 Hub、Remote 模块，执行 `sudo systemctl status baetyl` 来查看 `baetyl` 是否正常运行，成功加载的状态如下图示。

![Baetyl 状态](../images/install/systemctl-status.png)

此外，亦可通过执行命令 `docker stats` 查看系统当前正在运行的 docker 容器列表，具体如下图示。

![通过命令 docker ps 查看系统当前正在运行的 docker 容器列表](../images/guides/remote/docker-ps-after-remote-start.png)

成功启动 Baetyl 后，通过 MQTTBox 成功与 Hub 模块建立连接，并订阅主题 `t2`，成功订阅的状态如下图示。

![MQTTBox 成功订阅主题 t2](../images/guides/remote/mqttbox-sub-t2-success.png)

### Remote 消息远程同步

这里，将分别以 MQTT.fx、MQTTBox 作为消息发布方，另一方作为消息接收方进行测试。

**MQTT.fx 发布消息，MQTTBox 接收消息**

首先，通过 MQTT.fx 向主题 `t2` 发布消息 `This message is from MQTT.fx.`，具体如下图示。

![通过 MQTT.fx 向主题 t2 发布消息](../images/guides/remote/mqttfx-pub-t2-success.png)

同时，观察 MQTTBox 在订阅主题 `t2` 的消息接收状态，具体如下图示。

![MQTTBox 成功收到消息](../images/guides/remote/mqttbox-receive-t2-message-success.png)

**MQTTBox 发布消息，MQTT.fx 接收消息**

同理，通过 MQTTBox 作为发布端向主题 `t1` 发布消息 `This message is from MQTTBox.`，具体如下图示。

![通过 MQTTBox 向主题 t1 发布消息](../images/guides/remote/mqttbox-pub-t1-success.png)

同时，观察 MQTT.fx 在订阅主题 `t1` 的消息接收状态，具体如下图示。

![MQTT.fx 成功收到消息](../images/guides/remote/mqttfx-receive-t1-message-success.png)

综上，MQTT.fx 与 MQTTBox 均已正确接收到了对应的消息，且内容吻合。
