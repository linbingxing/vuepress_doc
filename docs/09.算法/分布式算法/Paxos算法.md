#  Paxos算法

## 1 什么是Paxos

Paxos算法是基于**消息传递**且具有**高度容错特性**的**一致性算法**，是目前公认的解决**分布式一致性**问题**最有效**的算法之一。

Paxos由 莱斯利·兰伯特(Leslie Lamport)于1998年在《The Part-Time Parliament》论文中首次公开，

最初的描述使用希腊的一个小岛Paxos，描述了Paxos小岛中通过决议的流程，并以此命名这个算法，

但是这个描述理解起来比较有挑战性。后来在2001年，莱斯利·兰伯特重新发表了朴实的算法描述版本《Paxos Made Simple》 。

Leslie Lamport提出的 Paxos 算法包含 2 个部分 ：

- Basic Paxos 算法，描述的是多节点之间如何就某个值达成共识；
- Multi-Paxos 思想，描述的是执行多个 Basic Paxos 实例，就一系列值达成共识。

> Basic Paxos 是 Multi-Paxos 思想的核心，说白了，Multi-Paxos 就是多执行几次 Basic Paxos。

##  2 Paxos 解决了什么问题

![Paxos-1](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-1.png)

在常见的分布式系统中，总会发生诸如**机器宕机**或**网络异常**（包括消息的延迟、丢失、重复、乱

序，还有网络分区）等情况。Paxos算法需要解决的问题就是如何在一个可能发生上述异常的分布式系

统中，快速且正确地在集群内部对**某个数据的值**达成**一致**，并且保证不论发生以上任何异常，都不会破

坏整个系统的一致性。

> 注：这里**某个数据的值**并不只是狭义上的某个数，它可以是一条日志，也可以是一条命令
>
> （command）。根据应用场景不同，**某个数据的值**有不同的含义。

在2PC 和 3PC的时候在一定程度上是可以解决数据一致性问题的. 但是并没有完全解决就是协调者宕机的情况。

![Paxos-2](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-2.png)

**如何解决2PC和3PC的存在的问题呢?** 

1. 引人多个协调者 ![Paxos-3](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-3.png)
2. 引人主协调者，其他以主协调者的命令为主

![Paxos-4](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-4.png)

其实在引入多个协调者之后又引入主协调者，那么这个就是最简单的一种Paxos算法。

Paxos的版本有: **Basic Paxos , Multi Paxos, Fast-Paxos**, 具体落地有Raft 和zk的ZAB协议。

![Paxos-5](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-5.png)

## 3 Basic Paxos相关概念

角色介绍

Basic Paxos算法在Leslie Lamport论文中的内容是比较精简的，核心是三种基本角色：Proposer、Acceptor及Learner：

![Paxos-6](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-6.png)

