# regexp  regex expression

是一本厚厚的书。

有个典型的笑话： 说，

如果要修改100处相同的代码，

一个普通的程序员用了2个小时去修改，他是 一条一条的修改

一个程序员高手，也用了2个小时去修改。前一小时59分，一直在调试正则表达式。 之后的1分钟，修改完毕。

##

```
irb(main):001:0> "abcde".index "abc"
=> 0
irb(main):002:0> "abcde".index "cde"
=> 2
irb(main):003:0> "abcde".index "casdfasl;dkfje"
=> nil
irb(main):004:0> "abcde".match /abc/
=> #<MatchData "abc">
irb(main):005:0> "abcde".match /cd/
=> #<MatchData "cd">
irb(main):006:0> "abcde".match /cd/

```

//
之间的内容，就是。

"", string
[],  array
//,  regexp

vim中： 跳到行末： $
替换，所有的行首的注释：  ^

```
# a = 1
# b = 2
```
如何替换成：
```
a = 1
b = 2
```

/name/  就代表了 所有的  "name"
135   =>    \d\d\d

\d : 数字
.  : 字母
*  : 任意一个字符
$ : 末尾
^: 开头
{}: 出现的字数, 例如：  {3},  {4,6} 表示出现： 4, 5 ， 6次。
\d* ： 表示出现任意次。

例如：
电话：   135-2222-8888
的正则表达式：
\d\d\d-\d\d\d\d-\d\d\d\d
\d{3}-\d{4}-\d{4}


所以，正则表达式的特点：
1。 用起来，高手都不知道怎么就弄成了。特别抽象。
2.  容易弄得特别复杂。


http://deerchao.net/tutorials/regex/regex.htm


常用的限定符
代码/语法       说明
*       重复零次或更多次
+       重复一次或更多次
?       重复零次或一次
{n}     重复n次
{n,}    重复n次或更多次
{n,m}   重复n到m次


a+       a, aa, aaa
a*      '',

贪婪与懒惰 （  多匹配与少匹配）

'==abcsalkdsfjals;dkfjalksdjfab='

贪婪模式：  /=.*=/     出现全部的字符
懒惰模式：  /=.*?=/    只出现： '=='

> '==abcsalkdsfjals;dkfjalksdjfab='.match( /=.*=/)
=> #<MatchData "==abcsalkdsfjals;dkfjalksdjfab=">
irb(main):006:0> '==abcsalkdsfjals;dkfjalksdjfab='.match( /=.*?=/)
=> #<MatchData "==">
