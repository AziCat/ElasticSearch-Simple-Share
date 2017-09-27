# ElasticSearch简单使用说明 #

## 目录 ##
* 简介
* 基础入门
    * 安装并运行Elasticsearch
    * 与Elasticsearch交互
    * 面向文档
    * 基本概念
    * 索引创建与删除
    * 文档简单的CURD操作
* 深入了解
    * 自定义配置
    * 集群内的原理
    * 映射和分析
    * 排序与相关性
* 实际使用中的Q&A
    * 检索包含中文的关键字时返回结果不准确
        * 中文分词器
        * 字典
        * 中文参数预处理
    * 索引重建
        * Reindex API
        * 索引别名实现零停机
    * 检索关键字包含特殊字符
        * 保留字消义
    * 检索关键字个数太多（https://github.com/elastic/elasticsearch/issues/482）
    * 权限控制&监控（X-Pack Kibana）
* 参考（~~Copy~~）内容
## ##

## 简介 ##

`ElasticSearch`是一个基于`Lucene`(路（第一声）森（第三声）)的搜索服务器。
它提供了一个`分布式`多用户能力的全文搜索引擎，基于RESTful web(**Delete、Post、Get、Put**)接口。
Elasticsearch是用**Java**开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。
Elasticsearch 不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎。 它可以被下面这样准确的形容：
* 一个分布式的实时文档存储，每个字段 可以被索引与搜索
* 一个分布式实时分析搜索引擎
* 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

## ##

## 基础入门 ##

### 安装并运行 Elasticsearch ###
安装 Elasticsearch 之前，你需要先安装一个较新的版本的 Java。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/warning.png)| **ElasticSearch 的5.x版本基于jdk1.8版本。**
------------|------

安装完jdk后可以从 elastic 的官网 [elastic.co/downloads/elasticsearch](elastic.co/downloads/elasticsearch) 获取最新版本的 Elasticsearch。
先下载并解压适合你操作系统的 Elasticsearch 版本。
本次以`Windows`的`ElasticSearch5.5`版本来进行说明 。

下载好ElasticSearch的压缩包后进行解压，运行
```
bin\elasticsearch.bat
```
测试 Elasticsearch 是否启动成功，可以打开另一个终端，执行以下操作：
```
curl 'http://localhost:9200/?pretty'
```
![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/tip.png)| 没有安装curl的可以用Sense或者Postman之类的工具发送Restful请求
------------|------

如果启动成功，会获得以下的返回内容
```json
{
  "name": "yjh-node1",
  "cluster_name": "es-cluster",
  "cluster_uuid": "KmI4nt3RSCmd2HDa6JD-aA",
  "version": {
    "number": "5.5.0",
    "build_hash": "260387d",
    "build_date": "2017-06-30T23:16:05.735Z",
    "build_snapshot": false,
    "lucene_version": "6.6.0"
  },
  "tagline": "You Know, for Search"
}
```
这就意味着你现在已经启动并运行一个 Elasticsearch 节点了，你可以用它做实验了。
单个`节点`可以作为一个运行中的 Elasticsearch 的实例。
而一个`集群`是一组拥有相同`cluster.name`的节点，他们能一起工作并共享数据，还提供容错与可伸缩性。
(当然，一个单独的节点也可以组成一个集群) 你可以在`elasticsearch.yml`配置文件中 修改`cluster.name`，该文件会在节点启动时加载(`重启服务后才会生效`)。

### 与ElasticSearch交互 ###
和 Elasticsearch 的交互方式取决于 你是否使用 Java

**JAVA API**

如果你正在使用 Java，在代码中你可以使用 Elasticsearch 内置的两个客户端：

* 节点客户端（Node client）
节点客户端作为一个非数据节点加入到本地集群中。
换句话说，它本身不保存任何数据，但是它知道数据在集群中的哪个节点中，并且可以把请求转发到正确的节点。
* 传输客户端（Transport client）
轻量级的传输客户端可以将请求发送到远程集群。
它本身不加入集群，但是它可以将请求转发到集群中的一个节点上。

两个 Java 客户端都是通过`9300`端口并使用本地 Elasticsearch 传输 协议和集群交互。
集群中的节点通过端口`9300`彼此通信。如果这个端口没有打开，节点将无法形成一个集群。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/tip.png)| Java 客户端作为节点必须和 Elasticsearch 有相同的 主要 版本
------------|------

更多的 Java 客户端信息可以在 [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html) 中找到。

**RESTful API with JSON over HTTP**

所有其他语言可以使用 RESTful API 通过端口 9200 和 Elasticsearch 进行通信，你可以用你最喜爱的 web 客户端访问 Elasticsearch 。事实上，正如你所看到的，你甚至可以使用 curl 命令来和 Elasticsearch 交互。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/note.png)| 多语言插件支持，所有这些都可以在 [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)  中找到。
------------|------

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：
```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

部件 | 说明
-----|------
VERB | 适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。
PROTOCOL | http 或者 https（如果你在 Elasticsearch 前面有一个https代理）
HOST|Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。
PORT|运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。
PATH|API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。
QUERY_STRING|任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)
BODY|一个 JSON 格式的请求体 (如果请求需要的话)

### 面向文档 ###
传统数据库都是关系型数据库，数据是存在于表中，通常一个字段>对应一列。

Elasticsearch 是`面向文档`的，意味着它存储整个对象或 文档。Elasticsearch 不仅存储文档，而且 _索引 每个文档的内容使之可以被检索。在 Elasticsearch 中，你 对文档进行索引、检索、排序和过滤--而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。

Elasticsearch 使用 JavaScript Object Notation 或者 JSON 作为文档的序列化格式。JSON 序列化被大多数编程语言所支持，并且已经成为 NoSQL 领域的标准格式。 它简单、简洁、易于阅读。

Elasticsearch与关系数据的类比对应关系如下：
```
Relational DB ⇒ Databases ⇒ Tables ⇒ Rows ⇒ Columns

Elasticsearch ⇒ Indices ⇒ Types ⇒ Documents ⇒ Fields
```

文档举例,它代表了一个 b_asj_aj 对象：
```
{
   "_index": "aj",
   "_type": "b_asj_aj",
   "_id": "PCS876298229941904",
   "_version": 1,
   "found": true,
   "_source": {
      "SL_JJFS": "110(120、122)接转",
      "RESERVATION18": null,
      "SLJJSJ": "20130610123800",
      "JA_JASJ": null,
      "RESERVATION17": null,
      "AJLARY": null,
      "YSDW": null,
      "ZYAQ": "test.test",
      "RESERVATION39_CN": null,
      "LA_ZHXGR": null,
      "YISHENSJ": null,
      "ZBDW": "440307000000",
      "ZABZ": "盗车内财物\n",
      "SLFXRQ": null,
      "SSSQ": null,
      "ZBDW_CN": "深圳市公安局龙岗分局",
      "AJXBRY_CN": null,
      "SLFAQH": null,
      "SLJJDW_CN": null,
      "AJSTATE": "已受理",
      "RESERVATION11": null,
      "RESERVATION14": null,
      "LA_ZHXGSJ": null,
      "AB": "贷款诈骗案",
      "SL_SLSJ": null,
      "ESSJ": null,
      "SDTD": null,
      "YSDW_CN": null,
      "AJLARY_CN": null,
      "PASJ": null,
      "LASJ": "20140506115625",
      "AJBH": "A351898396993344",
      "LA_LRR": null,
      "JZBH": null,
      "SLFACS": null,
      "SL_LRSJ": "20140427115625",
      "LASTUPDATEDTIME": "20170815171700",
      "REFRESHTIME": null,
      "CREATEDTIME": "20170518090500",
      "FASJZZ": "20110527033200",
      "AJWHCD": null,
      "RESERVATION22": null,
      "AJZBRY": null,
      "DEPARTMENTCODE_CN": "广州市公安局白云区分局三元里派出所出所",
      "RESERVATION38": null,
      "FADD_JD": "新塘镇仙村永发西街",
      "RESERVATION39": null,
      "SL_SLRXM": null,
      "LA_LRSJ": null,
      "AJZBRY_CN": null,
      "RESERVATION14_CN": null,
      "LA_PZR": null,
      "LA_PZSJ": null,
      "LA_LRR_CN": null,
      "SJGJDQ": null,
      "DEPARTMENTCODE": "440111540000",
      "YSSJ": null,
      "FADD": null,
      "XZCS": null,
      "SL_LRR": null,
      "SL_BJSLH": null,
      "RESERVATION36": null,
      "SLJJDW": null,
      "RESERVATION35": null,
      "RESERVATION05": null,
      "YSDWDH": null,
      "AJMC": "test",
      "RESERVATION04": null,
      "ZAGJ": null,
      "SYSTEMID": "PCS876298229941904",
      "RESERVATION09": null,
      "XA_XASJ": null,
      "RESERVATION08": null,
      "FASJCZ": "20120710095200",
      "AJLY": null,
      "LADW_CN": null,
      "SLJSDW_CN": "广州市公安局花都区分局经济犯罪侦查大队三中队",
      "AJLX": "刑事",
      "LADW": null,
      "FXXS": null,
      "RESERVATION17_CN": null,
      "XZSJ": null,
      "AJ_DELETEFLAG": null,
      "AJSSJQ": null,
      "SL_LRR_CN": null,
      "AJBARP": null,
      "AJXBRY": null,
      "RESERVATION45": null,
      "RESERVATION03": null,
      "RESERVATION47": null,
      "RESERVATION46": null
   }
}
```
### 基本概念 ###
* 集群 （Cluster）
* 节点 （Node）
* 索引 （Index）
* 类型 （Type）
* 文档 （Document）
* 分片与副本（Shards & Replicas）

**集群 （Cluster）**

一个ES集群可以由一个或者多个节点（nodes or servers）组成。
所有这些节点用来存储所有的数据以及提供联合索引，为我们提供跨节点查询的能力。
一个ES集群的名称是唯一的，默认情况下为“elasticsearch”。这个名称非常重要，因为一个节点（node）会通过这个名称来判断是否加入已有的集群。
`必须保证在不同环境下使用不同的集群名称，否则节点可能会加入错误的集群。`

**节点 （Node）**

一个节点是一个集群中的一台服务器，它用来存储数据，参与集群的索引以及提供搜索能力。
可以简单的理解为一个ElasticSearch的启动实例，在集群中，节点是由它的唯一名称来做为标识。
这个名称对于管理ES集群非常重要，我们用它来定位网络或集群中的某一节点。
一个节点可以通过指定集群名称让它加入某个集群。
在单集群下，我们可以有任意数量的节点，如果当前网络下没有任何ES节点，那么在启动节点后，当前节点会默认形成一个单节点集群。

**索引 （Index）**

一个索引是一组具有相似特性的文档的集合。
一个索引由它的名称唯一标识`（必须所有字母为小写字母）`。
在一个单集群下，我们可以定义任意多的索引。

**类型 （Type）**

在一个索引下，我们可以定义一个或多个类型（types）。
一个类型是一个索引逻辑分类或分区（category/partition），而分类或分区的划分方法由我们自己决定。
通常情况下，我们会为具有相类似的字段的一组文档定义类型。
比如，如果我们运行一个博客平台，所有的数据都使用同一索引，我们为用户数据定义一种类型，为博客数据定义另一种类型，同时为评论数据定义另一种类型。

**文档 （Document）**

一个文档是一个可以被索引的基本信息单元。
可以理解为传统关系型数据库的表中的一条数据。

**分片与副本（Shards & Replicas）**

一个索引可能会存储大量数据从而超过单个节点硬件的限制。
为了解决这个问题，ES提供了一种分片（shard）能力，让我们将一个索引切分成片。
当我们创建一个索引时，我们可以为它指定分片的数量。
每个分片自己都能独立工作，并且存在与集群的任一节点中。
至于副本（Replicas）则是分片的备份文件。
分片数和副本数可以通过配置来进行修改。

### 索引创建与删除 ###

一个 Elasticsearch 集群可以`包含多个`索引 ，相应的每个索引可以包含多个`类型`。
这些不同的类型存储着多个`文档`，每个文档又有多个`属性`。

下面举例创建一个保存员工信息的索引：

* 每个员工索引一个文档，包含该员工的所有信息。
* 每个文档都将是**employee**类型 。
* 该类型位于索引**sinobest**内。
* 该索引保存在我们的 Elasticsearch 集群中。

实践中这非常简单（尽管看起来有很多步骤），我们可以通过一条命令完成所有这些动作：
```
PUT /sinobest/employee/1
{
    "name" : "张三",
    "age" :  25,
    "position" : "程序员",
    "interests": [ "吃饭", "睡觉", "写bug" ],
    "jobNums" : 1234
}
```
注意，路径 /sinobest/employee/1 包含了三部分的信息：

* sinobest:索引名称
* employee:类型名称
* 1:特定雇员的ID

操作成功后返回：
```
{
   "_index": "sinobest",
   "_type": "employee",
   "_id": "1",
   "_version": 1,
   "result": "created",
   "_shards": {
      "total": 2,
      "successful": 1,
      "failed": 0
   },
   "created": true
}
```

插入索引后通过以下命令查看当前索引的状态：
```
GET /_cat/indices
```

返回结果：
```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   sinobest Q1el2g90QhKD0jDNQxgXag   5   1          1            0      5.8kb          5.8kb
```

当我们想删除这个索引时，可以用DELETE命令：
```
DELETE /my_index
```

你也可以这样删除多个索引：
```
DELETE /index_one,index_two
DELETE /index_*
```

你甚至可以这样删除`全部`索引：
```
DELETE /_all
DELETE /*
```

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/note.png)|对一些人来说，能够用单个命令来删除所有数据可能会导致可怕的后果。<br>如果你想要避免意外的大量删除, 你可以在你的 elasticsearch.yml 做如下配置：<br>action.destructive_requires_name: true<br>这个设置使删除只限于特定名称指向的数据,<br> 而不允许通过指定 _all 或通配符来删除指定索引库。<br>你同样可以通过 [Cluster State API](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_changing_settings_dynamically.html) 动态的更新这个设置。
----|----

### 文档简单的CURD操作 ###

**创建（Create）**
```
PUT /sinobest/employee/2
{
    "name" : "李四",
    "age" :  21,
    "position" : "项目经理",
    "interests": [ "吃饭", "睡觉", "改需求" ],
    "jobNums" : 4321
}
```

**更新（Update）**
```
PUT /sinobest/employee/2
{
    "name" : "李四",
    "age" :  22,
    "position" : "项目经理",
    "interests": [ "吃饭", "睡觉", "改需求" ],
    "jobNums" : 4321
}
```

**查找（Retrieve）**
```
GET /sinobest/employee/_search

GET /sinobest/employee/_search?q=age:22

GET /sinobest/employee/_search
{
    "query" : {
        "match" : {
            "name" : "李四"
        }
    }
}
```

**删除（Delete）**
```
DELETE /sinobest/employee/2
```

## ##

## 深入了解 ##

### 自定义配置 ###

elasticsearch的config文件夹里面有三个配置文件：
* elasticsearch.yml ES的基本配置文件，主要讲解这里的常用配置
* jvm.options JVM相关参数配置
* log4j2.properties 日志配置文件，es是使用log4j来记录日志的，所以logging.yml里的设置按普通log4j配置文件来设置就行了。

**Cluster**

cluster.name可以确定你的集群名称,当你的elasticsearch集群在同一个网段中elasticsearch会自动的找到具有相同cluster.name的elasticsearch服务。
所以当同一个网段具有多个elasticsearch集群时cluster.name就成为同一个集群的标识。
```
cluster.name: elasticsearch
```

节点名称同理,可自动生成也可手动配置
```
node.name: node-1
```
允许一个节点是否可以成为一个master节点,es是默认集群中的第一台机器为master,如果这台机器停止就会重新选举master.
```
node.master: true
```
允许该节点存储数据(默认开启)
```
node.data: true
```

**Index**

设置索引的分片数,默认为5
```
index.number_of_shards: 5
```
设置索引的副本数,默认为1:
```
index.number_of_replicas: 1
```
![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/note.png)|配置文件中提到的最佳实践是,如果服务器够多,可以将分片提高,尽量将数据平均分布到大集群中去<br>同时,如果增加副本数量可以有效的提高搜索性能 <br>需要注意的是,`number_of_shards`是索引创建后一次生成的,后续不可更改设置 <br>`number_of_replicas`是可以通过API去实时修改设置的
----|----

**Paths**

配置文件存储位置
```
path.conf: /path/to/conf
```
数据存储位置(单个目录设置)
```
path.data: /path/to/data
```
多个数据存储位置,有利于性能提升
```
path.data: /path/to/data1,/path/to/data2
```
临时文件的路径
```
path.work: /path/to/work
```
日志文件的路径
```
path.logs: /path/to/logs
```
插件安装路径
```
path.plugins: /path/to/plugins
```

**Network And HTTP**

设置绑定的ip地址,可以是ipv4或ipv6的,默认为0.0.0.0
```
network.bind_host: 192.168.0.1
```
设置其它节点和该节点交互的ip地址,如果不设置它会自动设置,值必须是个真实的ip地址
```
network.publish_host: 192.168.0.1
```
设置节点间交互的tcp端口,默认是9300
```
transport.tcp.port: 9300
```
设置对外服务的http端口,默认为9200
```
http.port: 9200
```
设置请求内容的最大容量,默认100mb
```
http.max_content_length: 100mb
```

**Discovery**

设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点.默认为1,官方推荐设置为(N/2)+1
```
discovery.zen.minimum_master_nodes: 1
```
探查的超时时间,默认3秒,提高一点以应对网络不好的时候,防止脑裂
```
discovery.zen.ping.timeout: 3s
```
这是一个集群中的主节点的初始列表,当节点(主节点或者数据节点)启动时使用这个列表进行探测
```
discovery.zen.ping.unicast.hosts: ["host1", "host2:port"]
```

### 集群内的原理 ###
介绍 Elasticsearch 在分布式环境中的运行原理。
在这里大概讲述下ES的扩容机制，以及如何处理硬件故障的内容(想起来市局停电带来的惨痛经历)。

#### 空集群 ####
如果我们启动了一个单独的节点，里面不包含任何的数据和 索引，那我们的集群看起来就是一个 `下图` “包含空内容节点的集群”。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/cp1.png)

一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同`cluster.name`配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为`主节点（Master）`时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到 集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。

#### 集群健康 ####
Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是`集群健康`， 它在`status`字段中展示为`green`、`yellow`或者`red`。
```
GET /_cluster/health
```
在一个不包含任何索引的空集群中，它将会有一个类似于如下所示的返回内容：
```
{
   "cluster_name":          "elasticsearch",
   "status":                "green",
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```
* green 所有的主分片和副本分片都正常运行。
* yellow 所有的主分片都正常运行，但不是所有的副本分片都正常运行。
* red 有主分片没能正常运行。

#### 添加索引 ####
我们往 Elasticsearch 添加数据时需要用到 索引 —— 保存相关数据的地方。 索引实际上是指向一个或者多个物理`分片`的逻辑命名空间 。

Elasticsearch 是利用分片将数据分发到集群内各处的。分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。 当你的集群规模扩大或者缩小时， Elasticsearch 会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。

一个分片可以是`主分片`或者`副本分片`。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

举例说明，创建索引`blogs`，设置主分片数为3，副本为1（副本1说明每个主分片有1个副本分片）
```
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```
我们的集群现在是下图“拥有一个索引的单节点集群”。所有3个主分片都被分配在 Node 1 。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/cp2.png)

如果我们现在查看集群健康， 我们将看到如下内容：
```
{
  "cluster_name": "elasticsearch",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 3,
  "active_shards": 3,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 3,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50
}
```
* status：集群 status 值为 yellow 。
* unassigned_shards ：没有被分配到任何节点的副本数。

集群的健康状况为`yellow`则表示全部`主分片`都正常运行（集群可以正常服务所有请求），
但是`副本`分片没有全部处在正常状态。
 实际上，所有3个副本分片都是 unassigned —— 它们都没有被分配到任何节点。
 在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据。

当前我们的集群是正常运行的，但是在硬件故障时有丢失数据的风险。

