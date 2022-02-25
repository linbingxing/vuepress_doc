#  Raft算法

> Raft算法讲解最生动的教程：[Raft: Understandable Distributed Consensus](http://thesecretlivesofdata.com/raft/)，也便于理解接下来要讲的内容。

## 1 什么是Raft算法

Paxos 是论证了一致性协议的可行性，但是论证的过程据说晦涩难懂，缺少必要的实现细节，而且工程实现难度比较高, 广为人知实现只有 zk 的实现 zab 协议。Paxos协议的出现为分布式强一致性提供了很好的理论基础，但是Paxos协议理解起来较为困难，实现比较复杂。然后斯坦福大学RamCloud项目中提出了易实现，易理解的分布式一致性复制协议 Raft。Java，C++，Go 等都有其对应的实现之后出现的Raft相对要简洁很多。

**引入主节点，通过竞选确定主节点。节点类型：Follower(从节点）、Candidate(候选节点)和Leader(主节点)**

 Leader 会周期性的发送心跳包给 Follower。每个 Follower 都设置了一个随机的竞选超时时间，一般为 150ms~300ms，如果在这个时间内没有收到 Leader 的心跳包，就会变成 Candidate，进入竞选阶段, 通过竞选阶段的投票多的人成为Leader。

![Raft-1](https://gitee.com/linbingxing/image/raw/master/distributed/Raft-1.png)

## 2 Raft的角色

- **Follower（跟随者）**

  Raft协议刚开始时，所有节点都是Follower，默默地接收和处理来自Leader的消息，当等待Leader心跳信息超时的时候，就主动站出来，推荐自己当Candidate。

- **Candidate（候选人）**

  Candidate将向其他节点发送投票请求（Request Vote），通知其他节点来投票，如果赢得了大多数（N/2+1）选票，就晋升Leader。

- **Leader（领导者）**

  Leader主要负责处理客户端请求，进行日志复制等操作，每一轮选举的目标就是选出一个Leader；Leader会不断地发送心跳信息，通知其他节点“我是领导者，我还活着，你们现在不要发起新的选举，找个新领导者来替代我。”

## 3 Raft选举流程

1.首先，在初始状态下，集群中所有的节点都是Follower状态，并被设定一个随机 election timeout（150ms-300ms）：

<img src="https://gitee.com/linbingxing/image/raw/master/distributed/Raft-2.png" alt="Raft-2" style="zoom:50%;" />

2.如果超时时间到期后没有收到来自Leader的心跳，节点就发起选举：将自己的状态切换为 Candidate，增加自己的任期编号，然后向集群中的其它 Follower 节点发送请求，询问其是否选举自己成为 Leader：<img src="https://gitee.com/linbingxing/image/raw/master/distributed/Raft-3.png" alt="Raft-3" style="zoom:50%;" />

3.其他节点接收到候选人 A 的请求投票消息后，如果在编号为 1 的这届任期内还没有进行过投票，那么它将把选票投给节点 A，并增加自己的任期编号：

<img src="Raft%E7%AE%97%E6%B3%95.assets/Raft-4.png" alt="Raft-4" style="zoom:50%;" />

4.当收到来自集群中过半数节点的接受投票后，节点即成为本届任期内 Leader，他将周期性地发送心跳消息，通知其他节点我是Leader，阻止Follower发起新的选举：<img src="Raft%E7%AE%97%E6%B3%95.assets/Raft-5.png" alt="Raft-5" style="zoom:50%;" />

> 如果在指定时间内（一个随机时间，每个节点都不同），Follower没有接收到来自Leader的心跳消息，那么它就认为当前没有Leader，推举自己为Candidate，发起新一轮选举。

## 4 Raft通信方式

在 Raft 算法中，节点之间的通信采用的是RPC方式，在Leader选举中，需要用到两类RPC：

1. 请求投票（RequestVote）RPC：由Candidate在选举期间发起，通知各Follower进行投票；
2. 日志复制（AppendEntries）RPC：由Leader发起，用来进行日志复制和心跳。

## 5 Raft任期

在选举流程中，我提到Leader节点是有任期的，每个任期由单调递增的数字标识，比如节点 A 的任期编号是 1。任期编号是随着选举的举行而变化的：

1. Follower在等待Leader的心跳信息超时后，会推举自己为Candidate，此时会增加自己的任期编号；
2. 如果一个节点发现自己的任期编号比其他节点小，那么它会更新自己的编号为较大的编号值。比如节点 B 的任期编号是 0，当收到来自节点 A 的请求投票消息时，因为消息中包含了节点 A 的任期编号，且编号为 1，那么节点 B 将把自己的任期编号更新为 1。
3. 当任期编号相同时，日志完整性高的Follower（也就是最后一条日志项对应的任期编号值更大，索引号更大），拒绝投票给日志完整性低的Candidate。

## 6 Raft日志同步

在 Raft 算法中，副本数据是以日志的形式存在的，Leader接收到来自客户端写请求后，处理写请求的过程就是一个日志复制的过程。Leader把请求作为日志条目（Log entries）加入到它的日志中，然后并行的向其他服务器发起 AppendEntries RPC 复制日志条目。当这条日志被复制到大多数服务器上，Leader将这条日志应用到它的状态机并向客户端返回执行结果。

Raft日志同步保证如下两点：

- 如果不同日志中的两个条目有着相同的索引和任期号，则它们所存储的命令是相同的。
- 如果不同日志中的两个条目有着相同的索引和任期号，则它们之前的所有条目都是完全一样的。

![Raft-7](https://gitee.com/linbingxing/image/raw/master/distributed/Raft-7.jpg)

**日志项**

日志是由日志项组成的，志项是一种数据格式，它主要包含客户端的指令（Command），还有索引值（Log index）、任期编号（Term）等信息：

<img src="https://gitee.com/linbingxing/image/raw/master/distributed/Raft-6.png" alt="Raft-6" style="zoom:50%;" />

**复制流程**

1. 首先，当Leader节点接收到客户端的写请求后，会创建一个新日志项，并附加到本地日志中；
2. Leader通过日志复制（AppendEntries）RPC 消息，将日志项复制到集群其它Follower节点上；
3. 如果Leader接收到大多数的“复制成功”响应后，它将日志项应用到自己的状态机，并返回成功给客户端。如果Leader没有接收到大多数的“复制成功”响应，那么就返回错误给客户端；
4. 当Follower接收到心跳信息，或者新的AppendEntries消息后，如果发现Leader已经提交了某条日志项，而自己还没应用，那么Follower就会将这条日志项应用到本地的状态机中。

![Raft-8](https://gitee.com/linbingxing/image/raw/master/distributed/Raft-8.png)

从上面这个过程可以看出，当Follower节点接受Leader的心跳消息或者AppendEntries消息后，会将日志项应用到自己的状态机。这个优化，降低了处理客户端请求的延迟，将二阶段提交优化为了一段提交，降低了一半的消息延迟。

**一致性检查**

在 Raft 算法中，Leader通过强制Follower直接复制自己的日志项，处理不一致日志。具体有 2 个步骤：

1. 首先，Leader通过AppendEntries消息，找到Follower节点上与自己相同日志项的最大索引值。也就是说，这个索引值之前的日志，Leader和Follower是一致的，之后的日志是不一致的了。
2. 然后，Leader强制Follower更新不一致日志项，实现日志的一致。

举个例子来理解下， 为了方便演示，我们引入 2 个新变量 ：

- PrevLogEntry：表示当前要复制的日志项的前一条日志项的索引值。比如下图中，如果Leader将索引值为8的日志项发送给Follower，那么此时 PrevLogEntry 值为 7；
- PrevLogTerm：表示当前要复制的日志项的前一条日志项的任期编号。比如下图中，如果Leader将索引值为8的日志项发送给Follower，那么此时 PrevLogTerm 值为 4。

![Raft-9](Raft%E7%AE%97%E6%B3%95.assets/Raft-9.png)

1. 首先，Leader通过AppendEntries消息，发送当前最新的日志项到Follower，这个日志项的 PrevLogEntry 值为 7，PrevLogTerm 值为 4；
2. 如果Follower在它的日志中，找不到PrevLogEntry 值为 7、PrevLogTerm 值为 4 的日志项，也就是说它的日志和Leader的不一致了，那么Follower就会拒绝接收新的日志项，并返回失败信息给Leader；
3. 此时Leader会递减要复制的日志项的索引值，并发送新的日志项到Follower，这个消息的 PrevLogEntry 值为 6，PrevLogTerm 值为 3；
4. 如果Follower在它的日志中，找到了 PrevLogEntry 值为 6、PrevLogTerm 值为 3 的日志项，那么AppendEntries消息返回成功，这样一来，Leader就知道在 PrevLogEntry 值为 6、PrevLogTerm 值为 3 的位置，Follower的日志项与自己相同；
5. 最后， Leader通过AppendEntries消息，复制并更新覆盖该索引值之后的日志项（也就是不一致的日志项），最终实现了集群各节点日志的一致。

## 7 Raft安全性

Raft增加了如下两条限制以保证安全性：

- 拥有最新的已提交的log entry的Follower才有资格成为Leader。

  这个保证是在RequestVote RPC中做的，Candidate在发送RequestVote RPC时，要带上自己的最后一条日志的term和log index，其他节点收到消息时，如果发现自己的日志比请求中携带的更新，则拒绝投票。日志比较的原则是，如果本地的最后一条log entry的term更大，则term大的更新，如果term一样大，则log index更大的更新。

- Leader只能推进commit index来提交当前term的已经复制到大多数服务器上的日志，旧term日志的提交要等到提交当前term的日志来间接提交（log index 小于 commit index的日志被间接提交）。

## 8 Raft日志压缩

在实际的系统中，不能让日志无限增长，否则系统重启时需要花很长的时间进行回放，从而影响可用性。Raft采用对整个系统进行snapshot来解决，snapshot之前的日志都可以丢弃。

每个副本独立的对自己的系统状态进行snapshot，并且只能对已经提交的日志记录进行snapshot。

Snapshot中包含以下内容：

- 日志元数据。最后一条已提交的 log entry的 log index和term。这两个值在snapshot之后的第一条log entry的AppendEntries RPC的完整性检查的时候会被用上。
- 系统当前状态。

当Leader要发给某个日志落后太多的Follower的log entry被丢弃，Leader会将snapshot发给Follower。或者当新加进一台机器时，也会发送snapshot给它。发送snapshot使用InstalledSnapshot RPC。

## 9 Raft成员变更

成员变更是在集群运行过程中副本发生变化，如增加/减少副本数、节点替换等。

