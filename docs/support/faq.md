# FAQ
**Q：在Mac系统下进行应用部署实践--部署mosquitto broker应用时，用户应用broker的状态显示已部署，最后使用mqtt.box连接边缘设备时，遇到Connection Error，请问是什么原因？**

**A**：目前框架仅支持Linux系统下的边缘设备，由于在Mac系统下宿主机端口无法正常工作，会引起mqtt.box连接失败，另外如果在节点预配时选择手动激活的方式，Mac系统下也无法打开浏览器进行激活。
