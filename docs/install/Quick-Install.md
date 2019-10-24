# 快速安装 Baetyl

Baetyl 快速安装支持的系统有: Ubuntu16.04、Ubuntu18.04、Debian9、CentOS7、Raspbian、Darwin，支持的平台有 amd64、i386、armv7l 和 arm64。

Baetyl 支持两种运行模式，分别是 **docker** 容器模式和 **native** 进程模式。本文基于 **docker** 容器模式进行快速安装。

## 安装容器运行时

在 **docker** 容器模式下，Baetyl 依赖于 docker 容器运行时。如果用户机器尚未安装 docker 容器运行时，可通过以下命令安装 docker 的最新版本（适用于类 Linux 系统）:

```shell
curl -sSL https://get.docker.com | sudo sh
```

对于 CentOS7 系统，在安装结束后需要手动启动 docker:

```shell
# 设置 docker 开机自启动
sudo systemctl enable docker
# 启动 docker
sudo systemctl start docker
```

在 Darwin 系统上安装 Docker 可参考 [docker.com/install](https://docs.docker.com/docker-for-mac/install/)。

然后设置 Baetyl 的配置目录 `/usr/local/var` 允许挂载到容器内。

![Mount path on Mac](../images/install/docker-path-mount-on-mac.png) 

安装结束后，用户可以查看 docker 的版本号:

```shell
docker version
```

**注意**：建议用户安装/更新 Docker 版本到 19.03 及以上。

**更多内容请参考 [官方文档](https://docs.docker.com/install/)。**

## 安装 Baetyl

Baetyl 发布新版本的同时，也会发布对应的二进制文件以及相应的 rpm、deb 包。使用以下命令，可以将 Baetyl 快速安装到设备上:

```shell
curl -sSL http://dl.baetyl.io/install.sh | sudo bash
```

执行完毕后，Baetyl 将会被安装到 /usr/local/bin 目录下。Baetyl 的运行配置存放在 /usr/local/etc/baetyl 和 /usr/local/var/db/baetyl 目录下，具体的配置方法可以参考 [配置文件解读文档](guides/Config-interpretation.md)。

## 导入示例配置包（可选）

作为一个边缘计算框架，Baetyl 通过 hub 模块提供基于 MQTT 的消息路由服务，通过 function-manager 模块和 python27、python36、nodejs85、sql 等运行时模块来提供本地函数计算服务。用户需要通过编辑配置文件，来让 baetyl 主程序加载相应的模块以及设定模块本身的运行参数。关于各个模块的配置详情，可参考 [配置解读](../guides/Config-interpretation.md) 中的内容进行进一步了解。

Baetyl 官方提供了一套示例配置，用户可通过以下命令导入示例配置文件:

```shell
# Darwin OS use Docker as a non-root user. So we don't run this script with "sudo" directly.
curl -O http://dl.baetyl.io/install_with_docker_example.sh && bash install_with_docker_example.sh
```

上述脚本将会检测当前系统上是否存在历史配置文件，用户根据提示选择**是否**删除历史配置文件。用户也可以根据脚本中的提示选择**是否**提前拉取示例配置中要用到的各个模块的镜像。

示例配置只用于学习和测试目的，用户应根据自己的实际工作场景按需配置。

如果用户不需要启动任何模块，那就不需要导入任何配置文件。

## 启动 Baetyl

采用快速安装方式后，用户可以下列直接执行命令来启动 Baetyl:

```shell
# Linux
sudo baetyl start
# Darwin
baetyl start
```

在 Linux 系统上，用户可以使用 Systemd 来管理 Baetyl 的启动（start）、停止（stop）和查看状态（status）:

启动 Baetyl:

```shell
sudo systemctl start baetyl
```

停止 Baetyl:

```shell
sudo systemctl stop baetyl
```

查看运行状态:

```shell
sudo systemctl status baetyl
```

如果用户之前安装过 Baetyl 或者导入了新的配置文件，请重启 Baetyl:

```shell
sudo systemctl restart baetyl
```

在 Darwin 系统上，用户可以使用 Launchctl 来管理 Baetyl 的启动（load）、停止（unload）:

启动 Baetyl:

```shell
ln -sfv /usr/local/etc/baetyl/baetyl.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/baetyl.plist 
```

停止 Baetyl:

```shell
launchctl unload ~/Library/LaunchAgents/baetyl.plist 
```

## 验证是否成功安装

快速安装 Baetyl 以后，用户可依据以下步骤验证 Baetyl 是否启动成功：

- Linux 系统上在终端中命令 `sudo systemctl status baetyl` 来查看 `baetyl` 是否正常运行。正常如下图所示，否则说明主程序 `baetyl` 启动失败；

![Baetyl](../images/install/systemctl-status.png)

- 在 Linux 或者 Darwin 系统上，均可在终端中执行命令 `docker stats` 查看各个模块的 docker 容器是否正常运行。如果事先已经拉取了各个模块的镜像，各个模块的容器可以很快运行起来。如果未事先拉取，主程序 `baetyl` 会先到镜像仓库拉取需要的镜像，用户需要等待 2~5 分钟执行此条命令。以上一步中导入的示例配置为例，待主程序拉取完成后，容器的运行状态如下图所示。如果用户本地的镜像与下述不一致，说明模块启动失败；

![当前运行 docker 容器查询](../images/install/docker-stats.png)

- 针对上述两种失败情况，用户需要查看主程序日志来了解具体的错误情况。主程序日志的默认存放位置为 `/usr/local/var/log/baetyl/baetyl.log`。针对日志中出现的错误，用户可先参考 [常见问题](../FAQ.md) 进行解决。必要时可以直接 [提交 Issue](https://github.com/baetyl/baetyl/issues)。
