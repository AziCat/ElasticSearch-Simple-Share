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
    * 检索关键字个数太多报错
    * 权限控制&监控
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
```json
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

## ##

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
```json
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
```json
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

#### 添加故障转移编辑 ####

当集群中只有一个节点在运行时，意味着会有一个单点故障问题。
幸运的是，我们只需再启动一个节点即可防止数据丢失。

启动第二个节点|
----|
为了测试第二个节点启动后的情况，你可以在同一个目录内，完全依照启动第一个节点的方式来启动一个新节点（参考[安装并运行 Elasticsearch](https://www.elastic.co/guide/cn/elasticsearch/guide/current/running-elasticsearch.html)）。多个节点可以共享同一个目录。当你在同一台机器上启动了第二个节点时，只要它和第一个节点有同样的 cluster.name 配置，它就会自动发现集群并加入到其中。 但是在不同机器上启动节点的时候，为了加入到同一集群，你需要配置一个可连接到的单播主机列表。 详细信息请查看[单播代替组播](https://www.elastic.co/guide/cn/elasticsearch/guide/current/important-configuration-changes.html#unicast)。|

如果启动了第二个节点，我们的集群将会如下图“拥有两个节点的集群——所有主分片和副本分片都已被分配”所示。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/cp3.png)

当第二个节点加入到集群后，3个`副本分片`将会分配到这个节点上——每个主分片对应一个副本分片。 这意味着当集群内任何一个节点出现问题时，我们的数据都完好无损。

所有新近被索引的文档都将会保存在主分片上，然后被并行的复制到对应的副本分片上。这就保证了我们既可以从主分片又可以从副本分片上获得文档。

`cluster-health`现在展示的状态为`green`，这表示所有6个分片（包括3个主分片和3个副本分片）都在正常运行。

```json
{
  "cluster_name": "elasticsearch",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 2,
  "number_of_data_nodes": 2,
  "active_primary_shards": 3,
  "active_shards": 6,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```

#### 水平扩容 ####

怎样为我们的正在增长中的应用程序按需扩容呢？ 当启动了第三个节点，我们的集群将会看起来如下图“拥有三个节点的集群——为了分散负载而对分片进行重新分配”所示。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/cp4.png)

`Node 1`和`Node 2`上各有一个分片被迁移到了新的`Node 3`节点，现在每个节点上都拥有2个分片，而不是之前的3个。 这表示每个节点的硬件资源（CPU, RAM, I/O）将被更少的分片所共享，每个分片的性能将会得到提升。

分片是一个功能完整的搜索引擎，它拥有使用一个节点上的所有资源的能力。 我们这个拥有6个分片（3个主分片和3个副本分片）的索引可以最大扩容到6个节点，每个节点上存在一个分片，并且每个分片拥有所在节点的全部资源。

#### 更多的扩容 ###

但是如果我们想要扩容超过6个节点怎么办呢？

主分片的数目在索引创建时 就已经确定了下来。实际上，这个数目定义了这个索引能够存储的最大数据量。（实际大小取决于你的数据、硬件和使用场景。） 但是，读操作——搜索和返回数据——可以同时被`主分片`或`副本分片`所处理，所以当你拥有越多的副本分片时，也将拥有越高的吞吐量。

在运行中的集群上是可以动态调整副本分片数目的 ，我们可以按需伸缩集群。让我们把副本数从默认的`1`增加到`2`：
```
PUT /blogs/_settings
{
   "number_of_replicas" : 2
}
```

如下图“将参数 number_of_replicas 调大到 2”所示， blogs 索引现在拥有9个分片：3个主分片和6个副本分片。 这意味着我们可以将集群扩容到9个节点，每个节点上一个分片。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/cp5.png)

#### 应对故障 ####

我们之前说过 Elasticsearch 可以应对节点故障，接下来让我们尝试下这个功能。 如果我们关闭第一个节点，这时集群的状态为下图“关闭了一个节点后的集群”

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/cp6.png)

我们关闭的节点是一个主节点。而集群必须拥有一个主节点来保证正常工作，所以发生的第一件事情就是选举一个新的主节点： Node 2 。

在我们关闭 Node 1 的同时也失去了主分片 1 和 2 ，并且在缺失主分片的时候索引也不能正常工作。 如果此时来检查集群的状况，我们看到的状态将会为 red ：不是所有主分片都在正常工作。

幸运的是，在其它节点上存在着这两个主分片的完整副本， 所以新的主节点立即将这些分片在 Node 2 和 Node 3 上对应的副本分片提升为主分片， 此时集群的状态将会为 yellow 。 这个提升主分片的过程是瞬间发生的，如同按下一个开关一般。

为什么我们集群状态是 yellow 而不是 green 呢？ 虽然我们拥有所有的三个主分片，但是同时设置了每个主分片需要对应2份副本分片，而此时只存在一份副本分片。 所以集群不能为 green 的状态，不过我们不必过于担心：如果我们同样关闭了 Node 2 ，我们的程序 依然 可以保持在不丢任何数据的情况下运行，因为 Node 3 为每一个分片都保留着一份副本。

如果我们重新启动 Node 1 ，集群可以将缺失的副本分片再次进行分配。如果 Node 1 依然拥有着之前的分片，它将尝试去重用它们，同时仅从主分片复制发生了修改的数据文件。

## ##

### 映射和分析 ###

#### 倒排索引(inverted index) ####
传统的关系型数据库，可以通过给某列建立一个B树索引来加快检索的速度。通常是文档->关键字。
而Elasticsearch与之相反，是关键字->文档。其使用一种称为`倒排索引`的结构，可能是因为将正常的索引倒过来了吧，所以大家叫他倒排索引，叫`反向索引`我觉得ok，它适用于快速的全文搜索。
一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

例如，假设我们有两个文档，每个文档的 content 域包含如下内容：

* The quick brown fox jumped over the lazy dog
* Quick brown foxes leap over lazy dogs in summer

为了创建倒排索引，我们首先将每个文档的 content 域`拆分`成单独的`词（我们称它为 词条 或 tokens ）`，创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。结果如下所示：
```
Term      Doc_1  Doc_2
-------------------------
Quick   |       |  X
The     |   X   |
brown   |   X   |  X
dog     |   X   |
dogs    |       |  X
fox     |   X   |
foxes   |       |  X
in      |       |  X
jumped  |   X   |
lazy    |   X   |  X
leap    |       |  X
over    |   X   |  X
quick   |   X   |
summer  |       |  X
the     |   X   |
------------------------
```

现在，如果我们想搜索 quick brown ，我们只需要查找包含每个词条的文档：
```
Term      Doc_1  Doc_2
-------------------------
brown   |   X   |  X
quick   |   X   |
------------------------
Total   |   2   |  1
```

两个文档都匹配，但是第一个文档比第二个匹配度更高。如果我们使用仅计算匹配词条数量的简单`相似性算法`，那么，我们可以说，对于我们查询的相关性来讲，第一个文档比第二个文档更佳。

#### 分析与分析器 ####

`分析`包含下面的过程：

* 首先，将一块文本分成适合于倒排索引的独立的 词条 ，
* 之后，将这些词条统一化为标准格式以提高它们的“可搜索性”

分析器执行上面的工作。 分析器 实际上是将三个功能封装到了一个包里：

**字符过滤器**

首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 `and`。

**分词器**

其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

**Token 过滤器**

最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a`， `and`， `the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）。

Elasticsearch提供了开箱即用的字符过滤器、分词器和token 过滤器。 这些可以组合起来形成自定义的分析器以用于不同的目的。

#### 内置分析器 ####
但是， Elasticsearch还附带了可以直接使用的预包装的分析器。 接下来我们会列出最重要的分析器。为了证明它们的差异，我们看看每个分析器会从下面的字符串得到哪些词条：
```
"Set the shape to semi-transparent by calling set_trans(5)"
```

**标准分析器**

标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 Unicode 联盟 定义的 单词边界 划分文本。删除绝大部分标点。最后，将词条小写。它会产生
```
set, the, shape, to, semi, transparent, by, calling, set_trans, 5
```
**简单分析器**

简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生
```
set, the, shape, to, semi, transparent, by, calling, set, trans
```
**空格分析器**

空格分析器在空格的地方划分文本。它会产生
```
Set, the, shape, to, semi-transparent, by, calling, set_trans(5)
```
**语言分析器**
特定语言分析器可用于[很多语言](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/analysis-lang-analyzer.html)(并不包括中文= =日文韩文这种块型语言也不包括)。
它们可以考虑指定语言的特点。
例如， `english`分析器附带了一组英语无用词（常用单词，例如 and 或者 the ，它们对相关性没有多少影响），它们会被删除。
由于理解英语语法的规则，这个分词器可以提取英语单词的`词干`。

`english`分词器会产生下面的词条：
```
set, shape, semi, transpar, call, set_tran, 5
```
注意看 transparent\`、 \`calling 和 set_trans 已经变为词根格式。

#### 什么时候使用分析器 ####
当我们`索引`一个文档，它的全文域被分析成词条以用来创建倒排索引。
但是，当我们在全文域`搜索`的时候，我们需要将查询字符串通过`相同的分析过程`，
以保证我们搜索的词条格式与索引中的词条格式一致。
* 当你查询一个`全文`域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。
* 当你查询一个`精确值`域时，不会分析查询字符串， 而是搜索你指定的精确值。

我们可以通过[Mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/mapping.html)来设置分析器

#### 映射(Mapping) ####

在传统数据库中，我们建表的时候要指定字段的类型、主键、长度、默认值等等，
而ES的映射操作就是类似这样的操作。
为ES的`文档`的字段设置类型、分析器、过滤器等等。

**字段数据类型**

ES支持以下多种的数据类型：
* 字符串
    * text and keyword
* 数字
    * long, integer, short, byte, double, float, half_float, scaled_float
* 日期
    * date
* 布尔型
    * boolean
* 二进制
    * binary
* 范围
    * integer_range, float_range, long_range, double_range, date_range

复杂的数据类型

* 数组
    * Array support does not require a dedicated type
* 对象
    * object for single JSON objects
* 嵌套
    * nested for arrays of JSON objects

**查看映射**

通过 /_mapping ，我们可以查看 Elasticsearch 在一个或多个索引中的一个或多个类型的映射 。
```
GET /index/_mapping/type
```

**analyzer**

对于`analyzed`字符串域，用`analyzer`属性指定在搜索和索引时使用的分析器。
默认， Elasticsearch 使用`standard`分析器，但你可以指定一个内置的分析器替代它，
例如`whitespace`、`simple` 和 `english`，也可以配置为第三方的分析器插件（需要另外安装）：
```
PUT /index/type/_mapping
{
   "properties": {
      "field": {
         "search_analyzer": "simple",
         "analyzer": "simple",
         "type": "string"
      }
   }
}
```

**更新映射**

当你首次创建一个索引的时候，可以指定类型的映射。你也可以使用`/_mapping`为新类型增加映射。

但是当索引有数据时，说明倒排索引已经创建，这时候不能修改映射，所以只能通过`删除`原来的索引，然后再重新建立索引和映射。。。

#### 排序与相关性 ####
默认情况下，返回的结果是按照`相关性`进行排序的——最相关的文档排在最前。
这里大概解释一下`相关性`意味着什么以及它是如何计算的，同时介绍一下使用`sort`参数进行自定义的排序。

**排序**

为了按照相关性来排序，需要将相关性表示为一个数值。
在 Elasticsearch 中，`相关性得分`由一个浮点数进行表示，并在搜索结果中通过`_score`参数返回， 默认排序是`_score`降序。
```json
{
   "took": 11,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 1,
      "max_score": 1,
      "hits": [
         {
            "_index": "sinobest",
            "_type": "employee",
            "_id": "1",
            "_score": 1,
            "_source": {
               "name": "张三",
               "age": 25,
               "position": "程序员",
               "interests": [
                  "吃饭",
                  "睡觉",
                  "写bug"
               ],
               "jobNums": 1234
            }
         }
      ]
   }
}
```
当我们想按实际的使用情况来排序，比如年龄的大小，这时候我们可以使用`sort`参数：
```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "name" : "张三" }}
        }
    },
    "sort": { "age": { "order": "desc" }}
}
```
返回结果中的`_score`为null：
```json
{
   "took": 11,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 1,
      "max_score": null,
      "hits": [
         {
            "_index": "sinobest",
            "_type": "employee",
            "_id": "1",
            "_score": null,
            "_source": {
               "name": "张三",
               "age": 25,
               "position": "程序员",
               "interests": [
                  "吃饭",
                  "睡觉",
                  "写bug"
               ],
               "jobNums": 1234
            }
         }
      ]
   }
}
```

**什么是相关性**

默认情况下，返回结果是按相关性倒序排列的。 但是什么是相关性？ 相关性如何计算？

Elasticsearch 的相似度算法被定义为`检索词频率/反向文档频率`，`TF/IDF`，包括以下内容：

项|说明
----|----
检索词频率|检索词在该字段出现的频率？出现频率越高，相关性也越高。 字段中出现过 5 次要比只出现过 1 次的相关性高。
反向文档频率|每个检索词在索引中出现的频率？频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。
字段长度准则|字段的长度是多少？长度越长，相关性越低。 检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大。

相关性并不只是全文本检索的专利。也适用于 yes|no 的子句，匹配的子句越多，相关性评分越高。

如果多条查询子句被合并为一条复合查询语句 ，比如`bool`查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。

**执行计划**

当在传统数据库调优一个复杂的sql的时候，通常会用到执行计划，来查看sql执行的时间损耗在哪个方面。
同理，ElasticSearch也可以通过`explain `参数来查看当次查询的细节。
```
GET /_search?explain
{
   "query"   : { "match" : { "name" : "张三" }}
}
```
![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/warning.png)| 输出 explain 结果代价是十分昂贵的，<br>它只能用作调试工具 。千万不要用于生产环境。
------------|------

## ##

## 实际使用中的Q&A ##

#### Q: 检索包含中文的关键字时返回结果不准确 ####
在使用的过程中，我查询关键字`天河区天河公园`的时候，会把包含以下内容的文档全部返回出来，
这显然是不对的。
```
天、河、区、公、园
```
ElasticSearch检索文档，是从`倒排索引`中查找包含此词条的文档，所以我在想是不是索引没建立好。
当初建立索引的时候没有设置相关的映射，所以插入ElasticSearch的时候用的是默认的分析器，于是使用`Analyze API`检验一下。
```
POST _analyze
{
  "tokenizer": "standard",
  "text":      "天河区天河公园"
}
```
返回结果如下：
```json
{
   "tokens": [
      {
         "token": "天",
         "start_offset": 0,
         "end_offset": 1,
         "type": "<IDEOGRAPHIC>",
         "position": 0
      },
      {
         "token": "河",
         "start_offset": 1,
         "end_offset": 2,
         "type": "<IDEOGRAPHIC>",
         "position": 1
      },
      {
         "token": "区",
         "start_offset": 2,
         "end_offset": 3,
         "type": "<IDEOGRAPHIC>",
         "position": 2
      },
      {
         "token": "天",
         "start_offset": 3,
         "end_offset": 4,
         "type": "<IDEOGRAPHIC>",
         "position": 3
      },
      {
         "token": "河",
         "start_offset": 4,
         "end_offset": 5,
         "type": "<IDEOGRAPHIC>",
         "position": 4
      },
      {
         "token": "公",
         "start_offset": 5,
         "end_offset": 6,
         "type": "<IDEOGRAPHIC>",
         "position": 5
      },
      {
         "token": "园",
         "start_offset": 6,
         "end_offset": 7,
         "type": "<IDEOGRAPHIC>",
         "position": 6
      }
   ]
}
```
关键字被分词成单独的每个汉字，然后排重后建立`倒排索引`，每个单独的汉字都为一个`倒排索引`。

#### A: 使用中文分析器，添加字典和中文关键字预处理 ####

**中文分词器的选择**
* [pinyin 分词器](https://github.com/medcl/elasticsearch-analysis-pinyin)
* [Mmseg 分词器](https://github.com/medcl/elasticsearch-analysis-mmseg/releases)
* ik-analyzer

**ik-analyzer**

安装方式请参考[gayhub地址](https://github.com/medcl/elasticsearch-analysis-ik)

IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包。
采用了特有的“正向迭代最细粒度切分算法“，支持细粒度和最大词长两种切分模式；具有83万字/秒（1600KB/S）的高速处理能力。
采用了多子处理器分析模式，支持：`英文字母、数字、中文词汇等分词处理，兼容韩文、日文字符`
优化的词典存储，更小的内存占用。支持用户词典扩展定义
针对Lucene全文检索优化的查询分析器IKQueryParser(作者吐血推荐)；引入简单搜索表达式，采用歧义分析算法优化查询关键字的搜索排列组合，能极大的提高Lucene检索的命中率。

ik 带有两个分词器
* ik_max_word：会将文本做最细粒度的拆分；尽可能多的拆分出词语
* ik_smart：会做最粗粒度的拆分；已被分出的词语将不会再次被其它词语占有

ik_max_word分词效果
```
POST _analyze
{
  "tokenizer": "ik_max_word",
  "text":      "天河区天河公园"
}
```

```json
{
   "tokens": [
      {
         "token": "天河区",
         "start_offset": 0,
         "end_offset": 3,
         "type": "CN_WORD",
         "position": 0
      },
      {
         "token": "天河",
         "start_offset": 0,
         "end_offset": 2,
         "type": "CN_WORD",
         "position": 1
      },
      {
         "token": "河",
         "start_offset": 1,
         "end_offset": 2,
         "type": "CN_WORD",
         "position": 2
      },
      {
         "token": "区",
         "start_offset": 2,
         "end_offset": 3,
         "type": "CN_CHAR",
         "position": 3
      },
      {
         "token": "天河",
         "start_offset": 3,
         "end_offset": 5,
         "type": "CN_WORD",
         "position": 4
      },
      {
         "token": "河",
         "start_offset": 4,
         "end_offset": 5,
         "type": "CN_WORD",
         "position": 5
      },
      {
         "token": "公园",
         "start_offset": 5,
         "end_offset": 7,
         "type": "CN_WORD",
         "position": 6
      }
   ]
}
```

ik_smart分析效果
```
POST _analyze
{
  "tokenizer": "ik_smart",
  "text":      "天河区天河公园"
}
```

```json
{
   "tokens": [
      {
         "token": "天河区",
         "start_offset": 0,
         "end_offset": 3,
         "type": "CN_WORD",
         "position": 0
      },
      {
         "token": "天河",
         "start_offset": 3,
         "end_offset": 5,
         "type": "CN_WORD",
         "position": 1
      },
      {
         "token": "公园",
         "start_offset": 5,
         "end_offset": 7,
         "type": "CN_WORD",
         "position": 2
      }
   ]
}
```

**添加字典**

上述的两种分词模式虽然能满足很多使用场景，但对于一些特殊情况，
例如`张小三`是一个用户的名字，分析结果如下：
```
{
   "tokens": [
      {
         "token": "张",
         "start_offset": 0,
         "end_offset": 1,
         "type": "CN_CHAR",
         "position": 0
      },
      {
         "token": "小三",
         "start_offset": 1,
         "end_offset": 3,
         "type": "CN_WORD",
         "position": 1
      }
   ]
}
```
我们想让分析器以`张小三`为一个词条而不是`张`和`小三`。这时候可以使用ik分词器的字典功能。

修改分析器目录下的`config/IKAnalyzer.cfg.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">custom/mydict.dic;custom/single_word_low_freq.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">custom/ext_stopword.dic</entry>
 	<!--用户可以在这里配置远程扩展字典 -->
	<entry key="remote_ext_dict">location</entry>
 	<!--用户可以在这里配置远程扩展停止词字典-->
	<entry key="remote_ext_stopwords">http://xxx.com/xxx.dic</entry>
</properties>
```

在使用中文分析器后，查询精度虽然提高了，但还是不准确，搜索`天河区天河公园`还是得不到想要的结果，
但我发现关键字输入`天河区 天河 公园`，用空格隔开就能查询到很准确的结果。所以针对中文的关键字，
还做了一下预处理（预分词）来提高查询准确度。
```java
//分词器
String analyzer = PropertiesUtil.getAsString(ES_CONFIG,"search.cn_analyzer");
//构建分析请求
AnalyzeRequestBuilder analyzeRequestBuilder = new AnalyzeRequestBuilder(client, AnalyzeAction.INSTANCE,
        simpleParam.getIndex(),chineseParamsList.toArray(new String[]{}));
analyzeRequestBuilder.setTokenizer(analyzer);

//查询
List<AnalyzeResponse.AnalyzeToken> ikTokenList = analyzeRequestBuilder.execute().actionGet().getTokens();

//处理结果
List<String> pretreatedCNList = new ArrayList<>();
ikTokenList.forEach(ikToken -> {
    String item = ikToken.getTerm();
    //排重
    if(!pretreatedCNList.contains(item)){
        pretreatedCNList.add(item);
    }
});
```

#### Q: 索引重建 ####
为了提高查询的精确度，经常会对索引的映射进行修改。但是对于有数据的索引来说，
必须要先删除索引才能重建，这样又要重新通过JDBC从数据库里获取数据。非常耗时而且要停机。

#### A: 使用Reindex API进行重建，索引别名实现零停机 ####
**Reindex API**

从Elasticsearch v2.3.0开始， [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/docs-reindex.html) 被引入。它能够对文档重建索引而不需要任何插件或外部工具。

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```
类似于传统数据库中的
```
CREATE TABLE NEW_TAB AS (SELECT * FROM TAB)
```
当然`Reindex API`也是支持按条件进行重建，此处不详讲。

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/warning.png)| Reindex 不会试图去设置目标索引。<br> 也不会复制源索引的设置与映射。<br> 在Reindex 操作之前，你应该要先设置目标索引的分片数、副本数、映射等相关设置。
----|----

**索引别名**

在前面提到的，重建索引的问题是必须更新应用中的索引名称。 索引别名就是用来解决这个问题的！

索引`别名`就像一个快捷方式或软连接，可以指向一个或多个索引，也可以给任何一个需要索引名的API来使用。
可以理解为`视图`

有两种方式管理别名：`_alias`用于单个操作，`_aliases`用于执行多个原子级操作。

假设你的应用有一个叫 my_index 的索引。事实上， my_index 是一个指向当前真实索引的别名。真实索引包含一个版本号： my_index_v1 ， my_index_v2 等等。

首先，创建索引 my_index_v1 ，然后将别名 my_index 指向它：
```
PUT /my_index_v1                    --创建索引 my_index_v1 。
PUT /my_index_v1/_alias/my_index    --设置别名 my_index 指向 my_index_v1 。
```
你可以检测这个别名指向哪一个索引：
```
GET /*/_alias/my_index
```
或哪些别名指向这个索引：
```
GET /my_index_v1/_alias/*
```
两者都会返回下面的结果：
```
{
    "my_index_v1" : {
        "aliases" : {
            "my_index" : { }
        }
    }
}
```
然后，我们决定修改`my_index_v1`中一个字段的映射。当然，我们不能修改现存的映射，所以我们必须重新索引数据。
首先, 我们用`新映射`创建索引`my_index_v2`，然后从`my_index_v1`中`Reindex`数据到`my_index_v2`。

一旦我们确定文档已经被正确地重索引了，我们就将别名指向新的索引。

一个别名可以指向多个索引，所以我们在添加别名到新索引的同时必须从旧的索引中删除它。这个操作需要原子化，这意味着我们需要使用 _aliases 操作：
```
POST /_aliases
{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}
```
此时别名`my_index`已经从`my_index_v1`指向到了`my_index_v2`。
外部访问`my_index`时，实际上是指向到了`my_index_v2`。
这时候把旧的`my_index_v1`索引删除掉。这样一套操作下来，开销很小，应该广泛使用。

#### Q: 检索关键字包含特殊字符 ####
大家都知道很多系统都有自己的保留字，有特殊含意，不能作为条件或者参数使用。
ElasticSearch也不例外，不小心中招的话，轻则报错，重则宕机。

#### A: 保留字消义 ####
现在列举一下ElasticSearch的保留字符：
```
+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /
```
如果你想查询`(1+1)=2`这个内容，关键字要处理成这样：
```
\(1\+1\)\=2
```

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/warning.png)| < 和 > 不能被消义。处理它的方法就是把它从关键字中去掉:smirk:<br>同时，也要注意空格' '也是保留字之一且不能消义，其作用是分隔关键字。<br>文档中的空格在分析器工作时已被去掉，所以不用担心文档会保存空格。
----|----

#### Q: 检索关键字个数太多 ####
有次随便瞎搞的时候，放了2000多个关键字去进行检索，结果报错
```
Query DSL: Allow to control (globally) the max clause count for `bool` query (defaults to 1024)
```

#### A: 控制关键字个数Or修改设置 ####
就跟select * from tab where field in (...)一样，in里面参数的数量不能超过1000。
ElasticSearch也有数量控制，也刚好是个整数1024。

为了避免这种情况，最好的处理方式是控制参数的个数，因为大数量的参数会影响检索效率。
如果一定要查这么多参数。ElasticSearch的[issues](https://github.com/elastic/elasticsearch/issues/482)给出了方法：
设置`match_phrase_prefix`的大小。

#### Q: 权限控制&监控 ####
不知道大家有没有发现，当我们启动了ElasticSearch之后，就能通过Restful API来进行相关操作。
并不需要登录，也没有什么权限控制，只要知道ElasticSearch的ip和port就能为所欲为。
这时怕是被人脱库或者删库都不知道是谁干的。

#### A: 使用官方提供的插件 ####
* Kibana
* X‑Pack

`Kibana`让您能够可视化`Elasticsearch`中的数据并对其进行操作。
有了`Kibana`，在为`Elastic Stack`管理设置或配置`X‑Pack security`功能来保护、监控和配置`Elastic Stack`。
命令行不再是唯一途径。
与此同时，得益于非常出色的 API，现在的`Elastic Stack`操作变得更加直观，
能够让更加广泛的受众上手操作。

详细可参见[官方网址](https://www.elastic.co/cn/products/kibana)

注：官方的插件并不是完全免费的喔:joy:

## ##

## 参考内容 ##
* [Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)（难得的中文文档喔，不过针对的是 Elasticsearch 2.x的，阅读时请注意）
* [Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/index.html)（官方文档，英语好的小伙伴可以看这个，此处是5.5版本的，目前最新的更新为5.6版本）
* [elastic/elasticsearch · GitHub](https://github.com/elastic/elasticsearch)（ElasticSearch的开源地址）
* [java客户端 API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/index.html)
* 百度百科
* 知乎