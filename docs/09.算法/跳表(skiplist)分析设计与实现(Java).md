# 跳表(skiplist)分析设计与实现(Java)

## 什么是跳跃表

跳跃表(简称跳表)由美国计算机科学家***William Pugh发明于1989年***。他在论文《Skip lists: a probabilistic alternative to balanced trees》中详细介绍了跳表的数据结构和插入删除等操作。

> 跳表(SkipList，全称跳跃表)是用于有序元素序列快速搜索查找的一个数据结构，跳表是一个随机化的数据结构，实质就是一种可以进行二分查找的有序链表。跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。跳表不仅能提高搜索性能，同时也可以提高插入和删除操作的性能。它在性能上和红黑树，AVL树不相上下，但是跳表的原理非常简单，实现也比红黑树简单很多。

![skiplist](https://gitee.com/linbingxing/image/raw/master/algorithm/skiplist.png)

