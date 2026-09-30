---
aliases:
- "Kubernetes 了解吗？"
- "工具与工程 3.3 Kubernetes 了解吗？"
---

# 03 Kubernetes 了解吗？

## 01 核心回答


**概念原理：**Kubernetes（K8s）是容器编排平台：声明式管理（写 YAML 描述期望状态，控制器持续调和实际状态）。核心对象：**Pod**（最小调度单元）、**Deployment**（无状态应用副本 + 滚动更新）、**Service**（稳定访问入口 + 负载均衡）、**ConfigMap/Secret**（配置）、**Ingress**（七层入口）、**HPA**（自动伸缩）。架构：Master（API Server/Controller/Scheduler）+ Worker（Kubelet/容器运行时）。

**实现流程（部署一个应用）：**① 写 Deployment YAML（镜像、副本数、资源限制、探针）；② `kubectl apply` 提交，Scheduler 把 Pod 调度到节点；③ Kubelet 拉镜像启动容器，探针（就绪/存活）通过后加入 Service 端点；④ 更新时滚动发布（逐个替换，新旧并存），失败报告状态，由发布系统或操作者决定暂停或回滚；⑤ 流量经 Ingress → Service → Pod。

**关键细节：**Pod 是**临时**的（重建后 IP 变，靠 Service 抽象）；资源 requests/limits 必须设（调度依据 + 限制）；探针决定流量摘除和重启；滚动发布要配 `maxUnavailable/maxSurge`；ConfigMap 变更不自动触发滚动重启；卷、环境变量和应用热加载需分别设计。

**风险与取舍：**K8s 运维复杂度高（学习成本、etcd 高可用）；有状态应用（数据库）用 Operator 或托管服务更省心；网络插件（CNI）和存储（PV/PVC）选型影响性能；排查问题链路长（Pod 状态 → 事件 → 日志 → 网络）。

**项目落地：**CI/CD：镜像推仓库 → GitOps（Argo CD）同步到 K8s；HPA 按 CPU/QPS 自动扩缩容；资源配额和命名空间隔离环境；监控用 Prometheus + Grafana 采集 Pod 指标。

**面试追问：**Deployment 和 StatefulSet 的区别（无状态 vs 有状态，稳定网络标识）；Service 怎么发现 Pod（标签选择器 + Endpoints）；滚动更新和蓝绿发布的区别；Pod 重启会丢什么（本地文件/内存状态，数据卷除外）。

---

## 02 理解补充与边界校订

Deployment 失败会报告停滞状态，原生控制器不会因进度期限超时就自动回滚，需发布工具或人工策略。ConfigMap 卷更新与环境变量更新行为不同，而且配置更新不自动触发 Deployment rollout。readiness 决定是否接流量，liveness 决定是否重启，startup 为慢启动提供保护；混淆探针可能导致故障重启风暴。

## 03 依据与延伸阅读

- [Kubernetes Deployment 行为](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [ConfigMap 更新边界](https://kubernetes.io/docs/concepts/configuration/configmap/)

## 04 相关问题

- [[八股/10-工程实践/06-容器与部署/01-Docker 了解吗？在项目中怎么使用|Docker 了解吗？在项目中怎么使用]]：容器打包与声明式编排

## 05 所属专题

- [[八股/10-工程实践/06-容器与部署/00-容器与部署导航|容器与部署导航]]
