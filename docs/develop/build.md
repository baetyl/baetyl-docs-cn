# 源码编译

云端管理套件源码编译介绍。

## 准备工作

- 安装 Golang 工具并启用 Modules 管理

Golang 的最低版本要求为 1.13。 下载安装 Golang 可参考 [golang.org](https://golang.org/dl/) 或者 [golang.google.cn](https://golang.google.cn/dl/)。我们现在采用 Go Modules 来管理依赖包，为了能够正常下载墙外的代码，可以参考 [goproxy.baidu.com](https://goproxy.baidu.com/) 来设置 GOPROXY，如下：

```shell
# 使用 go1.13 以上版本 
go env -w GONOSUMDB=\*                  ## 配置GONOSUMDB,暂不支持sumdb索引
go env -w GOPROXY=https://goproxy.cn    ## 配置GOPROXY,可以下载墙外代码
```

- 安装 Docker Engine 并打开 Buildx 功能

Docker 的最低版本要求为 19.03，因为从该版本开始内置了 Buildx 功能，可用于制作多平台的镜像。 安装 Docker 可参考 [docker.com/install](https://docs.docker.com/install/)，打开 Buildx 功能参考 [github.com/docker/buildx](https://github.com/docker/buildx) 。

## 下载代码

从 [Baetyl Github](https://github.com/baetyl/baetyl-cloud) 下载代码，执行如下命令：

```shell
git clone https://github.com/baetyl/baetyl-cloud.git
```

## 编译 baetyl-cloud

进入 baetyl-cloud 项目目录，执行 `make` 即可编译出当前系统平台的 baetyl-cloud 主程序。

```shell
make 
```

**注意**：国内请设置GOPROXY，否则可能导致依赖包下载不下来，参考开头的准备工作

上述命令执行完后，baetyl-cloud 主程序会生成在项目的 `output` 目录下。

## 制作镜像

如果使用容器模式运行 baetyl-cloud，我们推荐使用正式发布的官方镜像。如果你想自己制作镜像，可以使用下面提供的命令，但是前提是打开了最前面的准备工作中提到的 Buildx 功能。

进入 baetyl-cloud 项目目录，执行 `make image` 即可生成当前系统平台的模块镜像。

```shell
make image
```

**注意**：为了制作多平台的镜像，我们采用了 docker 的 buildx，参考开头的准备工作

执行结束后，你可以执行 `docker images` 来查看生成的镜像。

```shell
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
cloud                    git-be2c5a9         d70a7faf5443        About an hour ago   40.7MB
```

修改 scripts/demo/charts/baetyl-cloud/values.yaml 里的 image 配置项为上述镜像地址，执行 helm 安装，即可运行。