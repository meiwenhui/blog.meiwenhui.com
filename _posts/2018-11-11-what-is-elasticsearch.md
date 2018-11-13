---
layout: post
title: "ElasticSearch 是个什么鬼"
description: "ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口"
keywords: "ElasticSearch,es"
---

### 安装
用docker-compose.yml安装
```
version: '2.0'
services:
  elasticsearch:
    image: docker.io/elasticsearch:6.4.2
    container_name: es1
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /docker/es1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - esnet
  elasticsearch2:
    image: docker.io/elasticsearch:6.4.2
    container_name: es2
    depends_on:
      - elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /docker/es2:/usr/share/elasticsearch/data
    networks:
      - esnet

  kibana:
    image: docker.io/kibana:6.4.2
    container_name: kibana
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch
    environment:
      - "ELASTICSEARCH_URL=http://es1:9200"
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local

networks:
  esnet:
```

在root下修改/etc/sysctl.conf
```
sysctl -w vm.max_map_count=262144
// 或一劳永逸
sed -i '$a\vm.max_map_count=262144' /etc/sysctl.conf
```
启动
```
docker-compose up -d
```
然后就可以看到
```
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                                            NAMES
f0f6a32635ce        docker.io/kibana:6.4.2          "/usr/local/bin/ki..."   24 hours ago        Up 24 hours         0.0.0.0:5601->5601/tcp                           kibana
370a9de68024        docker.io/elasticsearch:6.4.2   "/usr/local/bin/do..."   24 hours ago        Up 24 hours         9200/tcp, 9300/tcp                               es2
83649c1c095b        docker.io/elasticsearch:6.4.2   "/usr/local/bin/do..."   24 hours ago        Up 24 hours         0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   es1
```

### IK分词器插件
地址
```
https://github.com/medcl/elasticsearch-analysis-ik
```
进入到容器中
```
docker exec -it 83649c1c095b /bin/sh
```
然后将目录切换到elasticsearch目录
```
sh-4.2# cd /usr/share/elasticsearch/
//然后
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.2/elasticsearch-analysis-ik-6.4.2.zip
```

重启es1,es2容器即可生效

##### 测试分词是否成功
```
GET _analyze?pretty
{
  "analyzer" : "ik_smart",
  "text" : "江苏凤凰文艺出版社"
}
```
输出如下说明成功
```
{
  "tokens": [
    {
      "token": "江苏",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "凤凰",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "文艺出版社",
      "start_offset": 4,
      "end_offset": 9,
      "type": "CN_WORD",
      "position": 2
    }
  ]
}
```

#### 几个核心概念
*   索引  
索引(index)是ElasticSearch存放具体数据的地方，是一类具有相似特征的文档的集合。ElasticSearch中索引的概念具有不同意思，这里的索引相当于关系数据库中的一个数据库实例。在ElasticSearch中索引还可以作为动词，表示对数据进行索引操作。

*   类型  
在6.0之前的版本，一个ElasticSearch索引中，可以有多个类型；从6.0版本开始，，一个ElasticSearch索引中，只有1个类型。一个类型是索引的一个逻辑上的分类，通常具有一组相同字段的文档组成。ElasticSearch的类型概念相当于关系数据库的数据表。

*   文档  
文档是ElasticSearch可被索引的基础逻辑单元，相当于关系数据库中数据表的一行数据。ElasticSearch的文档具有JSON格式，由多个字段组成，字段相当于关系数据库中列的概念。

*   分片  
当数据量较大时，索引的存储空间需求超出单个节点磁盘容量的限制，或者出现单个节点处理速度较慢。为了解决这些问题，ElasticSearch将索引中的数据进行切分成多个分片（shard），每个分片存储这个索引的一部分数据，分布在不同节点上。当需要查询索引时，ElasticSearch将查询发送到每个相关分片，之后将查询结果合并，这个过程对ElasticSearch应用来说是透明的，用户感知不到分片的存在。
一个索引的分片一旦指定，不再修改。

*   副本  
其实，分片全称是主分片，简称为分片。主分片是相对于副本来说的，副本是对主分片的一个或多个复制版本（或称拷贝），这些复制版本（拷贝）可以称为复制分片，可以直接称之为副本。当主分片丢失时，集群可以将一个副本升级为新的主分片。


https://blog.csdn.net/chengyuqiang/column/info/18392
#### Cheers!
