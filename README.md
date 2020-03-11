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
docker-compose up
docker-compose down
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
| SQL    |               |

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
+ 每一个节点都有名字，通过配置文件配置，或者启动时候 -E node.name = node1指定
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















































