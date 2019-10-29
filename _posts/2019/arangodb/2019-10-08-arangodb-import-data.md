---
layout:     post
title:      "ArangoDB导入数据"
subtitle:   " \"ArangoDB导入数据\""
date:       2019-10-08 12:01:00
author:     "gmg"
catalog: true
tags:
    - 工作
---

> ArangoDB

  # 导入数据
  ## 命令
  ```
  Usage: arangoimp [<options>]

  Section 'global options' (Global configuration)
    --backslash-escape <boolean>           使用反斜线作为转意字符(default: false)
    --batch-size <uint64>                  每次数据量大小 (in bytes) (default: 16777216)
    --collection <string>                  要导入的集合名字 (default: "")
    --create-collection <boolean>          当集合不存在时创建集合 (default: false)
    --create-collection-type <string>      创建集合类型 (edge or document).(default:  "document")
    --file <string>                        导入文件名
    --from-collection-prefix <string>      _from 集合名前缀
    --overwrite <boolean>                  当集合已经存在时覆盖集合(default: false)
    --separator <string>                   字段分隔符,csv或tsv使用(default: "")
    --to-collection-prefix <string>        _to 集合名前缀
    --type <string>                        导入文件类型. possible values: "auto", "csv", "json", "jsonl", "tsv" (default: "json")

  Section 'server' (Configure a connection to the server)
    --server.connection-timeout <double>   连接超时时间(default: 5)
    --server.database <string>             连接数据库名(default: "_system")
    --server.password <string>             连接密码. 如果没有特殊指定，需要使用密码，用户将被提醒索要一份密码(default: "")
    --server.request-timeout <double>      访问超时时间（秒） (default:1200)
    --server.username <string>             连接时用户名(default: "root")
  ```
  arangoimp --server.endpoint tcp://127.0.0.1:8529 --server.username root --server.password ××× --server.database _system --file test.csv --type csv --create-collection true --create-collection-type document --overwrite true --collection "test"

  ## 样例
  ### 顶点
  #### csv
  data.csv

  ```
  "first","last","age","active","dob"
  "John","Connor",25,true,
  "Jim","O'Brady",19,,
  "Lisa","Jones",,,"1981-04-09"
  Hans,dos Santos,0123,,
  Wayne,Brewer,,false,
  ```
  #### 命令
  ```
  arangoimp --server.endpoint tcp://192.168.254.134:8529 --server.username root --server.password root --server.database mydb --file data.csv --type csv --create-collection true --create-collection-type document --overwrite true --collection "users"
  ```
  ### 边
  #### csv
  edge.csv
  ```
  "_from","_to","desc"
  "users/44539","users/44545","1234 is connected to 4321"
  ```
  #### 导入命令
  ```
  arangoimp --server.endpoint tcp://192.168.254.134:8529 --server.username root --server.password root --server.database mydb --file edge.csv --type csv --create-collection true --create-collection-type edge --overwrite true --collection "usersEdge"
  ```
  ## 一些导入命令
  ```
  arangoimp --from-collection-prefix users --to-collection-prefix products

  arangoimp --file "data.csv" --type csv --collection "users"

  arangoimp --file "data.csv" --type csv --translate "id=_key"

  arangoimp --file "data.csv" --type csv --translate "from=_from" --translate "to=_to"

  arangoimp --file "data.csv" --type csv --remove-attribute "_key"
  ```







