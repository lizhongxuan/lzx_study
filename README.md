# 🚀 性能优化案例目录

为便于检索与维护，案例已按主题拆分到以下文件：

- 编程语言：`./programming_language.md`
- 云原生：`./cloud_native.md`
- 存储：`./storage.md`
- 数据库：`./database.md`
- 网络：`./network.md`
- 分布式架构与微服务：`./distributed_microservices.md`
- 系统设计：`./system_design.md`
- 架构设计：`./architecture_design.md`
- 算法：`./algorithm.md`
- Linux 内核：`./linux_kernel.md`
- 开源项目
  - Volcano：`./vocalno.md`

## 📂 分类目录

### 编程语言
- **[案例一：高频对象的池化与 Slab 分配](./programming_language.md#案例一高频对象的池化与slab分配)** - Object Pool, Slab, RAII

### 云原生
- **[案例一：采集资源调优 (Resource Optimization)](./cloud_native.md#案例一采集资源调优-resource-optimization)** - Staggered Start, Object Pool, GOMEMLIMIT

### 存储
- **[案例一：去重分片架构优化 (No-Reroute Strategy)](./storage.md#案例一去重分片架构优化-no-reroute-strategy)** - No-Reroute, Backpressure, Failover

### 数据库
- **[案例一：计算下推优化 (Computation Pushdown)](./database.md#案例一计算下推优化-computation-pushdown)** - Pushdown, Aggregation, Distributed
- **[案例二：采集端预聚合优化 (Pre-aggregation)](./database.md#案例二采集端预聚合优化-pre-aggregation)** - Pre-aggregation, Query Rewrite

### 网络
- **[案例一：基于 RSS Hash 的无锁流聚合架构](./network.md#案例一基于-rss-hash-的无锁流聚合架构)** - RSS, Share-Nothing, Lock-Free

### 分布式架构与微服务
- **[案例一：高并发场景下 Redis 分布式锁的安全性优化](./distributed_microservices.md#案例一高并发场景下-redis-分布式锁的安全性优化)** - Redis Lock, Watchdog, Lua

### 系统设计
- **[案例一：任务调度系统的强一致性锁迁移 (Redis -> Etcd)](./system_design.md#案例一任务调度系统的强一致性锁迁移-redis---etcd)** - Etcd Lock, Raft, Watch

### 架构设计
- **[案例一：基于多级缓存架构消除热点 Key 瓶颈](./architecture_design.md#案例一基于多级缓存架构消除热点-key-瓶颈)** - Multi-level Cache, Hot Key, BigCache

### 算法
- **[算法案例集合](./algorithm.md)** - 暂无案例

### Linux 内核
- **[案例一：基于 AF_PACKET V3 的零拷贝抓包优化](./linux_kernel.md#案例一基于-af_packet-v3-的零拷贝抓包优化)** - Zero-Copy, mmap
- **[案例二：基于 RCU 思想的配置无损热更新](./linux_kernel.md#案例二基于-rcu-思想的配置无损热更新)** - RCU, ArcSwap, Atomic Swap

### Volcano
- **[案例一：xxx](./volcano.md#案例一xxx)** - xxx