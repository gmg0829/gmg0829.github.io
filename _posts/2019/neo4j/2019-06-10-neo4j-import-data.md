---
layout:     post
title:      "neo4j 数据导入"
subtitle:   " \"neo4j 数据导入\""
date:       2019-06-10 18:20:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> neo4j


# neo4j数据导入
- load csv
- admin-import 或 neo4j-import

## load csv

 - 适用场景：0 ~ 1000w
 - 速度： 一般 5000/s
 - 优点：可以加载本地/远程CSV;可实时插入
 - 缺点:导入速度较慢;需要将数据转换成csv

### 示例
node.csv

```
b6b0ea842890425588d4d3cfb38139a9,"文烁"
5099c4f943d94fa1873165e3f6f3c2fb,"齐贺喜"
c83ed0ae9fb34baa956a42ecf99c8f6e,"李雄"
e62d1142937f4de994854fa1b3f0670a,"房玄龄"
```
脚本
```
using periodic commit 10000 load csv from "file:/node.csv" as line create (:test {id:line[0], name:line[1]})

```

node.csv
```
uuid,name,Label
b6b0ea842890425588d4d3cfb38139a9,"文烁",Label1
5099c4f943d94fa1873165e3f6f3c2fb,"齐贺喜",Label3
c83ed0ae9fb34baa956a42ecf99c8f6e,"李雄",Label2
e62d1142937f4de994854fa1b3f0670a,"房玄龄",Label
```

脚本
```
using periodic commit 10000 load csv with headers from "file:/neo4j/node.csv" as line with line create (:Test {uuid:line.uuid, name:line.name});
```


## admin-import 或 neo4j-import

 - 适用场景：千万以上 nodes
 - 速度：非常快 (xw/s)
 - 优点：官方出品，占用更少的资源
 - 缺点:需要转成CSV；必须停止neo4j；只能生成新的数据库，而不能在已存在的数据库中插入数据。
### admin-import 基本语法
```
neo4j-admin import [--mode=csv] [--database=<name>]
                          [--additional-config=<config-file-path>]
                          [--report-file=<filename>]
                          [--nodes[:Label1:Label2]=<"file1,file2,...">]
                          [--relationships[:RELATIONSHIP_TYPE]=<"file1,file2,...">]
                          [--id-type=<STRING|INTEGER|ACTUAL>]
                          [--input-encoding=<character-set>]
                          [--ignore-extra-columns[=<true|false>]]
                          [--ignore-duplicate-nodes[=<true|false>]]
                          [--ignore-missing-nodes[=<true|false>]]
                          [--multiline-fields[=<true|false>]]
                          [--delimiter=<delimiter-character>]
                          [--array-delimiter=<array-delimiter-character>]
                          [--quote=<quotation-character>]
                          [--max-memory=<max-memory-that-importer-can-use>]
                          [--f=<File containing all arguments to this import>]
                          [--high-io=<true/false>]
```
- --ignore-extra-columns=true 忽略多余列参数
- --ignore-missing-nodes=true 忽略失去节点参数
- --ignore-duplicate-nodes=true 忽略重复节点参数


### 导入数据示例

#### 示例一
三个csv

movies.csv

```
movie:ID,name,:LABEL
tt0133093,The Matrix,movie
tt0234215,The Matrix Reloaded,movie
tt0242653,The Matrix Revolutions,movie
```

actors.csv

```
person:ID,name,:LABEL
keanu,Keanu Reeves,person
laurence,Laurence Fishburne,person
carrieanne,Carrie-Anne Moss,person
```

roles.csv

```
:START_ID,role,:END_ID
keanu,Neo,tt0133093
keanu,Neo,tt0234215
keanu,Neo,tt0242653
laurence,Morpheus,tt0133093
laurence,Morpheus,tt0234215
laurence,Morpheus,tt0242653
carrieanne,Trinity,tt0133093
```

neo4j-import 导入
```
.\bin\neo4j-import --into data\databases\graph.db --nodes .\import\practice\actors.csv --nodes .\import\practice\movies.csv --relationships:ACTED_IN .\import\practice\roles.csv --skip-duplicate-nodes=true --skip-bad-relationships=true --stacktrace --bad-tolerance=500000

```

neo4j-admin 导入

```
.\bin\neo4j-admin import --database=graph.db --nodes .\import\practice\actors.csv --nodes .\import\practice\movies.csv --relationships:ACTED_IN .\import\practice\roles.csv 
```
#### 示例二

movies3-header.csv
```
movieId:ID,title,year:int,:LABEL
```

movies3.csv
```
tt0133093,"The Matrix",1999,Movie
tt0234215,"The Matrix Reloaded",2003,Movie;Sequel
tt0242653,"The Matrix Revolutions",2003,Movie;Sequel
```

actors3-header.csv
```
personId:ID,name,:LABEL
```

actors3.csv
```
keanu,"Keanu Reeves",Actor
laurence,"Laurence Fishburne",Actor
carrieanne,"Carrie-Anne Moss",Actor
```

roles3-header.csv
```
:START_ID,role,:END_ID,:TYPE
```

roles3.csv
```
keanu,"Neo",tt0133093,ACTED_IN
keanu,"Neo",tt0234215,ACTED_IN
keanu,"Neo",tt0242653,ACTED_IN
laurence,"Morpheus",tt0133093,ACTED_IN
laurence,"Morpheus",tt0234215,ACTED_IN
laurence,"Morpheus",tt0242653,ACTED_IN
carrieanne,"Trinity",tt0133093,ACTED_IN
carrieanne,"Trinity",tt0234215,ACTED_IN
carrieanne,"Trinity",tt0242653,ACTED_IN
```

导入脚本
```
neo4j_home$ bin/neo4j-admin import --nodes="import/movies3-header.csv,import/movies3.csv" --nodes="import/actors3-header.csv,import/actors3.csv" --relationships="import/roles3-header.csv,import/roles3.csv"
```
#### 示例三
```
neo4j-admin import --mode=csv --database=userMovie.db --nodes data_test\movies.csv --nodes data_test\actors.csv --relationships data_test\roles.csv


bin/neo4j-admin import --nodes:Movie import/movie_node.csv --relationships:ACTED_IN=import/roles5b.csv
```

### 注意

#### 注意一

连接据方式：建立软连接
```
move graph.db graph_copy.db

mklink /D graph.db userMovie.db windows

ln -s graph.db userMovie.db
```
#### 注意二

通过neo4j-admin方式导入的话，需要暂停服务，并且需要清除graph.db,这样才能导入进去数据。而且，只能在初始化数据时，导入一次之后，就不能再次导入。

所以这种方式，可以在初次建库的时候，导入大批量数据，等以后如果还需要导入数据时，可以采用上边的方法。

#### 注意三

所以最好把csv文件放到import目录下，注意，事先，进入$NEO_HOME/conf/neo4j.conf配置文件并取消这一行的注释：

dbms.directories.import=import

开启引入文件

```
apoc.import.file.enabled=true

dbms.security.procedures.unrestricted=apoc.*
dbms.security.allow_csv_import_from_file_urls=true
```
#### 注意四

因为neo4j是utf-8的，而CSV默认保存是ANSI的，需要用记事本另存为成UTF-8的

#### 注意五

在neo4j中，虽然有一个自增的id属性，但是要想使用它还是很麻烦的，尤其是在web管理端

因此在使用CSV创建关系时，需要我们自己指定或添加一个属性来作为“主键”，在创建关系时根据该属性来获取节点，并添加关系。






