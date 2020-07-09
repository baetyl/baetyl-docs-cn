# FAQ
**Q: 在Mac系统下进行应用部署实践--部署mosquitto broker应用时，用户应用broker的状态显示已部署，最后使用mqtt.box连接边缘设备时，遇到Connection Error，请问是什么原因？**

**A**: 由于Mac系统下宿主机端口映射无法正常工作，会导致mqtt.box连接失败。

**Q: 节点预配时采用SN文件方式激活时，部署状态一直只有baetyl-init在running，请问是什么原因？**

**A**: Mac系统下系统目录下的文件映射存在问题，只能选择将SN文件放置在用户目录下，请修改SN文件路径后再次部署。
