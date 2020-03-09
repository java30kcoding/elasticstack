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
kibana-plugin list
kibana remove
```

### DevTools























