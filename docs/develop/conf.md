# 配置说明

## baetyl-cloud 配置

默认配置文件是工作目录下的 etc/baetyl/service.yml，配置释义如下：

```YAML
adminServer:
  port: #云管理服务端口

nodeServer:
  port: #端云同步服务端口
  ca: #端云同步服务根证书路径
  cert: #端云同步服务证书路径
  key: #端云同步服务key文件路径

activeServer:
  port: #设备激活服务端口
  ca: #服务根证书路径
  cert: #服务端证书路径
  key: #服务端key文件路径

plugin: #在baetyl-cloud中，auth，lincese，pki，shadow，modelStorage是以插件形式实现，支持自定义
  auth: #鉴权插件，默认使用 defaultauth，不进行登录鉴权认证
  license: #license插件，默认使用 defaultlicense，不进行license限制
  pki: #证书管理插件，默认使用 defaultpki，自签证书
  shadow: #影子资源存储插件，默认使用 database
  modelStorage: #应用模型存储插件，默认使用 kubernetes
  databaseStorage: #数据库配置，默认使用 database

logger:
  filename: #日志文件路径
  level: #日志级别

kubernetes:
  inCluster: #是否使用k8s集群配置，true为使用，false为不使用
  configPath: #k8s配置文件路径，inCluster为true的情况下，不需要配置

database:
  type: #数据库类型，例如：mysql，sqlite3等
  url: #数据库的连接url

defaultpki:
  rootCAFile: #根证书路径，用于签发节点与云端连接证书的根证书，
  rootCAKeyFile: #根证书key文件路径，用于签发节点与云端连接证书的key文件
  persistent:
    kind: #存储类型 默认数据库:database
    database:
      type: #数据库类型，例如：mysql，sqlite3等
      url: #数据库的连接url
      
defaultauth:
  namespace: # baetyl-cloud使用的namespace，默认为baetyl-cloud
  keyFile: # 生成节点安装脚本时，签名token的key文件
```
