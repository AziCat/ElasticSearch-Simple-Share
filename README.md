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

![](https://github.com/AziCat/ElasticSearch-Simple-Share/raw/master/res/note.png)| Elasticsearch 为以下语言提供了官方客户端 --Groovy、JavaScript、.NET、 PHP、 Perl、 Python 和 Ruby--还有很多社区提供的客户端和插件，所有这些都可以在 [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)  中找到。
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

