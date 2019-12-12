# 源码安装 Baetyl

相比于快速安装 Baetyl，你可以采用源码编译的方式来使用 Baetyl 最新的功能。

## 准备工作

- 安装 Golang 工具并启用 Modules 管理

Golang 的最低版本要求为 1.12。 下载安装 Golang 可参考 [golang.org](https://golang.org/dl/) 或者 [golang.google.cn](https://golang.google.cn/dl/)。我们现在采用 Go Modules 来管理依赖包，可以参考 [goproxy.cn](https://goproxy.cn) 来启用。

- 安装 Docker Engine 并打开 Buildx 功能

Docker 的最低版本要求为 19.03，因为从该版本开始内置了 Buildx 功能，可用于制作多平台的镜像。 安装 Docker 可参考 [docker.com/install](https://docs.docker.com/install/)，打开 Buildx 功能参考 [github.com/docker/buildx](https://github.com/docker/buildx) 。

## 下载代码

从 [Baetyl Github](https://github.com/baetyl/baetyl) 下载代码，执行如下命令：

```shell
go get github.com/baetyl/baetyl
```

## 编译 Baetyl 和模块

进入 Baetyl 项目目录，执行 `make` 即可编译出当前系统平台的 Baetyl 主程序和模块程序。

```shell
cd $GOPATH/src/github.com/baetyl/baetyl
# 当前平台，所有模块
make # make all
```

上述命令执行完后，Baetyl 主程序和模块程序会生成在项目的 `output` 目录下，会按照平台分开放置。

如果需要指定平台和模块，参考如下命令：

```shell
# 所有平台，所有模块
make PLATFORMS=all
# 指定的平台，指定的模块
make PLATFORMS="linux/amd64 linux/arm64" MODULES="agent hub"
```

如下命令可以重新编译 Baetyl 主程序和模块程序。

```shell
# 当前平台，所有模块
make rebuild
# 所有平台，所有模块
make rebuild PLATFORMS=all
# 指定的平台，指定的模块
make rebuild PLATFORMS="linux/amd64 linux/arm64" MODULES="agent hub"
```

**注意**: 上述编译命令会自动读取 Git 的 revision 和 tag 作为程序的版本，所以在执行命令前请先提交或者移除本地的修改。

### 制作模块镜像

如果使用容器模式运行 Baetyl，我们推荐使用正式发布的官方镜像。如果你想自己制作镜像，可以使用下面提供的命令，但是前提是打开了最前面的准备工作中提到的 Buildx 功能。

进入 Baetyl 项目目录，执行 `make image` 即可生成当前系统平台的模块镜像。

```shell
cd $GOPATH/src/github.com/baetyl/baetyl
# 当前平台，所有模块
make image
# 当前平台，指定的模块
make image MODULES="agent hub"
```

执行结束后，你可以执行 `docker images` 来查看生成的镜像。

```shell
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
baetyl-function-python3   git-e8fe527         12b669a36a9c        54 minutes ago      202MB
baetyl-function-python2   git-e8fe527         278e5c465e17        About an hour ago   162MB
baetyl-hub                git-e8fe527         abd5bef8ba92        2 hours ago         16.9MB
baetyl-agent              git-e8fe527         7ac8dfecdb63        2 hours ago         18MB
baetyl-function-manager   git-e8fe527         e6564cd87768        2 hours ago         16.7MB
baetyl-remote-mqtt        git-e8fe527         0daa114b968d        2 hours ago         16MB
baetyl-timer              git-e8fe527         88a408e4512a        2 hours ago         16MB
baetyl-function-node8     git-e8fe527         d7bf1abb6d24        4 days ago          221MB
```

如果你需要制作多平台的镜像，你必须指定镜像发布的仓库地址和 `push` 参数，因为目前 Buildx 不支持将镜像的 manifest 加载到本地的镜像空间。

```shell
# 所有平台，所有模块
make image PLATFORMS=all XFLAGS=--push REGISTRY=<your docker image register>/
# 指定的平台，指定的模块
make image PLATFORMS="linux/amd64 linux/arm64" MODULES="agent hub" XFLAGS=--push REGISTRY=<your docker image register>/ 
```

### 安装 Baetyl 和示例

使用如下命令安装 Baetyl 和示例，默认安装到 `/usr/local`。

```shell
cd $GOPATH/src/github.com/baetyl/baetyl
sudo make install # 会同时安装 docker 容器模式的示例
sudo make install MODE=native # 会同时安装 native 进程模式的示例
```

你也可以指定安装的目录，例如项目下的 `output` ：

```shell
cd $GOPATH/src/github.com/baetyl/baetyl
make install PREFIX=output # 会同时安装 docker 容器模式的示例
make install MODE=native PREFIX=output # 会同时安装 native 进程模式的示例
```

在 Darwin 平台上，需要设置 Baetyl 使用到的 `/usr/local/var` 目录，允许挂载到容器内。

![Mount path on Mac](../images/install/docker-path-mount-on-mac.png) 

### 运行 Baetyl 和示例

如果 Baetyl 和示例已经安装到默认路径：`/usr/local`：

```shell
sudo baetyl start
```

如果 Baetyl 和示例已经安装到了指定路径，例如项目下的 `output`：

```shell
sudo ./output/bin/baetyl start
```

**注意**：启动方式根据安装方式的不同而不同，即，若选择 **docker** 运行模式安装，则上述命令会以 **docker** 容器模式运行 Baetyl。

**提示**：

- baetyl 启动后，可通过 `ps -ef | grep "baetyl"` 查看 baetyl 是否已经成功运行，并确定启动时所使用的参数。通过查看日志确定更多信息，日志文件默认存放在工作目录下的 `var/log/baetyl` 目录中。
- **docker** 容器模式运行，可通过 `docker ps` 或者 `docker stats` 命令查看容器运行状态。
- 如需使用自己的镜像，需要修改应用配置中的模块和函数的 `image`，指定自己的镜像。
- 如需自定义配置，请按照 [配置解读](../guides/Config-interpretation.md) 中的内容进行相关设置。

### 卸载

如果 Baetyl 安装在默认目录 `/usr/local`：

```shell
sudo make uninstall
```

如果 Baetyl 安装在指定目录下，例如项目下的 `output`：

```shell
make uninstall PREFIX=output
```
