---
layout: post
title: "[译文]跳表：一种平衡树的概率性替代品"
date: 2017-06-17 16:33:32 +0800
comments: true
categories: [data structure]
keywords: skip list,data structure
---

 **跳表**是一种可以替代**平衡树**的数据结构。跳表追求的是概率性平衡，而不是严格平衡。因此，跟平衡二叉树相比，跳表的**插入**和**删除**操作要简单得多，执行也更快。

二叉树可以用来实现字典和有序表等抽象数据结构。在元素随机插入的场景，二叉树可以很好应对。然而，在有序插入的情况下，二叉树就退化了(链表)，性能非常差。如果有办法对待插入元素进行随机排列，二叉树大概率可以运行良好。大部分情况下，插入是在线进行的，因此随机排列并不具有可行性。平衡树在操作时对树结构进行调整以满足平衡条件，因此获得理想性能。

跳表是一种概率性可行的平衡二叉树替代数据结构。跳表通过一个**随机数生成器**实现平衡。虽然跳表最坏情况下(`worst-case`)性能也很差，但是没有任何输入序列必然会导致最坏情况发生(这点类似划分元素(`pivot point`)随机选定的**快排**)。跳表极度不平衡发生的概率非常低(一个包含`250`个元素的字典，一次查找需要花`3`倍期望时间的概率小于百万分之一)。跳表平衡概率跟随机插入的二叉树差不多，好处是插入顺序不要求随机。

<!--more-->

实现概率性平衡比严格控制平衡要简单得多。对很多应用来说，跳表用起来比平衡树更自然，而且算法更简单。跳表算法简单性意味着更容易实现，而且与平衡树和自适应树相比有常数倍数的性能提升。跳表在空间上也比较高效。平均每个元素只需要额外耗费个2指针(甚至可以配置得更低)，并不需要在每个节点上都存与平衡和优先级相关的数据。

## 结构

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2740477-7dc366ea883cf19b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

搜索一个链表时，我们需要遍历每个节点(如图 1a)。如果列表是有序的，偶数节点另存一个指向下一个偶数节点的指针(如图 1b)，我们只需要检查最多`(n/2)+1`个节点(`n`是链表规模)。如果序号为`4`的倍数的节点都有一个往前跳`4`步的节点，那么最多只需要检查`(n/4)+2`次。如果，序号为`2^i`的节点有一个向前跳`2^i`步的指针，那么则需要检查`log2 n`次了！这种数据结构可以用来做快速搜索，但是插入和删除并没有可行性。

有`k`个前进指针的节点成为`k`层节点。如果第`2^i`个节点有一个向前跳`2^i`步的指针，那么每层节点数满足以下关系：第`1`层有`50%`的节点；第`2`层有`25%`的节点；第`3`层有`12.5%`的节点；以此类推。假设每层的比例还是一样，但是节点随机选择，会怎样呢(图 1e)？节点第`i`个前进指针不严格跳`2^i`步，而是可以跳任意步。由于不需要维持特殊条件，插入节点层数随机生成，插入和删除只需要做**局部修改**。极端情况下，有些层次分布会导致极差的性能，不过接下来我们会看到这种情况非常罕见。这种数据结构在链表的基础上加上额外指针以跳过一些中间节点，因此命名为**跳表**。

## 算法

这小节介绍用于**搜索**、**插入**、**删除**的算法。**搜索**操作返回与给定键(`key`)关联的值(`value`)，键不存在时则失败。**插入**操作将给定键关联到新的值，如果键不存在则插入新的节点。**删除**操作删除给定键。另外，类似**最小键**和**下一键**这类操作实现起来也非常简单。

每个元素由一个节点表示，层次由节点在插入时随机选定，与已有元素无关。层次为`i`的节点拥有`i`个前进指针，下标分别是`1`至`i`。节点不需要存储层数。选定一个合适的常量`MaxLevel`，层数在这个范围内。跳表的层数时当前所有节点层数的最大值，或者当跳表为空是，层数为`1`。用一个头向量存储从层次`1`到`MaxLevel`的向前指针。指针高于当前跳表层数的部分直接指向`NIL`。

### 初始化

约定`NIL`元素，其键比所有合法建都大(上限)。跳表的任意层都以`NIL`结尾。新的跳表初始化成层数只有`1`，并且所有表头所有前进指针都指向`NIL`。

### 查找

查找某个元素时，需要逐层遍历所有键不超过给定键的节点。如果当前层前进节点已经不符合条件了，往下一层开始遍历。当遍历进行到第`1`层时，下一个节点就是目标节点(如存在)。

```
Search(list, searchKey)
    x := list->header

    for i := list->level downto 1 do
        while x->forward[i]->key < searchKey do
            x = x->forward[i]

    x := x->forward[1]

    if x->key = searchKey
    then
        return x->value
    else
        return failure
```

### 插入/删除

**插入**或者**删除**节点，只需先执行搜索操作(图 3)，然后视情况重新拼接。伪代码如下所示：

```
Insert(list, searchKey, newValue)
    local update[1..MaxLevel]
    x := list-header

    for i := list->level downto 1 do
        while x->forward[i]->key < searchKey do
            x := x->forward[i]
        update[i] := x

    x := x->forward[i]

    if x->key = searchKey then
        x->value := newValue
    else
        lvl := randomLevel()
        if lvl > list->level then
            for i := list->level+1 to lvl do
                update[i] := list->header
            list->level = lvl
        x := makeNode(lvl, searchKey, value)
        for i := 1 to lvl do
            x->forward[i] = update[i]->forward[i]
            update[i]->forward[i] := x
```

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2740477-4c9045f1819efd42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图3展示了搜索过程。注意到，搜索的过程中维护了一个名为`update`的向量，在每次降层搜索时更新。搜索完成后，`update`刚好记录了各层在操作位置(图中环)左边最近的节点：

|元素|节点|
|:----|:----|
|update[1]|12|
|update[2]|9|
|update[3]|6|
|update[4]|6|

如果插入时生成了一个比当前最大层更大的层数，则需要更新跳表层数并且初始化`update`向量对应部分。

接下来，看看删除操作的伪代码：

```
Delete(list, searchKey)
    local update[1..MaxLevel]
    x := list-header

    for i := list->level downto 1 do
        while x->forward[i]->key < searchKey do
            x := x->forward[i]
        update[i] := x

    x := x->forward[i]

    if x->key < searchKey then
        for i := 1 to list->level do
            if update[i]->forward[i] != x then break
            update[i]->forward[i] = x->forward[i]

        free(x)

        while list->level > 1 and list->header->forward[list->level] = NIL do
            list->level := list->level - 1
```

在每次删除时，需要检查被删除节点是否是最大层节点。如果是，需要对跳表层数做对应调整。

### 随机函数

接下来，需啊确定一个随机数生成函数，其概率分布使得第`i`层中有`50%`的节点同时数据第`i+1`层。先抛开具体数值，我们在讨论一个分数`p`，对于有`i`层指针的节点中`p`部分，同时拥有`i+1`层指针。以下便是一个非常理想的随机数生成函数，随机层数生成与跳表元素及规模无关：

```
randomLevel()
    lvl := 1
    while random() < p and lvl < MaxLevel do
        lvl := lvl + 1
    return lvl
```
