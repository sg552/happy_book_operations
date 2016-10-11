# Git 里的边缘知识

不知道的话，循规蹈矩去做，也能正常使用。  (特别适合初创公司）
边缘知识： 知道了之后，可能会更加清晰一些。

## origin 是什么东东？

git push -u origin master

`origin` : 是远程服务器的“别名”。

例如：

小王， 参与了项目： library.



   o
  / \       ======>     Github.    library
  小王
   |
   |
   |
  /workspace/library


  小王 给 github 起个名字， 就叫：  origin .

所以， 小王在本地，运行：  $ git pull/push origin master 的时候，
小王本地的 git 就会 去 github 上 同步代码。


在如： 突然有一天， GFW， 把Github给屏蔽了。  或者说，国内访问github 太慢了。于是，我们就去了coding.net

   o
  / \   (origin)         ======>     Github.    library
  小王
   |    (new_source )    ======>     Coding.net library
   |
   |
  /workspace/library

所以，小王，就为  coding.net 在本地建立了一个别名：  `new_source`

git remote add new_source https://git.coding.net/Andrew_Du/happy_book_from_dashi.git
git push -u origin master



于是， 小王就可以同时使用 两个源头，来操作了。

$ git pull origin master  (  把代码，从github 更新过来）

$ git add/commit  (在本地做修改，提交）

$ git push new_source master ( 把代码， 放到 coding.net 上）


可以看出， git 功能极其强大。 但是： 这个 特点不实用。

git 特点： 只被英语好的程序员使用。 国内的公司， 用GIT的不多。 30%~50% （36kr 2015年统计）


## git push origin master 的具体含义？

完整写法： $ git push origin master:master

简略写法： $ git push origin master

## 更加简写的方法： $ git push

当我们本地只有一个分支, master, 远程也只有一个分支， master 的时候，

$ git push origin master:master 就可以省略成：

$ git push

在 git 的新版本中， 会有一个默认的设置。 见下面的 git push default 的警告问题。

## git remote add 的具体含义？

git remote add origin git@github.com/waterman112/git_study_from_liao_xue_feng.git

上面的语句的作用：

向 `.git/config` 文件中，增加一段话：

```
[remote "origin"]
    url = git@github.com/waterman112/git_study_from_liao_xue_feng.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```

## git@github.com/waterman112/git_study_from_liao_xue_feng.git 这串URL是干嘛的？

`git@` 相当于 协议（类似于： http:// )
`github.com`  git server 的网址
`waterman112` 用户
`git_study_from_liao_xue_feng.git`:  git repo 的名字。

使用的时候，把它看成一个整体。 整个复制下来。就够了。


##  push.default 的警告问题。

当我只输入下面命令的时候：

$ git push

系统会给出提示：

```
warning: push.default is unset; its implicit value has changed in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the traditional behavior, use:

  git config --global push.default matching

  To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

```

上面的意思就是： 对于git 2.0, 还是要指定： 默认的 本地分支和远程分支的。

使用
  git config --global push.default matching
就是让git 回归到 1.x 的行为。

## 最实用的办法： 把 git当成svn 来用


先讲个真实的故事。 发生在 裁员前的 motorola.

小S：
本地有2个分支： 1. master   2. siwei
远程有9个分支：
  1. master
  2. production
  3. staging
  4. siwei
  5. jiawei
  6. lily
  7. qi
  8. liang
  9. gao

总之， 远程上，每个同事，都有一个自己的分支。

小S。 每次提交代码时，都要先切换到 local - siwei 分支， 然后把commit 从master merge 到 siwei.
然后，$ git push origin siwei:siwei
然后： 告诉 老N:  我已经提交代码到remote:siwei了，请审核我的代码，可以的话合并，部署。
老N： 好的。 然后，code review, 如可以，  $ git checkout master,  git merge siwei.  (在远程， 把siwei分支的代码，合并到 master上）
老Q： 问老N： 我要部署了，有哪些commit 可以部署？ 老N说： master上的， commit:  a1b2c3 之前都可以部署。
老Q： 好的。把 master上 a1b2c3之前的commit 合并到  staging上。 （测试服务器）
一周后，
老Q： 可以把代码放到production吗？ 老N： 可以！
小S的代码，才会最终进入到 production上。


另外： 在美国，专门有公司，为了GIT MERGE，雇佣专门的人。

所以：

不要做很多个分支
不要每次部署前都要合并代码。

太浪费人力了。 而且你会发现： 太多分支，太多tag， 没有什么价值。

有的公司，专门养个人，每天只做git的分支合并。
不会解决问题， 反而，git 复杂的使用，会导致生产率的下降。

最好的办法：  只有一个 remote， 就叫origin .
只有一个分支： 就叫： master

这样的话 就很简单了。
最多，每次部署前，打个标签（tag)

## 没必要学的知识

rebase
cherry-pick


## gitlabs

这是私有的git服务器端。  （就好像： mysql server )

太复杂了。 特别不好配置，不好调试。 我在2010年的时候，做一个美国项目。 就在我们的AWS 上搭建过。 痛苦死了。
另外，客户端做很多操作时， 没有错误提示信息。

完全不可读。  所以不建议用这个。

而且github是世界上 代码托管独角兽。 我们用它就好了。
（国内的商业项目，用 coding.net, 因为它备案过了, 速度极快。）
