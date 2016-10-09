# Git 里的边缘知识

边缘知识： 知道了之后，可能会更加清晰一些。 但是不知道的话，循规蹈矩去做，也能正常使用。

## origin 是什么东东？

## git remote add 的具体含义？

git remote add origin git@github.com/waterman112/git_study_from_liao_xue_feng.git

上面的语句的作用：

向 `.git/config` 文件中，增加一段话：

```
[remote "origin"]
    url = git@github.com/waterman112/git_study_from_liao_xue_feng.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```


##  push.default 的警告问题。
warning: push.default is unset; its implicit value has changed in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the traditional behavior, use:

  git config --global push.default matching

  To squelch this message and adopt the new behavior now, use:

    git config --global push.default simple

## 最实用的办法： 把 git当成svn 来用

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
