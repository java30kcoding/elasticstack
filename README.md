# Elastic技术栈学习

## ElasticSearch的安装与简单配置

> 7.0版本以后自带jdk环境，无需任何依赖

### ElasticSearch下载地址

```http
https://www.elastic.co/downloads/elasticsearch
```

### ElasticSearch无法无法使用root用户启动

#### 1.新建elastic用户

```shell
adduser elastic
```

#### 2.修改密码

```shell
passwd elastic
```

#### 3.将文件夹权限赋予该用户

```shell
chown -R elastic yuanyl
```

#### 4.切换至elastic用户

```shell
su elastic
```

#### 5.后台启动模式

```shell
./elasticsearch -d
# 也可以不切换用户使用下面命令直接启动
su elastic -c "/yuanyl/elasticsearch-7.1.1/bin/elasticsearch -d"
```

### 启动问题

#### 1.文件描述

max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

> 修改/etc/security/limits.conf文件，增加配置，用户退出后重新登录生效

```shell
vim /etc/security/limits.conf
*               soft    nofile          65536
*               hard    nofile          65536
ulimit -Hn
ulimit -Sn
# 查看数量
```

#### 2.最大虚拟内存

max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

> 修改/etc/sysctl.conf文件，增加配置

```shell
vim /etc/sysctl.conf
vm.max_map_count=262144
sysctl -p
# 执行该命令生效
```

#### 3.默认配置

the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

> 把\#cluster.initial_master_nodes: ["node-1", "node-2"]注释打开

#### 4.无法外网访问

修改yml配置文件端口为0.0.0.0

## ElasticSearch插件

```
./elasticsearch-plugin list
./elasticsearch-plugin install analysis-icu
```

### 国内建议使用离线安装方法

```http
https://artifacts.elastic.co/downloads/elasticsearch-plugins/analysis-icu/analysis-icu-7.1.1.zip
```

```shell
./elasticsearch-plugin install file:///yuanyl/analysis-icu-7.1.1.zip
```

> 在web端可以通过该api查看所安装的插件http://shaking.top:9200/_cat/plugins

### Elastic提供插件的机制对系统进行扩展

+ Discovery Plugin
+ Analysis Plugin
+ Security Plugin
+ Management Plugin
+ Ingest Plugin
+ Mapper Plugin
+ Backup Plugin

## 启动多实例

```shell
./elasticsearch -E node.name=node1 -E cluster.name=shaking -E path.data=node1_data -d
./elasticsearch -E node.name=node2 -E cluster.name=shaking -E path.data=node2_data -d
./elasticsearch -E node.name=node3 -E cluster.name=shaking -E path.data=node3_data -d
```

### 查看已启动节点

```http
http://shaking.top:9200/_cat/nodes
```

### 出现问题

1. master节点启动成功，启动第二个节点的时候，查看nodes报错master_not_discovered_exception

   初始化配置文件中的cluster.initial_master_nodes: node1

## Kibana的安装与简单配置

### 使用华为云镜像下载安装

```http
wget https://mirrors.huaweicloud.com/kibana/7.1.1/kibana-7.1.1-linux-x86_64.tar.gz
```

### 修改Kibana配置指向ElasticSearch

### 外网访问同ElasticSearch

### Kibana插件

```shell
kibana-plugin install plugin_location
# 离线安装方式
kibana-plugin list
kibana remove
```

### DevTools

> API可以使用DevTools调用

## Docker的安装与使用

### Docker下载地址

```shell
yum update
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 阿里仓库
yum install docker-ce-18.03.1.ce
```

### 常用命令

```shell
systemctl start docker
systemctl enable docker
```

### Docker-Composer安装

```shell
curl -L https://github.com/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` -o /yuanyl/docker-compose
chmod +x /yuanyl/docker-compose
```

### Docker-Composer常用命令

```shell
docker-compose up -d
docker-compose down
docker ps # 查看镜像
```

> docker-compose.yaml文件在github上

## Logstash安装与简单配置

### Logstash下载地址

```shell
wget https://repo.huaweicloud.com/logstash/7.1.1/logstash-7.1.1.tar.gz
```

### 常用命令

```shell
logstash -f configLocation/logstash.conf
```

> Logstash需要java环境

## ElasticSearch的索引，文档和REST API

### 文档(Document)

+ ElasticSearch是面向文档的，文档是所有可搜索数据的最小单位
  + 日志文件中的日志项
  + 一本电影的具体信息 / 一张唱片的详细信息
  + MP3播放器里的一首歌 / 一篇PDF文档中的具体内容
+ 文档会被序列化成JSON格式，保存在ElasticSearch中
  + JSON对象由字段组成
  + 每个字段都有对应的字段类型(字符串 / 数值 / 布尔 / 日期 / 二进制 / 范围类型)
+ 每个文档都有一个Unique ID
  + 你可以自己指定ID
  + 或者通过ElasticSearch自动生成

#### JSON文档

+ 一篇文档包含了一系列的字段。类似数据库表中的一条记录
+ JSON文档格式灵活，不需要预先定义格式
  + 字段的类型可以指定或者通过ElasticSearch自动推算
  + 支持数组 / 支持嵌套

#### 文档的元数据

+ 元数据，用于标注文档的相关信息
  + _index 文档所属的索引名
  + _type 文档所属的类型名
  + _id 文档的唯一ID
  + _source 文档的原始JSON数据
  + _all 整合所有字段内容到该字段(目前已废除)
  + _version 文档的版本信息
  + _score 相关性打分

### 索引

+ Index 索引是文档的容器，是一类文档的结合
  + Index 体现了逻辑空间的概念：每个索引都有自己的Mapping定义，用于定义包含文档的字段名和字段类型
  + Shard 体现了物理空间的概念：索引中的数据分散在Shard上
+ 索引的Mapping与Settings
  + Mapping定义文档字段的类型
  + Setting定义不同的数据分布

### 与关系型数据库类比

| RDBMS  | ElasticSearch |
| ------ | ------------- |
| Table  | Index(Type)   |
| Row    | Document      |
| Column | Field         |
| Schema | Mapping       |
| SQL    | DSL           |

### 一些基本的API

```http
//查看索引相关信息
GET kibana_sample_data_ecommerce
//查看索引文档总数
GET kibana_sample_data_ecommerce/_count
//catAPI
//查看indices
GET /_cat/indices/kibana*?v&s=index
//查看状态为绿的索引
GET /_cat/indices?v&health=green
//按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc
//查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt
//每个索引占用的内存
GET /_cat/indices?v&h=i,tm&s=tm:desc
```

## ElasticSearch的节点，集群，分片和副本

### 分布式系统的可用性与扩展性

+ 高可用性
  + 服务可用性 - 允许有节点停止运行
  + 数据可用性 - 部分节点丢失，不会丢失数据
+ 可扩展性
  + 请求量提升 / 数据的不断增长(将数据分布到所有节点上)

### 分布式特性

+ ElasticSearch的分布式架构的好处
  + 存储的水平扩容
  + 提高系统的可用性，部分节点停止服务，整个集群的服务不受影响
+ ElasticSearch的分布式架构
  + 不同的集群通过不同的名字来区分，默认名字"elasticsearch"
  + 通过配置文件修改，或者在命令行中 -E cluster.name=shaking 进行设定
  + 一个集群可以有一个或者多个节点

### 节点

+ 节点是一个ElasticSearch的实例
  + 本质上就是一个Java进程
  + 一台机器上可以运行多个ElasticSearch进程，但是生产环境一般建议一台机器上只运行一个ElasticSearch实例
+ 每一个节点都有名字，通过配置文件配置，或者启动时候 -E node.name = node1指定(这时会覆盖ElasticSearch的yml对应配置文件)
+ 每一个节点在启动之后，会分配一个UID，保存在data目录下

### Master-eligible Nodes和Master Node

+ 每个节点启动后，默认就是一个Master eligible节点
  + 可以设置node.master:false禁止
+ Master-eligible节点可以参加选主流程，成为Master节点
+ 当一个节点启动时候，它将会自己选举成Master节点
+ 每个节点都保存了集群的状态，**只有Master节点才能修改集群的状态信息**
  + 集群状态(Cluster State)，维护了一个集群中必要的信息
    + 所有的节点信息
    + 所有的索引和其相关的Mapping与Setting信息
    + 分片的路由信息
  + 任意节点都能修改信息会导致数据的不一致性

### Data Node & Coordinating Node

+ Data Node
  + 可以保存数据的节点，叫做Data Node。负责保存分片数据。在数据扩展上起到至关重要的作用
+ Coordinating Node
  + 负责接收Client的请求，讲请求分发到合适的节点，最终把结果汇集到一起
  + 每个节点默认都起到了Coordinating Node的职责

### 其他的节点类型

+ Hot & Warm Node
  + 不同硬件配置的Data Node，用来实现Hot & Warm架构，降低集群部署的成本
+ Machine Learning Node
  + 负责跑机器学习的Job，用来做异常检测
+ Tribe Node
  + (5.3 开始使用Cross Cluster Search) Tribe Node连接到不同的ElasticSearch集群，并且支持讲这些集群当成一个独立的集群处理

### 配置节点类型

+ 开发环境中一个节点可以承担多种角色
+ 生产环境中，应该设置单一角色的节点(dedicated node)

| 节点类型          | 配置参数    | 默认值                                                    |
| ----------------- | ----------- | --------------------------------------------------------- |
| master eligible   | node.master | true                                                      |
| data              | node.data   | true                                                      |
| ingest            | node.ingest | true                                                      |
| coordinating only | 无          | 每个节点默认都时coordinating节点，设置其他类型全部为false |
| machine learning  | node.ml     | true(需要 enable x-pack)                                  |

### 分片(Primary Shard & Replica Shard)

+ 主分片，用以解决数据水平扩展的问题。通过主分片，可以将数据分布到集群内所有的节点之上
  + 一个分片是一个运行的Lucene的实例
  + 主分片数在索引创建时指定，后续不允许修改，除非Reindex
+ 副本，用以解决数据高可用的问题。分片是主分片的拷贝
  + 副本分片数，可以动态调整
  + 增加副本数，还可以在一定程度上提高服务的可用性(读取的吞吐)
+ 一个三节点的集群中，blogs索引的分片分布情况
  + 思考：增加一个节点或改大主分片数对系统的影响？

![image.png](https://i.loli.net/2020/03/11/lAXYKv2Fomk8Ntf.png)

### 分片的设定

+ 对于生产环境中分片的设定，需要提前做好容量规划
  + 分片数设置过小
    + 导致后续无法增加节点实现水平扩展
    + 单个分片的数据量太大，导致数据重新分配耗时
  + 分片数设置过大，7.0开始，默认主分片设置为1，解决了over - sharding问题
    + 影响搜索结果的相关性打分，影响统计结果的准确性
    + 单个节点上过多的分片，会导致资源浪费，同时也会影响性能

### 查看集群的健康状况

+ Green 主分片与副本都正常分配
+ Yellow 主分片全部正常分配，有副本分片未能正常分配
+ Red 有主分片未能分配
  + 例如，当服务器的磁盘容量超过85%时，去创建了一个新的索引

```http
GET _cluster/health
```

## ElasticSearch文档的基本CRUD与批量操作

### 文档的CRUD

| 操作类型 | 示例                                                         |
| -------- | ------------------------------------------------------------ |
| Index    | PUT my_index/_doc/1{"user":"shaking", "comment":"This is User!"} |
| Create   | PUT my_index/\_create/1{"user":"shaking", "comment":"This is User!"}<br />POST my_index/_doc(不指定ID，自动生成)<br />{"user":"shaking","comment":"This is User!"} |
| Read     | GET my_index/_doc/1                                          |
| Update   | POST my_index/_update/1{"doc":{"user":"shaking","comment":"This is User!"}} |
| Delete   | DELETE my_index/_doc/1                                       |

+ Type名，约定都用_doc
+ Create - 如果ID已存在，会失败
+ Index - 如果ID不存在，创建新的文档。否则先删除现有文档，在创建新的文档，版本会增加
+ Update - 文档必须已经存在，更新只会对响应字段做增量修改

### Create一个文档

+ 支持自动生成文档ID和指定文档ID俩种方式
+ 通过调用"post /user/_doc"
  + 系统会自动生成document ID
+ 使用HTTP PUT user/\_create/1时，URI中显示指定\_create，此时如果该ID的文档已经存在，操作失败

### Get一个文档

+ 找到文档，返回HTTP 200
  + 文档元信息
    + _index / _type / 
    + 版本信息，同一个ID的文档，即使被删除，Version号也会不断增加
    + _source中包含了文档的所有原始信息
+ 找不到文档返回404

![image.png](https://i.loli.net/2020/03/12/QqEk8BunGDJZ9CS.png)

### Index文档

+ Index和Create不一样的地方：如果文档不存在，就索引新的文档。否则现有文档会被删除，新的文档被索引。版本信息+ 1
+ 调用方式一个为POST，一个为PUT

![image.png](https://i.loli.net/2020/03/12/ub5tOs62THCvqdz.png)

### Update文档

+ Update方法不会删除原来的文档，而是实现真正的数据更新
+ Post方法 / payload需要包含在"doc"中

![image.png](https://i.loli.net/2020/03/12/qJL3BoG6u1lgvKi.png)

### 使用Kibana的Dev tools进行REST API的操作

```http
#create document，自动生成 _id
POST user/_doc
{
	"user":"shaking",
	"post_date":"2020-03-12T21:04:44",
	"message":"trying out Kibana"
}
```

![image.png](https://i.loli.net/2020/03/12/QdGKFujzcPEV4iD.png)

```http
#create document，指定ID。如果ID已存在，报错
PUT user/_doc/1?op_type=create
{
	"user":"shaking",
	"post_date":"2020-03-12T21:07:25",
	"message":"trying out Kibana"
}
```

> 第一次执行

![image.png](https://i.loli.net/2020/03/12/n8uQ5XZYUFPNvfq.png)

> 第二次执行

![image.png](https://i.loli.net/2020/03/12/n2rRuTAD9GW8Pgc.png)

```http
GET user/_doc/1
```

![image.png](https://i.loli.net/2020/03/12/rho397xwMcqTLbB.png)

```http
#Index方法
PUT user/_doc/1
{"user":"yuanyl"}
```

![image.png](https://i.loli.net/2020/03/12/yGF8czgoJ3XN9V1.png)

```http
GET user/_doc/1
```

![image.png](https://i.loli.net/2020/03/12/IkuRDPoxLTHeFrz.png)

```http
#在原文档上增加字段
POST user/_update/1/
{
	"doc":{
		"post_date":"2020-03-12T21:31:05",
		"message":"Trying out ElasticSearch!"
	}
}
```

![image.png](https://i.loli.net/2020/03/12/iXTW1OYH6973hMJ.png)

> 再次Get

![image.png](https://i.loli.net/2020/03/12/gJVbC3mpjKTxoGX.png)

### Bulk API

+ 支持在一次API调用中，对不同的索引进行操作
+ 支持四种类型操作
  + Index
  + Create
  + Update
  + Delete
+ 可以在URI中指定Index，也可以在请求的Payload中进行
+ 操作中单条操作失败，并不会影响其他操作
+ 返回结果包括了每一条操作的执行结果

```http
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

> 第一次调用结果

```json
{
  "took" : 354,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "delete" : {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 1,
        "result" : "not_found",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 404
      }
    },
    {
      "create" : {
        "_index" : "test2",
        "_type" : "_doc",
        "_id" : "3",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "update" : {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}
```

> 第二次执行结果

```json
{
  "took" : 15,
  "errors" : true,
  "items" : [
    {
      "index" : {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 3,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "delete" : {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 1,
        "result" : "not_found",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 4,
        "_primary_term" : 1,
        "status" : 404
      }
    },
    {
      "create" : {
        "_index" : "test2",
        "_type" : "_doc",
        "_id" : "3",
        "status" : 409,
        "error" : {
          "type" : "version_conflict_engine_exception",
          "reason" : "[3]: version conflict, document already exists (current version [1])",
          "index_uuid" : "UCONuTVTQbOpYvzaAfBLCA",
          "shard" : "0",
          "index" : "test2"
        }
      }
    },
    {
      "update" : {
        "_index" : "test",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 4,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 5,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}
```

### 批量读取 - mget

批量操作，可以减少网络连接所产生的开销，提高性能

```http
# mget 操作
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_id" : "2"
        }
    ]
}
```

![image.png](https://i.loli.net/2020/03/12/kZxsEUGJ5V1YTjd.png)

### 批量查询 - msearch

```http
# msearch 操作
POST kibana_sample_data_ecommerce/_msearch
{}
{"query" : {"match_all" : {}},"size":1}
{"index" : "kibana_sample_data_flights"}
{"query" : {"match_all" : {}},"size":2}
```

### 常见错误返回

| 问题         | 原因               |
| ------------ | ------------------ |
| 无法连接     | 网络故障或集群挂了 |
| 连接无法关闭 | 网络故障或节点出错 |
| 429          | 集群过于繁忙       |
| 4xx          | 请求体格式有错     |
| 500          | 集群内部错误       |

## 倒排索引

### 倒排索引的核心组成

+ 倒排索引包含两部分
  + 单词词典(Term Dictionary)，记录所有文档的单词，记录单词到倒排列表的关联关系
    + 单词词典一般比较大，可以通过B+树或哈希拉链法实现，以满足高性能的插入和查询
  + 倒排列表(Posting List)记录了单词对应的文档集合，由倒排索引项组成
    + 倒排索引项(Posting)
      + 文档ID
      + 词频 TF - 该单词在文档中出现的次数，用于相关性评分
      + 位置(Position) - 单词在文档中分词的位置。用于语句搜索(phrase query)
      + 偏移(Offset) - 记录单词的开始结束位置，实现高亮显示

### ElasticSearch的倒排索引

+ ElasticSearch的JSON文档中的每个字段，都有自己的倒排索引
+ 可以指定对某些字段不做索引
  + 优点：节省存储空间
  + 缺点：字段无法被搜索

## 通过Analyzer进行分词

### Analysis与Analyzer

+ Analysis - 文本分析是把全文本转换一些列单词(term / token)的过程，也叫分词
+ Analysis是通过Analyzer来实现的
  + 可以使用ElasticSearch内置的分词器 / 或者按需订制分词器
+ 除了在数据写入时转换词条，匹配Query语句的时候也需要用相同的分词器对查询语句进行分析

### Analyzer的组成

+ 分词器是专门处理分词的组件，Analyzer由三部分组成
  + Character Filters(针对原始文本处理，例如去除html)
  + Tokenizer(按照规则切分单词)
  + Token Filter(将切分的单词进行加工，小写，删除stopwords，增加同义词)

### ElasticSearch内置的分词器

+ Standard Analyzer - 默认分词器，按词切分，小写处理
+ Simple Analyzer - 按照非字母切分(符号被过滤)，小写处理
+ Stop Analyzer - 小写处理，停用词过滤(the, a, is)
+ Whitespace Analyzer - 按照空格切分，不转小写
+ Keyword Analyzer - 不分词，直接将输入当做输出
+ Patter Analyzer - 正则表达式，默认 \W+ (非字符分隔)
+ Language - 提供了30多种常见语言的分词器
+ Customer Analyzer 自定义分词器

### 使用_analyzer API

+ 直接指定Analyzer进行测试
+ 指定索引的字段进行测试
+ 自定义分词器进行测试

![image.png](https://i.loli.net/2020/03/14/2NPoBzvc9tI5KyH.png)

#### Standard Analyzer

![image.png](https://i.loli.net/2020/03/14/CYGsKHxlq34aIy6.png)

```http
#指定standard进行分词
GET _analyze
{
	"analyzer":"standard",
	"text":"2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

> 分词结果

```json
{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<NUM>",
      "position" : 0
    },
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "foxes",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "<ALPHANUM>",
      "position" : 9
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "<ALPHANUM>",
      "position" : 10
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "<ALPHANUM>",
      "position" : 11
    },
    {
      "token" : "evening",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "<ALPHANUM>",
      "position" : 12
    }
  ]
}
```

#### Simple Analyzer

![image.png](https://i.loli.net/2020/03/14/nZlxO56FLwuviar.png)

```http
#指定simple
GET _analyze
{
	"analyzer":"simple",
	"text":"2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

```json
{
  "tokens" : [
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "foxes",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "evening",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "word",
      "position" : 11
    }
  ]
}
```

#### Whitespace Analyzer

![image.png](https://i.loli.net/2020/03/14/zungOx1oZTSfYpX.png)

```http
#指定whitespace
GET _analyze
{
	"analyzer":"whitespace",
	"text":"2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

```json
{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "Quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "brown-foxes",
      "start_offset" : 16,
      "end_offset" : 27,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "evening.",
      "start_offset" : 62,
      "end_offset" : 70,
      "type" : "word",
      "position" : 11
    }
  ]
}
```

#### Stop Analyzer

+ 会把停用词过滤掉

![image.png](https://i.loli.net/2020/03/14/xDMJUIrpTkud3jl.png)

```http
#指定stop
GET _analyze
{
	"analyzer":"stop",
	"text":"2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

```json
{
  "tokens" : [
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "foxes",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "evening",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "word",
      "position" : 11
    }
  ]
}
```

#### Keyword Analyzer

```json
{
  "tokens" : [
    {
      "token" : "2 running Quick brown-foxes leap over lazy dogs in the summer evening.",
      "start_offset" : 0,
      "end_offset" : 70,
      "type" : "word",
      "position" : 0
    }
  ]
}
```

#### Pattern Analyzer

![image.png](https://i.loli.net/2020/03/14/TCGZeNWkBEHdl7a.png)

```json
{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "foxes",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 11
    },
    {
      "token" : "evening",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "word",
      "position" : 12
    }
  ]
}
```

#### Language

```http
#english
GET _analyze
{
	"analyzer":"english",
	"text":"2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

```json
{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<NUM>",
      "position" : 0
    },
    {
      "token" : "run",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "fox",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "lazi",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "dog",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "<ALPHANUM>",
      "position" : 11
    },
    {
      "token" : "even",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "<ALPHANUM>",
      "position" : 12
    }
  ]
}
```

#### ICU Analyzer

![image.png](https://i.loli.net/2020/03/14/n7dfmAxFNrCibze.png)

```http
POST _analyze
{
	"analyzer":"icu_analyzer",
	"text":"他说的确实在理"
}
```

```json
{
  "tokens" : [
    {
      "token" : "他",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "说的",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "确实",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "在",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "理",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    }
  ]
}
```

### 其他中文分词器

+ IK
  + 支持自定义词库，支持热更新分词字典
+ THULAC

## Search API概览

### 指定查询的索引

| 语法                   | 范围              |
| ---------------------- | ----------------- |
| /_search               | 集群上所有的索引  |
| /index1/_search        | index1            |
| /index1,index2/_search | index1和index2    |
| /index*/_search        | 以index开头的索引 |

### URI查询

+ 使用"q"，指定查询字符串
+ "query string syntax"，KV键值对

> curl XGET 
>
> "http://elasticsearch:9200/kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie"

### Request Body

![image.png](https://i.loli.net/2020/03/15/v8iGZBPwsFbN2Dg.png)

### 搜索Response

![image.png](https://i.loli.net/2020/03/15/aRkI4Ecz7KLmJtP.png)

### 搜索的相关性Relevance

+ 搜索是用户和搜索引擎的对话
+ 用户关心的是搜索结果的相关性
  + 是否可以找到所有相关的内容
  + 有多少不相关的内容被返回了
  + 文档的打分是否合理
  + 结合业务需求，平衡结果排名

### 衡量相关性

+ Information Retrieval
  + Precision(查准率) - 尽可能返回较少的无关文档
  + Recall(查全率) - 尽量返回较多的相关文档
  + Ranking - 是否能够按照相关度进行排序

























