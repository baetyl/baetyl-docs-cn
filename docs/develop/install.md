# Baetyl-cloud 安装

----

## 准备工作

- 安装k8s/k3s


----

## Helm 快速安装

### 1. 安装数据库

在安装 baetyl-cloud 之前，我们需要先安装数据库，可执行如下命令安装。

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mariadb --set rootUser.password=secretpassword,db.name=baetyl_cloud bitnami/mariadb
helm install phpmyadmin bitnami/phpmyadmin 
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

然后用浏览器打开 http://127.0.0.1:8080/index.php， 服务器输入：mariadb，账号输入：root，密码输入：secretpassword。登录后选择数据库 baetyl-cloud，点击 SQL按钮，将 baetyl-cloud 项目下 scripts/sql 目录中的所有文件的 sql 语句输入到页面执行。如果执行没有报错，则数据初始化成功。

### 3. 安装 baetyl-cloud

进入 baetyl-cloud 项目所在目录，执行如下命令。

```shell
# helm 3
helm install baetyl-cloud ./scripts/demo/charts/baetyl-cloud/
```

确认 baetyl-cloud 处于 Running 状态，也可查看日志是否报错。

```shell
kubectl get pod
# NAME                            READY   STATUS    RESTARTS   AGE
# baetyl-cloud-57cd9597bd-z62kb   1/1     Running   0          97s

kubectl logs -f baetyl-cloud-57cd9597bd-z62kb
```

成功后可通过http://0.0.0.0:30004操作baetyl-cloud Api。

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

安装mysql数据库，并初始化数据如下：

- 创建baetyl-cloud数据库及页表，具体sql语句见：*scripts/sql/tables.sql*
- 初始化页表数据，数据相关sql语句见：*scripts/sql/data.sql*
- 修改*baetyl-cloud-configmap.yml*中的数据库配置

### 2. 安装baetyl-cloud

```shell
cd scripts/demo/k8s
kubectl apply -f .
```

执行成功之后，可以通过`kubectl get pods |grep baetyl-cloud` 命令看到程序运行情况，之后就可以通过http://0.0.0.0:30004操作baetyl-cloud Api。 

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
cd scripts/demo/k8s
kubectl delete -f .
```

----

## 进程安装

### 1.安装数据库

安装mysql数据库，并初始化数据如下：

- 创建baetyl-cloud数据库及页表，具体sql语句见：*scripts/sql/tables.sql*
- 初始化页表数据，数据相关sql语句见：*scripts/sql/data.sql*
- 修改conf/cloud.yml中的数据库配置

### 2. 源码编译

参考[源码编译](../develop/build.html)

### git 3. 启动baetyl-cloud

```shell
# 导入k8s crd资源
kubectl apply -f scripts/demo/native/conf/crds.yml
cd scripts/demo/native
# 执行如下命令，然后替换conf/kubeconfig.yml文件中的example
kubectl config view --raw
# 然后执行如下命令：
nohup ../../../output/baetyl-cloud -c ./conf/cloud.yml > /dev/null &
# 执行成功后会返回成功建立的baetyl-cloud进程号
```

执行成功后可以通过http://0.0.0.0:9004 操作api服务。

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