# elasticsearch


# 最大的特点：

1. 数据库的 database, 就是  index

2. 数据库的 table,  就是 tag

3. 不要使用browser， 使用curl来进行客户端操作.  否则会出现 java heap ooxx...

curl:  -X 后面跟 RESTful ：  GET, POST ...

-d 后面跟数据。 (d = data to send)

## 基本用法


###. create:

指定 ID 来建立新记录。 （貌似PUT， POST都可以）
$ curl -XPOST localhost:9200/films/md/2 -d '
{ "name":"hei yi ren", "tag": "good"}'

使用自动生成的 ID 建立新纪录：
$ curl -XPOST localhost:9200/films/md -d '
{ "name":"ma da jia si jia3", "tag": "good"}'

### 查询：
2.1 查询所有的 index, type:
```
$ curl localhost:9200/_search?pretty=true
```

2.2 查询某个index下所有的type:
```
$ curl localhost:9200/films/_search
```

2.3 查询某个index 下， 某个 type下所有的记录：
```
$ curl localhost:9200/films/md/_search?pretty=true
```

2.4 带有参数的查询：
```
$ curl localhost:9200/films/md/_search?q=tag:good
# =>
{"took":7,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":2,"max_score":1.0,"hits":[{"_index":"film","_type":"md","_id":"2","_score":1.0, "_source" :
{ "name":"hei yi ren", "tag": "good"}},{"_index":"film","_type":"md","_id":"1","_score":0.30685282, "_source" :
{ "name":"ma da jia si jia", "tag": "good"}}]}}
```

2.5 使用JSON参数的查询： （注意 query 和 term 关键字）
```
$ curl localhost:9200/film/_search -d '
# =>
{"query" : { "term": { "tag":"bad"}}}'
```

### update
```
$ curl -XPUT localhost:9200/films/md/1 -d { ...(data)... }
```

### 删除。 删除所有的：
```
$ curl -XDELETE localhost:9200/films
```
