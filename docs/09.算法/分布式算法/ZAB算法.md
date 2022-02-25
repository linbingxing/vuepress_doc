#  ZAB协议详解

## 1 什么是Zab协议？

Zab协议 的全称是 Zookeeper Atomic Broadcast （Zookeeper原子广播）。

Zookeeper 是通过 Zab 协议来保证分布式事务的最终一致性。

Zab协议是为分布式协调服务Zookeeper专门设计的一种支持崩溃恢复的原子广播协议，是Zookeeper保证数据一致性的核心算法。

Zab借鉴了Paxos算法，但又不像Paxos那样，是一种通用的分布式一致性算法。

它是特别为Zookeeper设计的支持崩溃恢复的原子广播协议。

在Zookeeper中主要依赖Zab协议来实现数据一致性，基于该协议，zk实现了一种主备模型（即Leader和Follower模型）的系统架构来保证集群中各个副本之间数据的一致性。 这里的主备系统架构模型，就是指只有一台客户端（Leader）负责处理外部的写事务请求，然后Leader客户端将数据同步到其他Follower节点。

Zookeeper 客户端会随机的链接到 zookeeper 集群中的一个节点，如果是读请求，就直接从当前节点中读取数据；

如果是写请求，那么节点就会向 Leader 提交事务，Leader 接收到事务提交，会广播该事务，只要超过半数节点写入成功，该事务就会被提交。





[ZAB-一致性算法](https://houbb.github.io/2018/10/30/zab)

