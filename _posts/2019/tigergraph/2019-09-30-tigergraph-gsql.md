---
layout:     post
title:      "TigerGraph Gsql介绍"
subtitle:   " \"TigerGraph Gsql介绍\""
date:       2019-09-30 14:10:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> TigerGraph


## 基本类型

- INT
- UINT
- FLOAT
- DOUBLE
- BOOL
- STRING
- DATETIME

## 函数
parse_json_object(STRING str)  将str转换成JSON对象 JSONOBJECT

parse_json_array( STRING str ) 将str转换成JSON数组 JSONARRAY

## 控制流语句
### if else
```
IF x == 5 THEN y = 10;      
ELSE IF x == 7 THEN y = 5; 
ELSE y = 20; END;   
```
案例
```
CREATE QUERY countFriendsOf2(vertex<person> seed, BOOL includeCoworkers) FOR GRAPH friendNet
{
      SumAccum<INT> @@numFriends = 0;
      start = {seed};
 
      IF includeCoworkers THEN
         friends = SELECT v FROM start -((friend | coworker):e)-> :v
               ACCUM @@numFriends +=1;
      ELSE
         friends = SELECT v FROM start -(friend:e)-> :v
               ACCUM @@numFriends +=1;
      END;
      PRINT @@numFriends, includeCoworkers;
}
```
### case when
```
STRING drink = "Juice";
 CASE
 WHEN drink == "Juice" THEN @@calories += 50    
 WHEN drink == "Soda"  THEN @@calories += 120
 ...
 ELSE @@calories = 0       # Optional else-clause
END
```
### while
```
INT iter = 0;
WHILE v.size() !=0 DO
   IF iter == 5 THEN  BREAK;  END;
   # Some statements​
​iter = iter + 1;​
END;
```
### FOREACH
```
CREATE QUERY foreachRangeStep(INT a, INT b, INT c) FOR GRAPH minimalNet {
 ListAccum<INT> @@t;
 FOREACH i IN RANGE[a,b].step(c) DO
   @@t += i;
 END;
 PRINT @@t;
}
```
#### 查询主体集
```
CREATE QUERY employeesByCompany(STRING country) FOR GRAPH workNet {
 ListAccum<VERTEX<company>> @@companyList;
 start = {ANY};
 
 s = SELECT v FROM start:v
     WHERE v.type == "company"
     ACCUM @@companyList += v;
 
 FOREACH item IN @@companyList DO
   companyItem = {item};
   employees = SELECT t FROM companyItem -(worksFor)-> :t
               WHERE (t.locationId == country);
   PRINT employees;
 END;
}
```

## 访问属性
```
results = SELECT v FROM allVertices:s -(:e)-> :v WHERE v.type == "post" AND v.subject == "coffee";
   PRINT results;
```
## 顶点函数
```
int deg1, deg2, deg3, deg4;
 
 S = {m1};
 S2 = SELECT S
      FROM S - (posted:e) -> post:t
      ACCUM deg1 = S.outdegree(),
            deg2 = S.outdegree("posted"),
            deg3 = S.outdegree(e.type),  # same as deg2
            STRING str = "posted",
            deg4 = S.outdegree(str);  
```
## 删除修改语句
```
# Delete all "person" vertices with location equal to "us"
CREATE QUERY deleteEx() FOR GRAPH workNet {
 S = {person.*};
 DELETE s FROM S:s
   WHERE s.locationId == "us";
}
```
## 查询语句
### 基本查询
```
resultSet2 = SELECT v FROM S:v-(:e)->:t
​​​​​ WHERE t.type IN ("V1", "V2") AND
​​​​​​   t IN v.neighbors("E1|E2|E3")

resultSet3 = SELECT v FROM Source:v-(eType:e)->(V1|V2):t;
resultSet4 = SELECT v FROM Source:v-(eType:e)->:t;
resultSet5 = SELECT v FROM Source:v-(eType:e)->ANY:t;
resultSet6 = SELECT v FROM Source:v-(eType:e)->_:t;
resultSet7 = SELECT v FROM Source:v-((E1|E2|E3):e)->tType:t;
resultSet8 = SELECT v FROM Source:v-(:e)->tType:t;
resultSet9 = SELECT v FROM Source:v-(_:e)->tType:t;
resultSet10 = SELECT v FROM Source:v-(ANY:e)->tType:t;
```
### 
```
CREATE QUERY b(/* Parameters here */) FOR GRAPH aml {
 SetAccum<VERTEX> @@startOfVertices;
 SetAccum<VERTEX> @@endOfVertices;
 SetAccum<EDGE> @@postedSet;
 start = {CAccount.*};
 	
 Result = SELECT s
          FROM start:s-(PAY:e) -:tgt
          ACCUM  @@startOfVertices += s,
	        @@endOfVertices += tgt,
	        @@postedSet+=e;
	  PRINT @@startOfVertices+@@endOfVertices as Vertices;
	  PRINT @@postedSet;
}
```

```
CREATE QUERY topPopular() FOR GRAPH friendNet
{
​SumAccum<INT> @numFriends;
​SumAccum<INT> @numCoworkers;
​start = {person.*};
 
​result = SELECT v FROM start -((friend|coworker):e)-> person:v
​       ​ ACCUM CASE WHEN e.type == "friend" THEN v.@numFriends += 1
​​       ​    WHEN e.type == "coworker" THEN v.@numCoworkers += 1
​​       END
​​ ORDER BY v.@numFriends DESC, v.@numCoworkers DESC;
 
​PRINT result;
}
```
## 分页
```
result = SELECT v FROM S -(:e)-> :v LIMIT k;​​  
result = SELECT v FROM S -(:e)-> :v LIMIT j, k; ​  
result = SELECT v FROM S -(:e)-> :v LIMIT k OFFSET j;
```
## 累加器
```
 SumAccum<INT>    @@intAccum;
 SumAccum<FLOAT>  @@floatAccum;
 SumAccum<DOUBLE> @@doubleAccum;
 SumAccum<STRING> @@stringAccum;
 MinAccum<INT> @@minAccum;
 MaxAccum<FLOAT> @@maxAccum;
 AvgAccum @@averageAccum;
 ListAccum<INT> @@intListAccum;
 SetAccum<INT> @@intSetAccum;
 MapAccum<STRING, INT> @@intMapAccum;
```
### 深度查询
```
# return the k-hop neighbors count.
CREATE QUERY khop(VERTEX<MyNode> start_node, INT depth) for graph Twitter {

    OrAccum          @visited = false;
    SumAccum<int>    @@loop=0;

    Start = {start_node};
    Start = SELECT v
            FROM Start:v
            ACCUM v.@visited = true;

    WHILE (@@loop < depth) DO
        Start = SELECT v
                FROM Start:u - (MyEdge:e)->:v
                WHERE v.@visited == false
                ACCUM v.@visited = true;

        @@loop += 1;
   END;

   PRINT Start.size();
}
run query khop ("618593",8)
```
#### 例子
```
CREATE QUERY deepPath(INT depth,VERTEX<CCustomer> start_node) FOR GRAPH aml { 
	 OrAccum          @visited = false;
    SumAccum<int>    @@loop=0;
    SetAccum<VERTEX> @@startVertices;
	 SetAccum<VERTEX> @@endVertices;
    SetAccum<EDGE> @@edges;
	  
    StartCCustomer = {start_node};
	  Start = {ANY};
    Start = SELECT v
            FROM StartCCustomer:v
            ACCUM v.@visited = true;

    WHILE (@@loop < depth) DO
        Start = SELECT v
                FROM Start:u - (:e)->:v
                WHERE v.@visited == false
                ACCUM v.@visited = true,
	              @@edges+=e,
	              @@startVertices+=Start,
	              @@endVertices+=v;
        @@loop += 1;
   END;

   PRINT  @@edges;
	PRINT  @@startVertices union  @@endVertices;
}
```

## 其他函数
```
SelectVertex()读取一个包含特定顶点列表的数据文件，并返回相应的顶点集。
```







