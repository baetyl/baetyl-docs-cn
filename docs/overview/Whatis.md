# 什么是 Baetyl

[Baetyl](https://baetyl.io) 是百度云开源的边缘计算相关产品，可将云计算能力拓展至用户现场，提供临时离线、低延时的计算服务，包括设备接入、消息路由、消息远程同步、函数计算、设备信息上报、配置下发等功能。Baetyl 和 [智能边缘 BIE](https://cloud.baidu.com/product/bie.html)（Baidu-IntelliEdge）云端管理套件配合使用，通过在云端进行智能边缘核心设备的建立、存储卷创建、服务创建、函数编写，然后生成配置文件下发至 Baetyl 本地运行包，可达到云端管理和应用下发，边缘设备上运行应用的效果，满足各种边缘计算场景。

在[架构设计](./Design.md)上，Baetyl 一方面推行 **模块化**，拆分各项主要功能，确保每一项功能都是一个独立的模块，整体由主程序控制启动、退出，确保各项子功能模块运行互不依赖、互不影响；总体上来说，推行模块化的设计模式，可以充分满足用户 **按需使用、按需部署** 的切实要求；另一方面，Baetyl 在设计上还采用全面 **容器化** 的设计思路，基于各模块的镜像可以在 Docker 支持的各类操作系统上进行 **一键式构建**，依托 Docker 跨平台支持的特性，确保 Baetyl 在各系统、各平台的环境一致、标准化；此外，Baetyl 还针对 Docker 容器化模式赋予其 **资源隔离与限制** 能力，精确分配各运行实例的 CPU、内存等资源，提升资源利用效率。

## 优势

- **屏蔽计算框架**：Baetyl 提供主流运行时支持的同时，提供各类运行时转换服务，基于任意语言编写、基于任意框架训练的函数或模型，都可以在 Baetyl 中执行
- **简化应用生产**：[智能边缘 BIE](https://cloud.baidu.com/product/bie.html)云端管理套件配合 Baetyl，联合百度云，一起为 Baetyl 提供强大的应用生产环境，通过 [CFC](https://cloud.baidu.com/product/cfc.html)、[Infinite](https://cloud.baidu.com/product/infinite.html)、[Jarvis](http://di.baidu.com/product/jarvis)、[IoT EasyInsight](https://cloud.baidu.com/product/ist.html)、[TSDB](https://cloud.baidu.com/product/tsdb.html)、[IoT Visualization](https://cloud.baidu.com/product/iotviz.html) 等产品，可以在云端轻松生产各类函数、AI模型，及将数据写入百度云天工云端 TSDB 及物可视进行展示
- **简化运行环境部署**：Baetyl 推行 Docker 容器化，开发者可以根据 Baetyl 源码包中各模块的 DockerFile 快速构建 Baetyl 运行环境
- **按需部署**：Baetyl 推行功能模块化，各模块独立运行互相隔离，开发者完全可以根据自己的需求选择部署
- **丰富配置**：Baetyl 支持 X86、ARM 等多种硬件以及 Linux、Darwin 操作系统

## 贡献

欢迎来到 Baetyl 百度开源边缘计算项目，如果您想要向 Baetyl 贡献代码或文档，请遵循以下流程。

### 贡献流程

Baetyl 使用通用的 [Git 分支构建模型](http://nvie.com/posts/a-successful-git-branching-model/)。下面将为您提供通用的 Github 代码贡献方式。

1. Fork 代码库

   我们的开发社区非常活跃，感兴趣的开发者日益增多，因此，我们鼓励开发者采用 **fork** 方式向我们提交代码。关于如何 fork 一个代码库，请参考 Github 提供的官方帮助页面并点击 ["Fork" 按钮](https://help.github.com/articles/fork-a-repo/).

2. 准备开发环境

   如果您想要向 Baetyl 贡献代码，请参考如下命令准备相关本地开发环境：

   ```bash
   go get github.com/baetyl/baetyl # 获取 baetyl 代码库
   cd $GOPATH/src/github.com/baetyl/baetyl # 进入 baetyl 代码库目录
   git checkout master  # 校验当前处于 master 主分支
   git remote add fork https://github.com/<your_github_account>/baetyl  # 指定远程提交代码仓库
   ```

3. 提交代码到 fork 仓库

   这里，将改动的需求或修复的 bug 提交到步骤 2 中 fork 的远程仓库，具体请参考如下命令：

   ```bash
   git status   # 查看当前代码改变状态
   git add .
   git commit -c "modify description"  # 提交代码到本地仓库，并提交代码改动描述信息
   git push fork # 推送已提交本地仓库的代码要远程仓库
   ```

4. 创建代码合入请求

   基于 fork 的仓库地址直接向 Baetyl 官方仓库 [https://github.com/baetyl/baetyl](https://github.com/baetyl/baetyl) 提交 **pull request**（具体请参考[如何创建一个提交请求](https://help.github.com/articles/creating-a-pull-request/)），即可完成向 Baetyl 官方仓库的代码合入请求。一旦 Baetyl 代码仓库评审人员通过了您的代码提交、合入请求，您即可在 Baetyl 官方代码仓库中看到您贡献的代码。

### 代码评审规范

- Golang 的代码风格请参照 [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- 请在代码 CI 测试通过后及时通过 Email 向你的代码评审人发送代码提交请求URL
- 请及时回答评审人的每一个 comment，如果您采纳评审人给出的建议，请直接回复 **好的** 或是 **Done**；如果您不同意，请给出您的理由
- 如果您不想您的代码评审人被邮件通知频繁打扰，您可以通过 **交互框** 回复评审人提出的每一个建议，具体请参考 [如何使用交互框回复评审人信息](https://help.github.com/articles/reviewing-proposed-changes-in-a-pull-request/)
- 尽可能减少不必要的代码提交。一些开发者总是频繁提交代码。如果您想要向提交的代码中增加一个微小的改动，请使用命令 `git commit --amend` 代替 `git commit`

### 代码合入规范

无规矩不成方圆。这里规定，凡是提交 Baetyl 代码合入请求的代码，一律要求遵循以下规范：

- 建议您提交代码前再次执行命令 `govendor fmt +local`，具体请参考 [govendor](https://github.com/kardianos/govendor)
- 代码提交前 **必须** 进行单元测试（提交代码应包含）和竞争检测，参考执行命令 `make test`
- 仅有提交代码通过单元测试和竞争检测，才允许向 Baetyl 官方仓库提交
- 所有向 Baetyl 官方仓库提交的代码，**必须至少** 有 **1** 个代码评审员评审通过后，才可以将提交代码合入 Baetyl 官方代码仓库

**注意**：以上所有代码提交步骤要求及规范，同样适用文档贡献。

## 联系我们

Baetyl 作为中国首发的开源边缘计算框架，我们旨在打造一个 **轻量、安全、可靠、可扩展性强** 的边缘计算社区，为中国边缘计算技术的发展和不断推进营造一个良好的生态环境。为了更好的推进 Baetyl 的发展，如果您有更好的关于 Baetyl 的发展建议，欢迎选择如下方式与我们联系。

- 在线沟通，欢迎加入 [Baetyl 边缘计算开发者社区群](https://baetyl.bj.bcebos.com/Wechat/Wechat-Baetyl.png)
- 邮件沟通，发送到LFEDGE的BAETYL邮箱：<baetyl@lists.lfedge.org>
- 需求和BUG反馈，欢迎您在 [GitHub 提交 Issue](https://github.com/baetyl/baetyl/issues)
