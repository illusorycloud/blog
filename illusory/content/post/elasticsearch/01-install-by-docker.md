---
title: "Elasticsearch 系列(一)--使用docker-compose快速搭建 elasticsearch"
description: "elasticsearch 开发环境搭建"
date: 2020-03-20 22:00:00
draft: false
tags: ["elasticsearch"]
categories: ["elasticsearch"] 
---

本文主要记录了 Elasticsearch、Kibana 的安装部署流程，及其目录简单介绍。同时也记录了 Elasticsearch 分词器的安装即介绍等。

> 学之前首先还是需要把环境搭起来才方便嘛。

<!--more-->

## 1. 概述

**Elasticsearch** 是一个分布式的**开源搜索和分析引擎**，在 Apache Lucene 的基础上开发而成。以其简单的 REST 风格 API、分布式特性、速度和可扩展性而闻名，是 Elastic Stack 的核心组件

> ELK 中的 E

**Kibana**是一个针对 Elasticsearch 的**开源分析及可视化平台**，用来搜索、查看交互存储在Elasticsearch 索引中的数据。使用 Kibana，可以通过各种图表进行高级数据分析及展示。

> ELK 中的 K

Logstash TODO

## 2. 解压方式运行

### 1. 安装

`7.0` 版本开始已经自带 JDK 了，因此不需要准备 Java 环境。

下载压缩文件，解压即可使用。

官网地址

```text
https://www.elastic.co/cn/downloads/elasticsearch
```

下载后并解压，执行

```shell
./bin/elasticsearch
```

即可启动 elasticsearch。

> 不能用 root 启动 否则会报错 
>
> uncaught exception in thread [main]
> java.lang.RuntimeException: can not run elasticsearch as root

运行起来后浏览器访问  `http://localhost:9200` 应该可以看到以下信息

```json
{
  "name" : "docker",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "QWvfb7QyQr2rbdOO7iMFcg",
  "version" : {
    "number" : "7.8.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "757314695644ea9a1dc2fecd26d1a43856725e65",
    "build_date" : "2020-06-14T19:35:50.234439Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```

### 2. 目录文件结构

| 目录    | 配置文件          | 描述                                                         |
| ------- | ----------------- | ------------------------------------------------------------ |
| bin     |                   | 脚本文件，包括启动 Elasticsearch、安装插件，运行统计数据等。 |
| config  | elasticsearch.yml | 集群配置文件                                                 |
| JDK     |                   | Java 运行环境                                                |
| data    | path.data         | 数据文件                                                     |
| lib     |                   | Java 类库                                                    |
| logs    | path.logs         | 日志文件                                                     |
| modules |                   | 包含所有 ES 模块                                             |
| plugins |                   | 包含所有已安装插件                                           |

### 3. Kibana

同样的，下载压缩文件，直接解压后运行即可。

官网

```text
https://www.elastic.co/cn/downloads/kibana
```

执行

```shell
.//bin/kibana
```

即可启动，浏览器访问

```shell
http://localhost:5601
```

## 3. Docker 安装

使用 docker-compose 一键安装

> docker-compose 安装 [看这里](https://www.lixueduan.com/categories/Docker/)

### 1. 环境准备

调整用户 mmap 计数

> 否则启动时可能会出现内存不足的情况
查看当前限制

```shell
[root@iZ2zeahgpvp1oasog26r2tZ vm]# sysctl vm.max_map_count
vm.max_map_count = 65530
```

临时修改 - 不需要重启

```shell
[root@iZ2zeahgpvp1oasog26r2tZ vm]# sysctl -w vm.max_map_count=262144
vm.max_map_count = 262144
```

永久修改 - 需要重启

```shell
vi /etc/sysctl.cof
# 增加 如下内容
vm.max_map_count = 262144
```

**安装这些 大概需要 4GB 内存，否则可能无法启动**。

同时需要提前创建相关目录,大概结构是这样的

```shell
es/
  /data
  /plugins
   --docker-compose.yaml
```

### 2. docker-compose.yaml

```yaml
version: '2.2'
services:
  # cerebro 是一个简单的 ES 监控工具
  cerebro:
    image: lmenezes/cerebro:0.9.2
    container_name: cerebro
    ports:
      - "9000:9000"
    command:
      - -Dhosts.0.host=http://elasticsearch:9200
    networks:
      - es7net
  kibana:
    image: docker.elastic.co/kibana/kibana:7.8.0
    container_name: kibana7
    environment:
      - I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - es7net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
    container_name: es7_01
    environment:
      - cluster.name=dockeres
      - node.name=es7_01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type = single-node 
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data:/usr/share/elasticsearch/data
      - ./plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    networks:
      - es7net

networks:
  es7net:
    driver: bridge
```

### 3. 启动

一条命令即可启动

```shell
# -d 参数 指定后台启动
docker-compose up -d
```

## 4. 分词器安装

### 1. 安装

分词器安装很简单，一条命令搞定

```shell
./elasticsearch-plugin install url
```

其中 url 为对应分词器的下载地址

比如安装 ik 分词器

```shell
./elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.8.0/elasticsearch-analysis-ik-7.8.0.zip
```

常用分词器列表

* ​	IK 分词器
  * `https://github.com/medcl/elasticsearch-analysis-ik`
* 拼音分词器
  * `https://github.com/medcl/elasticsearch-analysis-pinyin`
* 其他
  * 还有很多分词器，不过这两个应该是比较常用的

**分词器版本需要和elasticsearch版本对应，并且安装完插件后需重启Es，才能生效**

**分词器版本需要和elasticsearch版本对应，并且安装完插件后需重启Es，才能生效**

**分词器版本需要和elasticsearch版本对应，并且安装完插件后需重启Es，才能生效**

插件安装其实就是下载 zip 包然后解压到 plugins 目录下。

Docker 安装的话可以通过 Volume 的方式放在宿主机。

比如前面创建的 plugins 目录就是存放分词器的，Elasticsearch 启动时会自动加载 该目录下的分词器。

### 2. 测试 

在 Kibana 中通过 Dev Tools 可以方便的执行各种操作。

```shell
#ik_max_word 会将文本做最细粒度的拆分
#ik_smart 会做最粗粒度的拆分
#pinyin 拼音
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": ["剑桥分析公司多位高管对卧底记者说，他们确保了唐纳德·特朗普在总统大选中获胜"]
} 
```

