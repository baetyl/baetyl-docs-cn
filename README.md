# cn.docs.baetyl.io

[![Documentation Status](https://img.shields.io/badge/docs-latest-brightgreen.svg?style=flat)](https://docs.baetyl.io/zh_CN/latest/) [![License](https://img.shields.io/github/license/baetyl/baetyl?color=brightgreen)](LICENSE)

![Baetyl-logo](./docs/images/logo/logo-with-name.png)

[Baetyl](https://baetyl.io) 是 [Linux Foundation Edge](https://www.lfedge.org) 旗下项目，旨在将云计算能力拓展至用户现场，提供临时离线、低延时的计算服务，包括设备接入、消息路由、消息远程同步、函数计算、设备信息上报、配置下发等功能。Baetyl 和 [智能边缘 BIE](https://cloud.baidu.com/product/bie.html)（Baidu-IntelliEdge）云端管理套件配合使用，通过在云端进行智能边缘核心设备的建立、存储卷创建、服务创建、函数编写，然后生成配置文件下发至 Baetyl 本地运行包，整体可达到 **边缘计算、云端管理、边云协同** 的效果，满足各种边缘计算场景。

## 项目内容

- 概述
  - [什么是 Baetyl](./docs/overview/Whatis.md)
  - [Baetyl 设计](./docs/overview/Design.md)
- 安装
  - [快速安装 Baetyl](./docs/install/Quick-Install.md)
  - [源码编译 Baetyl](./docs/install/Build-from-Source.md)
- 操作指南
  - [Baetyl 配置解读](./docs/guides/Config-interpretation.md)
  - [通过 Hub 服务将设备接入 Baetyl](./docs/guides/Device-connect-to-hub-module.md)
  - [利用 Hub 服务进行设备间消息转发](./docs/guides/Message-transfer-among-devices-with-hub-module.md)
  - [利用 Function 函数计算服务进行消息处理](./docs/guides/Message-handling-with-function-module.md)
  - [利用 Remote 远程服务实现 Baetyl 与百度 IoTHub 消息同步](./docs/guides/Message-synchronize-with-iothub-through-remote-module.md)
- 自定义开发
  - [如何针对 Python 运行时编写 Python 脚本](./docs/develop/How-to-write-a-python-script-for-python-runtime.md)
  - [如何针对 Node 运行时编写 Node 脚本](./docs/develop/How-to-write-a-node-script-for-node-runtime.md)
  - [如何针对 Python 运行时引入第三方包](./docs/develop/How-to-import-third-party-libraries-for-python-runtime.md)
  - [如何针对 Node 运行时引入第三方包](./docs/develop/How-to-import-third-party-libraries-for-node-runtime.md)
  - [如何开发一个自定义函数运行时](./docs/develop/How-to-develop-a-customize-runtime-for-function.md)
  - [如何开发一个自定义模块](./docs/develop/How-to-develop-a-customize-module.md)
- 最佳实践
  - [如何利用 Baetyl 上传数据到百度云 TSDB](./docs/practice/Write-data-to-TSDB.md)
- 常见问题
  - [FAQ](./docs/FAQ.md)
- 资源下载
  - [资源下载](./docs/Resources.md)

## 贡献

如果您热衷于开源社区贡献，具体请参考 [如何向 Baetyl 贡献代码和文档](./docs/overview/Contributing.md)。

## LICENSE

Baetyl 遵从 [Apache-2.0](./LICENSE) 协议。

## 联系我们

Baetyl 作为中国首发的开源边缘计算框架，我们旨在打造一个 **轻量、安全、可靠、可扩展性强** 的边缘计算社区，为中国边缘计算技术的发展和不断推进营造一个良好的生态环境。为了更好的推进 Baetyl 的发展，如果您有更好的关于 Baetyl 的发展建议，欢迎选择如下方式与我们联系。

- 欢迎加入 [Baetyl 边缘计算开发者社区群](https://baetyl.bj.bcebos.com/Wechat/Wechat-Baetyl.png)
- 欢迎加入 [Baetyl 的 LF Edge 讨论组](https://lists.lfedge.org/g/baetyl/topics)
- 欢迎发送邮件到：<baetyl@lists.lfedge.org>