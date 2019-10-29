---
layout:     post
title:      "初步了解ArangoDB"
subtitle:   " \"初步了解ArangoDB\""
date:       2019-10-01 11:10:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> ArangoDB

# ArangoDB
ArangoDB is a native multi-model database. Multi-model because ArangoDB provides the capabilities of a graph database, a document database, a key-value store in one C++ core. 

ArangoDB 是一个开源的分布式原生多模型数据库，是兼有图 (graph)、文档 (document)和键/值对 (key/value) 三种数据模型的 NoSQL 数据库。ArangoDB 使用类SQL的查询语言(AQL)构建出高性能应用程序。

ArangoDB 支持在 Windows、Linux、Dcoker、Mac 等多种系统下运行。本文以linux系统和docker为例安装。

## Linux 下 ArangoDB 的安装

### docker 安装
```
 docker run -p 8529:8529 -e ARANGO_ROOT_PASSWORD=gmg0829 arangodb/arangodb     
```

## rpm 安装

参考： https://my.oschina.net/electrician/blog/1799300



