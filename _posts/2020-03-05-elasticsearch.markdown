---
layout:     post
title:      "ElasticSearch"
subtitle:   ""
date:       2020-03-05 12:00:00
author:     "盈盈冲哥"
header-img: "img/fleabag.jpg"
mathjax: true
catalog: true
tags:
    - 学习
---

11. ElasticSearch

  - 节点

    - Master Node

      - 处理创建、删除索引请求 / 决定分⽚分配到哪个节点
      - 维护并更新Cluster State
    
    - Master Eligible Node

      - 在Master节点出现故障时，参与选主流程，成为Master Node
    
    - Data Node

      - 保存分⽚数据，通过增加Data Node，可以解决数据⽔平扩展
      - 和数据单点的问题
    
    - Coordinating Node

      - 负责将请求转发到正确的节点
      - 处理query AND fetch的结果集
  
  - 分⽚—分布式存储的基⽯

    - Primary Shard

      - 主分⽚在索引创建时指定，默认不能修改

    - Replica Shard
    
      - 数量可以动态调整，提⾼数据的可⽤性
      - 提升查询性能
      - 副本数量过多，会降低写⼊性能，增加磁盘负担
  
  - 倒排索引&正排索引

    倒排索引可以实现从term到document的快速查询，但是没法实现排序，这个时候就需要⽤到正排索引。es是通过doc_value和field_data来实现正排索引的。

    doc_value是在lucene构建倒排索引时，根据原始mapping配置判断是否额外建⽴⼀个有序的正排索引（基于documentId—>field value的映射列表）

    doc_value默认对所有字段启⽤，除了analyzed string，针对analyzed string，如果需要排序，需要开启field_data。

    doc_value本质上是⼀个序列化的列式存储，这个结构⾮常适⽤于聚合、排序、脚本等操作，也⾮常便于压缩。和倒排索引⼀样，doc_value也会序列化到磁盘，这样对性能和扩展性有很⼤帮助。

  - indexing的过程

    1. 将document写⼊index buffer，同时记录trans_log；
    2. 将index buffer写⼊segment，但不执⾏fsync，也不清理trans_log；
      - 这个过程叫做refresh，默认1s⼀次；
      - Index buffer占满时，也会触发refresh；
    3. 调⽤fsync，将缓存中的segments写⼊磁盘，清空trans_log；
      - 这个动作叫做flush，默认30min⼀次；
      - trans_log满（默认512M），也会触发flush；
  
  - 索引定义

    字段类型—>是否需要搜索—>是否需要分词—>是否需要聚合及排序

    字段类型：Text vs keyword

    - Text
      - ⽤于⽂本分析，⽂本会被Analyzer分词
      - 默认不⽀持聚合分析和排序，需要设置fielddata=true

    - Keyword
      - 适⽤于不需要分词的场景，精确匹配查询效率最⾼
      - ⽀持sorting和aggregations

  - 关于分⻚

    es提供三种分⻚查询的机制
    - from / size（es默认限定10000个⽂档）
    - search after（不⽀持跳⻚）
    - scroll API（遍历期间，更新的数据不会拿到）

    使⽤场景及建议
    - 需要全部⽂档，例如导出数据，允许⼀定误差（⽐如导出期间新增的数据）——scroll
    - 分⻚
      - 数据量⼩，且只查询最新的数据，使⽤from / size
      - 深度分⻚，使⽤search after，禁⽤跳⻚

  - 常⽤的优化设计建议

    - 基于时间序列拆分索引
    - 节点职责分离
      - eligible master node：使⽤低配的CPU，RAM和磁盘
      - data node：使⽤⾼配的CPU，RAM和磁盘
    - 写⼊性能优化
      - 客户端通过bulk API批量写
      - 服务端降低IO开销，如提⾼refresh interval
      - 降低CPU和存储开销，如减少不必要的索引 / 分词 / doc_value
      - 极致的性能追求（牺牲可靠性）
        - 临时将副本修改为0（针对初始化导⼊数据场景）
        - 修改trans_log配置（不每次落盘，60s fsync⼀次）
    - 定期做segment merge，segments过多，term_index会占⽤较⼤内存，影响集群性能

  - 学妹的整理

    特点：分布式，水平扩展，REST API

    功能：海量数据的存储管理以及几乎实时的搜索和数据分析

    分布式架构：存储水平扩容，PB级数据，一台机器可以多个es节点；可用性：部分节点宕机集群不受影响

    节点：至少三个master节点能够保证系统的正常运行状态（高可用性），主节点宕机剩下两个自主选举新的主节点，分为master&data节点，master对磁盘内存要求不高，不存数据，data反之，因此根据不同功能配置不同的机器；data node 通过分片来实现备份，即主分片和副本分片，如下“分片”所述

    master：处理创建，删除索引/决定分片的分配；维护更新cluster

    data：保存分片数据，增加此类型node可以水平扩展

    coordinating：请求转发到节点，处理query&fetch结果

    分片：一个分片是一个lucene索引，业务类不要超过20g，日志类不超过50g；太大了恢复起来会比较慢，保持合适大小能够提高集群的效率；主分片不能修改；副本分片数量可以动态调整，提升查询的性能，过多则影响磁盘写入性能；主分片和副本分布在不同节点，如果主分片宕机则内容会分到副本分片上

    索引：对term_dictionary做一个term_index，然后找到posting list再找到各个document，通过refresh来达到近实时的查询效率，refresh之前是读不到数据的，refresh的时间间隔即其近实时的效率（那要是refresh的时间间隔调短会怎样？refresh时间间隔变长为什么能够优化：提高interval可以降低IO的开销，因为索引是根据segment去写入的，会涉及到IO操作，如果很频繁则会比较消耗IO；refresh除了固定的时间间隔还有可以根据indexbox的大小，满足任意一个都会进行refresh。

    倒排索引：实现term到document的快速查询，但是没法排序，具体信息可参考↓

    官方：https://www.elastic.co/guide/cn/elasticsearch/guide/cn/inverted-index.html

    https://blog.csdn.net/hu948162999/article/details/81386384

    https://www.cnblogs.com/cjsblog/p/10327673.html

    正排索引：基于documentId->field value的映射列表，实现排序；doc_value是构建倒排时候，根据原始mapping配置判断是否有有序的正排索引，默认对所有字段启用，本质是一个序列化的列式存储；默认doc_value在磁盘当中，但是如果有词频之类的会加载到内存中

    query过程：query & fetch ，document在磁盘中的存储是按照块存储的，然后有一个文档位置的索引；从内存找到磁盘中对应的倒排索引的位置，然后在磁盘中找到倒排表->document ID->document block index->document block

    indexing：document->index buffer(with trans_log recorded); index buffer->segment (default refresh every one second); 

    内存分配：在JVM heap中地下有fieldData和其他缓存，可以设置断路器，节点中es的存在jvm heap，lucene的存在外部的heap中，会加载到jvm的缓存中提高之后的查询效率

    索引定义：搜索/分词/聚合&排序：text or keyword  ； 嵌套对象 & 父子结构；索引设计避免过多字段（不移维护，mapping存在cluster state过大集群性能受影响，reindex成本高）

    分页：from/size(<=10000 documents)数据量小且要最IN; search after(no jump)深度分页，禁止跳页； scroll API(no updating during iterating)需要全部文档，允许一定偏差

    优化：基于时间序列拆分索引；节点职责分离；写入优化；定期进行segment merge

  - ES和MySQL的区别

    1. 一个ES集群可以包含多个**索引（数据库）**，每个索引又包含了很多**类型（表）**，类型中包含了很多**文档（行）**，每个文档使用 JSON 格式存储数据，包含了很多**字段（列）**。

    2. ES**分布式搜索（一个索引包含多个分片，分片包括主分片和副本分片；选举master结点）**，传统数据库遍历式搜索

    3. ES采用**倒排索引（单词到文档）**，传统数据库采用B+树索引

    4. ES**没有用户验证和权限控制**

    5. ES**没有事务**的概念，不支持回滚，误删不能恢复
  
  - elasticsearch的倒排索引是什么

    传统的我们的检索是通过文章，逐个遍历找到对应关键词的位置。而倒排索引，是通过分词策略，形成了词和文章的映射关系表，这种词典+映射表即为倒排索引。有了倒排索引，就能实现o（1）时间复杂度 的效率检索文章了，极大的提高了检索效率。

    倒排索引的底层实现是基于：**FST（Finite State Transducer）数据结构**。lucene从4+版本后开始大量使用的数据结构是FST。FST有两个优点：

    1）**空间占用小。通过对词典中单词前缀和后缀的重复利用，压缩了存储空间；**

    2）**查询速度快。O(len(str))的查询时间复杂度。**
  
  - elasticsearch是如何实现master选举的

    选举流程大致描述如下：

    第一步：确认候选主节点数达标，elasticsearch.yml设置的值discovery.zen.minimum_master_nodes；

    第二步：比较：先判定是否具备master资格，具备候选主节点资格的优先返回；若两节点都为候选主节点，则id小的值会主节点。注意这里的id为string类型。
  
  - Elasticsearch写入数据的工作原理, 查询数据的工作原理, 搜索数据的工作原理分别是什么?

    **Elasticsearch写入数据的工作原理**

    - **客户端发送请求到任意一个协调节点(Coordinating Node), 然后协调节点将请求转发给master节点.**
    - **master节点对document进行路由, 将document写入主分片.**
    - **document写入主分片后, 将数据同步到副本分片.**
    - 主分片和副本分片都写入成功后, 返回响应结果给客户端.
    
    (1) document写入主分片的详细过程

    - **Document写入Index Buffer(ES进程缓冲), 同时将写命令记录到Transaction Log.**
    - **每隔1秒或Index Buffer空间被占满后, Index Buffer中的数据被写入新的Segment中, 并进入OS Cache, 这个过程叫Refresh.** (此时倒排索引已创建, 存在OS Cache中, 数据可被搜索)
    - 重复前面两个步骤.
    - 每隔30分钟或Transaction Log占满后, 先进行Refresh操作, 然后将OS Cache中的Segment刷入磁盘, 这个过程叫Flush.
    - 删除旧Transaction Log, 创建一个新的Transaction Log.
    - ES定期合并磁盘中的Segment File, 同时清除那些被标记为delete的文档.
    
    (2) **ES被称为近实时(Near Realtime)的原因**

    **从上文"document写入主分片的详细过程"中可以知道, Refresh操作每秒执行一次, 只有执行Refresh操作之后, 倒排索引才会被创建, 数据才能被搜索, 这就是ES被称为近实时的原因.**

    (3) ES存在数据丢失的问题

    从上文"document写入主分片的详细过程"中可以知道, document写入Index Buffer的同时会将写命令记录到Transaction Log, 目的是如果数据落盘之前机器宕机了, 可以从Transaction Log中恢复数据. 但在旧版本中Transaction Log不是默认落盘的, 它会先写入OS Cache中, 每隔5s才会被刷入磁盘, 所以如果在Transaction Log落盘前机器宕机了, 数据就完全丢失了.

    在新版本7.x中, Transaction Log是默认落盘的, 也就不会有数据丢失的问题. (index.translog.durability, index.translog.sync_interval)

    **Elasticsearch查询数据的工作原理(Get查询)**

    - **客户端发送请求到任意一个协调节点(Coordinating Node).**
    - **协调节点根据id进行路由, 找到对应的分片.**
    - **根据round-robin随机轮询算法, 在主分片和其他副本分片中随机选择一个, 进行查询.**
    - 将对应的document返回给协调节点.
    - 协调节点返回document给客户端.
    
    **Elasticsearch搜索数据的工作原理**

    - **客户端发送请求到任意一个协调节点(Coordinating Node).**
    - **协调节点将搜索请求转发给所有分片(主分片和副本分片采用随机轮询算法选一个)**
    - **每个分片将自己的搜素结果(这里只有id)返回给协调节点**
    - **协调节点对搜索结果进行合并, 排序, 分页等操作, 得出最终结果.**
    - **协调节点根据最终结果的id去各个分片上拉取document, 返回给客户端.**
  
  - Elasticsearch之四种搜索类型和搜索原理

    **Elasticsearch在搜索过程中存在以下几个问题：**

    第一、 **数量问题**。 比如， 用户需要搜索"衣服"， 要求返回符合条件的前 10 条。 但在 5个分片中， 可能都存储着衣服相关的数据。 所以 ES 会向这 5 个分片都发出查询请求， 并且要求每个分片都返回符合条件的 10 条记录。当ES得到返回的结果后，进行整体排序，然后取最符合条件的前10条返给用户。 这种情况， ES 中 5 个 shard 最多会收到 10*5=50条记录， 这样返回给用户的结果数量会多于用户请求的数量。
    
    第二、 **排名问题**。 上面说的搜索， 每个分片计算符合条件的前 10 条数据都是基于自己分片的数据进行打分计算的。计算分值使用的词频和文档频率等信息都是基于自己分片的数据进行的， 而 ES 进行整体排名是基于每个分片计算后的分值进行排序的(相当于打分依据就不一样， 最终对这些数据统一排名的时候就不准确了)， 这就可能会导致排名不准确的问题。如果我们想更精确的控制排序， 应该先将计算排序和排名相关的信息（ 词频和文档频率等打分依据） 从 5 个分片收集上来， 进行统一计算， 然后使用整体的词频和文档频率为每个分片中的数据进行打分， 这样打分依据就一样了。

    **Elasticsearch的搜索类型（SearchType类型）**

    1、 query and fetch
    
    向索引的所有分片 （ shard）都发出查询请求， 各分片返回的时候把元素文档 （ document）和计算后的排名信息一起返回。
    
    这种搜索方式是最快的。 因为相比下面的几种搜索方式， 这种查询方法只需要去 shard查询一次。 但是各个 shard 返回的结果的数量之和可能是用户要求的 size 的 n 倍。
    
    优点：这种搜索方式是最快的。因为相比后面的几种es的搜索方式，这种查询方法只需要去shard查询一次。
    
    缺点：返回的数据量不准确， 可能返回(N*分片数量)的数据并且数据排名也不准确，同时各个shard返回的结果的数量之和可能是用户要求的size的n倍。

    2、 query then fetch（ es 默认的搜索方式）

    如果你搜索时， 没有指定搜索方式， 就是使用的这种搜索方式。 这种搜索方式， 大概分两个步骤：

    第一步， 先向所有的 shard 发出请求， 各分片只返回文档 id(注意， 不包括文档 document)和排名相关的信息(也就是文档对应的分值)， 
    然后按照各分片返回的文档的分数进行重新排序和排名， 取前 size 个文档。
　　
    第二步， 根据文档 id 去相关的 shard 取 document。 这种方式返回的 document 数量与用户要求的大小是相等的。
    
    优点：返回的数据量是准确的。
　　
    缺点：性能一般，并且数据排名不准确。

    3、 DFS query and fetch
　　
    这种方式比第一种方式多了一个 DFS 步骤，有这一步，可以更精确控制搜索打分和排名。也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作。

    优点：数据排名准确

    缺点：性能一般；返回的数据量不准确， 可能返回(N*分片数量)的数据

    4、 DFS query then fetch
　　
    比第 2 种方式多了一个 DFS 步骤。也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作。

　　优点：返回的数据量是准确的，数据排名准确

　　缺点：性能最差【 这个最差只是表示在这四种查询方式中性能最慢， 也不至于不能忍受，如果对查询性能要求不是非常高， 而对查询准确度要求比较高的时候可以考虑这个】

　　DFS 是一个什么样的过程？

　　从 es 的官方网站我们可以发现， DFS 其实就是在进行真正的查询之前， 先把各个分片的词频率和文档频率收集一下， 然后进行词搜索的时候， 各分片依据全局的词频率和文档频率进行搜索和排名。 显然如果使用 DFS_QUERY_THEN_FETCH 这种查询方式， 5效率是最低的，因为一个搜索， 可能要请求 3 次分片。 但， 使用 DFS 方法， 搜索精度是最高的。
  
  - Elasticsearch在数据量很大的情况下（数十亿级别）如何提高搜索性能?

    (1) 善于利用OS Cache

    如果Elasticsearch的每次搜索都要落盘, 那搜索性能肯定很差, 将达到秒级. 但**如果ES集群中的数据量等于OS Cache的容量, 那每次搜索都会直接走OS Cache, 这样性能就会很高, 达到毫秒级.**

    ES集群中的数据量最好不要超过OS Cache的容量, 最低要求也不能超过OS Cache的两倍. 比如我们ES集群有3台机器, 每台机器64G内存, 为每个节点的ES JVM Heap分配32G内存, 最终集群的OS Cache为 32G * 3 = 96G内存. 我们ES集群中的数据量最优情况是不超过96G, 最低要求的情况是不超过192G.

    (2) 数据建模

    从上文"善于利用OS Cache"中我们知道, 我们要保证ES集群中的数据量不超过OS Cache的容量, 那么我们在数据建模的时候就要注意两点:

    - **不要将MySQL表中的所有字段都写入ES, 只写入一部分会被搜索的字段.**
    - **对于MySQL中具有关联关系的表, 我们直接将关联字段写入ES中或在应用端处理关联关系, 禁止在ES中处理关联关系.**

    (3) 数据预热

    如果我们无法做到让ES集群中的数据量不超过OS Cache的容量, **那我们做一个缓存预热子系统, 定时搜索"热数据", 让其进入OS Cache.**

    (4) 冷热分离

    在数据预热的基础上我们还可以进行冷热数据分离, 比如我们有6台机器, 创建两个索引, 每个索引3个分片, **一个索引放热数据, 一个索引放冷数据**. 热数据量一般只占总数据量的10%, 这样我们就能保证热数据都在OS Cache中. 而冷数据虽然占总数据的90%, 但却只有10%的用户访问, 性能差点是可以接受的.

    (5) 分页性能优化

    **深度分页的性能是很差的, 我们要防止出现深度分页的情况, 用滚动翻页来代替深度分页.**

    - search after
    - scroll

  - ES分页

    Elasticsearch分页api有三种:

    - from/size（深度分页。当我们进行一个分页查询from=990, size=10时:首先在每个分片上先都获取 1000 个文档，通过 Coordinating Node 聚合所有结果，再通过排序选取前 1000 个文档。页数越深，占用内存就越多。）
    - search after（用来实时的获取下一页文档信息, 它不支持指定页数(from), 只能往下翻.）
    - scroll（用来处理大数据量的分页搜索, 不适用于实时的分页搜索. scroll分页搜索每次都会创建一个快照, 新数据写入只能影响到后续的分页搜索.）

  - Elasticsearch生产集群的部署架构是什么？每个索引的数据量大概有多少？每个索引大概有多少个分片？

    (1) ES生产集群我们部署了5台机器, 每台机器是6核64G的, 集群总内存是320G.

    (2) 我们ES集群的日增量数据大概是2000万条, 500MB左右; 每月增量数据大概是6亿条, 15G左右. 目前系统已经运行了几个月, 现在ES集群里数据总量大概是100G左右.

    (3) 目前线上有5个索引（这个结合你们自己业务来, 看看自己有哪些数据可以放ES的）, 每个索引的数据量大概是20G, 所以这个数据量之内, 我们每个索引分配的是8个shard, 比默认的5个shard多了3个shard.
  
  - 常用命令

    创建索引模板 PUT _template/...
      settings: 分片数、副本数
      mappings: properties 字段类型
    创建索引 PUT ...
    查询 GET _search
      query: match_all, term (精确查找), match(模糊匹配), range (范围查找), [bool: filter, should (或), must (与)]
    java中的写法：jsonbuilder，client发送请求；类似JPA

#### ElasticSearch & Solr

- 常见问题

    - lucena引擎，一次查询的过程
    - 近实时刷新的原理
    - 分页怎么使用的，各个查询的原理，为什么有不同

    - 底层原理
    - 常用的命令
    - 倒排索引
    - es的打分机制，tfidf
    - 分页命令、scroll命令
    - 还有一些命令的区别，match啊什么的条件命令(https://www.cnblogs.com/caoweixiong/p/11792049.html)
    - 怎么在java里使用的
    - 跟关系型 数据库的差别， 这些mysql不行嘛
    - es优化
    - es基本的查询原理，一次查询的过程
    - es高并发会不会数据冲突(https://blog.csdn.net/r_p_j/article/details/78388783)
    - es为什么是近实时的
    - 具体使用没问，就是几个命令的区别，这还是美团面试官问的，阿里只问原理
  
  - 目录

    - ES和MySQL的区别
    - ES的原理，lucene原理
    - 分布式架构的原理
    - 写入（近实时刷新）、查询、搜索的过程（搜索类型和原理）
    - 如何优化提高搜索性能

  - ES和MySQL的区别

    1. 一个ES集群可以包含多个**索引（数据库）**，每个索引又包含了很多**类型（表）**，类型中包含了很多**文档（行）**，每个文档使用 JSON 格式存储数据，包含了很多**字段（列）**。

    2. ES**分布式搜索（一个索引包含多个分片，分片包括主分片和副本分片；选举master结点）**，传统数据库遍历式搜索

    3. ES采用**倒排索引（单词到文档）**，传统数据库采用B+树索引

    4. ES**没有用户验证和权限控制**

    5. ES**没有事务**的概念，不支持回滚，误删不能恢复
  
  - elasticsearch的倒排索引是什么

    传统的我们的检索是通过文章，逐个遍历找到对应关键词的位置。而倒排索引，是通过分词策略，形成了词和文章的映射关系表，这种词典+映射表即为倒排索引。有了倒排索引，就能实现o（1）时间复杂度 的效率检索文章了，极大的提高了检索效率。

    倒排索引的底层实现是基于：**FST（Finite State Transducer）数据结构**。lucene从4+版本后开始大量使用的数据结构是FST。FST有两个优点：

    1）**空间占用小。通过对词典中单词前缀和后缀的重复利用，压缩了存储空间；**

    2）**查询速度快。O(len(str))的查询时间复杂度。**

    > 列式存储DocValues：分组统计
  
  - elasticsearch是如何实现master选举的

    选举流程大致描述如下：

    第一步：确认候选主节点数达标，elasticsearch.yml设置的值discovery.zen.minimum_master_nodes；

    第二步：比较：先判定是否具备master资格，具备候选主节点资格的优先返回；若两节点都为候选主节点，则id小的值会主节点。注意这里的id为string类型。
  
  - Elasticsearch的分布式架构原理是什么?

    Elasticsearch是分布式的, 在多台机器上启动ES进程实例, 组成一个ES集群. **ES集群中的节点会选举出一个master节点来管理集群, 比如负责索引的创建与删除, 负责主分片与副本分片的身份切换等等.** ES集群的master节点选举算法如下:

    - 查找当前活跃的Master节点.
    - 如果存在活跃的Master节点, 选择其中nodeId最小的那个作为Master节点. (脑裂问题可能导致多个Master节点)
    - 如果不存在活跃的Master节点, 则获取集群中活跃的Master Eligible Nodes, 进行投票选举.
    - 如果集群中活跃的候选节点数超过一半(正常情况下候选节点数的一半), 则选择clusterStateVersion(集群状态版本)最新的那个作为Master节点. 如果clusterStateVersion相同, 则选择nodeId最小的那个作为Master节点.
    - 如果如果集群中活跃的候选节点数没有超过一半, 则无法选举.
    
    **ES存储数据的基本单位是索引, 每个索引都会被拆分为多个分片, 每个分片分布在不同节点上. 同时为了保证可用性, 每个分片都会设置副本, 主分片提供读写服务, 副本分片提供读服务.** 当主分片所在的节点宕机后, master节点会从副本分片中选出一个作为主分片, 然后当宕机的节点修复后, master节点会将缺失的副本分片分配过去, 同步数据后, 集群恢复正常.
  
  - 详细描述一下Elasticsearch索引文档的过程

    这里的索引文档应该理解为文档写入ES，创建索引的过程。
    
    第一步：客户写集群某节点写入数据，发送请求。（如果没有指定路由/协调节点，请求的节点扮演路由节点 的角色。）

    第二步：节点1接受到请求后，使用文档_id来确定文档属于分片0。请求会被转到另外的节点，假定节点3。因此分片0的主分片分配到节点3上。

    第三步：节点3在主分片上执行写操作，如果成功，则将请求并行转发到节点1和节点2的副本分片上，等待结果返回。所有的副本分片都报告成功，节点3将向协调节点（节点1）报告成功，节点1向请求客户端报告写入成功。

    如果面试官再问：第二步中的文档获取分片的过程？回答：借助路由算法获取，路由算法就是根据路由和文档id计算目标的分片id的过程。

    `1shard = hash(_routing) % (num_of_primary_shards)`
  
  - 详细描述一下Elasticsearch搜索的过程？

    **搜索拆解为“query then fetch” 两个阶段。query阶段的目的：定位到位置，但不取。**步骤拆解如下：

    1）假设一个索引数据有5主+1副本 共10分片，一次请求会命中（主或者副本分片中）的一个。

    2）每个分片在本地进行查询，结果返回到本地有序的优先队列中。

    3）第2）步骤的结果发送到协调节点，协调节点产生一个全局的排序列表。

    **fetch阶段的目的：取数据。路由节点获取所有文档，返回给客户端。**

  - Elasticsearch写入数据的工作原理, 查询数据的工作原理, 搜索数据的工作原理分别是什么?

    **Elasticsearch写入数据的工作原理**

    - **客户端发送请求到任意一个协调节点(Coordinating Node), 然后协调节点将请求转发给master节点.**
    - **master节点对document进行路由, 将document写入主分片.**
    - **document写入主分片后, 将数据同步到副本分片.**
    - 主分片和副本分片都写入成功后, 返回响应结果给客户端.
    
    (1) document写入主分片的详细过程

    - **Document写入Index Buffer(ES进程缓冲), 同时将写命令记录到Transaction Log.**
    - **每隔1秒或Index Buffer空间被占满后, Index Buffer中的数据被写入新的Segment中, 并进入OS Cache, 这个过程叫Refresh.** (此时倒排索引已创建, 存在OS Cache中, 数据可被搜索)
    - 重复前面两个步骤.
    - 每隔30分钟或Transaction Log占满后, 先进行Refresh操作, 然后将OS Cache中的Segment刷入磁盘, 这个过程叫Flush.
    - 删除旧Transaction Log, 创建一个新的Transaction Log.
    - ES定期合并磁盘中的Segment File, 同时清除那些被标记为delete的文档.
    
    (2) **ES被称为近实时(Near Realtime)的原因**

    **从上文"document写入主分片的详细过程"中可以知道, Refresh操作每秒执行一次, 只有执行Refresh操作之后, 倒排索引才会被创建, 数据才能被搜索, 这就是ES被称为近实时的原因.**

    (3) ES存在数据丢失的问题

    从上文"document写入主分片的详细过程"中可以知道, document写入Index Buffer的同时会将写命令记录到Transaction Log, 目的是如果数据落盘之前机器宕机了, 可以从Transaction Log中恢复数据. 但在旧版本中Transaction Log不是默认落盘的, 它会先写入OS Cache中, 每隔5s才会被刷入磁盘, 所以如果在Transaction Log落盘前机器宕机了, 数据就完全丢失了.

    在新版本7.x中, Transaction Log是默认落盘的, 也就不会有数据丢失的问题. (index.translog.durability, index.translog.sync_interval)

    **Elasticsearch查询数据的工作原理(Get查询)**

    - **客户端发送请求到任意一个协调节点(Coordinating Node).**
    - **协调节点根据id进行路由, 找到对应的分片.**
    - **根据round-robin随机轮询算法, 在主分片和其他副本分片中随机选择一个, 进行查询.**
    - 将对应的document返回给协调节点.
    - 协调节点返回document给客户端.
    
    **Elasticsearch搜索数据的工作原理**

    - **客户端发送请求到任意一个协调节点(Coordinating Node).**
    - **协调节点将搜索请求转发给所有分片(主分片和副本分片采用随机轮询算法选一个)**
    - **每个分片将自己的搜素结果(这里只有id)返回给协调节点**
    - **协调节点对搜索结果进行合并, 排序, 分页等操作, 得出最终结果.**
    - **协调节点根据最终结果的id去各个分片上拉取document, 返回给客户端.**
  
  - Elasticsearch之四种搜索类型和搜索原理

    **Elasticsearch在搜索过程中存在以下几个问题：**

    第一、 **数量问题**。 比如， 用户需要搜索"衣服"， 要求返回符合条件的前 10 条。 但在 5个分片中， 可能都存储着衣服相关的数据。 所以 ES 会向这 5 个分片都发出查询请求， 并且要求每个分片都返回符合条件的 10 条记录。当ES得到返回的结果后，进行整体排序，然后取最符合条件的前10条返给用户。 这种情况， ES 中 5 个 shard 最多会收到 10*5=50条记录， 这样返回给用户的结果数量会多于用户请求的数量。
    
    第二、 **排名问题**。 上面说的搜索， 每个分片计算符合条件的前 10 条数据都是基于自己分片的数据进行打分计算的。计算分值使用的词频和文档频率等信息都是基于自己分片的数据进行的， 而 ES 进行整体排名是基于每个分片计算后的分值进行排序的(相当于打分依据就不一样， 最终对这些数据统一排名的时候就不准确了)， 这就可能会导致排名不准确的问题。如果我们想更精确的控制排序， 应该先将计算排序和排名相关的信息（ 词频和文档频率等打分依据） 从 5 个分片收集上来， 进行统一计算， 然后使用整体的词频和文档频率为每个分片中的数据进行打分， 这样打分依据就一样了。

    **Elasticsearch的搜索类型（SearchType类型）**

    1、 query and fetch
    
    向索引的所有分片 （ shard）都发出查询请求， 各分片返回的时候把元素文档 （ document）和计算后的排名信息一起返回。
    
    这种搜索方式是最快的。 因为相比下面的几种搜索方式， 这种查询方法只需要去 shard查询一次。 但是各个 shard 返回的结果的数量之和可能是用户要求的 size 的 n 倍。
    
    优点：这种搜索方式是最快的。因为相比后面的几种es的搜索方式，这种查询方法只需要去shard查询一次。
    
    缺点：返回的数据量不准确， 可能返回(N*分片数量)的数据并且数据排名也不准确，同时各个shard返回的结果的数量之和可能是用户要求的size的n倍。

    2、 query then fetch（ es 默认的搜索方式）

    如果你搜索时， 没有指定搜索方式， 就是使用的这种搜索方式。 这种搜索方式， 大概分两个步骤：

    第一步， 先向所有的 shard 发出请求， 各分片只返回文档 id(注意， 不包括文档 document)和排名相关的信息(也就是文档对应的分值)， 
    然后按照各分片返回的文档的分数进行重新排序和排名， 取前 size 个文档。
　　
    第二步， 根据文档 id 去相关的 shard 取 document。 这种方式返回的 document 数量与用户要求的大小是相等的。
    
    优点：返回的数据量是准确的。
　　
    缺点：性能一般，并且数据排名不准确。

    3、 DFS query and fetch
　　
    这种方式比第一种方式多了一个 DFS 步骤，有这一步，可以更精确控制搜索打分和排名。也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作。

    优点：数据排名准确

    缺点：性能一般；返回的数据量不准确， 可能返回(N*分片数量)的数据

    4、 DFS query then fetch
　　
    比第 2 种方式多了一个 DFS 步骤。也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作。

　　优点：返回的数据量是准确的，数据排名准确

　　缺点：性能最差【 这个最差只是表示在这四种查询方式中性能最慢， 也不至于不能忍受，如果对查询性能要求不是非常高， 而对查询准确度要求比较高的时候可以考虑这个】

　　DFS 是一个什么样的过程？

　　从 es 的官方网站我们可以发现， DFS 其实就是在进行真正的查询之前， 先把各个分片的词频率和文档频率收集一下， 然后进行词搜索的时候， 各分片依据全局的词频率和文档频率进行搜索和排名。 显然如果使用 DFS_QUERY_THEN_FETCH 这种查询方式， 5效率是最低的，因为一个搜索， 可能要请求 3 次分片。 但， 使用 DFS 方法， 搜索精度是最高的。

  - Elasticsearch在数据量很大的情况下（数十亿级别）如何提高搜索性能?

    (1) 善于利用OS Cache

    如果Elasticsearch的每次搜索都要落盘, 那搜索性能肯定很差, 将达到秒级. 但**如果ES集群中的数据量等于OS Cache的容量, 那每次搜索都会直接走OS Cache, 这样性能就会很高, 达到毫秒级.**

    ES集群中的数据量最好不要超过OS Cache的容量, 最低要求也不能超过OS Cache的两倍. 比如我们ES集群有3台机器, 每台机器64G内存, 为每个节点的ES JVM Heap分配32G内存, 最终集群的OS Cache为 32G * 3 = 96G内存. 我们ES集群中的数据量最优情况是不超过96G, 最低要求的情况是不超过192G.

    (2) 数据建模

    从上文"善于利用OS Cache"中我们知道, 我们要保证ES集群中的数据量不超过OS Cache的容量, 那么我们在数据建模的时候就要注意两点:

    - **不要将MySQL表中的所有字段都写入ES, 只写入一部分会被搜索的字段.**
    - **对于MySQL中具有关联关系的表, 我们直接将关联字段写入ES中或在应用端处理关联关系, 禁止在ES中处理关联关系.**

    (3) 数据预热

    如果我们无法做到让ES集群中的数据量不超过OS Cache的容量, **那我们做一个缓存预热子系统, 定时搜索"热数据", 让其进入OS Cache.**

    (4) 冷热分离

    在数据预热的基础上我们还可以进行冷热数据分离, 比如我们有6台机器, 创建两个索引, 每个索引3个分片, **一个索引放热数据, 一个索引放冷数据**. 热数据量一般只占总数据量的10%, 这样我们就能保证热数据都在OS Cache中. 而冷数据虽然占总数据的90%, 但却只有10%的用户访问, 性能差点是可以接受的.

    (5) 分页性能优化

    **深度分页的性能是很差的, 我们要防止出现深度分页的情况, 用滚动翻页来代替深度分页.**

    - search after
    - scroll

  - ES分页

    Elasticsearch分页api有三种:

    - from/size（深度分页。当我们进行一个分页查询from=990, size=10时:首先在每个分片上先都获取 1000 个文档，通过 Coordinating Node 聚合所有结果，再通过排序选取前 1000 个文档。页数越深，占用内存就越多。）
    - search after（用来实时的获取下一页文档信息, 它不支持指定页数(from), 只能往下翻.）
    - scroll（用来处理大数据量的分页搜索, 不适用于实时的分页搜索. scroll分页搜索每次都会创建一个快照, 新数据写入只能影响到后续的分页搜索.）

  - Elasticsearch生产集群的部署架构是什么？每个索引的数据量大概有多少？每个索引大概有多少个分片？

    (1) ES生产集群我们部署了5台机器, 每台机器是6核64G的, 集群总内存是320G.

    (2) 我们ES集群的日增量数据大概是2000万条, 500MB左右; 每月增量数据大概是6亿条, 15G左右. 目前系统已经运行了几个月, 现在ES集群里数据总量大概是100G左右.

    (3) 目前线上有5个索引（这个结合你们自己业务来, 看看自己有哪些数据可以放ES的）, 每个索引的数据量大概是20G, 所以这个数据量之内, 我们每个索引分配的是8个shard, 比默认的5个shard多了3个shard.

  > [https://www.cnblogs.com/jajian/p/9801154.html](https://www.cnblogs.com/jajian/p/9801154.html)
  
  全文搜索引擎的索引建立都是根据**倒排索引**的方式生成索引。

  > [https://blog.csdn.net/playgrrrrr/article/details/79008124](https://blog.csdn.net/playgrrrrr/article/details/79008124)

  ES和MySQL的区别

  > [https://juejin.im/entry/5c46d7c2e51d4551df6f2338](https://juejin.im/entry/5c46d7c2e51d4551df6f2338)

  ES面试题

  > [https://developer.51cto.com/art/201904/594615.htm](https://developer.51cto.com/art/201904/594615.htm)

  ES原理

  > [https://www.cnblogs.com/sessionbest/articles/8689030.html](https://www.cnblogs.com/sessionbest/articles/8689030.html)

  Lucene原理

  > [https://blog.csdn.net/litianxiang_kaola/article/details/104147340](https://blog.csdn.net/litianxiang_kaola/article/details/104147340)

  Elasticsearch面试必知必会

  > [https://blog.csdn.net/wangyunpeng0319/article/details/78218332](https://blog.csdn.net/wangyunpeng0319/article/details/78218332)

  Elasticsearch之四种查询类型和搜索原理

  > [https://blog.csdn.net/zkyfcx/article/details/79998197](https://blog.csdn.net/zkyfcx/article/details/79998197)

  ElasticSearch底层原理浅析

  > [https://blog.csdn.net/litianxiang_kaola/article/details/103808522](https://blog.csdn.net/litianxiang_kaola/article/details/103808522)

  ES分页

  > [https://blog.csdn.net/chen_2890/article/details/83895646](https://blog.csdn.net/chen_2890/article/details/83895646)

  SpringBoot整合Elasticsearch

#### 分词算法

> [https://zhuanlan.zhihu.com/p/33261835](https://zhuanlan.zhihu.com/p/33261835)

- 基于词表的分词方法
  - 正向最大匹配法(forward maximum matching method, FMM)
  - 逆向最大匹配法(backward maximum matching method, BMM)
  - N-最短路径方法
- 基于统计模型的分词方法
  - 基于N-gram语言模型的分词方法
- 基于序列标注的分词方法
  - 基于HMM的分词方法
  - 基于CRF的分词方法
  - 基于词感知机的分词方法
  - 基于深度学习的端到端的分词方法