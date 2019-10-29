---
layout:     post
title:      "初步了解TigerGraph"
subtitle:   " \"初步了解TigerGraph\""
date:       2019-09-20 13:20:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> TigerGraph

# TigerGraph
通过其原生并行图技术，TigerGraph代表了图数据库演进的大趋势：一个完整的，分布式的并行计算平台，支持互联网规模的实时数据分析。 

TigerGraph 采用最新的高效大数据平台的设计理念(大规模并行处理架构和快速数据压缩算法)从零打造，为您提供了一个高速的、易扩展的，可进行深度探索和查询数据商业价值的企业级图数据平台。


## 创建点和边

```
CREATE VERTEX person (PRIMARY_ID name STRING, name STRING, age INT, gender STRING, state STRING)

CREATE UNDIRECTED EDGE friendship (FROM person, TO person, connect_day DATETIME)

```
## 创建图

```
CREATE GRAPH social (person, friendship)
```
## 载入数据

```
USE GRAPH social
BEGIN
CREATE LOADING JOB load_social FOR GRAPH social {
  DEFINE FILENAME file1="/home/tigergraph/person.csv";
  DEFINE FILENAME file2="/home/tigergraph/friendship.csv";
 
  LOAD file1 TO VERTEX person VALUES ($"name", $"name", $"age", $"gender", $"state") USING header="true", separator=",";
  LOAD file2 TO EDGE friendship VALUES ($0, $1, $2) USING header="true", separator=",";
}
END
RUN LOADING JOB load_social
```
## 执行查询
```
use  graph social

SELECT count() FROM person

````

##  gadmin

```
查看服务状态：gadmin status  ##如果某个服务出现问题，可以先用此条命令查看状态
启动服务： gadmin start
停止服务： gadmin stop -fy
重启服务： gadmin restart -fy
修改配置： gadmin --configure
应用配置： gadmin config-apply
修改runtime变量： gadmin --configure runtime
查看license有效期： gadmin status license
更新license： gadmin set-license-key [new_license]
查看gse log位置： gadmin log -v gse
查看节点和边总个数： gadmin status -v graph
详细信息可以使用 gadmin -h 查看
```
## 参考
https://blog.csdn.net/haveanybody/article/details/86720591

https://blog.csdn.net/weixin_33725515/article/details/86717929

https://service.tigergraph.com.cn/



