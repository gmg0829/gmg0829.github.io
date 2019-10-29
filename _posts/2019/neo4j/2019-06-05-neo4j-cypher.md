---
layout:     post
title:      "neo4j cypher 介绍"
subtitle:   " \"neo4j cypher 介绍\""
date:       2019-06-05 13:10:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> neo4j


# cypher 

##  CREATE命令

- 创建没有属性的节点

- 使用属性创建节点

- 在没有属性的节点之间创建关系

- 使用属性创建节点之间的关系

- 为节点或关系创建单个或多个标签

### 例子

- 创建一个标签，即“Dept”
- 创建一个节点，即“dept”
- 创建三个属性，即deptno，dname，location

```
CREATE (dept:Dept { deptno:10,dname:"Accounting",location:"Hyderabad" })
```

##   MATCH & RETURN匹配和返回

- 检索节点的某些属性
- 检索节点的所有属性
- 检索节点和关联关系的某些属性
- 检索节点和关联关系的所有属性

### 例子

- dept是节点名称
- 这里Dept是一个节点标签名
- deptno是dept节点的属性名称
- dname是dept节点的属性名

```
MATCH (dept: Dept)
RETURN dept.deptno,dept.dname
```

##  CREATE+MATCH+RETURN命令

例如：本示例演示如何使用属性和这两个节点之间的关系创建两个节点。

- 客户节点包含：ID，姓名，出生日期属性

- CreditCard节点包含：id，number，cvv，expiredate属性

- 客户与信用卡关系：DO_SHOPPING_WITH

- CreditCard到客户关系：ASSOCIATED_WITH


我们将在以下步骤中处理此示例： 

- 创建客户节点
- 创建CreditCard节点
- 观察先前创建的两个节点：Customer和CreditCard
- 创建客户和CreditCard节点之间的关系
- 查看新创建的关系详细信息
- 详细查看每个节点和关系属性

```
CREATE (e:Customer{id:"1001",name:"Abc",dob:"01/10/1982"})

MATCH (e:Customer)
RETURN e.id,e.name,e.dob

CREATE (cc:CreditCard{id:"5001",number:"1234567890",cvv:"888",expiredate:"20/17"})

MATCH (cc:CreditCard)
RETURN cc.id,cc.number,cc.cvv,cc.expiredate
```

## 关系基础

基于方向性，Neo4j关系被分为两种主要类型。

- 单向关系
- 双向关系

### 没有属性的关系与现有节点

- 这里关系名称为“DO_SHOPPING_WITH”
- 关系标签为“r”。
- e和Customer分别是客户节点的节点名称和节点标签名称。
- cc和CreditCard分别是CreditCard节点的节点名和节点标签名。

```
MATCH (e:Customer),(cc:CreditCard) 
CREATE (e)-[r:DO_SHOPPING_WITH ]->(cc) 

MATCH (e)-[r:DO_SHOPPING_WITH ]->(cc) 
RETURN r

```

### 与现有节点的属性的关系

```
MATCH (cust:Customer),(cc:CreditCard) 
CREATE (cust)-[r:DO_SHOPPING_WITH{shopdate:"12/12/2014",price:55000}]->(cc) 
RETURN r
```

- 这里关系名称为“DO_SHOPPING_WITH”
- 关系标签为“r”。
- shopdate和price是关系“r”的属性。
- e和Customer分别是客户节点的节点名称和节点标签名称。
- cc和CreditCard分别是CreditCard节点的节点名和节点标签名。

### 新节点无属性关系

```
CREATE (fb1:FaceBookProfile1)-[like:LIKES]->(fb2:FaceBookProfile2) 

MATCH (fb1:FaceBookProfile1)-[like:LIKES]->(fb2:FaceBookProfile2) 
RETURN like
```

### 新节点与属性的关系
```
CREATE (video1:YoutubeVideo1{title:"Action Movie1",updated_by:"Abc",uploaded_date:"10/10/2010"})
-[movie:ACTION_MOVIES{rating:1}]->
(video2:YoutubeVideo2{title:"Action Movie2",updated_by:"Xyz",uploaded_date:"12/12/2012"}) 

MATCH (video1:YoutubeVideo1)-[movie:ACTION_MOVIES]->(video2:YoutubeVideo2) 
RETURN movie
```
## CREATE创建标签

这里m是一个节点名

Movie, Cinema, Film, Picture是m节点的多个标签名称

```
CREATE (m:Movie:Cinema:Film:Picture)
```

## 检索关系节点的详细信息

```
MATCH (cust)-[r:DO_SHOPPING_WITH]->(cc) 
RETURN cust,cc
```

## WHERE子句

```
MATCH (emp:Employee) 
WHERE emp.name = 'Abc'
RETURN emp

MATCH (emp:Employee) 
WHERE emp.name = 'Abc' OR emp.name = 'Xyz'
RETURN emp
```

## DELETE删除
- 删除节点。
- 删除节点及相关节点和关系。
```
MATCH (e: Employee) DELETE e
MATCH (e: Employee) RETURN e
```
我们应该使用逗号（，）运算符来分隔节点名称和关系名称

```
DELETE <node1-name>,<node2-name>,<relationship-name>

MATCH (cc: CreditCard)-[rel]-(c:Customer) 
DELETE cc,c,rel
```

## REMOVE删除

有时基于我们的客户端要求，我们需要向现有节点或关系添加或删除属性。

- 我们使用Neo4j CQL SET子句向现有节点或关系添加新属性。
- 我们使用Neo4j CQL REMOVE子句来删除节点或关系的现有属性。

Neo4j CQL REMOVE命令用于

- 删除节点或关系的标签
- 删除节点或关系的属性

Neo4j CQL DELETE和REMOVE命令之间的主要区别 - 

- DELETE操作用于删除节点和关联关系。
- REMOVE操作用于删除标签和属性。

从书节点中删除“price”属性

```
CREATE (book:Book {id:122,title:"Neo4j Tutorial",pages:340,price:250}) 
MATCH (book { id:122 })
REMOVE book.price
RETURN book
```
##  SET子句

- 向现有节点或关系添加新属性
- 添加或更新属性值

```
MATCH (dc:DebitCard)
SET dc.atm_pin = 3456
RETURN dc
```

##  Sorting排序

```
MATCH (emp:Employee)
RETURN emp.empid,emp.name,emp.salary,emp.deptno
ORDER BY emp.name DESC
```

## UNION
与SQL一样，Neo4j CQL有两个子句，将两个不同的结果合并成一组结果
- UNION
- UNION ALL

```
MATCH (cc:CreditCard)
RETURN cc.id as id,cc.number as number,cc.name as name,
   cc.valid_from as valid_from,cc.valid_to as valid_to
UNION
MATCH (dc:DebitCard)
RETURN dc.id as id,dc.number as number,dc.name as name,
   dc.valid_from as valid_from,dc.valid_to as valid_to
```

##  LIMIT和SKIP子句
LIMIT: 只返回Top的两个结果，因为我们定义了limit = 2。这意味着前两行。

```
MATCH (emp:Employee) 
RETURN emp
LIMIT 2
```

SKIP: 它只返回来自Bottom的两个结果，因为我们定义了skip = 2。这意味着最后两行。
```
MATCH (emp:Employee) 
RETURN emp
SKIP 2
```
## 合并

MERGE命令是CREATE命令和MATCH命令的组合。

Neo4j CQL MERGE命令在图中搜索给定模式，如果存在，则返回结果
如果它不存在于图中，则它创建新的节点/关系并返回结果。

```
MERGE (gp2:GoogleProfile2{ Id: 201402,Name:"Nokia"})
MERGE (gp2:GoogleProfile2{ Id: 201402,Name:"Nokia"})
MATCH  (gp1:GoogleProfile2) 
RETURN gp1.Id,gp1.Name
```

CQL CREATE命令检查此节点是否可用，它只是在数据库中创建新节点。
CQL MERGE命令将新的节点添加到数据库，只有当它不存在。

## NULL值

```
MATCH (e:Employee) 
WHERE e.id IS NOT NULL
RETURN e.id,e.name,e.sal,e.deptno
```

## IN操作符

```
MATCH (e:Employee) 
WHERE e.id IN [123,124]
RETURN e.id,e.name,e.sal,e.deptno
```


## cql函数

### 字符串函数
```
MATCH (e:Employee) 
RETURN e.id,UPPER(e.name),e.sal,e.deptno
```
### AGGREGATION聚合
 它类似于SQL中的GROUP BY子句。

 ```
 MATCH (e:Employee) RETURN COUNT(*)
 ```
 ### 关系函数
 
 以在获取开始节点，结束节点等细节时知道关系的细节。
 - STARTNODE 它用于知道关系的开始节点。
 - ENDNODE 它用于知道关系的结束节点。
 - ID 	它用于知道关系的ID。
 - TYPE  它用于知道字符串表示中的一个关系的TYPE。

```
 MATCH (video1:YoutubeVideo1)-[movie:ACTION_MOVIES]->(video2:YoutubeVideo2) 
 RETURN movie

MATCH (a)-[movie:ACTION_MOVIES]->(b) 
RETURN STARTNODE(movie)

RETURN ENDNODE(movie)
RETURN ID(movie),TYPE(movie)
```
## CQL - 索引

Neo4J索引操作
- Create Index 创建索引
- Drop Index 丢弃索引

```
CREATE INDEX ON :Customer (name)
DROP INDEX ON :Customer (name)
```
## UNIQUE约束

```
CREATE CONSTRAINT ON (cc:CreditCard)
ASSERT cc.number IS UNIQUE

DROP CONSTRAINT ON (cc:CreditCard)
ASSERT cc.number IS UNIQUE
```

参考neo4j docs: https://neo4j.com/docs/

https://neo4j.com/docs/cypher-refcard/current/