---
layout: post
title: "分布式系统漫谈 Google三驾马车：GFS"
category: Reading Notes 
tags: ["读文章", "分布式系统", "GFS"]
---
{% include JB/setup %}

## Google File System

GFS是一个**可扩展**的分布式文件系统，用于大型的，**分布式**，**对大量数据进行访问**的应用。它运行于**廉价的普通硬件**上，提供容错功能。

Google设置**一个主（master）来保存目录和索引信息**，这是为了简化系统结果，提高性能来考虑的，**但是这就会造成主（master）成为单点故障或者瓶颈**。为了消除主（master）的单点故障Google**把每个chunk设置的很大**（64M），这样，由于**代码访问数据的本地性，application端和master的交互会减少**，而**主要数据流量都是Application和chunkserver之间的访问**。

master所有信息都储存在内存里，启动时信息从chunkserver中获取。

**The master stores thress major types of metadata: the file and chunk namespaces,the mapping from files to chunks, and the locations of each chunk's replicas**.

Having a single master vastly simplifies our design and enables the master to make sophisticated chunk placement and replication decisions using global knowledge. However, **we must minimize its involvement in reads and writes so that it does not become a bottleneck**. **Clients never read and write file data through the master**. Instead, **a client asks the master which chunkservers it should contact**. It caches this information for a limited time and interacts with the chunkservers directly for many subsequent operations.

Neither the client nor the chunkserver caches file data. Client caches offer little benefit because most applications stream through huge files or have working sets too large to be cached. Not having them simplifies the client and the overall system by eliminating cache coherence issues. Clients do cache metadata, however. Chunkservers need not cache file data because chunks are stored as local files and so Linux's buffer cache already keeps frequently accessed data in memory.

宿主储存三种元文件，文件和chunk的名字，文件和chunk的关联关系，还有每个chunk的位置。

拥有一个宿主很大程度上简化了设计并且能让宿主利用全局的知识来做必要的chunk处理。但是我们也必须减少宿主在读写操作中的参与度以防止宿主变成瓶颈。clients不从宿主中读写文件，而是只向宿主查询chunkserver的信息。client 缓存这个信息一段时间并且直接对chunkserver进行操作。

client和chunkserver都不缓存文件。**之所以都不缓存文件是因为很多应用所用的文件都太大去让client缓存**，但client缓存元文件。chunkserver也不去缓存文件，因为chunks都储存为本地文件并且linux系统已经将经常使用的数据缓存在内存中了。

GFS对外提供create, delete, open, close, read和write操作，同时还有snapshot和record append.

**Snapshot creates a copy of a file or a directory tree at low cost.**

Record append allows multiple clients to applend data to the same file concurrently while guaranteeing the atomicity of each individual client's append. 能让多个clients在同一文件后同时添加数据只要每个client所要加的东西都在一起，不会client a和client b要加的数据交叉。client a写了一半写起client b，client b写了一半又开始写client a了。
