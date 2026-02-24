# Week 2: Hadoop HA 高可用集群实战

![Status](https://img.shields.io/badge/Status-Completed-success)
![Hadoop](https://img.shields.io/badge/Hadoop-3.2.1-blue)
![ZooKeeper](https://img.shields.io/badge/ZooKeeper-3.6-orange)
![Docker](https://img.shields.io/badge/Docker-Compose-blue)

## 1. 项目概述
本周目标是构建一个**生产级别的最小化 Hadoop 高可用 (HA) 集群**。通过引入 ZooKeeper 和 JournalNode，消除 NameNode 和 ResourceManager 的单点故障 (SPOF)。

**核心成果：**
- ✅ 搭建 2 名 NameNode (Active/Standby) 实现 HDFS HA。
- ✅ 搭建 2 名 ResourceManager 实现 YARN HA。
- ✅ 部署 3 节点 ZooKeeper 集群用于选举与状态管理。
- ✅ 部署 3 节点 JournalNode 集群用于 EditsLog 共享。
- ✅ 验证自动故障转移 (Failover) 机制。

## 2. 集群架构拓扑

```text
+---------------------+       +---------------------+
|   ZooKeeper Cluster |       |   JournalNode Cluster|
|  (zk1, zk2, zk3)    |       |   (jn1, jn2, jn3)   |
|  Port: 2181         |       |   Port: 8485        |
+----------+----------+       +----------+----------+
           |                             |
           +-------------+---------------+
                         |
           +-------------+---------------+
           |                             |
+----------v----------+       +----------v----------+
|    NameNode 1       |       |    NameNode 2       |
|    (Active)         |       |    (Standby)        |
|    Port: 9870       |       |    Port: 9870       |
+----------+----------+       +----------+----------+
           |                             |
+----------v----------+       +----------v----------+
|    DataNode 1       |       |    DataNode 2       |
+---------------------+       +---------------------+

           +---------------------+
           |   ResourceManager   |
           |   (rm1: 8088)       |
           |   (rm2: 8088)       |
           +---------------------+
```

## 3. 目录结构

```text
hadoop-week2/
├── docker-compose.yml       # 容器编排文件
├── .env                     # 环境变量
├── config/                  # Hadoop 配置文件挂载目录
│   ├── core-site.xml
│   ├── hdfs-site.xml
│   └── yarn-site.xml
└── data/                    # 数据持久化目录
    ├── zk/                  # ZK 数据 (可选，本教程使用容器内)
    ├── jn1, jn2, jn3        # JournalNode 数据
    ├── nn1, nn2             # NameNode 数据
    └── dn1, dn2             # DataNode 数据
```

## 4. 快速启动指南

### 4.1 前置准备
确保已安装 Docker 和 Docker Compose，并清理了 Week 1 的残留容器。
```bash
# 进入项目目录
cd hadoop-week2
```

### 4.2 启动基础设施 (ZK + JN)
```bash
docker compose up -d zk1 zk2 zk3 jn1 jn2 jn3
```
*等待 30 秒确保 ZK 集群选举完成，JN 启动成功。*

### 4.3 初始化 HA 环境
**步骤 1: 格式化 ZooKeeper (创建 HA 锁节点)**
```bash
docker exec -it nn1 bash
hdfs zkfc -formatZK
exit
```

**步骤 2: 格式化 NameNode (仅主节点)**
```bash
docker exec -it nn1 bash
hdfs namenode -format -force
hdfs --daemon start namenode
hdfs --daemon start zkfc
exit
```

**步骤 3: 引导 Standby NameNode**
```bash
docker exec -it nn2 bash
hdfs namenode -bootstrapStandby
hdfs --daemon start namenode
hdfs --daemon start zkfc
exit
```

### 4.4 启动剩余组件
```bash
docker compose up -d dn1 dn2 rm1 rm2 nm1 nm2
```

## 5. 验证与测试

### 5.1 检查 HA 状态
```bash
# 查看 NN1 状态 (应为 active 或 standby)
docker exec -it nn1 hdfs haadmin -getServiceState nn1

# 查看 NN2 状态 (应与 NN1 相反)
docker exec -it nn2 hdfs haadmin -getServiceState nn2
```

### 5.2 Web UI 访问
| 组件 | 节点 | 地址 | 说明 |
| :--- | :--- | :--- | :--- |
| **HDFS NN1** | nn1 | http://localhost:9870 | 查看 Active/Standby 状态 |
| **HDFS NN2** | nn2 | http://localhost:9871 | 端口映射为 9871 |
| **YARN RM** | rm1 | http://localhost:8088 | 资源调度界面 |
| **ZooKeeper** | zk1 | localhost:2181 | 客户端连接端口 |

### 5.3 故障转移测试 (Failover)
1.  确认当前 Active 节点 (假设是 `nn1`)。
2.  模拟宕机：`docker stop nn1`。
3.  等待 10-20 秒。
4.  检查 `nn2` 状态：`docker exec -it nn2 hdfs haadmin -getServiceState nn2` (应变为 `active`)。
5.  恢复 `nn1`：`docker start nn1`，观察其是否自动变为 `standby`。

## 6. 深度思考：ZooKeeper 在 Hadoop HA 中的作用

基于你已有的 ZK 知识，请关注以下实现细节：

1.  **临时节点 (Ephemeral Nodes):**
    *   ZKFC 在 ZK 中创建 `/hadoop-ha/mycluster/ActiveStandbyElectorLock` 临时节点。
    *   **原理:** 只有持有锁的 NN 才是 Active。当 NN 宕机，会话断开，临时节点消失，触发 Watcher 通知其他 ZKFC 参与选举。
2.  **健康检测:**
    *   ZKFC 不仅监控 ZK 锁，还通过 RPC 定期心跳检测本地 NameNode 健康状态。
    *   如果本地 NN 挂掉，ZKFC 会主动删除 ZK 锁，触发切换。
3.  **JournalNode (QJM):**
    *   ZK 不负责存储大量 EditsLog，只负责选举。
    *   NN 将 EditsLog 写入多数派 JN (3 个中写成功 2 个)，保证元数据一致性。

## 7. 常见问题排查 (Troubleshooting)

| 问题 | 可能原因 | 解决方案 |
| :--- | :--- | :--- |
| **NN 启动失败** | ClusterID 不一致 | 确保 nn2 执行了 `-bootstrapStandby` 而非 `-format` |
| **ZKFC 无法启动** | 未初始化 ZK | 执行 `hdfs zkfc -formatZK` |
| **Web UI 无法访问** | 端口冲突 | 检查 `docker-compose.yml` 端口映射，修改 host 端口 |
| **脑裂 (Split-Brain)** | Fencing 配置缺失 | 生产环境需配置 `sshfence`，本教程使用 `shell(/bin/true)` 简化 |
| **DN 无法注册** | 网络不通 | 确保所有容器在 `hadoop-net` 网络中，检查 `core-site.xml` |

## 8. 清理环境
**警告：这将删除所有数据！**
```bash
docker compose down -v
rm -rf data/*
```

## 9. 下周计划 (Week 3)
- [ ] 深入 HDFS 读写流程源码分析
- [ ] MapReduce Shuffle 机制详解
- [ ] 使用 Wireshark 或日志分析数据包流向

---
*Author: Hadoop Learner*
*Date: 2024*