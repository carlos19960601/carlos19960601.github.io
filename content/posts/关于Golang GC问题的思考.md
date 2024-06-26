---
title: "关于Golang GC问题的思考"
date: 2021-07-27T23:45:51+08:00
draft: false
original: true
tags: 
  - Golang
---

由于GC复杂，我也没有仔细研究过GC的源码，所以只能站在巨人的肩上学习，如果想了解GC的具体实现请移步文末的参考资料。本文只是记录我在阅读完大佬文章中自己的一些问题与思考，可能有一些不对的地方。欢迎大家一起讨论。

<!--more-->

# 清理阶段，新产生的对象被标记成白色，岂不是会被回收掉？

在[参考文章1](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)留言中也有人提了这个问题

![](/关于GolangGC问题的思考/Untitled.png)

我再将问题描述一下

在标记完成的时候会STW，将状态切换成_GCoff，然后就会进入清理阶段，进入清理阶段的时候，已经结束标记阶段，所以这个时候就没有颜色的区分了(当然也可以说新产生的对象时白色的，因为GC开始的时候默认所有的对象都是白色)。

那新产生的对象，在清理阶段会被回收吗？肯定是不会的，要不然早就崩了，为什么没有被回收呢

我个人理解是这样的，清理阶段，当Goroutine申请新内存的时候就会触发清理，先进行清理再进行对象的分配，这样就没问题了。因为已经清理了，所以后续也不会清理这块内存了，新对象在清理后再分配也就不会有问题了。

# 栈上的黑色对象指向了堆中的白色对象？

在[参考文章1](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)留言中也有人提了这个问题，我就不再重新描述了

![](/关于GolangGC问题的思考/Untitled%201.png)

我还是说下我的理解，首先栈分为3种

- 还没没有扫描的栈
- 扫描中的栈
- 扫描过的栈

如果是扫描中的栈，Groutine是暂停的，也就是栈上的对象不会发生变化，就不再讨论

如果是还没有扫描的栈，因为栈在最开始的时候都被标记成灰色，不管栈上的对象引用怎么变化，最后都会扫描栈，所以也不会有啥问题

如果是扫描过的栈，如果发生栈上的对象引用的变化，由于已经扫描过了，不会再扫描，所以就会有问题。但事实上却没有问题，为什么？

在参考资料[golang 混合写屏障原理深入剖析，这篇文章给你梳理的明明白白！！！](https://mp.weixin.qq.com/s?__biz=Mzg3NTU3OTgxOA==&mid=2247486752&idx=1&sn=2eafb908903e408b85bb9d8a10deb100&chksm=cf3e1de5f84994f3388a0526f9a8b508c28acaee6d887eadd29de77cfee24bb01404a007d57c&scene=178&cur_album_id=1749948750287978500#rd)中说明这个问题的原因

![](/关于GolangGC问题的思考/Untitled%202.png)

问题在图片中描述得比较清楚了，栈上的A想要指向堆中的C，有多种方式

- A.C = new(C)
- A.C = B.C
- A.C = A.X.C

第1种方式，直接在堆中初始化C，然后A.C指向过去；如果是这样，C就不会是白色的，因为现在正在标记阶段，新对象都是黑色的

第2种方式，这种是不可能的，Goroutine1栈中能访问到Goroutine2栈中的对象根本就不可能

第3种方法是A能访问到X，X引用了C，如果是这样，因为Goroutine1已经被扫描过了，A还能访问到X，说明X一定不是白色的，X引用了C，当扫描X的时候C一定不会被标记成白色

综上所述，Goroutine1栈中的A想要指向堆中的白色对象是不可能。

# 删除写屏障到底是把什么对象染成灰色

![](/关于GolangGC问题的思考/Untitled%203.png)

![](/关于GolangGC问题的思考/Untitled%204.png)

![](/关于GolangGC问题的思考/Untitled%205.png)

![](/关于GolangGC问题的思考/Untitled%206.png)

![](/关于GolangGC问题的思考/Untitled%207.png)

![](/关于GolangGC问题的思考/Untitled%208.png)

# 理解混合写屏障一定要先理解删除写屏障

![](/关于GolangGC问题的思考/Untitled%209.png)

很多人都能理解插入写屏障，插入写屏障的流程个人理解如下：

1. STW，扫描栈上的对象，标记成灰色
2. 恢复运行，开始并发标记
3. STW，重新扫描栈，完成标记阶段

由于栈上的对象被当作根对象，栈上的对象发生变更的时候，需要触发插入写屏障，由于栈上的操作频繁，而且要实现栈上的插入写屏障很复杂。所以Golang并没有实现栈上的插入写屏障

但此时可能会出现如下的情况，栈上的黑色对象会指向白色对象。于是需要在标记完成的时候对栈进行了重新扫描，这时需要STW，这个过程大概需要花费10ms~100ms

![](/关于GolangGC问题的思考/Untitled%2010.png)

删除写屏障

1. 删除写屏障也叫基于快照的写屏障方案，必须在起始时，STW 扫描整个栈（注意了，是所有的 goroutine 栈），保证**所有堆上在用的对象**都处于灰色保护下，保证的是弱三色不变式；
2. 由于起始快照的原因，起始也是执行 STW，删除写屏障不适用于栈特别大的场景，栈越大，STW 扫描时间越长，对于现代服务器上的程序来说，栈地址空间都很大，所以删除写屏障都不适用，一般适用于很小的栈内存，比如嵌入式，物联网的一些程序；
3. 并且删除写屏障会导致扫描进度（波面）的后退，所以扫描精度不如插入写屏障；

**思考问题**：我不整机暂停 STW 栈，而是一个栈一个栈的快照，这样也没有 STW 了，是否可以满足要求？（这个就是当前 golang 混合写屏障的时候做的哈，虽然没有 STW 了，但是扫描到某一个具体的栈的时候，还是要暂停这一个 goroutine 的）

不行，纯粹的删除写屏障，起始必须整个栈打快照，要把所有的堆对象都处于灰色保护中才行。

**举例**：如果没有把栈完全扫黑，那么可能出现丢数据，如下：

**初始状态**：

1. A 是 g1 栈的一个对象，g1栈已经扫描完了，并且 C 也是扫黑了的对象；
2. B 是 g2 栈的对象，指向了 C 和 D，g2 完全还没扫描，B 是一个灰色对象，D 是白色对象；

![](/关于GolangGC问题的思考/Untitled%2011.png)

**步骤一**：g2 进行赋值变更，把 C 指向 D 对象，这个时候黑色的 C 就指向了白色的 D（由于是删除屏障，这里是不会触发hook的）

**步骤二**：把 B 指向 C 的引用删除，由于是栈对象操作，不会触发删除写屏障；

![](/关于GolangGC问题的思考/Untitled%2012.png)

步骤三：清理，因为 C 已经是黑色对象了，所以不会再扫描，所以 D 就会被错误的清理掉。

**解决办法有如下**：

方法一：栈上对象也 hook，所有对象赋值（插入，删除）都 hook（这个就不实际了）;

所有的插入，删除如果都 hook ，那么一定都不会有问题，虽然本轮精度很差，但是下轮回收可以回收了。但是还是那句话，栈，寄存器的赋值 hook 是不现实的。

方法二：起始快照整栈扫黑，使得整个堆上的在用对象都处于灰色保护；

整栈扫黑，那么在用的堆上的对象是一定处于灰色堆对象的保护下的，之后配合堆对象删除写屏障就能保证在用对象不丢失。

方法三：加入插入写屏障的逻辑，C 指向 D 的时候，把 D 置灰，这样扫描也没问题。这样就能去掉起始 STW 扫描，从而可以并发，一个一个栈扫描。

**细品下，这不就成了当前在用的混合写屏障了，所以我觉得正确的理解方式应该是：混合写屏障 = 删除写屏障 + 插入写屏障，必须先理解下删除写屏障，你才能理解混合写屏障。**

# 混合写屏障current stack is  grey的 理解

在混合写屏障中许多文章都这么描述

```go
writePointer(slot, ptr):
    shade(*slot)
    if current stack is grey:
        shade(ptr)
    *slot = ptr
```

current stack is grey该怎么理解

Proposal中是这么描述的

if the current goroutine's stack has not yet been scanned, also shades the reference being installed.

翻译过来就是当前栈还没有被被扫描的时候，需要使用插入写屏障

首先看一下栈已经被扫描的情况

正在被扫描/未被扫描的栈

![](/关于GolangGC问题的思考/Untitled%2013.png)

栈被扫描过的存在上面2种情况

- 栈上以及相关联堆上的对象都已经被标记成黑色
- 栈上的是黑色，堆上部分对象还没有被标记成黑色

这2种情况下即使不使用插入写屏障也不会出现黑色对象指向白色对象的情况

![](/关于GolangGC问题的思考/Untitled%2014.png)

要想让黑色对象指向白色对象有2种途径

- 当前栈中的白色对象（这种情况下就是上图所示，E对象在灰色D对象的保护下，最后还是会被扫描成黑色，所以不会有问题）
- 其他栈引用了当前栈的黑色对象，将该黑色对象指向了白色对象(这个属于栈未被扫描，会正在扫描的情况，后面说这种情况)

接下来是栈正在被扫描或者未被扫描的情况，如下图

![](/关于GolangGC问题的思考/Untitled%2015.png)

![](/关于GolangGC问题的思考/Untitled%2016.png)

![](/关于GolangGC问题的思考/Untitled%2017.png)

![](/关于GolangGC问题的思考/Untitled%2018.png)

如果不开启插入写屏障就会出现黑色对象指向白色对象的情况，所以这种情况下需要开启插入写屏障

# 参考资料

[Go 语言垃圾收集器的实现原理](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)

[Go语言GC实现原理及源码分析 - luozhiyun`s Blog](https://www.luozhiyun.com/archives/475)

[Proposal: Eliminate STW stack re-scanning](https://go.googlesource.com/proposal/+/master/design/17503-eliminate-rescan.md)

[golang 垃圾回收 （一）概述篇](https://mp.weixin.qq.com/s/GYYLLlVWMoI-ls8IgrzndA)

[golang 垃圾回收（二）屏障技术](https://mp.weixin.qq.com/s?__biz=Mzg3NTU3OTgxOA==&mid=2247486744&idx=1&sn=fdd53010c7751029943b8eaeaefeb8da&chksm=cf3e1dddf84994cba288591b179dd0ca231dcb48756046b8e39b307c26f4a3ead8ea87f3f197&scene=178&cur_album_id=1749948750287978500#rd)

[golang 垃圾回收（三）插入写屏障](https://mp.weixin.qq.com/s?__biz=Mzg3NTU3OTgxOA==&mid=2247486746&idx=1&sn=3b08ffd0e0bc01c112623c42e784b940&chksm=cf3e1ddff84994c9f14222547a6b7080a9e35a1efa9bcb90df2363f9c2426072b624cfe84ea7&scene=178&cur_album_id=1749948750287978500#rd)

[golang 垃圾回收 - 删除写屏障](https://mp.weixin.qq.com/s?__biz=Mzg3NTU3OTgxOA==&mid=2247486749&idx=1&sn=dd11d207c5c585c969b9556c17d4df15&chksm=cf3e1dd8f84994ce09e610d8c1d1e97861fda59c01a26cb9abca3206e5425d0b30e9b3bb0ce8&scene=178&cur_album_id=1749948750287978500#rd)

[golang 混合写屏障原理深入剖析，这篇文章给你梳理的明明白白！！！](https://mp.weixin.qq.com/s?__biz=Mzg3NTU3OTgxOA==&mid=2247486752&idx=1&sn=2eafb908903e408b85bb9d8a10deb100&chksm=cf3e1de5f84994f3388a0526f9a8b508c28acaee6d887eadd29de77cfee24bb01404a007d57c&scene=178&cur_album_id=1749948750287978500#rd)

[Golang垃圾回收 屏障技术](https://zhuanlan.zhihu.com/p/74853110)