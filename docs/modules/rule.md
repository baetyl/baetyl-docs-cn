# rule 模块

## 简介

Baetyl-Rule 可以实现 Baetyl 框架端侧的消息流转，在 [Baetyl-Broker](https://github.com/baetyl/baetyl-broker)(端侧消息中心)、函数服务、Iot Hub(云端 Mqtt Broker) 进行消息交换。

支持以下的消息流转方式：
- 订阅 Baetyl-Broker 的消息主题，发送到自身的其他消息主题，支持函数处理
- 订阅 Baetyl-Broker 的消息主题，发送到其他消息节点（比如Iot Hub）的消息主题，支持函数处理
- 订阅其他消息节点的消息主题，发送到 Baetyl-Broker 的消息主题，支持函数处理

其中 Baetyl 支持 Python、Node、Sql 等多种运行时，可以配置相关的脚本函数对消息进行过滤、处理、转换以及丰富等，具体可以参考 [Baetyl-Function](https://github.com/baetyl/baetyl-function) 模块。

## 配置

Baetyl-Rule 的全量配置文件如下，并对配置字段做了相应解释：

```yaml
clients: # 消息节点，可以从节点订阅消息，也可以发送消息节点
  - name: iothub # 名称
    kind: mqtt # mqtt 类型
    address: 'ssl://u7isgiz.mqtt.iot.bj.baidubce.com:1884' # 地址
    username: client.example.org # 用户名
    ca: ../example/var/lib/baetyl/testcert/ca.pem # 连接节点的 CA
    key: ../example/var/lib/baetyl/testcert/client.key # 连接节点的私钥
    cert: ../example/var/lib/baetyl/testcert/client.pem # 连接节点的公钥
    insecureSkipVerify: true # 是否跳过服务端证书校验
rules: # 消息规则
  - name: rule1 # 规则名称，必须保持唯一
    source: # 消息源
      topic: broker/topic1 # 消息主题
      qos: 1 # 消息质量
    target: # 消息目的地
      client: iotcore # 消息节点，如果不设置，默认为 baetyl-broker
      topic: iotcore/topic2 # 消息主题
      qos: 0 # 消息质量
  - name: rule2 # 规则名称，必须保持唯一
    source: # 消息源
      topic: broker/topic5 # 消息主题
      qos: 0 # 消息质量
    target: # 消息目的地
      topic: broker/topic6 # 消息主题
      qos: 1 # 消息质量
    function: # 处理函数
      name: node85 # 函数名称
```

其中，Baetyl-Rule 会自动在节点信息列表中添加 Baetyl-Broker 的节点信息。当一条规则的的 client 字段未配置时，会默认设置为 Baetyl-Broker，从 Baetyl-Broker 订阅消息或者转发消息到 Baetyl-Broker。