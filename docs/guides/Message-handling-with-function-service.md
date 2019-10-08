# 利用 Function 函数计算服务进行消息处理

**声明**：

- 本文测试所用设备系统为 Ubuntu 18.04
- python 版本为 3.6，2.7 版本配置流程相同，但需要在 python 脚本中注意语言差异
- 模拟 MQTT client 行为的客户端为 [MQTTBox](../Resources.html#下载-MQTTBox-客户端)
- 本文所用镜像为依赖 Baetyl 源码自行编译所得，具体请查看 [如何从源码构建镜像](../install/Build-from-Source.md)

_**提示**：Darwin 系统可以通过源码安装 Baetyl，可参考 [源码编译 Baetyl](../install/Build-from-Source.md)。_

与基于 Hub 服务实现设备间消息转发不同的是，本文主要介绍利用本地函数计算服务进行消息处理。其中 Hub 服务用于建立 Baetyl 与 MQTT 客户端之间的连接，Python 运行时服务用于处理 MQTT 消息，而本地函数计算服务则通过 MQTT 消息上下文衔接本地 Hub 服务与 Python 运行时服务。

本文将以 TCP 连接方式为例，展示本地函数计算服务的消息处理、计算功能。

## 操作流程

- 步骤一：安装 Baetyl，**并导入示例配置包**。参考 [快速安装 Baetyl](../install/Quick-Install.md) 进行操作；
- 步骤二：依据测试需求修改导入的配置信息，执行 `sudo systemctl start baetyl` 以容器模式启动 Baetyl，然后执行 `sudo systemctl status baetyl` 来查看 Baetyl 是否正常运行。如果 Baetyl 已经启动，执行 `sudo systemctl start baetyl` 重启来加载新的配置。
- 步骤三：通过 MQTTBox 以 TCP 方式与 Baetyl Hub 服务 [建立连接](./Device-connect-to-hub-service.md)；
  - 若成功与 Hub 服务建立连接，则依据配置的主题权限信息向有权限的主题发布消息，同时向拥有订阅权限的主题订阅消息，并观察 Baetyl 日志信息；
    - 若 Baetyl 日志显示已经启动 Python 运行时服务，则表明发布的消息受到了预期的函数处理；
    - 若 Baetyl 日志显示未成功启动 Python 运行时服务，则重复上述步骤，直至看到 Baetyl 主程序成功启动了 Python 运行时服务。
  - 若与 Baetyl Hub 建立连接失败，则重复步骤三操作，直至 MQTTBox 与 Baetyl Hub 服务成功建立连接为止。
- 步骤四：通过 MQTTBox 查看对应主题消息的收发状态。

![基于本地函数计算服务实现设备消息处理流程](../images/guides/process/python-flow.png)

## 消息处理测试

依据 `步骤一` 导入示例配置包后，确认一下应用配置、 Hub 服务配置以及函数计算服务配置。

将 Baetyl 应用配置改成如下配置：

```yaml
# /usr/local/var/db/baetyl/application.yml
version: v0
services:
  - name: localhub
    image: hub.baidubce.com/baetyl/baetyl-hub
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
  - name: function-manager
    image: hub.baidubce.com/baetyl/baetyl-function-manager
    replica: 1
    mounts:
      - name: function-manager-conf
        path: etc/baetyl
        readonly: true
      - name: function-manager-log
        path: var/log/baetyl
  - name: function-python27-sayhi
    image: hub.baidubce.com/baetyl/baetyl-function-python27
    replica: 0
    mounts:
      - name: function-sayhi-conf
        path: etc/baetyl
        readonly: true
      - name: function-sayhi-code
        path: var/db/baetyl/function-sayhi
        readonly: true
  - name: function-python36-sayhi
    image: hub.baidubce.com/baetyl/baetyl-function-python36
    replica: 0
    mounts:
      - name: function-sayhi-conf
        path: etc/baetyl
        readonly: true
      - name: function-sayhi-code
        path: var/db/baetyl/function-sayhi
        readonly: true
  - name: function-node85-sayhi
    image: hub.baidubce.com/baetyl/baetyl-function-node85
    replica: 0
    mounts:
      - name: function-sayjs-conf
        path: etc/baetyl
        readonly: true
      - name: function-sayjs-code
        path: var/db/baetyl/function-sayhi
        readonly: true
  - name: function-sql-filter
    image: hub.baidubce.com/baetyl/baetyl-function-sql
    replica: 0
    mounts:
      - name: function-filter-conf
        path: etc/baetyl
        readonly: true
volumes:
  # hub
  - name: localhub-conf
    path: var/db/baetyl/localhub-conf
  - name: localhub-data
    path: var/db/baetyl/localhub-data
  - name: localhub-cert
    path: var/db/baetyl/localhub-cert-only-for-test
  - name: localhub-log
    path: var/db/baetyl/localhub-log
  # function
  - name: function-manager-conf
    path: var/db/baetyl/function-manager-conf
  - name: function-manager-log
    path: var/db/baetyl/function-manager-log
  - name: function-sayhi-conf
    path: var/db/baetyl/function-sayhi-conf
  - name: function-sayhi-code
    path: var/db/baetyl/function-sayhi-code
  - name: function-sayjs-conf
    path: var/db/baetyl/function-sayjs-conf
  - name: function-sayjs-code
    path: var/db/baetyl/function-sayjs-code
  - name: function-filter-conf
    path: var/db/baetyl/function-filter-conf
```

Baetyl Hub 服务配置改成如下配置：

```yaml
# /usr/local/var/db/baetyl/localhub-conf/service.yml
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
subscriptions:
  - source:
      topic: 't'
    target:
      topic: 't/topic'
logger:
  path: var/log/baetyl/service.log
  level: "debug"
```

Baetyl 本地函数计算服务相关配置无需修改，具体配置如下：

```yaml
# /usr/local/var/db/baetyl/function-manager-conf/service.yml
hub:
  address: tcp://localhub:1883
  username: test
  password: hahaha
rules:
  - clientid: func-python27-sayhi-1
    subscribe:
      topic: t
    function:
      name: python27-sayhi
    publish:
      topic: t/py2hi
  - clientid: func-sql-filter-1
    subscribe:
      topic: t
      qos: 1
    function:
      name: sql-filter
    publish:
      topic: t/sqlfilter
      qos: 1
  - clientid: func-python36-sayhi-1
    subscribe:
      topic: t
    function:
      name: python36-sayhi
    publish:
      topic: t/py3hi
  - clientid: func-node85-sayhi-1
    subscribe:
      topic: t
    function:
      name: node85-sayhi
    publish:
      topic: t/node8hi
functions:
  - name: python27-sayhi
    service: function-python27-sayhi
    instance:
      min: 0
      max: 10
  - name: sql-filter
    service: function-sql-filter
  - name: python36-sayhi
    service: function-python36-sayhi
  - name: node85-sayhi
    service: function-node85-sayhi
logger:
  path: var/log/baetyl/service.log
  level: "debug"

# /usr/local/var/db/baetyl/function-filter-conf/service.yml
functions:
  - name: sql-filter
    handler: 'select qos() as qos, topic() as topic, * where id < 10'

# /usr/local/var/db/baetyl/function-sayhi-conf/service.yml
functions:
  - name: 'python27-sayhi'
    handler: 'index.handler'
    codedir: 'var/db/baetyl/function-sayhi'
  - name: 'python36-sayhi'
    handler: 'index.handler'
    codedir: 'var/db/baetyl/function-sayhi'

# /usr/local/var/db/baetyl/function-sayjs-conf/service.yml
functions:
  - name: 'node85-sayhi'
    handler: 'index.handler'
    codedir: 'var/db/baetyl/function-sayhi'
```

Python 函数代码无需修。`/usr/local/var/db/baetyl/function-sayhi-code/index.py` 实现如下：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
"""
function to say hi in python
"""


def handler(event, context):
    """
    function handler
    """ 
    res = {}
    if isinstance(event, dict):
        if "err" in event:
            raise TypeError(event['err'])
        res = event
    elif isinstance(event, bytes):
        res['bytes'] = event.decode("utf-8")

    if 'messageQOS' in context:
        res['messageQOS'] = context['messageQOS']
    if 'messageTopic' in context:
        res['messageTopic'] = context['messageTopic']
    if 'messageTimestamp' in context:
        res['messageTimestamp'] = context['messageTimestamp']
    if 'functionName' in context:
        res['functionName'] = context['functionName']
    if 'functionInvokeID' in context:
        res['functionInvokeID'] = context['functionInvokeID']

    res['Say'] = 'Hello Baetyl'
    return res
```

Node 函数代码无需修。`/usr/local/var/db/baetyl/function-sayjs-code/index.js` 实现如下：

```js
#!/usr/bin/env node

const hasAttr = (obj, attr) => {
    if (obj instanceof Object && !(obj instanceof Array)) {
        if (obj[attr] != undefined) {
            return true;
        }
    }
    return false;
};

const passParameters = (event, context) => {
    if (hasAttr(context, 'messageQOS')) {
        event['messageQOS'] = context['messageQOS'];
    }
    if (hasAttr(context, 'messageTopic')) {
        event['messageTopic'] = context['messageTopic'];
    }
    if (hasAttr(context, 'messageTimestamp')) {
        event['messageTimestamp'] = context['messageTimestamp'];
    }
    if (hasAttr(context, 'functionName')) {
        event['functionName'] = context['functionName'];
    }
    if (hasAttr(context, 'functionInvokeID')) {
        event['functionInvokeID'] = context['functionInvokeID'];
    }
};

exports.handler = (event, context, callback) => {
    // support Buffer & json object
    if (Buffer.isBuffer(event)) {
        const message = event.toString();
        event = {}
        event["bytes"] = message;
    }
    else if("err" in event) {
        return callback(new TypeError(event['err']))
    }

    passParameters(event, context);
    event['Say'] = 'Hello Baetyl'
    callback(null, event);
};
```

如上配置，假若 MQTTBox 基于上述配置信息已与 Hub 服务建立连接，向 Hub 发送主题为 `t` 的消息，函数计算服务会降消息分别路由给 `python27-sayhi`、`python36-sayhi`、`node85-sayhi` 和 `sql-filter` 函数处理，并分别输出主题为 `t/py2hi`、`t/py3hi`、`t/node8hi` 和 `t/sqlfilter` 的消息。这时订阅主题 `#` 的 MQTT client 将会接收到这这些消息，以及原消息 `t` 和 Hub 服务直接转主题的消息 `t/topic`。

_**提示**：凡是在 `rules` 消息路由配置项中出现、用到的函数，必须在 `functions` 配置项中进行函数实例的配置，否则无法正常启动函数运行时实例。_

### Baetyl 启动

依据 `步骤二`，执行 `sudo systemctl start baetyl` 以容器模式启动 Baetyl，如果 Baetyl 已经启动，执行 `sudo systemctl restart baetyl` 来重启。

_**提示**：Darwin 系统通过源码安装 Baetyl，可执行 `sudo baetyl start` 以容器模式启动 Baetyl。_

查看 Baetyl 主程序的日志，执行 `sudo tail -f -n 40 /usr/local/var/log/baetyl/baetyl.log` 显示如下：

![Baetyl 加载、启动日志](../images/guides/process/function-start-log.png)

同样，我们也可以通过执行命令 `docker ps` 查看系统当前正在运行的 docker 容器列表，具体如下图示。

![通过 `docker ps` 命令查看系统当前运行 docker 容器列表](../images/guides/process/docker-ps.png)

经过对比，不难发现，本次 Baetyl 启动时已经成功加载了 Hub 服务和函数计算服务，函数运行时服务实例并没有启动，因为函数运行时服务实例在有消息触发时才会动态创建。

### MQTTBox 建立连接

本次测试中，我们采用 TCP 连接方式对 MQTTBox 进行连接信息配置，然后点击 `Add subscriber` 按钮订阅主题 `#` ，该主题用于接收所有 Hub 服务收到的消息。

### 消息处理验证

通过查看 `/usr/local/var/db/baetyl/function-sayhi-code/index.py` 代码文件可以发现，在接收到某字典类格式的消息后，函数 `handler` 会对其进行一系列处理，然后将处理结果返回。返回的结果中包括各种追加的上下文信息，比如 `messageTopic`、`functionName` 等。

这里，我们通过 MQTTBox 将消息 `{"id":1}` 发布给主题 `t` ，然后观察 MQTTBox 接收到的消息如下。

![MQTTBox 接收消息](../images/guides/process/mqttbox-tcp-process-success.png)

发送消息后，我们快速执行命令 `docker ps` 查看系统当前正在运行的容器列表，所有函数运行时服务实例都被启动了，其结果如下图示。

![通过 `docker ps` 命令查看系统当前正在运行的容器列表](../images/guides/process/docker-ps-after-trigger.png)

综上，我们通过 Hub 服务和函数计算服务模拟了消息在本地处理的过程，可以看出该框架非常适合用于边缘处理消息流。
