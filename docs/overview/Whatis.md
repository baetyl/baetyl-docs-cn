# 什么是 Baetyl

[Baetyl](https://baetyl.io) 是 [Linux Foundation Edge](https://www.lfedge.org) 旗下项目，旨在将云计算能力拓展至用户现场，提供临时离线、低延时的计算服务，包括设备接入、消息路由、消息远程同步、函数计算、设备信息上报、配置下发等功能。Baetyl 和 [智能边缘 BIE](https://cloud.baidu.com/product/bie.html)（Baidu-IntelliEdge）云端管理套件配合使用，通过在云端进行智能边缘核心设备的建立、存储卷创建、服务创建、函数编写，然后生成配置文件下发至 Baetyl 本地运行包，整体可达到 **边缘计算、云端管理、边云协同** 的效果，满足各种边缘计算场景。

在[架构设计](Design.md)上，Baetyl 一方面推行 **模块化**，拆分各项主要功能，确保每一项功能都是一个独立的模块，整体由主程序控制启动、退出，确保各项子功能模块运行互不依赖、互不影响；总体上来说，推行模块化的设计模式，可以充分满足用户 **按需使用、按需部署** 的切实要求；另一方面，Baetyl 在设计上还采用全面 **容器化** 的设计思路，基于各模块的镜像可以在支持 Docker 的各类操作系统上进行 **一键式构建**，依托 Docker 跨平台支持的特性，确保 Baetyl 在各系统、平台的环境一致；此外，Baetyl 还针对 Docker 容器化模式赋予其 **资源隔离与限制** 能力，精确分配各运行实例的 CPU、内存等资源，提升资源利用效率。

## 优势

- **屏蔽计算框架**：Baetyl 提供主流运行时支持的同时，提供各类运行时转换服务，基于任意语言编写、基于任意框架训练的函数或模型，都可以在 Baetyl 中执行
- **简化应用生产**：[智能边缘 BIE](https://cloud.baidu.com/product/bie.html)云端管理套件配合 Baetyl，联合百度云，一起为 Baetyl 提供强大的应用生产环境，通过 [CFC](https://cloud.baidu.com/product/cfc.html)、[Infinite](https://cloud.baidu.com/product/infinite.html)、[EasyEdge](https://ai.baidu.com/easyedge/home)、[TSDB](https://cloud.baidu.com/product/tsdb.html)、[IoT Visualization](https://cloud.baidu.com/product/iotviz.html) 等产品，可以在云端轻松生产各类函数、AI模型，及将数据写入百度云天工云端 TSDB 及物可视进行展示
- **服务按需部署**：Baetyl 推行容器化和模块化，各模块独立运行互相隔离，开发者完全可以根据自己的需求选择部署
- **支持多种平台**：Baetyl 支持多种软硬件平台，比如 X86 和 ARM 架构的CPU，Linux 和 Darwin 操作系统等

## 组成

Baetyl 作为一个边缘计算框架，除了提供底层服务管理能力外，还提供一些基础功能模块，具体如下：

- Baetyl [主程序](Design.html#id3) 负责服务实例的管理，如启动、退出、守护等，由引擎系统、API、命令行构成。目前支持两种运行模式：Native 进程模式和 Docker 容器模式
- 官方模块 [baetyl-agent](Design.html#baetyl-agent) 负责和 BIE 云端管理套件通讯，可以进行应用下发，设备信息上报等。强制证书认证，保证传输安全；
- 官方模块 [baetyl-hub](Design.html#baetyl-hub) 提供基于 [MQTT 协议](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html) 的消息订阅和发布功能，支持 4 种接入方式：TCP、SSL、WS 及 WSS；
- 官方模块 [baetyl-remote-mqtt](Design.html#baetyl-remote-mqtt) 用于桥接两个 MQTT Server 进行消息同步，支持配置多路消息转发；
- 官方模块 [baetyl-function-manager](Design.html#baetyl-function-manager) 提供基于 MQTT 消息机制，弹性、高可用、扩展性好、响应快的计算能力；
- 官方模块 [baetyl-function-python27](Design.html#baetyl-function-python27) 提供 Python2.7 函数运行时，可由 `baetyl-function-manager` 动态启动实例；
- 官方模块 [baetyl-function-python36](Design.html#baetyl-function-python36) 提供 Python3.6 函数运行时，可由`baetyl-function-manager` 动态启动实例；
- 官方模块 [baetyl-function-node85](Design.html#baetyl-function-node85) 提供 Node 8.5 函数运行时，可由`baetyl-function-manager` 动态启动实例；
- SDK (Golang) 可用于开发自定义模块。

### 架构图

![架构图](../images/overview/design/design_overview.png)

## 快速安装

- [快速安装 Baetyl](../install/Quick-Install.md)
- [源码编译 Baetyl](../install/Build-from-Source.md)

## 开发文档

- [Baetyl 设计](Design.md)
- [Baetyl 配置解读](../guides/Config-interpretation.md)
- [如何针对 Python 运行时编写 Python 脚本](../develop/How-to-write-a-python-script-for-python-runtime.md)
- [如何针对 Node 运行时编写 Node 脚本](../develop/How-to-write-a-node-script-for-node-runtime.md)
- [如何针对 Python 运行时引入第三方包](../develop/How-to-import-third-party-libraries-for-python-runtime.md)
- [如何针对 Node 运行时引入第三方包](../develop/How-to-import-third-party-libraries-for-node-runtime.md)
- [如何开发一个自定义函数运行时](../develop/How-to-develop-a-customize-runtime-for-function.md)
- [如何开发一个自定义模块](../develop/How-to-develop-a-customize-module.md)

## 如何贡献

如果您热衷于开源社区贡献，Baetyl 将为您提供两种贡献方式，分别是代码贡献和文档贡献。具体请参考 [如何向 Baetyl 贡献代码和文档](Contributing.md)。

## 联系我们

Baetyl 作为中国首发的开源边缘计算框架，我们旨在打造一个 **轻量、安全、可靠、可扩展性强** 的边缘计算社区，为中国边缘计算技术的发展和不断推进营造一个良好的生态环境。为了更好的推进 Baetyl 的发展，如果您有更好的关于 Baetyl 的发展建议，欢迎选择如下方式与我们联系。

- 欢迎加入 [Baetyl 边缘计算开发者社区群](https://baetyl.bj.bcebos.com/Wechat/Wechat-Baetyl.png)
- 欢迎加入 [Baetyl 的 LF Edge 讨论组](https://lists.lfedge.org/g/baetyl/topics)
- 欢迎发送邮件到：<baetyl@lists.lfedge.org>
- 欢迎到 [GitHub 提交 Issue](https://github.com/baetyl/baetyl/issues)
