---
title: "ElasticSearch笔记"
date: 2024-03-06T13:17:10+08:00
draft: false
original: true
categories: 
  - 技术
tags: 
  - ElasticeSearch
---

# ElastiSearch集群的认识

Cluster -> Node -> Shard(Lucene Index) -> Segment

Segment:

* Inverted Index(倒排索引)
* Stored Fields(简单的键值对，默认存储整个JOSN source)
* Document Values(用于排序、聚合、facet，本质是一个列式存储)

Segment是不可变的，随着数据的操作，Segement会合并，删除，压缩等

问题转换：

* suffix -> xiffus *
如果我们想以后缀作为搜索条件，可以为Term做反向处理。

(60.6384, 6.5017) -> u4u8gyykk
对于GEO位置信息，可以将它转换为GEO Hash。

123 -> {1-hundreds, 12-tens, 123}
对于简单的数字，可以为它生成多重形式的Term。

# 索引写入流程

1. 请求来到一个Node，该Node被称为协调节点
2. 协调节点更具文档_id计算Shard，根据路由表分发请求到都相应Primary Shard的Node上
3. 在Shard上写入 In-memory buffer & Translog，还没写入Segment，此时文档还搜索不到
4. refresh过程（默认1s执行一次）: In-memory buffer写入到Segment中，此时还没写入磁盘，还在文件系统的缓存中，此时文档可以被搜索到。清空In-memory buffer，Translog此时还不会清空
5. flush过程(每隔一段时间或者translog文件过大时): 内存缓冲区的文档被写入Segment中，缓冲区清空，生成Commit Point，文件系统缓存刷新到磁盘，删除老的translog
6. Primary Shard写入成功后，将请求发送给Replica Shard，Replica Shard上成功返回后，写入请求完成

写入请求到达Shard后，先写Lucene文件，创建好索引，此时索引还在内存里面，接着去写TransLog，写完TransLog后，刷新TransLog数据到磁盘上，写磁盘成功后，请求返回给用户。这里有几个关键点:

* 一是和数据库不同，数据库是先写CommitLog，然后再写内存，而Elasticsearch是先写内存，最后才写TransLog，一种可能的原因是Lucene的内存写入会有很复杂的逻辑，很容易失败，比如分词，字段长度超过限制等，比较重，为了避免TransLog中有大量无效记录，减少recover的复杂度和提高速度，所以就把写Lucene放在了最前面。
* 二是写Lucene内存后，并不是可被搜索的，需要通过Refresh把内存的对象转成完整的Segment后，然后再次reopen后才能被搜索，一般这个时间设置为1秒钟，导致写入Elasticsearch的文档，最快要1秒钟才可被从搜索到，所以Elasticsearch在搜索方面是NRT（Near Real Time）近实时的系统。
* 三是当Elasticsearch作为NoSQL数据库时，查询方式是GetById，这种查询可以直接从TransLog中查询，这时候就成了RT（Real Time）实时系统。四是每隔一段比较长的时间，比如30分钟后，Lucene会把内存中生成的新Segment刷新到磁盘上，刷新后索引文件已经持久化了，历史的TransLog就没用了，会清空掉旧的TransLog。
