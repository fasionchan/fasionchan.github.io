---
layout: post
title: "【译文】MongoDB WiredTiger引擎调优技巧"
date: 2017-03-09 18:44:40 +0800
comments: true
categories: MongoDB WiredTiger
keywords: mongodb,wiretiger,cache,tickets
---

`MongoDB`从`3.0`开始引入可插拔存储引擎的概念。
当前，有不少存储引擎可供选择：`MMAPV1`、`WiredTiger`、`MongoRocks`、`TokuSE`等等。
每个存储引擎都有自己的优势，你需要根据性能要求及应用特征挑选最适合的一个。

从`3.2.x`开始，`WiredTiger`成为默认的存储引擎。
最为`MongoDB`目前最流行的存储引擎，`WiredTiger`与原先的`MMAPV1`相比有以下优势：

- **性能&并发**：在大多数工作负载下，`WiredTiger`的性能要比`MMAPV1`高很多。
`WiredTiger`引擎为现代多核系统量身定制，更好地发挥多核系统的处理能力。
`MMAPV1`引擎使用表级锁，因此，当某个单表上有并发的操作，吞吐将受到限制。
`WiredTiger`使用文档级锁，由此带来并发及吞吐的提高。
对于典型的应用，切到`WiredTiger`引擎，可带来5-10倍的性能提升。

<!--more-->

- **压缩&加密**：`MMAPV1`引擎要求数据在内存和在磁盘的形式一致(map磁盘内存映射)。
因此，它并不支持压缩和加密。`WiredTiger`并没有这层限制，可以更好地支持。

- **索引前缀压缩**：`WiredTiger`存储索引时使用前缀压缩——相同的前缀只存一次。
由此带来的效果是：索引更小了，对物理内存使用也更少了。

接下来，我会展示几个用来调优`WiredTiger`引擎性能的关键参数。

# 调优Cache Size

`WiredTiger`最重要的调优参数就是`cache`规模。
默认，`MongoDB`从`3.x`开始会保留可用物理内存的50%(`3.2`是60%)作为数据`cache`。
虽然，默认的设置可以应对大部分的应用，通过调节为特定应用找到最佳配置值还是非常值得的。
`cache`的规模必须足够大，以便保存应用整个工作集(working set)。

除了这个`cache`，`MongoDB`在做诸如聚合、排序、连接管理等操作时需要额外的内存。
因此，必须确保有足够的内存可供使用，否则，`MongoDB`进程有被`OOM killer`杀死的风险。

调节这个参数，首先要理解在默认配置下，`cache`的使用情况。运行以下命令，可以获得`cache`统计：

```
db.serverStatus().wiredTiger.cache
```

命令输出结果例子如下：

```
{
   "tracked dirty bytes in the cache" : 409861,
   "tracked bytes belonging to internal pages in the cache" : 738956332,
   "bytes currently in the cache" : 25769360777,
   "tracked bytes belonging to leaf pages in the cache" : 31473298388,
   "maximum bytes configured" : 32212254720,
   "tracked bytes belonging to overflow pages in the cache" : 0,
   "bytes read into cache" : 29628550664,
   "bytes written from cache" : 34634778285,
   "pages evicted by application threads" : 0,
   "checkpoint blocked page eviction" : 102,
   "unmodified pages evicted" : 333277,
   "page split during eviction deepened the tree" : 0,
   "modified pages evicted" : 437117,
   "pages selected for eviction unable to be evicted" : 44825,
   "pages evicted because they exceeded the in-memory maximum" : 74,
   "pages evicted because they had chains of deleted items" : 33725,
   "failed eviction of pages that exceeded the in-memory maximum" : 1518,
   "hazard pointer blocked page eviction" : 34814,
   "internal pages evicted" : 21623,
   "maximum page size at eviction" : 10486876,
   "eviction server candidate queue empty when topping up" : 8235,
   "eviction server candidate queue not empty when topping up" : 3020,
   "eviction server evicting pages" : 191708,
   "eviction server populating queue, but not evicting pages" : 2996,
   "eviction server unable to reach eviction goal" : 0,
   "pages split during eviction" : 8821,
   "pages walked for eviction" : 157970002,
   "eviction worker thread evicting pages" : 563015,
   "in-memory page splits" : 52,
   "percentage overhead" : 8,
   "tracked dirty pages in the cache" : 9,
   "pages currently held in the cache" : 1499798,
   "pages read into cache" : 2260232,
   "pages written from cache" : 3018846
}
```

第一个要关注的数值试，`cache`中脏数据的百分比。
如果这个百分比比较高，那么调大`cache`规模很有可能可以提升性能。
如果应用是重读的，可再关注`bytes read into cache`这个指标。
如果这个指标比较高，那么调大`cache`规模很有可能可以提升读性能。

调节`cache`规模不一定非得重启服务，我们可以动态调整：

```
db.adminCommand( { "setParameter": 1, "wiredTigerEngineRuntimeConfig": "cache_size=xxG"})
```

如果你想让调整在重启后也有效，那么你需要将配置文件也相应调整一下。

# 控制Read/Write Tickets

WiredTiger使用`tickets`来控制可以同时被存储引擎处理的读/写操作数。
默认值是128，在大部分情况下表现良好。
如果这个值经常掉到0，所有后续操作将会被排队等待。
例如，观察到读`tickets`下降，系统可能有大量长耗时的操作(未索引操作)。
如果你想找出有哪些慢操作，可以用一些第三方工具。
你可以根据系统需要和性能影响上下调节`tickets`。

运行以下命令可以确认`tickets`的使用情况：

```
db.serverStatus().wiredTiger.concurrentTransactions
```

下面是一个输出例子：

```
{
   "write" : {
      "out" : 0,
      "available" : 128,
      "totalTickets" : 128
   },
   "read" : {
      "out" : 3,
      "available" : 128,
      "totalTickets" : 128
   }
}
```

同样，可以动态调节`tickets`：

```
db.adminCommand( { setParameter: 1, wiredTigerConcurrentReadTransactions: xx } )
db.adminCommand( { setParameter: 1, wiredTigerConcurrentWriteTransactions: xx } )
```

一旦做出调整，注意要观察系统的性能监控确保影响是符合预期的。
