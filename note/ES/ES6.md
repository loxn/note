## 1.概述

### 1.1 一些概念

**Index**

　　类似于mysql数据库中的database
　　

**Type**

　　类似于mysql数据库中的table表，es中可以在Index中建立type（table），通过mapping进行映射。
　　

**Document**

　　由于es存储的数据是文档型的，一条数据对应一篇文档即相当于mysql数据库中的一行数据row，一个文档中可以有多个字段也就是mysql数据库一行可以有多列。
　　
**Field**
　　es中一个文档中对应的多个列与mysql数据库中每一列对应
　　

**Mapping**

　　可以理解为mysql或者solr中对应的schema，只不过有些时候es中的mapping增加了动态识别功能，感觉很强大的样子，其实实际生产环境上不建议使用，最好还是开始制定好了对应的schema为主。
　　

**indexed**

　　就是名义上的建立索引。mysql中一般会对经常使用的列增加相应的索引用于提高查询速度，而在es中默认都是会加上索引的，除非你特殊制定不建立索引只是进行存储用于展示，这个需要看你具体的需求和业务进行设定了。

**Query DSL**

　　类似于mysql的sql语句，只不过在es中是使用的json格式的查询语句，专业术语就叫：QueryDSL

**GET/PUT/POST/DELETE**

　　分别类似与mysql中的select/update/delete......



### 1.2 架构

![image](D:\笔记\note\ES\980882-20170208171207963-1711795457.png)

**Gateway层**

es用来存储索引文件的一个文件系统且它支持很多类型，例如：本地磁盘、共享存储（做snapshot的时候需要用到）、hadoop的hdfs分布式存储、亚马逊的S3。它的主要职责是用来对数据进行长持久化以及整个集群重启之后可以通过gateway重新恢复数据。

**Distributed Lucene Directory**

Gateway上层就是一个lucene的分布式框架，lucene是做检索的，但是它是一个单机的搜索引擎，像这种es分布式搜索引擎系统，虽然底层用lucene，但是需要在每个节点上都运行lucene进行相应的索引、查询以及更新，所以需要做成一个分布式的运行框架来满足业务的需要。

**四大模块组件**

districted lucene directory之上就是一些es的模块，Index Module是索引模块，就是对数据建立索引也就是通常所说的建立一些倒排索引等；Search Module是搜索模块，就是对数据进行查询搜索；Mapping模块是数据映射与解析模块，就是你的数据的每个字段可以根据你建立的表结构通过mapping进行映射解析，如果你没有建立表结构，es就会根据你的数据类型推测你的数据结构之后自己生成一个mapping，然后都是根据这个mapping进行解析你的数据；River模块在es2.0之后应该是被取消了，它的意思表示是第三方插件，例如可以通过一些自定义的脚本将传统的数据库（mysql）等数据源通过格式化转换后直接同步到es集群里，这个river大部分是自己写的，写出来的东西质量参差不齐，将这些东西集成到es中会引发很多内部bug，严重影响了es的正常应用，所以在es2.0之后考虑将其去掉。

**Discovery、Script**

es4大模块组件之上有 Discovery模块：es是一个集群包含很多节点，很多节点需要互相发现对方，然后组成一个集群包括选主的，这些es都是用的discovery模块，默认使用的是 Zen，也可是使用EC2；es查询还可以支撑多种script即脚本语言，包括mvel、js、python等等。

 **Transport协议层**

再上一层就是es的通讯接口Transport，支持的也比较多：Thrift、Memcached以及Http，默认的是http，JMX就是java的一个远程监控管理框架，因为es是通过java实现的。

**RESTful接口层**

最上层就是es暴露给我们的访问接口，官方推荐的方案就是这种Restful接口，直接发送http请求，方便后续使用nginx做代理、分发包括可能后续会做权限的管理，通过http很容易做这方面的管理。如果使用java客户端它是直接调用api，在做负载均衡以及权限管理还是不太好做。



### 1.3 倒排索引

**正排索引（正向索引）**：正排表是以文档的ID为关键字，表中记录文档中每个字的位置信息，查找时扫描表中每个文档中字的信息直到找出所有包含查询关键字的文档

尽管**正排表的工作原理非常的简单**，但是由于其检索效率太低，除非在特定情况下，否则实用性价值不大

![img](D:\笔记\note\ES\855959-20170422144908837-629623654.png)



　**倒排索引（反向索引）**：倒排表以字或词为关键字进行索引，表中关键字所对应的记录表项记录了出现这个字或词的所有文档，一个表项就是一个字表段，它记录该文档的ID和字符在该文档中出现的位置情况

![img](D:\笔记\note\ES\855959-20170422144951009-413072227.png)



## 3.安装

### 3.1 安装ES

1. 配置jdk

2. 下载地址：https://www.elastic.co/cn/downloads/elasticsearch

3. bin/elasticsearch，启动，使用非root账户

4. 下面情况需要清理磁盘空间

   后台运行：./bin/elasticsearch -d

   ```sh
   [INFO ][o.e.c.r.a.DiskThresholdMonitor] [ZAds5FP] low disk watermark [85%] exceeded on     [ZAds5FPeTY-ZUKjXd7HJKA][ZAds5FP][/opt/elasticsearch-6.2.4/data/nodes/0] free: 1.2gb[14.2%],     replicas will not be assigned to this node
   ```

5. 实现远程访问

   a. vim config/elasticsearch.yml

   ​	network.host: node3

   b. vim /etc/security/limits.conf

   ​	esuser soft nofile 65536

   ​	esuser hard nofile 65536

   ​	esuser soft nproc 4096

   ​	esuser hard nproc 4096

   c. vim /etc/sysctl.conf

   ​	vm.max_map_count=655360

   ​	执行 sysctl -p 生效

6. 关闭防火墙

   systemctl stop firewalld.service
   
7. 访问 node3:9200验证



### 3.2 安装Kibana

1. 下载https://www.elastic.co/downloads/kibana

2. 解压缩，修改配置文件config/kibana.yml

   将server.host,elasticsearch.url修改成所在服务器的ip地址

3. 后台启动，nohup bin/kibana &



### 3.3 配置中文分词器

1. git clone https://github.com/medcl/elasticsearch-analysis-ik.git
2. man package
3. 解压 target 目录下的 elasticsearch-analysis-ik-7.0.0.zip 文件 到 es-home 的 plugins/ik 目录下



## 4. 简单API

GET，获取

PUT，更新，要指定id

POST，指定id为更新，不指定为新增

DELETE，删除

### 4.1 基本操作

```json
# 不指定id，会自动生成id
POST /lib/user/
{
  "name":"老王",
  "age":23
}
# 指定id，若id不存在，则新增，若id存在，则覆盖
POST /lib/user/1
{
  "name":"老李",
  "age":23
}
# put操作，必须指定id，效果同post
PUT /lib/user/1
{
  "name":"罗鑫",
  "age":24
}
# 根据id获取
GET /lib/user/1
# 查询指定的字段
GET /lib/user/1?_source=age

# 更新指定字段，可添加新的字段
POST /lib/user/1/_update
{
  "doc": {
    "age":18,
    "field1":"新增的字段"
  }
}
# 删除指定的id
DELETE /lib/user/1
# 删除索引
DELETE /lib/
```

### 4.2 批量获取，_mget

```json
# 指定索引和类型
GET /_mget
{
  "docs":[
   {
       "_index": "lib",
       "_type": "user",
       "_id": 1,
       "_source": "age"
   },
   {
       "_index": "lib",
       "_type": "user",
       "_id": 2,
       "_source": ["age","name"]
   }
  ]
}
# 获取相同类型的文档
GET /lib/user/_mget
{
"docs":[
   {
      "_id": 1
   },
   {
       "_type": "user",
       "_id": 2
   }
 ]
}

GET /lib/user/_mget
{
   "ids": ["1","2"]
}
```

### 4.3 批量操作，_bulk

**bulk 的 格式：**

```
{action:{metadata}}\n
{requstbody}\n
```

**action:**

```
create：文档不存在时创建
update:更新文档
index:创建新文档或替换已有文档
delete:删除一个文档

create 和index的区别:
如果数据存在，使用create操作失败，会提示文档已经存在，使用index则可以成功执行
```

**metadata：_index,_type,_id**

示例：

```json
# 批量添加
POST /lib2/books/_bulk
{"index":{"_id":1}}
{"title":"JAVA","price":90}
{"index":{"_id":2}}
{"title":"Html5","price":45}
{"index":{"_id":3}}
{"title":"Php","price":35}
{"index":{"_id":4}}
{"title":"Python","price":50}
# 批量获取
GET /lib2/books/_mget
{
   "ids": ["1","2","3","4"]
}
# 多个操作
POST /lib2/books/_bulk
{"delete":{"_index":"lib2","_type":"books","_id":4}}
{"create":{"_index":"tt","_type":"ttt","_id":"100"}}
{"name":"lisi"}
{"index":{"_index":"tt","_type":"ttt"}}
{"name":"zhaosi"}
{"update":{"_index":"lib2","_type":"books","_id":"4"}}
{"doc":{"price":58}}
```

bulk一次最大处理多少数据量:

　　bulk会把将要处理的数据载入内存中，所以数据量是有限制的，最佳的数据量不是一个确定的数值，它取决于你的硬件，你的文档大小以及复杂性，你的索引以及搜索的负载。

　　一般建议是1000-5000个文档，大小建议是5-15MB，默认不能超过100M，可以在es的配置文件（即$ES_HOME下的config下的elasticsearch.yml）中。



## 5.版本控制

ElasticSearch采用了乐观锁来保证数据的一致性

使用内部版本号时，

操作数据的时候，指定版本号（_version），若指定的版本和数据版本一致，则操作成功，版本号 +1，否则失败

使用外部版本号时，

指定的版本号（version_type=external），必须大于现有的版本，成功则更新为指定的版本，否则失败

```json
PUT /lib/user/1?version=4
{
  "name":"罗鑫",
  "age":25
}
PUT /lib/user/1?version=6&version_type=external
{
  "name":"罗鑫",
  "age":26
}
```



## 6.数据类型





## 7.query

数据准备

```json
PUT /lib2/
{
    "settings":{
      "number_of_shards" : 3,
      "number_of_replicas" : 0
    },
    "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text","index": false},
            "address": {"type":"text","analyzer":"ik_max_word"},
            "age": {"type":"integer"},
            "interests": {"type":"text","analyzer":"ik_max_word"},
            "birthday": {"type":"date"}
        }
      }
    }
}
```



### **term 和 terms**

term 不会对输入的词进行分词，会直接在索引中找是否有输入的内容，适合keyword 、numeric、date

```json
GET /lib/user/_search/
{
  "query": {
      "term": {"address": "shanxi"}
  }
}
# terms，某个字段是否有一个或多个词
GET /lib/user/_search
{
    "from":0,
    "size":2,
    "version":true,
    "query":{
        "terms":{
            "interests": ["bigdata","lol"]
        }
    }
}
```

### **match**

会对输入的词进行分词，然后去索引中找，这样地址中包含xian 或者 baoding 都会出来，此时用term就没有结果

```json
GET /lib/user/_search
{
    "query":{
        "match":{
            "address": "xian baoding"
        }
    }
}
# 查询所有
GET _search
{
  "query": {
    "match_all": {}
  }
}
```

### **multi_match**

指定多个字段查询，此时 interests 或者 name 包含lol 的都会查出来

```json
GET /lib/user/_search
{
    "query":{
        "multi_match": {
            "query": "lol",
            "fields": ["interests","name"]
         }
    }
}
```

### **match_phrase**

短语查询，查询 interests 包含 lol,laogun 这个整体的文档

```json
GET lib/user/_search
{
  "query":{  
      "match_phrase":{  
         "interests": "lol,laogun"
      }
   }
}
```

### **指定返回的字段**

```json
# 指定返回 address 和 name 字段
GET /lib/user/_search
{
    "_source": ["address","name"],
    "query": {
        "match": {
            "interests": "changge"
        }
    }
}
# includes 指定排除的字段，或者指定需要的字段
GET /lib3/user/_search
{
    "query": {
        "match_all": {}
    },
	"_source": {
      "includes": ["name","address"],
      "excludes": ["age","birthday"]
  }
}
# 如果忘了字段的全名，使用 通配符 *
GET /lib3/user/_search
{
    "_source": {
        "includes": "addr*",
        "excludes": ["name","bir*"]
	},
    "query": {
        "match_all": {}
    }
}
```

### **排序sort**

```json
GET /lib/user/_search
{
    "query": {
        "match_all": {}
    },
    "sort": [
        {"age": {"order":"asc"}}
    ]
}

# 采用基于_doc(不使用_score)进行排序的方式，性能较高
"sort":["_doc"],
```

### **前缀匹配查询**

```json
# 查询name 是 lao 开头的
GET /lib/user/_search
{
  "query": {
    "match_phrase_prefix": {
        "name": {
            "query": "lao"
        }
    }
  }
}
```

### **range:实现范围查询**

```json
# 含 左 不含 右
GET /lib/user/_search
{
    "query": {
        "range": {
            "birthday": {
                "from": "1993-01-01",
                "to": "1993-02-01",
                "include_lower": true,
                "include_upper": false
            }
        }
    }
}
```

### **wildcard 通配符**

允许使用通配符* 和 ?来进行查询

*代表0个或多个字符

？代表任意一个字符

```json
GET /lib/user/_search
{
    "query": {
        "wildcard": {
             "name": "lao*"
        }
    }
}
GET /lib/user/_search
{
    "query": {
        "wildcard": {
             "name": "shi?iang"
        }
    }
}
```

### **fuzzy实现模糊查询**

value：查询的关键字

boost：查询的权值，默认值是1.0

min_similarity:设置匹配的最小相似度，默认值为0.5，对于字符串，取值为0-1(包括0和1);对于数值，取值可能大于1;对于日期型取值为1d,1m等，1d就代表1天

prefix_length:指明区分词项的共同前缀长度，默认是0

max_expansions:查询中的词项可以扩展的数目，默认可以无限大

```json
# 也能查出 java
GET /lib/user/_search
{
    "query": {
        "fuzzy": {
             "interests": {
                 "value": "jaa"
             }
        }
    }
}
```

### 结果高亮 highlight

```json
GET /lib3/user/_search
{
    "query":{
        "match":{
            "interests": "changge"
        }
    },
    "highlight": {
        "fields": {
             "interests": {}
        }
    }
}
```



## 8. Filter查询

和query 的区别是，query要打分，而 filter 不用打分，同时可以cache。因此，filter速度要快于query。

如果希望某个条件不影响打分，就用 filter

### 8.1 简单过滤查询

```json
GET /lib3/items/_search
{ 
       "post_filter": {
             "term": {
                 "price": 40
             }
       }
}
GET /lib3/items/_search
{
      "post_filter": {
          "terms": {
                 "price": [25,40]
              }
        }
}
# 此处 id 要用小写
GET /lib3/items/_search
{
    "post_filter": {
        "term": {
            "itemID": "id100123"
          }
      }
}
```



### 8.2 bool过滤查询

可以实现组合过滤查询

格式：

```json
{
    "bool": {
        "must": [],
        "should": [],
        "must_not": []
    }
}
```

must:必须满足的条件---and

should：可以满足也可以不满足的条件--or

must_not:不需要满足的条件--not

```json
# 满足一个即可
GET /lib3/items/_search
{
  "post_filter": {
    "bool": {
      "should":[
        {"term":{"price":25}},
        {"term":{"itemID":"id100123"}}
      ]
    }
  }
}
# 嵌套使用
GET /lib3/items/_search
{
    "post_filter": {
        "bool": {
            "should": [
                {"term": {"itemID": "id100123"}},
                {
                    "bool": {
                        "must": [
                            {"term": {"itemID": "id100124"}},
                            {"term": {"price": 50}}
                        ]
                    }
                }
            ]
        }
    }
}
GET /lib/user/_search
{
  "post_filter":{
    "bool": {
      "must":[
          {"wildcard":{"name":"lao*"}}
        ],
      "must_not":[
          {"match":{"interests":"game"}}
        ],
      "filter":{
        "range":{"age":{"gt":22}}
      }
    }
  }
}
```



### 8.3 范围过滤

gt: >

lt: <

gte: >=

lte: <=

```json
GET /lib3/items/_search
{
     "post_filter": {
          "range": {
              "price": {"gt": 25,"lt": 50}                
            }
      }
}
```



### 8.4 过滤非空

```json
GET /lib3/items/_search
{
  "query": {
    "bool": {
      "filter": {
          "exists":{"field":"price"}
      }
    }
  }
}
GET /lib3/items/_search
{
    "query" : {
        "constant_score" : {
            "filter": {
                "exists" : { "field" : "price" }
            }
        }
    }
}

```



### 8.5 过滤器缓存

ElasticSearch提供了一种特殊的缓存，即过滤器缓存（filter cache），用来存储过滤器的结果，被缓存的过滤器并不需要消耗过多的内存（因为它们只存储了哪些文档能与过滤器相匹配的相关信息），而且可供后续所有与之相关的查询重复使用，从而极大地提高了查询性能。

注意：ElasticSearch并不是默认缓存所有过滤器，
以下过滤器默认不缓存：

```
numeric_range
script
geo_bbox
geo_distance
geo_distance_range
geo_polygon
geo_shape
and
or
not
```

exists,missing,range,term,terms默认是开启缓存的

开启方式：在filter查询语句后边加上
"_catch":true



## 9. 聚合查询

```json
# 求和，"size":0，去除无关信息
GET /lib3/items/_search
{
  "size":0,
  "aggs": {
     "price_of_sum": {"sum": {"field": "price"}}
  }
}

# 最小值
GET /lib3/items/_search
{
  "size": 0, 
  "aggs": {
     "price_of_min": {"min": {"field": "price"}}
  }
}

# 最大值
GET /lib3/items/_search
{
  "size": 0, 
  "aggs": {
     "price_of_max": {"max": {"field": "price"}}
  }
}
# 平均值
GET /lib3/items/_search
{
  "size":0,
  "aggs": {
     "price_of_avg": {"avg": {"field": "price"}
     }
  }
}
```

```json
# 有 lol 爱好的人，按年龄降序分组，同时求每组的平均值
GET /lib/user/_search
{
  "query": {
      "match": {"interests": "lol"}
   },
   "size": 0, 
   "aggs":{
       "age_group_by":{
           "terms": {
             "field": "age",
             "order": {"avg_of_age": "desc"}
           },
           "aggs": {
             "avg_of_age": {"avg": {"field": "age"}}
           }
       }
   }
}
```



## 10.ES原理



### 10.1 解析es的分布式架构

#### 10.1.1 分布式架构的透明隐藏特性

ElasticSearch是一个分布式系统，隐藏了复杂的处理机制

分片机制：我们不用关心数据是按照什么机制分片的、最后放入到哪个分片中

分片的副本：

集群发现机制(cluster discovery)：比如当前我们启动了一个es进程，当启动了第二个es进程时，这个进程作为一个node自动就发现了集群，并且加入了进去

shard负载均衡：比如现在有10shard，集群中有3个节点，es会进行均衡的进行分配，以保持每个节点均衡的负载请求

请求路由

#### 3.1.2 扩容机制

垂直扩容：购置新的机器，替换已有的机器

水平扩容：直接增加机器

#### 10.1.3 rebalance

增加或减少节点时会自动均衡

#### 10.1.4 master节点

主节点的主要职责是和集群操作相关的内容，如创建或删除索引，跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点。稳定的主节点对集群的健康是非常重要的。

#### 10.1.5 节点对等

每个节点都能接收请求
每个节点接收到请求后都能把该请求路由到有相关数据的其它节点上
接收原始请求的节点负责采集数据并返回给客户端

### 10.2 分片和副本机制

1.index包含多个shard

2.每个shard都是一个最小工作单元，承载部分数据；每个shard都是一个lucene实例，有完整的建立索引和处理请求的能力

10.增减节点时，shard会自动在nodes中负载均衡

4.primary shard和replica shard，每个document肯定只存在于某一个primary shard以及其对应的replica shard中，不可能存在于多个primary shard

5.replica shard是primary shard的副本，负责容错，以及承担读请求负载

6.primary shard的数量在创建索引的时候就固定了，replica shard的数量可以随时修改

7.primary shard的默认数量是5，replica默认是1，默认有10个shard，5个primary shard，5个replica shard

8.primary shard不能和自己的replica shard放在同一个节点上（否则节点宕机，primary shard和副本都丢失，起不到容错的作用），但是可以和其他primary shard的replica shard放在同一个节点上

### 10.3 单节点环境下创建索引分析

PUT /myindex
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}

这个时候，只会将3个primary shard分配到仅有的一个node上去，另外3个replica shard是无法分配的（一个shard的副本replica，他们两个是不能在同一个节点的）。集群可以正常工作，但是一旦出现节点宕机，数据全部丢失，而且集群不可用，无法接收任何请求。

### 10.4 两个节点环境下创建索引分析

将3个primary shard分配到一个node上去，另外3个replica shard分配到另一个节点上

primary shard 和replica shard 保持同步

primary shard 和replica shard 都可以处理客户端的读请求

### 10.5 水平扩容的过程

1.扩容后primary shard和replica shard会自动的负载均衡

2.扩容后每个节点上的shard会减少，那么分配给每个shard的CPU，内存，IO资源会更多，性能提高

3.扩容的极限，如果有6个shard，扩容的极限就是6个节点，每个节点上一个shard，如果想超出扩容的极限，比如说扩容到9个节点，那么可以增加replica shard的个数

### 10.6容错性

3个分片，一个副本，共6个shard，3个节点，最多能承受几个节点所在的服务器宕机？(容错性)

最多容忍一个节点宕机

![1565934370782](D:\笔记\note\ES\1565934370782.png)

为了提高容错性，增加shard的个数：3个分片，2个副本，共9个shard
这样就能容忍最多两台服务器宕机了

![1565934499969](D:\笔记\note\ES\1565934499969.png)

总结：扩容是为了提高系统的吞吐量，同时也要考虑容错性，也就是让尽可能多的服务器宕机还能保证数据不丢失

### 10.7ElasticSearch的容错机制

以9个shard，3个节点为例：

1.如果master node 宕机，此时不是所有的primary shard都是Active status，所以此时的集群状态是red。

容错处理的第一步:是选举一台服务器作为master
容错处理的第二步:新选举出的master会把挂掉的primary shard的某个replica shard 提升为primary shard,此时集群的状态为yellow，因为少了一个replica shard，并不是所有的replica shard都是active status

容错处理的第三步：重启故障机，新master会把所有的副本都复制一份到该节点上，（同步一下宕机后发生的修改），此时集群的状态为green，因为所有的primary shard和replica shard都是Active status

### 10.8文档的核心元数据

1._index:

说明了一个文档存储在哪个索引中

同一个索引下存放的是相似的文档(文档的field多数是相同的)

索引名必须是小写的，不能以下划线开头，不能包括逗号

2._type:

表示文档属于索引中的哪个类型

一个索引下只能有一个type

类型名可以是大写也可以是小写的，不能以下划线开头，不能包括逗号

10._id:

文档的唯一标识，和索引，类型组合在一起唯一标识了一个文档

可以手动指定值，也可以由es来生成这个值

### 10.9 文档id生成方式

1.手动指定

  put /index/type/66

  通常是把其它系统的已有数据导入到es时

2.由es生成id值

  post /index/type

 es生成的id长度为20个字符，使用的是base64编码，URL安全，使用的是GUID算法，分布式下并发生成id值时不会冲突

### 10.9 _source元数据分析

其实就是我们在添加文档时request body中的内容

指定返回的结果中含有哪些字段：

get /index/type/1?_source=name

### 10.10 改变文档内容原理解析

![1565943040703](D:\笔记\note\ES\1565943040703.png)



### 10.11 基于groovy脚本执行partial update

es有内置的脚本支持，可以基于groovy脚本实现复杂的操作

```json
1.修改年龄
POST /lib/user/4/_update
{
  "script": "ctx.__source.age+=1"
}

2.修改名字
POST /lib/user/4/_update
{
  "script": "ctx.__source.last_name+='hehe'"
}

10.添加爱好
POST /lib/user/4/_update
{
  "script": {
    "source": "ctx._source.interests.add(params.tag)",
    "params": {
      "tag":"picture"
    }
  }
}
4.删除爱好
POST /lib/user/4/_update
{
  "script": {
    "source": "ctx._source.interests.remove(ctx._source.interests.indexOf(params.tag))",
    "params": {
      "tag":"picture"
    }
  }
}

5.删除文档
POST /lib/user/4/_update
{
  "script": {
    "source": "ctx.op=ctx._source.age==params.count?'delete':'none'",
    "params": {
        "count":29
    }
  }
}

6.upsert
POST /lib/user/4/_update
{
  "script": "ctx._source.age += 1",

  "upsert": {
     "first_name" : "Jane",
     "last_name" :   "Lucy",
     "age" :  20,
     "about" :       "I like to collect rock albums",
     "interests":  [ "music" ]
  }
}
```



### 10.12 partial update 处理并发冲突

使用的是乐观锁:_version

retry_on_conflict:

POST /lib/user/4/_update?retry_on_conflict=3

重新获取文档数据和版本信息进行更新，不断的操作，最多操作的次数就是retry_on_conflict的值

### 10.13 文档数据路由原理解析

1.文档路由到分片上：

 一个索引由多个分片构成，当添加(删除，修改)一个文档时，es就需要决定这个文档存储在哪个分片上，这个过程就称为数据路由(routing)

2.路由算法：

```
shard=hash(routing) % number_of_pirmary_shards
```

示例：一个索引，3个primary shard

(1)每次增删改查时，都有一个routing值，默认是文档的_id的值

(2)对这个routing值使用哈希函数进行计算

(3)计算出的值再和主分片个数取余数

余数肯定在0---（number_of_pirmary_shards-1）之间，文档就在对应的shard上

routing值默认是文档的_id的值，也可以手动指定一个值，手动指定对于负载均衡以及提高批量读取的性能都有帮助

3.primary shard个数一旦确定就不能修改了



### 10.14 文档增删改内部原理

1:发送增删改请求时，可以选择任意一个节点，该节点就成了协调节点(coordinating node)

2.协调节点使用路由算法进行路由，然后将请求转到primary shard所在节点，该节点处理请求，并把数据同步到它的replica shard

3.协调节点对客户端做出响应

### 10.15 写一致性原理和quorum机制

1.任何一个增删改操作都可以跟上一个参数
consistency

可以给该参数指定的值：

one: (primary shard)只要有一个primary shard是活跃的就可以执行

all: (all shard)所有的primary shard和replica shard都是活跃的才能执行

quorum: (default) 默认值，大部分shard是活跃的才能执行 （例如共有6个shard，至少有3个shard是活跃的才能执行写操作）

2.quorum机制：多数shard都是可用的，

int((primary+number_of_replica)/2)+1

例如：3个primary shard，1个replica

int((3+1)/2)+1=3

至少3个shard是活跃的

注意：可能出现shard不能分配齐全的情况

比如：1个primary shard,1个replica
int((1+1)/2)+1=2
但是如果只有一个节点，因为primary shard和replica shard不能在同一个节点上，所以仍然不能执行写操作

再举例：1个primary shard,3个replica,2个节点

int((1+3)/2)+1=3

最后:当活跃的shard的个数没有达到要求时，
es默认会等待一分钟，如果在等待的期间活跃的shard的个数没有增加，则显示timeout

put /index/type/id?timeout=60s

### 10.16 文档查询内部原理

第一步：查询请求发给任意一个节点，该节点就成了coordinating node，该节点使用路由算法算出文档所在的primary shard

第二步：协调节点把请求转发给primary shard也可以转发给replica shard(使用轮询调度算法(Round-Robin Scheduling，把请求平均分配至primary shard 和replica shard)

第三步：处理请求的节点把结果返回给协调节点，协调节点再返回给应用程序

特殊情况：请求的文档还在建立索引的过程中，primary shard上存在，但replica shar上不存在，但是请求被转发到了replica shard上，这时就会提示找不到文档

### 10.17 bulk批量操作的json格式解析

bulk的格式：

{action:{metadata}}\n

{requstbody}\n

为什么不使用如下格式：

[{

"action": {

},

"data": {

}

}]

这种方式可读性好，但是内部处理就麻烦了：

1.将json数组解析为JSONArray对象，在内存中就需要有一份json文本的拷贝，另外还有一个JSONArray对象。

2.解析json数组里的每个json，对每个请求中的document进行路由

3.为路由到同一个shard上的多个请求，创建一个请求数组

4.将这个请求数组序列化

5.将序列化后的请求数组发送到对应的节点上去

耗费更多内存，增加java虚拟机开销

1.不用将其转换为json对象，直接按照换行符切割json，内存中不需要json文本的拷贝

2.对每两个一组的json，读取meta，进行document路由

3.直接将对应的json发送到node上去

### 10.18 查询结果分析

{
  "took": 419,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "lib3",
        "_type": "user",
        "_id": "3",
        "_score": 0.6931472,
        "_source": {
          "address": "bei jing hai dian qu qing he zhen",
          "name": "lisi"
        }
      },
      {
        "_index": "lib3",
        "_type": "user",
        "_id": "2",
        "_score": 0.47000363,
        "_source": {
          "address": "bei jing hai dian qu qing he zhen",
          "name": "zhaoming"
        }
      }
      
took：查询耗费的时间，单位是毫秒

_shards：共请求了多少个shard

total：查询出的文档总个数

max_score： 本次查询中，相关度分数的最大值，文档和此次查询的匹配度越高，_score的值越大，排位越靠前

hits：默认查询前10个文档

timed_out：设置timeout可以减少响应时间，本来查100条是20ms，设置10ms的 timeout 就只会拿到 小于 100的数据

GET /lib3/user/_search?timeout=10ms
{
    "_source": ["address","name"],
    "query": {
        "match": {
            "interests": "changge"
        }
    }
}

### 10.19 多index，多type查询模式

GET _search

GET /lib/_search

GET /lib,lib3/_search

GET /*3,*4/_search

GET /lib/user/_search

GET /lib,lib4/user,items/_search

GET /_all/_search

GET /_all/user,items/_search

### 10.20 分页查询中的deep paging问题

GET /lib3/user/_search
{
    "from":0,
    "size":2,
    "query":{
        "terms":{
            "interests": ["hejiu","changge"]
        }
    }
}

GET /_search?from=0&size=3

deep paging:查询的很深，比如一个索引有三个primary shard，分别存储了6000条数据，我们要得到第100页的数据(每页10条)，类似这种情况就叫deep paging

如何得到第100页的10条数据？

在每个shard中搜索990到999这10条数据，然后用这30条数据排序，排序之后取10条数据就是要搜索的数据，这种做法是错的，因为3个shard中的数据的_score分数不一样，可能这某一个shard中第一条数据的_score分数比另一个shard中第1000条都要高，所以在每个shard中搜索990到999这10条数据然后排序的做法是不正确的。

正确的做法是每个shard把0到999条数据全部搜索出来（按排序顺序），然后全部返回给coordinate node，由coordinate node按_score分数排序后，取出第100页的10条数据，然后返回给客户端。

deep paging性能问题

1.耗费网络带宽，因为搜索过深的话，各shard要把数据传送给coordinate node，这个过程是有大量数据传递的，消耗网络，

2.消耗内存，各shard要把数据传送给coordinate node，这个传递回来的数据，是被coordinate node保存在内存中的，这样会大量消耗内存。

3.消耗cpu coordinate node要把传回来的数据进行排序，这个排序过程很消耗cpu.

鉴于deep paging的性能问题，所以应尽量减少使用。

### 10.21 query string查询及copy_to解析

GET /lib3/user/_search?q=interests:changge

GET /lib3/user/_search?q=+interests:changge

GET /lib3/user/_search?q=-interests:changge

copy_to字段是把其它字段中的值，以空格为分隔符组成一个大字符串，然后被分析和索引，但是不存储，也就是说它能被查询，但不能被取回显示。

![1566054874365](D:\笔记\note\ES\1566054874365.png)

注意:copy_to指向的字段字段类型要为：text

当没有指定field时，就会从copy_to字段中查询
GET /lib3/user/_search?q=changge

### 10.22字符串排序问题

对一个字符串类型的字段进行排序通常不准确，因为已经被分词成多个词条了

解决方式：对字段索引两次，一次索引分词（用于搜索），一次索引不分词(用于排序)

```json
PUT /lib3
{
    "settings":{
        "number_of_shards" : 3,
        "number_of_replicas" : 0
      },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text"},
            "address": {"type":"text"},
            "age": {"type":"integer"},
            "birthday": {"type":"date"},
            "interests": {
                "type":"text",
                "fields": {"raw":{"type": "keyword"}},
                "fielddata": true
             }
          }
        }
     }
}
# 查询使用 字段.raw
GET /lib3/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "interests.raw": {"order": "asc"}
    }
  ]
}
```



### 10.23 如何计算相关度分数

使用的是TF/IDF算法(Term Frequency&Inverse Document Frequency)

**1. Term Frequency**:我们查询的文本中的词条在document本中出现了多少次，出现次数越多，相关度越高

搜索内容： hello world，结果：第二句分数更高，因为有两个词

doc1：*Hello，I love china*.

doc2：*Hello world,how are you*!

**2.Inverse Document Frequency**：我们查询的文本中的词条在索引的所有文档中出现了多少次，出现的次数越多，相关度越低

搜索内容：hello world，结果：按照第一个规则，两句的分数一样，但是按照第二个规则，hello在所有的文档中出现的次数更多，相关度更低，所以第二句分数更高

doc1：hello，what are you doing?

doc2：I like the world.

hello 在索引的所有文档中出现了500次，world出现了100次

**3.Field-length**(字段长度归约) norm:field越长，相关度越低

搜索内容：hello world

doc1：{"title":"hello,what's your name?","content":{"owieurowieuolsdjflk"}}

doc2：{"title":"hi,good morning","content":{"lkjkljkj.......world"}}

结果：按照前两个规则分数相同， word 所在的字段过长，所以第一句的分数更高

```json
# 查看分数是如何计算的：相当于 mysql 中的 explain + sql
GET /lib3/user/_search?explain=true
{
    "query":{
        "match":{
            "interests": "duanlian,changge"
        }
    }
}
```

```json
# 查看一个文档能否匹配上某个查询：

GET /lib3/user/2/_explain
{
    "query":{
        "match":{
            "interests": "duanlian,changge"
        }
    }
}
```



### 10.24 Doc Values 解析

DocValues其实是Lucene在构建倒排索引时，会额外建立一个有序的正排索引(基于document => field value的映射列表)

{"birthday":"1985-11-11",age:23}

{"birthday":"1989-11-11",age:29}

document     age       birthday

doc1         23         1985-11-11

doc2         29         1989-11-11

存储在磁盘上，节省内存 

对排序，分组和一些聚合操作能够大大提升性能 

注意：默认对不分词的字段是开启的，对分词字段无效（需要把fielddata设置为true）

```json
PUT /lib3
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text"},
            "address": {"type":"text"},
            "age": {
              "type":"integer",
              "doc_values":false
            },
            "interests": {"type":"text"},
            "birthday": {"type":"date"}
        }
      }
     }
}
```



### 10.25 基于scroll技术滚动搜索大量数据

如果一次性要查出来比如10万条数据，那么性能会很差，此时一般会采取用scoll滚动查询，一批一批的查，直到所有数据都查询完为止。

1.scoll搜索会在第一次搜索的时候，保存一个当时的视图快照，之后只会基于该旧的视图快照提供数据搜索，如果这个期间数据变更，是不会让用户看到的

2.采用基于_doc(不使用_score)进行排序的方式，性能较高

3.每次发送scroll请求，我们还需要指定一个scoll参数，指定一个时间窗口，每次搜索请求只要在这个时间窗口内能完成就可以了

```json
GET /lib3/user/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort":["_doc"],
  "size":3
}

GET /_search/scroll
{
   "scroll": "1m",
   "scroll_id": "DnF1ZXJ5VGhlbkZldGNoAwAAAAAAAAAdFkEwRENOVTdnUUJPWVZUd1p2WE5hV2cAAAAAAAAAHhZBMERDTlU3Z1FCT1lWVHdadlhOYVdnAAAAAAAAAB8WQTBEQ05VN2dRQk9ZVlR3WnZYTmFXZw=="
}
```



### 10.26 dynamic mapping策略

**dynamic**:

1.true:遇到陌生字段就 dynamic mapping

2.false:遇到陌生字段就忽略

3.strict:约到陌生字段就报错

```json
PUT /lib8
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "dynamic":strict,
        "properties":{
            "name": {"type":"text"},
            "address":{
                "type":"object",
                "dynamic":true
            },
        }
      }
     }
}

#会报错

PUT  /lib8/user/1
{
  "name":"lisi",
  "age":20,
  "address":{
    "province":"beijing",
    "city":"beijing"
  }
}
```



**date_detection**:默认会按照一定格式识别date，比如yyyy-MM-dd

可以手动关闭某个type的date_detection

```json
PUT /lib8
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "date_detection": false,
        }
    }
}
```



**定制 dynamic mapping template(type)**

```json
PUT /my_index
{ 
  "mappings": { 
    "my_type": { 
      "dynamic_templates": [ 
        { 
          "en": { 
            "match": "*_en", 
            "match_mapping_type": "string", 
            "mapping": { 
              "type": "text", 
              "analyzer": "english" 
            } 
          } 
        } 
      ] 
     } 
  } 
}
#使用了模板,使用了english分词器，is是无意义的词，查不出来
PUT /my_index/my_type/3
{
  "title_en": "this is my dog"

}
#没有使用模板，使用默认的stander分词器，is能查出
PUT /my_index/my_type/5
{
  "title": "this is my cat"
}

GET my_index/my_type/_search
{
  "query": {
    "match": {
      "title": "is"
    }
  }
}
```



### 10.27重建索引

一个field的设置是不能修改的，如果要修改一个field，那么应该重新按照新的mapping，建立一个index，然后将数据批量查询出来，重新用bulk api写入到index中。

批量查询的时候，建议采用scroll api，并且采用多线程并发的方式来reindex数据，每次scroll就查询指定日期的一段数据，交给一个线程即可。

```json
# 当前的 content 字段是 日期类型
PUT /index1/type1/4
{
   "content":"1990-12-12"
}
# 添加 字符类型的数据 报错
PUT /index1/type1/4
{
   "content":"I am very happy."
}
#修改content的类型为string类型,报错，不允许修改
PUT /index1/_mapping/type1
{
  "properties": {
    "content":{"type": "text"}
  }
}
# 正确的方式，先创建新的索引，把index1索引中的数据查询出来导入到新的索引中
PUT /newindex
{
  "mappings": {
    "type1":{
      "properties": {
        "content":{"type": "text"}
      }
    }
  }
}
#但是应用程序使用的是之前的索引，为了不用重启应用程序，给index1这个索引起个#别名
#如果程序中使用的是 index2 这个别名，就不用重启程序，这个别名直接赋给 newindex
POST /_aliases
{
  "actions": [
    	{"remove": {"index":"index1","alias":"index2"}},
    	{"add": {"index": "newindex","alias": "index2"}}
	]
}
```



### 10.28 索引不可变的原因

倒排索引包括：

   文档的列表，文档的数量，词条在每个文档中出现的次数，出现的位置，每个文档的长度，所有文档的平均长度

索引不变的原因：

不需要锁，提升了并发性能

可以一直保存在缓存中（filter）

节省cpu和io开销

## 第四节 在Java应用中访问ElasticSearch

pom中加入ElasticSearch6.2.4的依赖：

```xml
 <dependency>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>transport</artifactId>
      <version>6.2.4</version>
 </dependency>    
```

```java
public class Test1 {

    private static TransportClient client;
    static {
        Settings settings = Settings.builder().put("cluster.name", "myes").build();

        client = new PreBuiltTransportClient(settings);
        try {
            client.addTransportAddress(new TransportAddress(InetAddress.getByName("node3"), 9300));
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        System.out.println(client);
    }


    // 添加文档
    @Test
    public void testAdd() throws Exception{
        List<DiscoveryNode> discoveryNodes = client.connectedNodes();
        XContentBuilder doc1 = XContentFactory.jsonBuilder()
                .startObject()
                .field("id","3")
                .field("title","Java设计模式之单例模式")
                .field("content","枚举单例模式可以防反射攻击。")
                .field("postdate","2018-02-02")
                .field("url","csdn.net/79247746")
                .endObject();
        IndexResponse response = client.prepareIndex("index1", "blog", null)
                .setSource(doc1).get();
        System.out.println(response.status()); // CREATED
    }


    // 根据id 删除
    @Test
    public void testDel() throws Exception{
        DeleteResponse response=client.prepareDelete("index1","blog","ERVkl2wBaTeKTVp8_UO4").get();
        //删除成功返回OK，否则返回NOT_FOUND
        System.out.println(response.status());
    }

    // 根据id更新
    @Test
    public void testUpdate1() throws Exception{
        // 没有数据会报错
        UpdateRequest request=new UpdateRequest();
        request.index("index1")
                .type("blog")
                .id("ExWhl2wBaTeKTVp8mkP1Z")
                .doc(XContentFactory.jsonBuilder().startObject().field("title","单例模式解读").endObject());
        UpdateResponse response=client.update(request).get();

        //更新成功返回OK，否则返回NOT_FOUND
        System.out.println(response.status());
    }

    @Test
    public void testUpdate2() throws Exception{
        // 没有数据会新增
        IndexRequest request1 =new IndexRequest("index1","blog","3")
                .source(
                        XContentFactory.jsonBuilder().startObject()
                                .field("id","3")
                                .field("title","装饰模式")
                                .field("content","动态地扩展一个对象的功能")
                                .field("postdate","2018-05-23")
                                .field("url","csdn.net/79239072")
                                .endObject()
                );
        UpdateRequest request2=new UpdateRequest("index1","blog","3")
                .doc(
                        XContentFactory.jsonBuilder().startObject()
                                .field("title","装饰模式解读")
                                .endObject()
                ).upsert(request1);

        UpdateResponse response=client.update(request2).get();
        //upsert操作成功返回OK，否则返回NOT_FOUND
        System.out.println(response.status());
    }

    // 批量 获取
    @Test
    public void testMultGet() throws Exception{
        MultiGetResponse mgResponse = client.prepareMultiGet()
                .add("index1","blog","ExWhl2wBaTeKTVp8mkPZ")
                .add("lib3","items","1","2","3")
                .get();

        for(MultiGetItemResponse response:mgResponse){
            GetResponse rp=response.getResponse();
            if(rp!=null && rp.isExists()){
                System.out.println(rp.getSourceAsString());
            }
        }

    }

    @Test
    public void testBulk() throws Exception{
        // 批量添加
        BulkRequestBuilder bulkRequest = client.prepareBulk();

        bulkRequest.add(client.prepareIndex("lib2", "books", "4")
                .setSource(XContentFactory.jsonBuilder()
                        .startObject()
                        .field("title", "python")
                        .field("price", 68)
                        .endObject()
                )
        );
        bulkRequest.add(client.prepareIndex("lib2", "books", "5")
                .setSource(XContentFactory.jsonBuilder()
                        .startObject()
                        .field("title", "VR")
                        .field("price", 38)
                        .endObject()
                )
        );
        //批量执行
        BulkResponse bulkResponse = bulkRequest.get();

        System.out.println(bulkResponse.status());
        if (bulkResponse.hasFailures()) {
            System.out.println("存在失败操作");
        }
    }

    // 根据条件删除
    @Test
    public void testD() throws Exception{
        BulkByScrollResponse response =
                DeleteByQueryAction.INSTANCE.newRequestBuilder(client).filter(QueryBuilders.matchQuery("title", "单例"))
                        .source(
                                "index1").get();
        long deleted = response.getDeleted();
        System.out.println(deleted);
    }
}
```

查询

```java
public class TestQuery {
    private static TransportClient client;

    static {
        Settings settings = Settings.builder().put("cluster.name", "myes").build();

        client = new PreBuiltTransportClient(settings);
        try {
            client.addTransportAddress(new TransportAddress(InetAddress.getByName("node3"), 9300));
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        System.out.println(client);
    }

    // 跟据id查询
    @Test
    public void testGet() throws Exception {
        GetResponse documentFields = client.prepareGet("index1", "blog", "ExWhl2wBaTeKTVp8mkPZ").execute().actionGet();
        System.out.println(documentFields.getSourceAsString());
    }

    // 查询
    @Test
    public void testAll() throws Exception {
        // 查询全部
        // MatchAllQueryBuilder query = QueryBuilders.matchAllQuery();

        // match 查询
        // MatchQueryBuilder query = QueryBuilders.matchQuery("address", "河北");

        // match 多个字段
        // MultiMatchQueryBuilder query = QueryBuilders.multiMatchQuery("河北", "address", "interests");

        // term 查询
        // TermQueryBuilder query = QueryBuilders.termQuery("address", "河北");

        // terms
        // TermsQueryBuilder query = QueryBuilders.termsQuery("address", "保定", "西安");

        // range
        // RangeQueryBuilder query =
        //        QueryBuilders.rangeQuery("birthday").from("1993-01-01").to("1993-04-01").format("yyyy-MM-dd");

        // prefix
        // PrefixQueryBuilder query = QueryBuilders.prefixQuery("name", "老");

        // wildcard
        // WildcardQueryBuilder query = QueryBuilders.wildcardQuery("name", "老*");

        // fuzzy
        // FuzzyQueryBuilder query = QueryBuilders.fuzzyQuery("interests", "bigdta");

        // type 查询
        // TypeQueryBuilder query = QueryBuilders.typeQuery("user");

        // ids
        IdsQueryBuilder query = QueryBuilders.idsQuery().addIds("1", "2");

        SearchResponse lib3 = client.prepareSearch("lib2").setQuery(query).setSize(6).get();
        SearchHits hits = lib3.getHits();
        for (SearchHit hit : hits) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            System.out.println(sourceAsMap);
        }
    }

    // 聚合查询
    @Test
    public void testAgg() throws Exception {
        // 最大值
        // MaxAggregationBuilder field = AggregationBuilders.max("maxAge").field("age");

        // 最小值
        // MinAggregationBuilder field = AggregationBuilders.min("minAge").field("age");

        // avg
        // AvgAggregationBuilder field = AggregationBuilders.avg("avg").field("age");

        // sum
        // SumAggregationBuilder field = AggregationBuilders.sum("sum").field("age");

        // 基数
        // CardinalityAggregationBuilder field = AggregationBuilders.cardinality("cardinality").field("age");

        // 统计 按age 分组，然后求和
        TermsAggregationBuilder field = AggregationBuilders.terms("terms").field("age");

        SearchResponse lib2 = client.prepareSearch("lib2").addAggregation(field).get();
        // Max minAge = lib2.getAggregations().get("maxAge");
        // Min minAge = lib2.getAggregations().get("minAge");
        // Avg minAge = lib2.getAggregations().get("avg");
        // Sum minAge = lib2.getAggregations().get("sum");
        // Cardinality minAge = lib2.getAggregations().get("cardinality");
        Terms filter = lib2.getAggregations().get("terms");

        for (Terms.Bucket bucket : filter.getBuckets()) {
            System.out.println(bucket.getKey() + ":" + bucket.getDocCount());
        }
    }

    @Test
    public void testAggFilter() throws Exception{
        // 查询
        FilterAggregationBuilder field = AggregationBuilders.filter("filter", QueryBuilders.termQuery("age", "23"));
        SearchResponse lib2 = client.prepareSearch("lib2").addAggregation(field).get();
        Filter filter = lib2.getAggregations().get("filter");
        System.out.println(filter.getDocCount());

        // 指定多个条件
        FiltersAggregationBuilder filtersBuidler = AggregationBuilders.filters("filters",
                new FiltersAggregator.KeyedFilter("lol",QueryBuilders.termQuery("interests", "lol")),
                new FiltersAggregator.KeyedFilter("bigdata",QueryBuilders.termQuery("interests", "bigdata")));

        SearchResponse lib3 = client.prepareSearch("lib2").addAggregation(filtersBuidler).get();
        Filters filters = lib3.getAggregations().get("filters");
        for (Filters.Bucket bucket : filters.getBuckets()) {
            System.out.println(bucket.getKey() + ":" + bucket.getDocCount());
        }

        // 范围统计
        RangeAggregationBuilder range = AggregationBuilders.range("range").field("age")
                .addUnboundedTo(24) // 小于等于24 的
                .addUnboundedFrom(10) // 大于等于10 的
                .addRange(22, 24); // 22 dc 24 之间的

        SearchResponse rangeResp = client.prepareSearch("lib2").addAggregation(range).get();
        Range ra = rangeResp.getAggregations().get("range");
        for (Range.Bucket bucket : ra.getBuckets()) {
            System.out.println(bucket.getKey() + ":" + bucket.getDocCount());
        }

        // 统计某个字段为空  的数据
        MissingAggregationBuilder missing = AggregationBuilders.missing("missing").field("price");
        SearchResponse missingResp = client.prepareSearch("lib3").addAggregation(missing).get();
        Aggregation missing1 = missingResp.getAggregations().get("missing");
        System.out.println(missing1.toString());
    }

    @Test
    public void testQueryStr() throws Exception {
        // 查询有 lol 兴趣，没有 老滚 兴趣
        // QueryStringQueryBuilder query = QueryBuilders.queryStringQuery("+lol -老滚");

        // bool 查询
        // BoolQueryBuilder query = QueryBuilders.boolQuery().must(QueryBuilders.matchQuery("interests", "老滚"))
        //        .mustNot(QueryBuilders.matchQuery("interests", "java"));

        ConstantScoreQueryBuilder query =
                QueryBuilders.constantScoreQuery(QueryBuilders.matchQuery("name", "老王"));

        SearchResponse lib3 = client.prepareSearch("lib2").setQuery(query).setSize(6).get();
        SearchHits hits = lib3.getHits();
        for (SearchHit hit : hits) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            System.out.println(sourceAsMap);
        }
    }
}
```











































​              











































