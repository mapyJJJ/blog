>对于 kubernetes 来说，对外的api接口由 **kube-apiserver** 服务提供 
作为前端接口，各种第三方 cli 以及 前端页面 可用接入这些 api 提供操作各种资源的能力
对于目前主流的监控方案而言，也都是依赖于此

---

##### 时序数据库 - Prometheus
> 时序记录简单来说就是 存入每个时刻的数据点 后面按照周期，展示趋势，规律

特点
- 高并发写入
- 快速进行大量数据的聚合或分组等运算

不同于 innodb 的 b tree 结构，b树对于插入和查询 其时间复杂度 都是 log(N)
主要耗时在寻道 随机io 
所以主流的时序数据库都是 LSM tree 结构，主要思想是 先 内存写 在到 顺序io写入

核心组件:
- Prometheus Server: 抓取数据，存入时序数据库，提供查询接口
- client libraries : 客户端，上报数据 查询数据
- metrics端 : Server会定期从 exporter 拉取数据，以及Pushgateway的汇报
- alertmanager : 通过rules 指标，达到告警指标，进行相应的提醒

主要场景:
- 主要针对性能和可用性监控，不适用于针对日志（Log）、事件（Event）、调用链（Tracing）等的监控。


**helm 部署:**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
```

**将部署 6 个应用**
- alertmanager (statefulSet)
- grafana (Deployment)
- kube-prometheus-operator(deployment)
- kube-state-metrics (deployment)
- node-exporter (DaemonSet) 
- prometheus (statefulSet)

此时 node-exporter，kube-state-metrics 已经开始收集数据提供给 prometheus 
kube-state-metrics 主要收集应用，pod，node内部对象的状态
node-exporter 提供cpu，磁盘，内存 信息

**通过grafana可视化观察数据**

通过nodeport service对外提供访问grafana入口
修改默认的service, prometheus-grafana 改为 NodePort类型

默认grafana的账号密码:
```
username: admin
password: prom-operator
```

**整理一些常用的kube-state-metrics查询语句**

[官方指标文档](https://github.com/kubernetes/kube-state-metrics/tree/master/docs)

```
count(kube_pod_status_phase{phase="Running/Pending/Failed",namespace="xxx"}) # 查询namespace中不同状态的pod数量

sum(kube_deployment_status_replicas{deployment="xxx"})  # deployment replicas数量
sum(kube_deployment_status_replicas_available{deployment="xxx"})
sum(kube_deployment_status_replicas_unavailable{deployment="xxx"})

...
```


---

##### 集群完整视图 - weave

主要功能:
- 展示节点，pod，应用之间的逻辑结构
- 实时占用资源监控
- 浏览器打开命令行
- 搜索功能定位资源

```
kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"
# 修改weave service改为nodePort类型，进行远程访问
```

由于可以直接通过weave修改登入集群环境，需要开启前端页面鉴权

修改 yaml

weave-scope-app deployment加入env参数 :

```yaml
env:
  - name: ENABLE_BASIC_AUTH
    value: 'true'
  - name: BASIC_AUTH_USERNAME
    value: admin
  - name: BASIC_AUTH_PASSWORD
    value: password
```
weave-scope-agent DaemonSet 加入同样的env参数，该守护进程是用于抓取数据的


---

#### 获取pod运行日志 - loki

```
helm repo add loki https://grafana.github.io/loki/charts
helm upgrade --install loki --namespace=monitoring loki/loki-stack
```

promtail(抓取数据) , loki(提供服务)
promtail主要从 /var/log/pods/ 获取各个pod的运行日志

配合 grafana 用于可视化日志