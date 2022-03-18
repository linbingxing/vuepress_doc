---
title: 一致性Hash算法
date: 2022-02-24 20:35:17
permalink: /pages/147d06/
categories:
  - 算法
  - 分布式算法
tags:
  - 
---
# 分布式算法 - 一致性哈希算法

## 一致性哈希算法简介

一致性哈希算法在1997年由[麻省理工学院](https://baike.baidu.com/item/麻省理工学院/117999)提出，是一种特殊的哈希算法，目的是解决分布式缓存的问题。 在移除或者添加一个服务器时，能够尽可能小地改变已存在的服务请求与处理请求服务器之间的映射关系。一致性哈希解决了简单哈希算法在分布式[哈希表](https://baike.baidu.com/item/哈希表/5981869)( Distributed Hash Table，DHT) 中存在的动态伸缩等问题。

一致性哈希算法提出了在动态变化的Cache环境中，判定哈希算法好坏的四个定义:

- `平衡性(Balance)`: 平衡性是指哈希的结果能够尽可能分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。很多哈希算法都能够满足这一条件。
- `单调性(Monotonicity)`: 单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲加入到系统中。哈希的结果应能够保证原有已分配的内容可以被映射到原有的或者新的缓冲中去，而不会被映射到旧的缓冲集合中的其他缓冲区。
- `分散性(Spread)`: 在分布式环境中，终端有可能看不到所有的缓冲，而是只能看到其中的一部分。当终端希望通过哈希过程将内容映射到缓冲上时，由于不同终端所见的缓冲范围有可能不同，从而导致哈希的结果不一致，最终的结果是相同的内容被不同的终端映射到不同的缓冲区中。这种情况显然是应该避免的，因为它导致相同内容被存储到不同缓冲中去，降低了系统存储的效率。分散性的定义就是上述情况发生的严重程度。好的哈希算法应能够尽量避免不一致的情况发生，也就是尽量降低分散性。
- `负载(Load)`: 负载问题实际上是从另一个角度看待分散性问题。既然不同的终端可能将相同的内容映射到不同的缓冲区中，那么对于一个特定的缓冲区而言，也可能被不同的用户映射为不同 的内容。与分散性一样，这种情况也是应当避免的，因此好的哈希算法应能够尽量降低缓冲的负荷。

**优点**

- 可扩展性。一致性哈希算法保证了增加或减少服务器时，数据存储的改变最少，相比传统哈希算法大大节省了数据移动的开销 。

- 更好地适应数据的快速增长。采用一致性哈希算法分布数据，当数据不断增长时，部分虚拟节点中可能包含很多数据、造成数据在虚拟节点上分布不均衡，此时可以将包含数据多的虚拟节点分裂，这种分裂仅仅是将原有的虚拟节点一分为二、不需要对全部的数据进行重新哈希和划分。

  虚拟节点分裂后，如果物理服务器的负载仍然不均衡，只需在服务器之间调整部分虚拟节点的存储分布。这样可以随数据的增长而动态的扩展物理服务器的数量，且代价远比传统哈希算法重新分布所有数据要小很多。

## 一致性哈希算法原理

### 环形哈希空间

一致性哈希算法通过一个叫作一致性哈希环的数据结构实现。这个环的起点是 0，终点是 2^32 - 1，并且起点与终点连接，故这个环的整数分布范围是 [0, 2^32-1]，如下图所示：

![hash-1](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-1.jpg)

整个空间按顺时针方向组织，0和2^32-1在零点中方向重合。

假设我们有3台服务器节点，把服务器节点通过hash算法，加入到上述的环中。一般情况下是根据机器的IP地址或者唯一的计算机别名进行哈希。

```sh
n1=hash(node1);
n2=hash(node2);
n3=hash(node3);
```

![hash-2](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-2.png)

假设我们现在有key1,key2,key3,key4 4个key值，我们通过一定的hash算法，计算出对应的key值，将其对应到上面的环形hash空间中。

```bash
k1=hash(key1);
k2=hash(key2);
k3=hash(key3);
k4=hash(key4);
```

![hash-3](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-3.png)

接下来就是数据如何存储到服务器节点上了，key值哈希之后的结果顺时针找上述环形hash空间中，距离自己最近的机器节点，然后将数据存储到上面， 如上图所示，k1 存储到 n2 服务器上， k2,k3存储到n3服务器上， k4存储在n1服务器上。用图表示如下:

![hash-4](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-4.png)

###  删除服务节点

假设n2服务器宕机，这时候需要从集群中将其摘除。那么，之前存储在n2上的k1，将会顺时针寻找距离它最近的一个节点，也就是n3节点，这样，k1就会存储到n3上了，看一看下下面的图，比较清晰。

![hash-5](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-5.png)

摘除n2节点之后，只影响到了原先存储在n3上的k1，而k3、k4、k2都没有受到影响，也就意味着解决了最开始的解决方案(hash(key)%N)中可能带来的雪崩问题。

###  增加服务节点

新增n4节点之后，原先存储到n3的k2，迁移到了n4，分担了n3上的存储压力和流量压力。干扰的也只有n3而已，其他数据不会受到影响。

![hash-6](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-6.png)

因此，一致性哈希算法对于节点的增减都只需重定位换空间的一小部分即可，具有较好的容错性和可扩展性。

### 虚拟节点

**出现不平衡问题**

上面的简单的一致性hash的方案在某些情况下但依旧存在问题: 一个节点宕机之后，数据需要落到距离他最近的节点上，会导致下个节点的压力突然增大，可能导致雪崩，整个服务挂掉。如图所示

![hash-5](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-5-1645709501054.png)

当节点n2宕机之后，之前在n2上的k1就要迁移到n3上，这时候带来了两部分的压力:

- 之前请求到n2上的流量转嫁到了n3上,会导致n3的流量增加，如果之前n2上存在热点数据，则可能导致n3扛不住压力挂掉。
- 之前存储到n2上的key值转义到了n3，会导致n3的内容占用量增加，可能存在瓶颈。

当上面两个压力发生的时候，可能导致n3节点也宕机了。那么压力便会传递到n1上，又出现了类似滚雪球的情况，服务压力出现了雪崩，导致整个服务不可用。这一点违背了最开始提到的四个原则中的 **平衡性**， 节点宕机之后，流量及内存的分配方式打破了原有的平衡。

针对这个问题，我们可以通过引入虚拟节点来解决负载不均衡的问题。即将每台物理服务器虚拟为一组虚拟服务器，将虚拟服务器放置到哈希环上，如果要确定对象的服务器，需先确定对象的虚拟服务器，再由虚拟服务器确定物理服务器。

假设存在以下的真实节点和虚拟节点的对应关系。

![hash-7](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-7-1645710606290.png)

映射到具体环形上来看

![hash-8](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-8.png)

假设n1机器宕机，则会发生一下情况。

- 原先存储在虚拟节点V1上的k1数据将迁移到V3上，也就意味着迁移到了n2机器上。
- 原先存储再虚拟节点V2上的k2数据将迁移到V6上，也就意味着迁移到了n3机器上。

结果如下图:

![hash-9](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-9.png)

这个就解决之前的问题了，某个节点宕机之后，存储及流量压力并没有全部转移到某台机器上，而是分散到了多台节点上。解决了节点宕机可能存在的雪崩问题。

当物理节点多的时候，虚拟节点多，这个的雪崩可能就越小。

一个物理节点应该拆分为多少虚拟节点，下面可以先看一张图：

![hash-10](%E4%B8%80%E8%87%B4%E6%80%A7Hash%E7%AE%97%E6%B3%95.assets/hash-10.png)

横轴表示需要为每台福利服务器扩展的虚拟节点倍数，纵轴表示的是实际物理服务器数。可以看出，物理服务器很少，需要更大的虚拟节点；反之物理服务器比较多，虚拟节点就可以少一些。比如有10台物理服务器，那么差不多需要为每台服务器增加100~200个虚拟节点才可以达到真正的负载均衡。

## 一致性哈希算法的Java实现

**不带虚拟节点的**

```java
import java.util.SortedMap;
import java.util.TreeMap;

public class ConsistentHashingWithoutVirtualNode {
    //待添加入Hash环的服务器列表
    private static String[] servers = {"192.168.0.1:8888", "192.168.0.2:8888", 
      "192.168.0.3:8888"};

    //key表示服务器的hash值，value表示服务器
    private static SortedMap<Integer, String> sortedMap = new TreeMap<Integer, String>();

    //程序初始化，将所有的服务器放入sortedMap中
    static {
        for (int i = 0; i < servers.length; i++) {
            int hash = getHash(servers[i]);
            System.out.println("[" + servers[i] + "]加入集合中, 其Hash值为" + hash);
            sortedMap.put(hash, servers[i]);
        }
    }

    //得到应当路由到的结点
    private static String getServer(String key) {
        //得到该key的hash值
        int hash = getHash(key);
        //得到大于该Hash值的所有Map
        SortedMap<Integer, String> subMap = sortedMap.tailMap(hash);
        if (subMap.isEmpty()) {
            //如果没有比该key的hash值大的，则从第一个node开始
            Integer i = sortedMap.firstKey();
            //返回对应的服务器
            return sortedMap.get(i);
        } else {
            //第一个Key就是顺时针过去离node最近的那个结点
            Integer i = subMap.firstKey();
            //返回对应的服务器
            return subMap.get(i);
        }
    }

    //使用FNV1_32_HASH算法计算服务器的Hash值
    private static int getHash(String str) {
        final int p = 16777619;
        int hash = (int) 2166136261L;
        for (int i = 0; i < str.length(); i++)
            hash = (hash ^ str.charAt(i)) * p;
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;

        // 如果算出来的值为负数则取其绝对值
        if (hash < 0)
            hash = Math.abs(hash);
        return hash;
    }

    public static void main(String[] args) {
        String[] keys = {"semlinker", "kakuqo", "fer"};
        for (int i = 0; i < keys.length; i++)
            System.out.println("[" + keys[i] + "]的hash值为" + getHash(keys[i])
                    + ", 被路由到结点[" + getServer(keys[i]) + "]");
    }

}
```

**带虚拟节点的**

```java
/**
 * 带虚拟节点的一致性Hash算法
 * @author 五月的仓颉 http://www.cnblogs.com/xrq730/
 */
public class ConsistentHashingWithVirtualNode
{
    /**
     * 待添加入Hash环的服务器列表
     */
    private static String[] servers = {"192.168.0.0:111", "192.168.0.1:111", "192.168.0.2:111",
            "192.168.0.3:111", "192.168.0.4:111"};

    /**
     * 真实结点列表,考虑到服务器上线、下线的场景，即添加、删除的场景会比较频繁，这里使用LinkedList会更好
     */
    private static List<String> realNodes = new LinkedList<String>();

    /**
     * 虚拟节点，key表示虚拟节点的hash值，value表示虚拟节点的名称
     */
    private static SortedMap<Integer, String> virtualNodes =
            new TreeMap<Integer, String>();

    /**
     * 虚拟节点的数目，这里写死，为了演示需要，一个真实结点对应5个虚拟节点
     */
    private static final int VIRTUAL_NODES = 5;

    static
    {
        // 先把原始的服务器添加到真实结点列表中
        for (int i = 0; i < servers.length; i++)
            realNodes.add(servers[i]);

        // 再添加虚拟节点，遍历LinkedList使用foreach循环效率会比较高
        for (String str : realNodes)
        {
            for (int i = 0; i < VIRTUAL_NODES; i++)
            {
                String virtualNodeName = str + "&&VN" + String.valueOf(i);
                int hash = getHash(virtualNodeName);
                System.out.println("虚拟节点[" + virtualNodeName + "]被添加, hash值为" + hash);
                virtualNodes.put(hash, virtualNodeName);
            }
        }
        System.out.println();
    }

    /**
     * 使用FNV1_32_HASH算法计算服务器的Hash值,这里不使用重写hashCode的方法，最终效果没区别
     */
    private static int getHash(String str)
    {
        final int p = 16777619;
        int hash = (int)2166136261L;
        for (int i = 0; i < str.length(); i++)
            hash = (hash ^ str.charAt(i)) * p;
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;

        // 如果算出来的值为负数则取其绝对值
        if (hash < 0)
            hash = Math.abs(hash);
        return hash;
    }

    /**
     * 得到应当路由到的结点
     */
    private static String getServer(String node)
    {
        // 得到带路由的结点的Hash值
        int hash = getHash(node);
        // 得到大于该Hash值的所有Map
        SortedMap<Integer, String> subMap =
                virtualNodes.tailMap(hash);
        // 第一个Key就是顺时针过去离node最近的那个结点
        Integer i = subMap.firstKey();
        // 返回对应的虚拟节点名称，这里字符串稍微截取一下
        String virtualNode = subMap.get(i);
        return virtualNode.substring(0, virtualNode.indexOf("&&"));
    }

    public static void main(String[] args)
    {
        String[] nodes = {"127.0.0.1:1111", "221.226.0.1:2222", "10.211.0.1:3333"};
        for (int i = 0; i < nodes.length; i++)
            System.out.println("[" + nodes[i] + "]的hash值为" +
                    getHash(nodes[i]) + ", 被路由到结点[" + getServer(nodes[i]) + "]");
    }
}
```

资料参考

[一致性Hash算法详解](https://zhuanlan.zhihu.com/p/98030096)

[5分钟理解一致性哈希算法](https://juejin.cn/post/6844903750860013576)

[图解一致性哈希算法](https://segmentfault.com/a/1190000021199728)

[对一致性Hash算法，Java代码实现的深入研究](https://www.cnblogs.com/xrq730/p/5186728.html)