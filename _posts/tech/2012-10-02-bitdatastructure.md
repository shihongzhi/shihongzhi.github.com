---
layout:     post
title:      大规模数据学习小结
category: blog
description: 大规模数据学习小结
tags: 大规模, 数据结构, 算法
---
因为准备面试的原因，最近看了不少大规模数据处理相关的资料。在这里顺便做一下记录。

##B树

大规模数据存储应用中主要性能瓶颈是磁盘I/O读写过于频繁，这也就是普通二分查找树不适用于大规模数据存储应用的原因，因为它们的树的高度过大，会导致频繁读取磁盘，进而导致效率低下。

基于这个思想，`B树`(B为`Balance`的简称)被设计成了一种多叉平衡查找树。这样会使得数的高度相比较于平衡二分查找树低很多。

`m阶B树`的定义：

>1. 树中每个节点最多含有m个孩子(m>=2)
>2. 除根结点和叶子结点外，其它每个结点至少有[ceil(m / 2)]个孩子（其中ceil(x)是一个取上限的函数）；
>3. 若根结点不是叶子结点，则至少有2个孩子（特殊情况：没有孩子的根结点，即根结点为叶子结点，整棵树只有一个根节点）；
>4. 所有叶子结点都出现在同一层，叶子结点不包含任何关键字信息(可以看做是外部接点或查询失败的接点，实际上这些结点不存在，指向这些结点的指针都为null)；
>5. 非叶子节点中就直接保存记录，该记录不会再出现在其孩子节点中，这也是`B树`与`B+树`的主要区别

简单的情况下，可以把每个节点想象成磁盘中的一个block，每次查找都需要读取一个block，如果没有找到，则再读取相应的子节点block，递归查找。

当然对于那些经常读取的节点，或者说root节点附近的节点一般情况下都是常存于内存的，这样可以节省不少的磁盘I/O读写操作

`B树`的插入删除操作都可能会引发节点记录的分裂和合并。具体的操作这里就不叙述了。可以查看[该博客](http://blog.csdn.net/v_july_v/article/details/6530142)

B树实际上是为了降低树的高度，牺牲了CPU的效率。所以当整个数据结构都是在内存中时，一个普通的2-3数或AVL数足矣。B树更适合外存查找

##B+数

B+数是一个B树的变形，它更适合文件系统和数据库的设计

>1. 有n棵子树的结点中含有n个关键字； (而B 树是n棵子树有n-1个关键字)
>2. 所有的叶子结点中包含了全部关键字的信息，及指向含有这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大的顺序链接。 (而B 树的叶子节点并没有包括全部需要查找的信息)
>3. 所有的非终端结点可以看成是索引部分，结点中仅含有其子树根结点中最大（或最小）关键字。 (而B 树的非终节点也包含需要查找的有效信息)

至于为什么B+树比较适合数据库的数据保存，主要原因在于数据库中经常会进行range query，也就是范围查找。而B树不支持这种快速扫库的行为

##B\*树

B\*树是B+树的变种，在B+ 树非根和非叶子结点再增加指向兄弟的指针；

##R树

R树是B树在高维空间的扩展，也是一棵平衡树。在二维情况下的具体组织结构如下图所示：

![R树][R-img-reg]
[R-img-reg]:{{ BASE_PATH }}/images/2012/R-tree.jpg

R树通过MBR(Minimal Bounding Rectangle)的方法进行空间分割。

R树的具体定义为：

>1. 除非它是根结点之外，所有叶子结点包含有m至M个记录索引（条目）。作为根结点的叶子结点所具有的记录个数可以少于m。通常，m=M/2。
>2. 对于所有在叶子中存储的记录（条目），I是最小的可以在空间中完全覆盖这些记录所代表的点的矩形（注意：此处所说的“矩形”是可以扩展到高维空间的）。
>3. 每一个飞叶子结点拥有m至M个孩子结点，除非它是根结点。
>4. 对于在非叶子结点上的每一个条目，i是最小的可以在空间上完全覆盖这些条目所代表的店的矩形（同性质2）。
>5. 所有叶子结点都位于同一层，因此R树为平衡树。

##LSM-Tree

B+树的插入记录的效率比较低下，需要进行节点索引的重建，因此在查询频率远低于插入频率的情况下(如日志文件)，LSM-Tree应运而生。LSM-Tree能够有效的降低索引插入的开销。大部分的NoSQL数据库都用到了LSM-Tree(如BigTable)

LSM-Tree通过使用某种算法(两组件C0C1及多组件算法)，对索引变更进行`延迟`及`批量`处理，并通过一种类似于归并排序的方式高效地将更新迁移到磁盘。

下图为LSM-Tree的插入过程：

![LSM-Tree][LSM-img-reg]
[LSM-img-reg]:{{ BASE_PATH }}/images/2012/LSM-tree.jpg

上图一个具有组件C0，C1，C2…Ck-1和Ck的多组件LSM-tree，索引树的大小伴随着下标的增加而增大，其中只有C0是驻留在内存中的，其他则是在磁盘上。在所有的组件对(Ci-1,Ci)之间都有一个异步的rolling merge过程负责在较小的组件Ci-1超过阈值大小时，将它的记录移到Ci中

###LSM-Tree插入 

Rolling merge将记录的插入批量化了。并不是一个记录就进行一次I/O操作，而是在先在C0中保存插入的记录，当C0中的记录满了之后，在通过一次磁盘I/O将C0中的记录批量处理到Ci中。因此对索引延迟批量处理的过程中，rolling merge起到了关键性的作用。

如果把那些最近插入的值（添加时间戳）都放在C0中，而不要移动到Ci去，因为C0一直保存在内存中，C0就充当了内存缓冲池的作用了，所以处理速度能够相当快速。这种策略针对短期事务UNDO日志的索引访问模式特别有效。

###LSM-Tree删除 

如果删除记录不在C0中(在C0中则直接删除)，可以记录一个删除标记记录。延迟删除，实际的删除可以在后面的rolling merge过程中碰到实际的那个索引entry时执行；与此同时，查询请求必须在通过该删除标记的过滤，以避免返回一个已经删除的记录

针对记录的索引Key被改变的情况，之前是在B树中是要重新建立索引的，但是针对LSM-tree可以简化成两个操作：删除一个记录+插入一个记录

###Tips
为了提高LSM-Tree的随机读的效率，可以通过`Level LRU`和`Bloom Filter`策略，还可以用`Fractional cascading`方法。

`Bloom Filter`策略是通过为每颗Ci树中的记录建立一个Bloom Filter，这样就可以快速判断某记录是否在该Ci树中出现，如果不在该Ci树中出现的话，可以直接排除对该Ci树的遍历了。

`Fractional cascading`方法通过为相邻的Ci树中的记录设置`Forwarding pointers`来实现，当然指向每个block的Forwarding pointer只需要一个，所以可以去除那些重复指向同一个block的指针。针对没有被Forwarding poiner指向的block，需要添加一个`ghost pointer`

上面提到的Ci树可以任何数，但是一般情况下C0常存于内存，所以一般C0用普通的2-3数或AVL数足矣，而其他Ci树保存于磁盘中，所以应该采用B+树。

##TokuDB

首先提一下CacheOblivious算法(缓存忘却算法)，该算法在不需要明确知道存储器层次中数据传输规模的情况下，也可以高效的工作。

###Fractal Tree Indexes

然后就是TokuDB提出的替代B+树的`Fractal Tree Indexes`.

具体`Fractal Tree Indexes`的定义如下：

>1. logN arrays, one array for each power of two.
>2. Each array is completely full or empty.
>3. Each array is sorted.

####查询

下图是包含10个元素的Fractal Tree Indexes:

![fractal-Tree][fractal10-img-reg]
[fractal10-img-reg]:{{ BASE_PATH }}/images/2012/fractal-tree-10.jpg

如果只是这种结构的话，查找的复杂度为O(log^2N),即logN的树高度乘以array的长度logN。可以通过增加`Forward Pointer`来加速查找速度，添加Forward Pointers之后的结构会变成如下：

![fractalforward][fractalforward-img-reg]
[fractalforward-img-reg]:{{ BASE_PATH }}/images/2012/fractal-forward.jpg

通过添加`Forward Pointer`之后，查找的复杂度降低到了O(logN)

####插入

具体的插入操作为开辟一块空的Temp arrays，然后进行merge操作。如下图所示：

![fractalinsert][fractalinsert-img-reg]
[fractalinsert-img-reg]:{{ BASE_PATH }}/images/2012/fractal-insert.jpg

平均的插入复杂度为O(logN/B)

###COLA

`LSM tree + forward + ghost = COLA`

##Spanner

Spanner是一个多月前Google在OSDI会议上公开的一个分布式系统。Spanner与其他系统的比较：

![Spanner][spanner-img-reg]
[spanner-img-reg]:{{ BASE_PATH }}/images/2012/spanner.jpg

从上图的比较中可见Spanner的强大。Spanner的这篇论文我还在看，等看完了再跟大家分享！