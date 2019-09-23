/*
Title: Baetyl 0.1.4
Sort: 20
*/

# Pre-release 0.1.4(2019-07-05)

## 功能

- [#251](https://github.com/baidu/openedge/issues/251) 增加 Nodejs85 函数计算运行时
- [#260](https://github.com/baidu/openedge/issues/260) 主程序收集 IP 和 MAC 信息
- [#263](https://github.com/baidu/openedge/issues/263) 优化应用 OTA，不再重启配置未变化的服务
- [#264](https://github.com/baidu/openedge/issues/264) 优化存储卷清理逻辑，将其从 主程序迁移至 Agent 模块，会清除所有不在应用的存储卷列表中的目录
- [#266](https://github.com/baidu/openedge/issues/266) 采集服务实例的 CPU 与内存资源使用信息

## Bug 修复

- [#246](https://github.com/baidu/openedge/issues/246) Agent 模块状态上报周期从 1 分钟变更为 20 秒

## 其他

- [#269](https://github.com/baidu/openedge/issues/269) [#273](https://github.com/baidu/openedge/issues/273) [#280](https://github.com/baidu/openedge/issues/280) 更新 Makefile 文件，支持选择性部署
