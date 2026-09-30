---
aliases:
- "Docker 了解吗？在项目中怎么使用？"
- "工具与工程 3.1 Docker 了解吗？在项目中怎么使用？"
---

# 01 Docker 了解吗？在项目中怎么使用？

## 01 核心回答


**概念原理：**Docker 是容器化技术：用**命名空间（Namespace）**隔离进程视图（PID/网络/文件系统），用**Cgroups**限制资源（CPU/内存），镜像分层复用文件系统，实现"一次构建、到处运行"。与虚拟机区别：共享宿主机内核、无完整 OS 开销、秒级启动。

**实现流程（项目中使用）：**① 写 `Dockerfile`（基础镜像 → 复制产物 → 启动命令），多阶段构建减小镜像；② `docker build` 构建镜像，推送到私有仓库（Harbor）；③ `docker run` 启动；④ 多容器用 `docker compose` 编排（应用 + Redis + MySQL 本地一键起）；⑤ 生产由 K8s 调度，实际容器运行时由集群选择，常见为 containerd 等 CRI 实现。常用命令：

```bash
# 构建与推送
docker build -t registry/app:v1.0.0 .
docker push registry/app:v1.0.0

# 运行容器（端口映射 / 数据卷 / 环境变量 / 资源限制）
docker run -d --name app \
  -p 8080:8080 \
  -v /data/app:/app/data \
  -e SPRING_PROFILES_ACTIVE=prod \
  --memory 2g --cpus 2 \
  registry/app:v1.0.0

# 查看与调试
docker ps                 # 运行中的容器
docker logs -f app        # 跟随日志
docker exec -it app bash  # 进入容器
docker compose up -d      # 多容器编排启动
```

**关键细节：**容器**无状态化**（数据卷外置，重启不丢）；使用受保护的版本标签或固定镜像 digest 保证版本可追溯；资源限制必须设（`--memory --cpus`）防容器吃光宿主机；健康检查（`HEALTHCHECK`）。

**风险与取舍：**容器内日志要输出到 stdout 由采集器收集（别写容器内文件）；镜像体积影响拉取和启动速度（多阶段构建 + 精简基础镜像）；共享内核意味着内核漏洞影响所有容器（隔离弱于虚拟机）。

**项目落地：**CI 流水线：代码 → 测试 → 构建镜像 → 推仓库 → K8s 滚动发布；本地开发用 compose 起依赖；环境一致性（开发/测试/生产同一镜像）。

**面试追问：**容器和虚拟机的区别（共享内核 vs 独立内核）；镜像分层原理（只读层 + 写时复制）；Namespace 和 Cgroups 各管什么（隔离 vs 限制）；容器里 PID 1 为什么特殊（init 职责）。

---

## 02 理解补充与边界校订

镜像 tag 通常可以被重新指向，并非天然不可变；发布固定 digest 可提升可追溯性。容器共享宿主内核，镜像也受 CPU 架构和内核能力影响，因此「到处运行」有兼容前提。现代 Kubernetes 通过 CRI 使用 containerd 等运行时，Docker 构建的 OCI 镜像可用，不等于集群必须运行 Docker Engine。

## 03 依据与延伸阅读

- [Docker 构建最佳实践](https://docs.docker.com/build/building/best-practices/)

## 04 相关问题

- [[八股/10-工程实践/06-容器与部署/03-Kubernetes 了解吗|Kubernetes 了解吗]]：容器打包与声明式编排

## 05 所属专题

- [[八股/10-工程实践/06-容器与部署/00-容器与部署导航|容器与部署导航]]
