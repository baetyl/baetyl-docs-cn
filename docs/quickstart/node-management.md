# 边缘节点

## 功能简介
节点是边缘设备在云端的映射，云端一个节点代表一个边缘设备。通过在云端创建节点，在边缘设备完成节点安装后，就可以对边缘设备进行管理。此外，还可以在云端查看已有的节点，编辑节点的信息，删除节点。

## 使用说明
对节点的操作包括创建节点，删除节点，对节点编辑，以及安装节点。

### 创建节点
左侧菜单栏点击边缘节点，进入边缘节点页面并点击 **创建节点**，进入节点创建页面

![创建节点1](../images/quickstart/node-management/01-create-node-1.png)

![创建节点2](../images/quickstart/node-management/02-create-node-2.png)

* 名称: 节点名称，不可为空，不可重复
* 描述：描述可以为空
* 标签：对节点进行标识，用于关联应用，可以不绑定标签，也可以绑定多个标签
* 认证方式: 节点端云同步使用证书（强制）

点击确定完成节点创建后可以在节点列表看到已创建节点。

![节点列表](../images/quickstart/node-management/03-node-list.png)

### 删除节点
在节点列表页面点击删除，弹出确定窗口

![删除节点](../images/quickstart/node-management/04-delete-node.png)

点击确定后完成节点删除。

### 节点编辑

点击节点进入节点详情页可对节点描述信息和标签进行编辑，可以添加新的标签，修改已有标签键值，或删除已有标签。

![编辑节点标签](../images/quickstart/node-management/05-node-update-labels.png)

### 节点安装
节点安装目前支持 Kube 模式在线安装。baetyl运行模式包括 **k3s+docker** 和 **k3s+containerd** 两种，用户根据实际需要选择运行环境安装命令，需要注意的是，若用户选择 **k3s+docker** 运行环境，需先安装 docker 后再安装 k3s。

- 节点安装-containerd
![节点安装-containerd](../images/quickstart/node-management/06-node-install-command-containerd.png)

- 节点安装-docker
![节点安装-docker](../images/quickstart/node-management/07-node-install-command-docker.png)

运行环境准备就绪后，执行节点安装命令，即可安装边缘节点，效果如下：

![节点安装效果](../images/quickstart/node-management/08-node-install.png)


采用上述两种模式安装都会在 **baetyl-edge-system** 部署 **baetyl-core** 和 **baetyl-function** 两个服务。查看 **baetyl-edge-system** 命名空间下的pod均处于运行状态即表示节点安装完成。

![节点检查](../images/quickstart/node-management/10-node-check.png)


### 查看节点
在节点详情页可以看到节点已连接，显示了节点的详细信息与资源使用情况，这里需要手动刷新状态，默认20秒会上报一次状态，无需频繁刷新。

![节点详情1](../images/quickstart/node-management/11-node-detail-1.png)

点击左边栏应用部署菜单可以查看已部署的应用相关信息和资源使用情况。

![节点详情2](../images/quickstart/node-management/12-node-detail-2.png)

