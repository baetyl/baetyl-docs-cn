# Baetyl-cloud 安装

----

## 准备工作

- 安装k8s/k3s，关于 k8s 介绍请参考 [kubernetes 官网](https://kubernetes.io)。

## 声明

- 撰写本文时所用 k8s 相关信息如下：
```
// kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.3", GitCommit:"06ad960bfd03b39c8310aaf92d1e7c12ce618213", GitTreeState:"clean", BuildDate:"2020-02-11T18:14:22Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.5", GitCommit:"20c265fef0741dd71a66480e35bd69f18351daea", GitTreeState:"clean", BuildDate:"2019-10-15T19:07:57Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
```

- 撰写本文使用 baetyl-cloud 版本信息如下：
```
// git log
commit fd97c860be11a748fa124aac721f0697312b27cf
Author: hannatao <413024870@qq.com>
Date:   Tue Feb 23 11:46:18 2021 +0800

    Update sql to the current struct (#259)
```
因为 baetyl-cloud 代码在快速迭代，最新的代码无法做到实时适配。所以用户在下载 baetyl-cloud 代码后需要切换到此版本：
```shell script
git reset --hard fd97c860be11a
```
另外本文会定期更新来适配最新的 baetyl-cloud 代码。

----

## Helm 快速安装

本文支持使用 helm v2/v3 版本进行安装，测试时相关版本信息如下：
```
// helm v3: helm version
version.BuildInfo{Version:"v3.2.3", GitCommit:"8f832046e258e2cb800894579b1b3b50c2d83492", GitTreeState:"clean", GoVersion:"go1.13.12"}

// helm v2: helm version
Client: &version.Version{SemVer:"v2.16.9", GitCommit:"8ad7037828e5a0fca1009dabe290130da6368e39", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.9", GitCommit:"8ad7037828e5a0fca1009dabe290130da6368e39", GitTreeState:"clean"}
```
关于 helm 的安装，可以参考 [helm 安装链接](https://helm.sh/docs/intro/install )。

### 1. 安装数据库

在安装 baetyl-cloud 之前，我们需要先安装数据库，可执行如下命令安装。

```shell
// helm v3
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mariadb --set rootUser.password=secretpassword,db.name=baetyl_cloud bitnami/mariadb
helm install phpmyadmin bitnami/phpmyadmin 

// helm v2
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --name mariadb --set rootUser.password=secretpassword,db.name=baetyl_cloud bitnami/mariadb
helm install --name phpmyadmin bitnami/phpmyadmin 
```

**注意**：这里为了演示方便，我们 hardcode 了密码，请自行修改，可全局替换 secretpassword。

### 2. 初始化数据

确认 mariadb 和 phpmyadmin 都进入 Runing 状态。

```shell
kubectl get pod
# NAME                            READY   STATUS             RESTARTS   AGE
# mariadb-master-0                1/1     Running            0          2m56s
# mariadb-slave-0                 1/1     Running            0          2m56s
# phpmyadmin-55f4f964d7-ctmxj     1/1     Running            0          117s
```

然后执行如下命令，保持终端不要退出。

```shell
export POD_NAME=$(kubectl get pods --namespace default -l "app=phpmyadmin,release=phpmyadmin" -o jsonpath="{.items[0].metadata.name}")
echo "phpMyAdmin URL: http://127.0.0.1:8080"
kubectl port-forward --namespace default svc/phpmyadmin 8080:80
```

然后用浏览器打开 http://127.0.0.1:8080/index.php ， 服务器输入：mariadb，账号输入：root，密码输入：secretpassword。登录后选择数据库 baetyl-cloud，点击 SQL按钮，先后将 baetyl-cloud 项目下 scripts/sql 目录中的 tables.sql 和 data.sql 输入到页面执行。如果执行没有报错，则数据初始化成功。如果之前使用本教程安装过，再次安装时请注意删除 baetyl-cloud 数据库下的历史数据。

### 3. 安装 baetyl-cloud

对于 helm v3，直接进入 baetyl-cloud 项目所在目录，执行如下命令。

```shell
# helm v3
helm install baetyl-cloud ./scripts/charts/baetyl-cloud/
```

对于 helm v2, 在上述目录下用户需要额外做些操作：

- 将 ./scripts/charts/baetyl-cloud/Chart.yaml 中 apiVersion 版本从 v2 修改为 v1 并保存，如下所示:
```yaml
apiVersion: v1
name: baetyl-cloud
description: A Helm chart for Kubernetes

...
```
- 手动导入 crd:
```shell script
# k8s版本为v1.16或更高版本 
# k3s版本为v1.17.4或更高版本执行
kubectl apply -f ./scripts/charts/baetyl-cloud/apply/
# k8s版本小于v1.16
# k3s版本小于v1.17.4
kubectl apply -f ./scripts/charts/baetyl-cloud/apply_v1beta1/
```
- helm v2 安装 baetyl-cloud:
```shell script
// helm v2
helm install --name baetyl-cloud ./scripts/charts/baetyl-cloud/
```

接下来需要确认 baetyl-cloud 处于 Running 状态，也可查看日志是否报错。

```shell
kubectl get pod
# NAME                            READY   STATUS    RESTARTS   AGE
# baetyl-cloud-57cd9597bd-z62kb   1/1     Running   0          97s

kubectl logs -f baetyl-cloud-57cd9597bd-z62kb
```

成功后可通过 http://0.0.0.0:30004 操作 baetyl-cloud API。

### 4. 安装边缘节点

调用 RESTful API 创建节点。

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

获取边缘节点的在线安装脚本。

```shell
curl http://0.0.0.0:30004/v1/nodes/demo-node/init
# {"cmd":"curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh"}
```

在 baetyl-cloud 部署地机器上执行安装脚本.

```shell
curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh
```

**注意**：如果需要在 baetyl-cloud 部署地机器以外的设备上安装边缘节点，请修改数据库将 baetyl_system_config 表中的 node-address 和 active-address 修改成真实的地址。

查看边缘节点的状态，最终会有两个边缘服务处于 Running 状态，也可调用云端 RESTful API 查看边缘节点状态，可以看到边缘节点已经在线（"ready":true）。

```shell
kubectl get pod -A
# NAMESPACE            NAME                                      READY   STATUS    RESTARTS   AGE
# baetyl-edge-system   baetyl-core-8668765797-4kt7r              1/1     Running   0          2m15s
# baetyl-edge-system   baetyl-function-5c5748957-nhn88           1/1     Running   0          114s

curl http://0.0.0.0:30004/v1/nodes/demo-node
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1939112",...,"report":{"time":"2020-07-22T07:25:27.495362661Z","sysapps":...,"node":...,"nodestats":...,"ready":true}
```

### 5. 卸载baetyl-cloud

```shell
helm delete baetyl-cloud
```

----

## K8s 安装

### 1. 安装数据库

安装 mysql 数据库，并初始化数据如下：

- 创建 baetyl-cloud 数据库及表，具体 sql 语句见：*scripts/common/tables.sql*

- 初始化表数据，数据相关 sql 语句见：*scripts/k8s/sql/data.sql*

  ```shell
  # 注意修改 baetyl_property 中 sync-server-address 和 init-server-address 为实际的服务器地址：
  # 比如服务部署在本机，则地址可配置如下：
  # sync-server-address : https://0.0.0.0:30005
  # init-server-address : https://0.0.0.0:30003
  # 若服务部署在非本机，请将IP更改为实际的服务器IP地址
  ```

- 修改 *baetyl-cloud-configmap.yml* 中的数据库配置

### 2. 安装 baetyl-cloud

```shell
cd scripts/k8s
# k8s版本为v1.16或更高版本 
# k3s版本为v1.17.4或更高版本执行
kubectl apply -f ./apply/
# k8s版本小于v1.16
# k3s版本小于v1.17.4
kubectl apply -f ./apply_v1beta1/
```

执行成功之后，可以通过`kubectl get pods |grep baetyl-cloud` 命令看到程序运行情况，之后就可以通过 http://0.0.0.0:30004 操作 baetyl-cloud API。 

### 3. 安装边缘节点

调用 RESTful API 创建节点。

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

获取边缘节点的在线安装脚本。

```shell
curl http://0.0.0.0:30004/v1/nodes/demo-node/init
# {"cmd":"curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh"}
```

在 baetyl-cloud 部署地机器上执行安装脚本.

```shell
curl -skfL 'https://0.0.0.0:30003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh
```

**注意**：如果需要在 baetyl-cloud 部署地机器以外的设备上安装边缘节点，请修改数据库将 baetyl_system_config 表中的 node-address 和 active-address 修改成真实的地址。

查看边缘节点的状态，最终会有两个边缘服务处于 Running 状态，也可调用云端 RESTful API 查看边缘节点状态，可以看到边缘节点已经在线（"ready":true）。

```shell
kubectl get pod -A
# NAMESPACE            NAME                                      READY   STATUS    RESTARTS   AGE
# baetyl-edge-system   baetyl-core-8668765797-4kt7r              1/1     Running   0          2m15s
# baetyl-edge-system   baetyl-function-5c5748957-nhn88           1/1     Running   0          114s

curl http://0.0.0.0:30004/v1/nodes/demo-node
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1939112",...,"report":{"time":"2020-07-22T07:25:27.495362661Z","sysapps":...,"node":...,"nodestats":...,"ready":true}
```

### 4. 卸载baetyl-cloud

```shell
cd scripts/k8s
# k8s版本为v1.16或更高版本 
# k3s版本为v1.17.4或更高版本执行
kubectl delete -f ./apply/
# k8s版本小于v1.16
# k3s版本小于v1.17.4
kubectl delete -f ./apply_v1beta1/
```

----

## 进程安装

### 1.安装数据库

安装 mysql 数据库，并初始化数据如下：

- 创建 baetyl-cloud 数据库及表，具体sql语句见：*scripts/common/tables.sql*

- 初始化表数据，数据相关 sql 语句见：*scripts/native/sql/data.sql*

    ```shell
  # 注意修改 baetyl_property 中 sync-server-address 和 init-server-address 为实际的服务器地址：
  # 比如服务部署在本机，则地址可配置如下：
  # sync-server-address : https://0.0.0.0:9005
  # init-server-address : https://0.0.0.0:9003
  # 若服务部署在非本机，请将IP更改为实际的服务器IP地址
  ```

- 修改 *native/conf/conf.yml* 中的数据库配置

### 2. 源码编译

参考[源码编译](../develop/build.html)

### 3. 启动 baetyl-cloud

```shell
cd scripts/native
# 导入 k8s crd 资源
# k8s版本为v1.16或更高版本 
# k3s版本为v1.17.4或更高版本执行
kubectl apply -f ./apply/
# k8s版本小于v1.16
# k3s版本小于v1.17.4
kubectl apply -f ./apply_v1beta1/
# 执行如下命令，然后替换 conf/kubeconfig.yml 文件中的 example
kubectl config view --raw
# 然后执行如下命令：
nohup ../../output/baetyl-cloud -c ./conf/conf.yml > /dev/null &
# 执行成功后会返回成功建立的 baetyl-cloud 进程号
```

执行成功后可以通过 http://0.0.0.0:9004 操作 baetyl-cloud API。

### 4. 安装边缘节点

调用 RESTful API 创建节点。

```shell
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:9004/v1/nodes
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1931564","createTime":"2020-07-22T06:25:05Z","labels":{"baetyl-node-name":"demo-node"},"ready":false}
```

获取边缘节点的在线安装脚本。

```shell
curl http://0.0.0.0:9004/v1/nodes/demo-node/init
# {"cmd":"curl -skfL 'https://0.0.0.0:9003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh"}
```

在 baetyl-cloud 部署地机器上执行安装脚本.

```shell
curl -skfL 'https://0.0.0.0:9003/v1/active/setup.sh?token=f6d21baa9b7b2265223a333630302c226b223a226e6f6465222c226e223a2264656d6f2d6e6f6465222c226e73223a2262616574796c2d636c6f7564222c227473223a313539353430323132367d' -osetup.sh && sh setup.sh
```

**注意**：如果需要在 baetyl-cloud 部署地机器以外的设备上安装边缘节点，请修改数据库将 baetyl_system_config 表中的 node-address 和 active-address 修改成真实的地址。

查看边缘节点的状态，最终会有两个边缘服务处于 Running 状态，也可调用云端 RESTful API 查看边缘节点状态，可以看到边缘节点已经在线（"ready":true）。

```shell
kubectl get pod -A
# NAMESPACE            NAME                                      READY   STATUS    RESTARTS   AGE
# baetyl-edge-system   baetyl-core-8668765797-4kt7r              1/1     Running   0          2m15s
# baetyl-edge-system   baetyl-function-5c5748957-nhn88           1/1     Running   0          114s

curl http://0.0.0.0:9004/v1/nodes/demo-node
# {"namespace":"baetyl-cloud","name":"demo-node","version":"1939112",...,"report":{"time":"2020-07-22T07:25:27.495362661Z","sysapps":...,"node":...,"nodestats":...,"ready":true}
```

### 5. 进程退出

```shell
# 根据创建成功时的进程号杀死进程：
sudo kill 进程号
```