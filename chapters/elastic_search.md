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




# 全文检索  lucene, solr , elastic search .

概念： 通过查预先设定好的字典的方式，来找到结果。

在 被查询的 记录，保存到数据库的同时，就被分析，分析出里面的 keywords, 然后，做字典的排表。
以后查询时，直接查 keywords就可以。

例如： ”我爱北京天安门“
在被保存时，会被 “分词”， 成： 我， 爱， 北京， 天安门“ （实际上的分词，取决于  你用的词库）

有的会分成： 我 爱 北京天安门
有的会分成：  我爱   北京   天安门
有的会分成：  我 爱 北京 天安 门

所以，对于不同的词库，它的分词效果，会直接影响到搜索结果。 例如：
(  对于中文，百度搜索肯定比google搜得准确）
（对于英文， GOOGLE肯定比百度准确）

开源的词库就那么几种，特别是中文词库。 推崇的词库有：javaeye的词库。

例如： 昨天外面一直在下雪，下了一天。

分词：
1. 了，在，这样的语气助词， 一般不分词。
2. 分词只分重要的词。

所以，对于上面的句子的分词，很可能是： 昨天，外面，一直，下雪。

所以，在这个分词情况下，我搜索“昨天”， 瞬间搜到，搜索“了”， 就搜不到。

## 当今的elastic search是3年前最好的 全文检索工具。

lucene: 是JAVA世界最早的全文检索   05年以前就出现了。 辉煌在07~ 09
solr : 跟lucene 差不多。 用的人不多。
elastic search; 似乎是google把XX技术开元之后， lucene的原作者，弄出来的。

所以，大家要用elastic search. + 一个好的中文词库。

##  集群
可以轻易地组件集群。

我们做全文搜索时， 不是通过SOCKET的方式，来访问数据库的。

  socket: /tmp/mysql.sock

而 elastic search 是使用HTTP 的 restful request来访问的。

  http://localhost:3456

那么，运行elastic search集群的话，跟thin 一样，多跑几个实例，就可以了：

  http://localhost:3456
  http://localhost:3457
  http://localhost:3458

基本不需要配置，这些集群，就会自动知道 当前的网络中，又多少 “伙伴”， 用户访问任意一个地址，都可以获得同样地效果。

优点： 可以瞬间扩展。

## 按照 相似度 来显示搜索结果。

正常的搜索 “北京天安门“， 很可能会匹配到两个keyword， ”北京“， ”天安门“， 甚至有“门”，
elastic search会按照这个匹配度来显示。

