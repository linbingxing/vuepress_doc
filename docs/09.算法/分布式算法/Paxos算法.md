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

### 3.1 角色介绍

Basic Paxos算法在Leslie Lamport论文中的内容是比较精简的，核心是三种基本角色：Proposer、Acceptor及Learner：

![Paxos-6](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-6.png)

- **Proposer（提议者）**
  	Proposer， 提出提案[n, v]，用于投票表决。一般来说，集群中收到客户端请求的节点，就是提议者。Proposer可以有多个， 代表的是接入和协调功能，收到客户端请求后，发起二阶段提交（Basic Paxos算法本质是一种二阶段提交算法，后面我们就会明白），进行共识协商。
- **Acceptor（接受者）**
  	Acceptor， 对每个提案进行投票，并存储接受的值。 一般来说，集群中的所有节点都在扮演接受者的角色，参与共识协商，并接受和存储数据。 Acceptor 之间完全对等独立，对于Proposer 提出的提案，必须获得超过半数以上(N/2+1)的 Acceptor批准后才能通过。

- **Learner（学习者）**
  	Learner， 被告知投票的结果，接受达成共识的值，存储保存，不参与投票的过程。一般来说，学习者是数据备份节点，比如“Master-Slave”模型中的 Slave，被动地接受数据，容灾备份。

### 3.2 决策模型

![Paxos-7](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-7.png)

## 4 Basic Paxos流程

Basic Paxos流程一共分为4个步骤:

- Prepare

  Proposer提出一个提案,编号为N, 此N大于这个Proposer之前提出所有提出的编号, 请求Accpetor的多数人接受这个提案。

- Promise

  如果编号N大于此Accpetor之前接收的任提案编号则接收, 否则拒绝。

- Accept

  如果达到多数派, Proposer会发出accept请求, 此请求包含提案编号和对应的内容。

- Accepted

  如果此Accpetor在此期间没有接受到任何大于N的提案,则接收此提案内容, 否则忽略。

###  4.1 Basic Paxos流程图

1. 无故障的Basic Paxos

​         ![Paxos-8](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-8.png)

2. Acceptor失败时的Basic Paxos

   在下图中，多数派中的一个Acceptor发生故障，因此多数派大小变为2。在这种情况下，Basic Paxos协议仍然成功。

   ![Paxos-9](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-9.png)

2. Proposer失败时的Basic Paxos

   Proposer在提出提案之后但在达成协议之前失败。具体来说，传递到Acceptor的时候失败了,这个时候需要选出新的Proposer（提案人）,那么 Basic Paxos协议仍然成功。

   ![Paxos-10](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-10.png)

3. 当多个提议者发生冲突时的basic Paxos，最复杂的情况是多个Proposer都进行提案,导致Paxos的活锁问题。

   ![Paxos-11](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-11.png)

**针对活锁问题解决起来非常简单 只需要在每个Proposer再去提案的时候随机加上一个等待时间即可。**

## 5 Basic Paxos算法示例

假设集群中有A、B、C三个节点，它们需要对客户端写入的某个值X达成共识。

![Paxos-12](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-12.jpg)

假设客户端 1 的提案编号为 1，节点 A、B 先收到来自客户端 1 的Prepare请求；客户端 2 的提案编号为 5，节点 C 先收到来自客户端 2 的Prepare请求。

### 5.1 **阶段一（准备阶段）**

首先来看第一个阶段，客户端 1、2 作为Proposer，分别向所有Acceptor发送包含提案编号的Prepare请求：

![Paxos-13](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-13.jpg)

> 这里要注意的是，在Prepare请求中，只需要携带提案编号就可以了，不需要提案值。

按照上图的时间线顺序，站在节点的角度看，节点 A、B 先收到了提案编号为 1 的Prepare请求，节点 C 先收到了提案编号为 5 的Prepare请求，它们会做如下处理：

- 由于节点 A、B 之前没有通过任何提案，所以将返回一个 “尚无提案”的响应。也就是说节点 A 和 B 在告诉Proposer，我之前没有通过任何提案，并承诺以后不再响应提案编号小于等于 1 的Prepare请求，不会通过编号小于 1 的提案。
- 节点 C 也是如此，它将返回一个 “尚无提案”的响应，并承诺以后不再响应提案编号小于等于 5 的Prepare请求，不会通过编号小于 5 的提案。

![Paxos-14](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-14.jpg)

然后过了一段时间， 节点 A、B 收到了提案编号为 5 的Prepare请求，节点 C 收到了提案编号为 1 的Prepare请求，将执行下面的处理过程：

- 节点 A、B 收到提案编号为 5 的Prepare请求时，因为提案编号 5 大于它们之前响应的Prepare请求的提案编号 1，而且两个节点都没有通过任何提案，所以它将返回一个 “尚无提案”的响应，并承诺以后不再响应提案编号小于等于 5 的Prepare请求，不会通过编号小于 5 的提案。
- 当节点 C 收到提案编号为 1 的Prepare请求时，由于提案编号 1 小于它之前响应的Prepare请求的提案编号 5，所以丢弃该Prepare请求，不做响应。

![Paxos-15](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-15.jpg)

### 5.2 **阶段二（批准阶段）**

第二个阶段，客户端 1、2 在收到大多数节点的Prepare响应之后，会分别发送Accept请求：

![Paxos-16](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-16.jpg)

- 当客户端 1 收到大多数的接受者（节点 A、B）的Prepare响应后，根据响应中提案编号最大的提案的值，设置Accept请求中的值。因为该值在来自节点 A、B 的Prepare响应中都为空（也就是“尚无提案”），所以就把自己的提议值 3 作为提案的值，发送Accept请求[1, 3]。
- 当客户端 2 收到大多数的接受者的Prepare响应后（节点 A、B 和节点 C），根据响应中提案编号最大的提案的值，来设置接受请求中的值。因为该值在来自节点 A、B、C 的准备响应中都为空（“尚无提案”），所以就把自己的提议值 7 作为提案的值，发送Accept请求[5, 7]。

然后，当三个节点收到 2 个客户端的Accept请求时，会进行这样的处理：

![Paxos-17](https://gitee.com/linbingxing/image/raw/master/distributed/Paxos-17.jpg)

- 当节点 A、B、C 收到接受请求[1, 3]的时候，由于提案的提案编号 1 小于三个节点承诺能通过的提案的最小提案编号 5，所以提案[1, 3]将被拒绝。
- 当节点 A、B、C 收到接受请求[5, 7]的时候，由于提案的提案编号 5 不小于三个节点承诺能通过的提案的最小提案编号 5，所以就通过提案[5, 7]，也就是接受了值 7，三个节点就 X 值为 7 达成了共识。

通过上面的示例演示可以看到， Basic Paxos 是通过二阶段提交的方式来达成共识的。 除了共识，Basic Paxos 还实现了容错，在少于一半的节点出现故障时，集群也能工作。

本质上，提案编号的大小代表着优先级，你可以这么理解，根据提案编号的大小，接受者保证三个承诺：

1. 如果Prepare请求的提案编号，小于等于接受者已经响应的Prepare请求的提案编号，那么接受者将承诺不响应这个Prepare请求；
2. 如果Accept请求中的提案的提案编号，小于接受者已经响应的Prepare请求的提案编号，那么接受者将承诺不通过这个提案；
3. 如果接受者之前有通过提案，那么接受者将承诺，会在Prepare请求的响应中，包含已经通过的最大编号的提案信息。

## 6 **Basic Paxos的问题**

1. 首先，如果多个Proposer同时提交提案，可能出现提案冲突，导致在Prepare阶段没有Proposer者接收到大多数Prepare响应，协商失败，需要重新协商，而不断的重新协商最终导致了“**活性问题**”。

   想象一下，一个 5 节点的集群，如果 3 个节点作为Proposer同时提案，就可能发生因为没有Proposer接收大多数响应（比如 1 个Proposer接收到 1 个Prepare响应，另外 2 个Proposer分别接收到 2 个Prepare响应）而准备失败，需要重新协商。

2. 此外，Basic Paxos 是通过二阶段提交来达成共识的，2 轮 RPC 通讯本身就十分消耗性能，存在可能的延时问题。如果因为活性问题再多次往返通讯，对整个系统性能的影响可想而知。

## 7 Multi-Paxos思想

Basic Paxos 只能就单个值（Value）达成共识，一旦遇到为一系列的值达成共识的时候，它就不管用了。Leslie Lamport提到的 Multi-Paxos 是一种思想，基于该思想，可以通过多个 Basic Paxos 实例实现一系列值的共识算法（比如 Chubby 的 Multi-Paxos 实现、Raft 算法等）。

针对活性问题，一种解决方案就是引入领导者节点，领导者节点作为唯一Proposer，这样就不存在多个Proposer同时提交提案的情况，也就不存在因提案冲突而引发的活性问题了。

针对二阶段提交的性能问题，我们可以采用“当领导者处于稳定状态时，省掉Prepare阶段，直接进入Accept阶段”这个优化机制。

![Multi_Paxos-1](https://gitee.com/linbingxing/image/raw/master/distributed/Multi_Paxos-1.jpg)

通过上图，可以看到，Multi-Paxos 引入领导者节点之后，因为只有领导者节点一个Proposer，只有它说了算，所以就不存在提案冲突。另外，当领导者节点处于稳定状态时，就省掉了Prepare阶段，直接进入Accept阶段，所以在很大程度上减少了网络通信的往返开销，提升了性能，降低了延迟。

## **8 Multi-Paxos流程图**

1. 第一次流程-确定Leader

![Multi_Paxos-2](https://gitee.com/linbingxing/image/raw/master/distributed/Multi_Paxos-2.png)

2.  第二次流程-直接由Leader确认

![Multi_Paxos-3](https://gitee.com/linbingxing/image/raw/master/distributed/Multi_Paxos-3.png)

## **9 Multi-Paxos角色重叠流程图**

Multi-Paxos在实施的时候会将Proposer，Acceptor和Learner的角色合并统称为“服务器”。因此，最后只有“客户端”和“服务器”。

![](https://gitee.com/linbingxing/image/raw/master/distributed/Multi_Paxos-4.png)

## 10 Chuppy的Multi-Paxos实现

在 Chubby 中，主节点（领导者节点）是通过执行 Basic Paxos 算法，进行投票选举产生的，并且在运行过程中，主节点会通过不断续租的方式来延长租期（Lease）。比如在实际场景中，几天内都是同一个节点作为主节点。如果主节点故障了，那么其他的节点又会投票选举出新的主节点，也就是说主节点是一直存在的，而且是唯一的。

Chubby 实现了Leslie Lamport提到的 “当领导者处于稳定状态时，省掉了准备阶段，直接进入接受阶段” 这个优化机制，提升了数据的提交效率，但是所有写请求都在主节点处理，限制了集群处理写请求的并发能力，约等于单机。

Chubby 为了实现了强一致性，读操作也只能在主节点上执行。 也就是说，只要数据写入成功，之后所有的客户端读到的数据都是一致的。具体的过程如下：

1.所有的读请求和写请求都由主节点来处理。当主节点从客户端接收到写请求后，作为Proposer，执行 Basic Paxos 实例，将数据发送给所有的节点，并且在大多数节点接受了这个写请求之后，再响应给客户端成功；

![Multi_Paxos-5](https://gitee.com/linbingxing/image/raw/master/distributed/Multi_Paxos-5.jpg)

2.当主节点接收到读请求后，主节点只需要查询本地数据，然后返回给客户端就可以了：

![Multi_Paxos-6](https://gitee.com/linbingxing/image/raw/master/distributed/Multi_Paxos-6.jpg)

