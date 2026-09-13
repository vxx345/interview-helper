# 后端高频考点清单

题面命中 Java / 集合 / 并发 / JVM / MySQL / Redis / Kafka / Spring / 分布式 / 网络 时读取本文件。

每个考点列出**必答关键词**（漏了会被追问）和**一句话破题方向**。本文件不是完整答案，是**防止遗漏的检查表**——生成答案时用它做覆盖度自检。

---

## 1. Java 基础与集合

| 考点 | 必答关键词 |
|---|---|
| HashMap | 数组+链表+红黑树、**扩容 resize**、hash 扰动函数、负载因子 0.75、**并发下死循环**（JDK7 头插扩容，JDK8 已改尾插但仍非线程安全） |
| ConcurrentHashMap | JDK7 分段锁 Segment vs JDK8 **CAS + synchronized**（锁桶头节点）、size() 用 CounterCell 分段计数 |
| String | 不可变（final char[]/byte[]）、字符串常量池、JDK9 后从 char[] 改 byte[] 省空间 |
| equals / hashCode | 为什么必须一起重写：先用 hashCode 定位桶，再用 equals 比较，只重写 equals 会导致相同对象散落在不同桶 |
| ArrayList vs LinkedList | 随机访问 O(1) vs O(n)；实际工程几乎不用 LinkedList（内存局部性差、指针开销大） |

**加分点**：HashMap 的「为什么是 8 才转红黑树」——源码注释里给了泊松分布的计算，链表长度达到 8 的概率约千万分之六，是在「树化成本」和「查找效率」之间的工程折中。

---

## 2. 并发

| 考点 | 必答关键词 |
|---|---|
| volatile | **可见性**（MESI + 总线嗅探）、**禁止指令重排**（内存屏障）、**不保证原子性**（i++ 仍不安全） |
| synchronized |  monitorenter/monitorexit 字节码、对象头 Mark Word、**锁升级**（无锁→偏向→轻量→重量，JDK15 后移除偏向锁）、可重入 |
| AQS | **state + CLH 双向队列 + CAS**；独占/共享两种模式；ReentrantLock 的公平非公平差别只在「是否先 tryAcquire 插队」 |
| 线程池 | 7 个参数、**执行流程**（核心→队列→最大→拒绝）、拒绝策略 4 种、**为什么不用 Executors 创建**（无界队列 OOM） |
| ThreadLocal | 弱引用 key + **内存泄漏**（value 强引用链）、用完必须 remove，尤其在线程池场景 |
| CountDownLatch / CyclicBarrier / Semaphore | 一次性 vs 可复用、计数 vs 信号量 |

**加分点**：线程池参数怎么设——CPU 密集型设 `N+1`，IO 密集型设 `2N` 或按 `N × (1 + 等待时间/计算时间)`；但更工程的做法是**动态配置 + 监控队列积压**，因为拍脑袋算出来的数通常不准。

---

## 3. JVM

| 考点 | 必答关键词 |
|---|---|
| 内存区域 | 堆 / 元空间（JDK8 取代永久代）/ 虚拟机栈 / 本地方法栈 / 程序计数器 / 直接内存 |
| GC 算法 | 标记-清除（碎片）/ 标记-复制（浪费空间）/ 标记-整理（慢）；**分代收集**的依据是弱分代假说 |
| 垃圾回收器 | Serial / Parallel / CMS（已废弃）/ G1（**Region + 可预测停顿模型**）/ ZGC（**染色指针 + 读屏障**，停顿 <10ms） |
| 判断对象死亡 | 引用计数（循环引用问题）→ 可达性分析，GC Roots 有哪些 |
| OOM 排查 | 先 `-XX:+HeapDumpOnOutOfMemoryError` 拿 dump → MAT 分析 → 看是哪个类加载器/大对象 |
| CPU 飙高排查 | `top` 找进程 → `top -Hp` 找线程 → `jstack` 转 16 进制 tid 定位栈 |

**加分点**：不只是「用什么收集器」，而是「停顿时间和吞吐量的权衡」——G1 的价值在于可以设 `MaxGCPauseMillis`，把「不可控的停顿」变成「可控的成本」。

---

## 4. MySQL

| 考点 | 必答关键词 |
|---|---|
| 索引数据结构 | **B+ 树**：非叶子只存索引、叶子用链表相连 → **扇出大、树高 3 层就能存千万级**、范围查询天然友好。对比 B 树（数据分散在各层）、Hash（不支持范围）、红黑树（树太高，IO 次数多） |
| 聚簇 vs 二级索引 | 聚簇索引叶子存整行，二级索引叶子存主键 → **回表**；覆盖索引避免回表 |
| 最左前缀 | 联合索引 `(a,b,c)` 能走 `a`、`a,b`、`a,b,c`，跳列则失效；范围查询后的列无法继续走索引 |
| MVCC | **隐藏列（DB_TRX_ID / DB_ROLL_PTR）+ undo log 版本链 + ReadView**；快照读 vs 当前读 |
| 幻读 | 快照读靠 MVCC、**当前读靠 Next-Key Lock（行锁+间隙锁）**，两者配合才完整 |
| RC vs RR | RR 只在第一次快照读生成 ReadView；**很多公司主动降 RC**——没有间隙锁（外键除外），并发度更高、死锁更少 |
| 锁 | 行锁 / 表锁 / 意向锁 / 间隙锁；两阶段锁协议；死锁检测与超时 |
| 慢 SQL 排查 | 慢查询日志 → `explain` 看 type/key/rows/Extra → 关注 Using filesort、Using temporary |
| 事务 | ACID；redo log（崩溃恢复，WAL）、undo log（回滚 + MVCC）、binlog（主从复制）；**两阶段提交**保证 redo 与 binlog 一致 |

**加分点**：索引失效的典型场景——函数/隐式类型转换/前置通配符 `%xx`/or 连接非索引列/违反最左前缀。以及「明明有索引却全表扫描」——优化器判断回表代价高于全表扫描时会放弃索引。

---

## 5. Redis

| 考点 | 必答关键词 |
|---|---|
| 数据结构 | String / List / Hash / Set / **ZSet（跳表+dict）**；底层：SDS、双向链表、压缩列表 ziplist→listpack、哈希表、跳表、intset |
| 为什么快 | 纯内存 + **单线程避免锁竞争和上下文切换** + **IO 多路复用**（epoll）；注意 Redis 6.0 的多线程只处理网络 IO，命令执行仍单线程 |
| 缓存三大问题 | 穿透（布隆过滤器 / 缓存空值）、击穿（互斥锁 / 逻辑过期）、雪崩（随机 TTL / 多级缓存 / 熔断降级） |
| 过期与淘汰 | 惰性删除 + 定期删除；8 种淘汰策略，重点 **LRU vs LFU**（Redis 用近似 LRU，随机采样 N 个 key 淘汰最久未用的） |
| 持久化 | RDB（fork 子进程、快照、恢复快但丢数据多）vs AOF（追加日志、可配置刷盘策略、支持重写）；**生产通常混合持久化** |
| 分布式锁 | `set nx px` + **唯一 value（防误删别的线程的锁）** + **Lua 脚本保证释放原子性**；Redlock 争议（时钟跳跃问题）；续期用看门狗 |
| 一致性 | Cache Aside 模式（读：命中返回、未命中读库回写；写：**先更新数据库再删除缓存**）；延迟双删、MQ 重试删除兜底 |

**加分点**：为什么是「删缓存」而不是「更新缓存」——两个并发写操作时，更新缓存的顺序无法保证，会导致脏数据；删除则最多是一次 cache miss，代价小得多。

---

## 6. Kafka / MQ

| 考点 | 必答关键词 |
|---|---|
| 为什么快 | **顺序磁盘 IO**（比随机内存访问还快）+ **零拷贝 sendfile** + 批量压缩 + 分区并行 |
| 消息不丢 | 生产端 `acks=all` + 重试；Broker 端 `replication.factor≥3` + `min.insync.replicas≥2`；消费端**手动提交 offset**（处理完再提交） |
| 重复消费 | 幂等（唯一键/去重表）或事务；**Kafka 只能保证 at-least-once，exactly-once 要靠幂等生产者 + 事务** |
| 消息积压 | 分层治理：**先把 consumer 数打到分区数上限 → 再优化单实例吞吐 → 最后才考虑加分区**。consumer 数 > 分区数纯属浪费 |
| 顺序性 | 分区内有序，全局有序只能单分区（牺牲吞吐）；业务上常用「同一业务键路由到同一分区」保证局部有序 |
| 延迟队列 | Kafka 不适合做延迟队列（消费者无法按时间跳过）；用 **RocketMQ 延迟消息** 或 **Redis ZSet 轮询** |

**加分点**：加分区不是银弹——分区数过多会导致文件句柄暴涨、副本同步开销上升，**反而可能更慢**。另外要建可观测闭环：Prometheus + exporter 监控 consumer lag，lag 持续增长即告警，避免「悄悄积压」。

---

## 7. Spring

| 考点 | 必答关键词 |
|---|---|
| IoC | 控制反转 / DI；Bean 生命周期：实例化 → 属性填充 → 初始化（Aware / BeanPostProcessor / InitializingBean）→ 使用 → 销毁；**三级缓存解决循环依赖**（singletonFactories 提前暴露工厂对象） |
| AOP | JDK 动态代理（实现接口）vs CGLIB（子类继承）；核心是 BeanPostProcessor 在初始化后生成代理对象 |
| 事务 | 声明式事务基于 AOP；**失效场景**：自调用（没走代理）、非 public 方法、异常被 catch 吞掉、rollbackFor 配置不当 |
| 循环依赖 | 三级缓存只能解决**单例 + 属性注入**；构造器注入和 prototype 无法解决 |

---

## 8. 分布式

| 考点 | 必答关键词 |
|---|---|
| CAP | 分区容错 P 必须选，实际是在 **C 和 A 之间**权衡；CP（ZooKeeper / etcd）vs AP（Eureka） |
| BASE | 基本可用、软状态、最终一致 |
| 分布式事务 | 2PC（同步阻塞、协调者单点）/ 3PC / **TCC**（Try-Confirm-Cancel，业务侵入大）/ **本地消息表 + MQ**（最终一致，最常用）/ Seata AT |
| 分布式 ID | UUID（无序、索引碎片）/ 数据库号段（Leaf-segment）/ **雪花算法**（时间戳+机器位+序列位，趋势递增，注意**时钟回拨**问题） |
| 一致性哈希 | 虚拟节点解决数据倾斜；扩缩容只迁移 1/N 数据 |
| 限流 | 固定窗口（临界突刺）/ 滑动窗口 / **令牌桶**（允许突发）/ 漏桶（恒定速率）；单机 Guava RateLimiter vs 分布式 Redis + Lua |
| 熔断降级 | 三态机：Closed → Open → Half-Open；Sentinel / Resilience4j |
| 接口幂等 | 唯一索引 / Token 机制 / 状态机 / 分布式锁 |

**加分点**：分布式锁的坑——「锁过期但业务没执行完」（用看门狗续期）、「主从切换导致锁丢失」（Redis 主从异步复制，极端场景用 Redlock 或改用 etcd/ZooKeeper）。
