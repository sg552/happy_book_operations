
# SQL, NO-SQL, 缓存， 和 全文索引。

可以认为，全文索引，也是一种数据库。

sql:                    database,       table,         row
no-sql:                 db,             collection,    document
full-text-index:        ??              ??             document

完全可以把它作为数据库来使用。


## 我的观点：
会用MYSQL， 其他的ORACLE, POSTGRES, SQL SERVER ，都会了。
会用ELASTIC SEARCH， 其他的全文检索，都会了。
会用一个CACHE,其他的CACHE就都会了。


## 建模时，先考虑什么？

是先考虑数据库的表？ 还是先考虑MODEL？

必须先考虑MODEL。

不会持久层的人，才会去关注 数据库。
了解持久层（hibernate, rails中得AR）的人，会首先关注MODEL。
因为MODEL 更加易读，易懂。

sql:(上来就要表示出表的结构）

User
----------
id: int(8)
name: varchar(255)

model:(更加注重可读性）

User:
id
name


<?php
   sql = 'select * from users ...'
?>

   result = User.all

## cache
所有的cache ，都把内容保存在内存里。
甚至一些数据库(hbase, mongo? )， 也都把数据保存在内容中。（有一些从内存同步到硬盘的机制，这个就要看文档了）


