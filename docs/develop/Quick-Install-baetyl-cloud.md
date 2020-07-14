# 源码安装 baetyl-cloud

快速安装baetyl-cloud，你可以采用源码编译的方式来使用 baetyl-cloud 最新的功能。

## 准备工作

- 安装 Golang 工具并启用 Modules 管理

Golang 的最低版本要求为 1.13。 下载安装 Golang 可参考 [golang.org](https://golang.org/dl/) 或者 [golang.google.cn](https://golang.google.cn/dl/)。我们现在采用 Go Modules 来管理依赖包，为了能够正常下载墙外的代码，可以参考 [goproxy.baidu.com](https://goproxy.baidu.com/) 来设置 GOPROXY，如下：

```shell
# 使用 go1.13 以上版本 
go env -w GONOPROXY=\*\*.baidu.com\*\*              ## 配置GONOPROXY环境变量,所有百度内代码,不走代理
go env -w GONOSUMDB=\*                              ## 配置GONOSUMDB,暂不支持sumdb索引
go env -w GOPROXY=https://goproxy.baidu.com         ## 配置GOPROXY,可以下载墙外代码
```

- 安装 Docker Engine 并打开 Buildx 功能

Docker 的最低版本要求为 19.03，因为从该版本开始内置了 Buildx 功能，可用于制作多平台的镜像。 安装 Docker 可参考 [docker.com/install](https://docs.docker.com/install/)，打开 Buildx 功能参考 [github.com/docker/buildx](https://github.com/docker/buildx) 。

- 安装数据库

本示例默认已预安装mysql，[mysql下载](https://dev.mysql.com/downloads/mysql/)。

- 安装k8s/k3s

本示例默认已预安装k3s
```shell
curl -sfL https://get.k3s.io | sh -
```
如果国内上述安装很慢的话，可以参考使用如下命令使用国内镜像
```bash
curl -sfL https://docs.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

- 安装helm

参见[helm官网](https://helm.sh/)

```shell
brew install helm
```

## 下载代码

从 [Baetyl Github](https://github.com/baetyl/baetyl-cloud) 下载代码，执行如下命令：

```shell
git clone https://github.com/baetyl/baetyl-cloud.git
```

## 编译 baetyl-cloud

进入 baetyl-cloud 项目目录，执行 `make` 即可编译出当前系统平台的 baetyl-cloud 主程序。

```shell
# 国内请设置GOPROXY，否则无法下载被墙依赖代码，参考开头的准备工作
make 
```

上述命令执行完后，baetyl-cloud 主程序会生成在项目的 `output` 目录下。

## baetyl-cloud配置

baetyl-cloud配置定义说明参见 [配置解读](./Baetyl-cloud-config-interpretation.md)。
参考配置解读进行相关设置。service.yml参考如下：
 ```shell
activeServer:
  port: ":9003"
  ca: "./client_ca.crt"
  cert: "./server.crt"
  key: "./server.key"
  insecureSkipVerify: true
adminServer:
  port: ":9004"

nodeServer:
  port: ":9005"
  ca: "./server_ca.crt"
  cert: "./server.crt"
  key: "./server.key"

plugin: 
  auth: "defaultauth"
  license: "defaultlicense"
  pki: "defaultpki"
  shadow: "database"
  modelStorage: "kubernetes"
  databaseStorage: "database"

logger:
  filename: "./baetyl/baetyl-cloud.log"
  level: debug

kubernetes:
  inCluster: false
  configPath: "./kubernetes.yml" #需设置为准备工作里的k8s的访问配置文件

database:
  type: "mysql"
  url: "username:password@(localhost:port)/baetyl_cloud?charset=utf8&parseTime=true" #需要改为准备工作里的数据库地址

defaultpki:
  rootCAFile: "./client_ca.crt"
  rootCAKeyFile: "./client_ca.key"
  persistent:
    kind: "database"
    database:
      type: "mysql"
      url: "username:password@(localhost:port)/baetyl_cloud?charset=utf8&parseTime=true" #需要改为准备工作里的数据库地址
       
defaultauth:
  namespace: "baetyl-cloud"
  keyFile: "./token.key"
```


## 在kubernetes里创建baetyl-cloud的CRD

设置k8s集群为准备工作安装的k8s/k3s的集群配置, 运行scripts/crd/crd.yml创建自定义资源。

```shell
kubectl apply -f scripts/crd/crd.yml
```

## 数据库初始化

创建baetyl-cloud库及表结构，见scripts/sql/tables.sql
初始化数据，见examples/sql/data.sql

## 创建证书文件

通过openssl在output目录下创建根证书(nodeca.pem)及根证书key文件(nodeca.key)，通过根证书签发cloudserver.pem证书及cloudserver.key；通过openssl使用rsa算法生成加密使用的token.key。

也可以参考examples/charts/baetyl-cloud/certs/目录下的示例证书及examples/charts/baetyl-cloud/conf/token.key；将证书文件和token.key拷贝到output目录

## 启动baetyl-cloud

### 进程模式启动baetyl-cloud

baetyl-cloud编译成功之后, 在output目录下创建配置文件service.yml并进行设置，设置完成之后通过如下命令启动：

```shell
cd output
nohup ./baetyl-cloud -c ./service.yml > /dev/null &
```
启动后就可以通过http://127.0.0.1:9004访问openapi服务，http://127.0.0.1:9005访问端云同步服务，http://127.0.0.1:9003访问激活服务。

  
### 镜像模式启动baetyl-cloud

* 替换examples/charts/baetyl-cloud/conf/cloud.yml中的数据库地址为准备工作中数据库的地址；
* 替换examples/charts/baetyl-cloud/conf/k8s.yml为准备工作中k8s/k3s配置；
* 将在examples/charts/baetyl-cloud/目录复制到待部署机器上，执行helm install baetyl-cloud . 即可安装；
* 可以通过kubectl get po 命令看到程序运行情况；
* 通过http://0.0.0.0:30004可以访问openapi、通过https://0.0.0.0:30005可以访问端云同步服务、通过https://0.0.0.0:30003可以访问激活服务

#### 制作镜像

如果使用容器模式运行baetyl-cloud，我们推荐使用正式发布的官方镜像。如果你想自己制作镜像，可以使用下面提供的命令，但是前提是打开了最前面的准备工作中提到的 Buildx 功能。

进入 Baetyl 项目目录，执行 `make image` 即可生成当前系统平台的模块镜像。

```shell
# 为了制作多平台的镜像，我们采用了 docker 的 buildx，参考开头的准备工作
make image
```

执行结束后，你可以执行 `docker images` 来查看生成的镜像。

```shell
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
cloud                    git-be2c5a9         d70a7faf5443        About an hour ago   40.7MB
```
修改examples/charts/baetyl-cloud/values.yaml里image的配置为上述镜像路径，执行helm安装，即可运行。