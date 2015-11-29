#  non-sql no-sql

不使用sql的数据库。

SQL: 数据库：MYSQL, ORACLE, SQL SERVER, POSTGRES， 都是传统数据库。
新型数据库：
  non-sql:   mongodb, cassandra, ...
  key-value: redis

除了这个，还有elastic search , lucene等（全文检索）

## 看个例子：

比如，数据库，有个表： User,  该表有个列： id, name

## 查询：
传统：  select * from users;
non-sql:   User.all

## 新建：

insert into users values(default, "Jim");

nonsql:  User.create({id: default, name: 'Jim'});

## 删除：

sql:  delete from users where id = 1;
non-sql:  User.delete({id: 1})

## 更新

sql: update users set name = "Mike" where id = 1;
nonsql:  User.where({id: 1}).update({ name: "Mike"})
nonsql:  User.update({id: 1}, { name: "Mike"})

## non sql 的实质

就是在数据库的层面，使用了对象的持久化。
（之前，都是在代码的层面，使用专门的框架来做，例如： active record(ruby),  hibernate (java)

## 个人感觉，不需要非得要求使用NON-SQL。 它的好处，在rails下不明显。 甚至，会降低开发效率。提高调试的难度。

用起来 : 跟Rails狠相似。 但是又有不同：

1. RAILS的更加简洁。  mongo 不太简介
2. RAILS的更好理解。 MONGO的语法，有的不好理解。例如：  User.find({age: { gt: 20, lt: 30}})
gt:  greater than,
lt:  less than.

有很多这样的语法，你必须拼成，  { id: ...,  { gt:.. }, }  这样的JSON。

3. 调试性能比较麻烦。
SQL调试： 看slow log, 分析join, 多表查询， 看里面的逻辑。
NO-SQL调试： 不好调试。

例如：在100W条数据中，查到一些记录，需要90秒。（跟传统的SQL相比，在性能上没有任何优势）

$ show dbs

> use hello;

## 另一个特点：
不需要声明数据库的结构（声明了更好用(用于查询的优化），不声明也可以用）

## 特别好用的地方：

把一种不确定结构的数据结构存储进来，是特别有效的。  （postgres也支持类似的功能）
但是，建议： 不要因为这一个功能，就冒然使用NO-SQL。

id:   ObjectId('alsdkfjaslkdjf')

不如：MYSQL好记。 但是，用法基本一样。

```
# ruby, mysql, postgres ...
User.find(1)

# ruby, mongodb:
User.find(ObjectId('a;dklsfjalksjfalsdkjf'))

sql:  database,  table,         row
no-sql:     db,  collection,    document

## 分片： replication

概念自己看吧， 基本用不到。
