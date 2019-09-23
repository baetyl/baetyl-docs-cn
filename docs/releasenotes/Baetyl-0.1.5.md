/*
Title: Baetyl 0.1.5
Sort: 10
*/

# Pre-release 0.1.5(2019-08-15)

## 功能

- [#302](https://github.com/baidu/openedge/issues/302) 支持主程序 OTA 升级
- [#305](https://github.com/baidu/openedge/issues/305) golang sdk 支持配置模板
- [#300](https://github.com/baidu/openedge/issues/300) [#301](https://github.com/baidu/openedge/issues/301) deb 包安装支持 systemd
- [#298](https://github.com/baidu/openedge/issues/298) 采集宿主机 CPU 核数
- [#297](https://github.com/baidu/openedge/issues/297) 支持 sock 文件配置和文件挂载
- [#290](https://github.com/baidu/openedge/issues/290) 支持 deb 包的构建和发布
- [#289](https://github.com/baidu/openedge/issues/289) openedge 程序前台运行（openedge 程序需要被进程守护工具管理，比被 systemd 守护）
- [#293](https://github.com/baidu/openedge/issues/293) 为处理图片或分析 AI 推断的结果，增加带 opencv 4.1.0 的 openedge-function-python36 函数运行时

## Bug 修复

- [#303](https://github.com/baidu/openedge/issues/303) 修复 openedge.sock 的 "address already in use" 问题
- [#292](https://github.com/baidu/openedge/issues/292) agent 模块检查应用配置当中的服务列表，不允许为空

## 其他

- N/A
