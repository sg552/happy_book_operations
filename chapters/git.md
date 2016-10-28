# git 入门

想要完整的掌握git, 请看<<pro git>> 中文版: [https://github.com/progit/progit](https://github.com/progit/progit)

一点提示：Git是一本书， 我们要随时使用 git status 来处理git add/commit/push/merge中遇到的各种问题。


git , 虽然名字上叫 饭桶， 但是实际在功能上却可以秒杀传统的各种SCM， 比如 VSS,
CVS, SVN.

git跟上面三种SCM最大的区别，在于: git 可以完全脱离远程服务器。 在本地使用。

## 工作过程

git 的典型工作过程是；

- 程序员A修改了代码。(通过git status/diff查看修改的情况)
- 程序员A把修改后的文件，(通过git add/rm)放到了git的"待提交代码列表"中。
- 程序员A把“待提交的代码”提交(通过git commit)
- 程序员A把“本地的commit“ 同步到远程的git服务器上（通过git push)
- 程序员B 把程序员A的代码(通过git pull/merge)同步到本地。

## 几个概念

- working directory: 当前的代码目录

- branch: 代码的分支。 在本地可以有，在远程也可以有。

- commit: 每次的代码的提交。应该对应着某一个bug的修复, 或者对应 某个新增的功能.
每个commit 在git中都用40位的字符串来表示，例如`e4b924e3e0c03a6d71effc3bc3075c2596c14cf3`
每次提交的时候,必须有log. 例如: "增加了某某功能".
commit 是保存在本地的.

- staging area: 待提交的区域.  每次代码在被 git add/rm 之后，都会被放到这里.
这里的代码只是一个“待提交状态”。只有在 git commit 之后，本地的git系统才会
认为有一个专门的commit 号 对应刚才提交的代码。

- cherrypick:
(不用学)  在任意的 时间线上, 把任意一个 "不连贯"的 commit , 合并过来. 例如,

- rebase
(不用学) 破坏时间线.

- merge 合并. 如果对于同样一个文件, 本地有修改, 远程也有修改, 那么 在pull 的时候,
如果无法自动合并, git 就会提示, 不知道如何合并. 需要人肉手工做.

- push 把本地的新增的commit, 放到远程.  (让其他同事使用)

- pull 把远程新增的commit(往往是其他同事的提交的代码), 放到本地.

- 合并冲突 对于某些修改, 如果 两个不同的人, 修改了同个文件的同一行, 那么git 就无法自动合并.
(因为git是把每一行作为一个最小单位)  所以这个时候,就要人肉的合并冲突.

## Git 极速入门

这里非常不错： http://www.lovecloud.info/index.php/2010/02/08/%E6%9C%80%E8%BF%91%E5%AD%A6%E5%88%B0%E7%9A%84%E5%87%A0%E4%B8%AAgit%E7%9A%84%E7%94%A8%E6%B3%95/

基本步骤是：

## 获取最新代码  git pull

## merge.


1. 在github上创建仓库


2. 在本地创建仓库

cool@cool:/workspace$ mkdir git_test
cool@cool:/workspace$ cd git_test/
cool@cool:/workspace/git_test$ git init .
Initialized empty Git repository in /workspace/git_test/.git/
cool@cool:/workspace/git_test$

3. 把本地的代码 先push 到远程:

todo 若干图片.
```
cool@cool:/workspace/git_test$ git status
On branch master

Initial commit

nothing to commit (create/copy files and use "git add" to track)
cool@cool:/workspace/git_test$ vim apple.rb
cool@cool:/workspace/git_test$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	apple.rb

nothing added to commit but untracked files present (use "git add" to track)
cool@cool:/workspace/git_test$ git add .
cool@cool:/workspace/git_test$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   apple.rb

cool@cool:/workspace/git_test$ git commit -m '第一次的提交'

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'cool@cool.(none)')
cool@cool:/workspace/git_test$ git config --global  user.email  "330469102@qq.com"
cool@cool:/workspace/git_test$ vim
cool@cool:/workspace/git_test$ git config --global  user.name "yuancheng"
cool@cool:/workspace/git_test$ vim
cool@cool:/workspace/git_test$ git commit -m '第一次提交'
[master (root-commit) 0995254] 第一次提交
 1 file changed, 6 insertions(+)
 create mode 100644 apple.rb
cool@cool:/workspace/git_test$ git status
On branch master
nothing to commit, working directory clean
cool@cool:/workspace/git_test$ git log
commit 099525404b46a0203f82acb3c183923ba13558a0
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:33:01 2016 +0800

    第一次提交
cool@cool:/workspace/git_test$ git push origin master
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
cool@cool:/workspace/git_test$ git push -u origin master
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
cool@cool:/workspace/git_test$ git remote add origin https://github.com/fodcool/my_git_test.git
cool@cool:/workspace/git_test$ cat .git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = https://github.com/fodcool/my_git_test.git
	fetch = +refs/heads/*:refs/remotes/origin/*
cool@cool:/workspace/git_test$ git branch -a
* master
cool@cool:/workspace/git_test$ git push origin master
Username for 'https://github.com': ^C
cool@cool:/workspace/git_test$ ^C
cool@cool:/workspace/git_test$ git push origin master:master
Username for 'https://github.com': cool22336
Password for 'https://cool22336@github.com':
cool@cool:/workspace/git_test$ git push origin master:master
Username for 'https://github.com': fodcool
Password for 'https://fodcool@github.com':
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 273 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/fodcool/my_git_test.git
 * [new branch]      master -> master
cool@cool:/workspace/git_test$ git branch -a
* master
  remotes/origin/master
cool@cool:/workspace/git_test$ git log
commit 099525404b46a0203f82acb3c183923ba13558a0
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:33:01 2016 +0800

    第一次提交


```


假设远程有个 小李的同学,推送了一个修改:

在同个文件(apple.rb )中的第二行,也加上自己的修改信息:



小王, git pull , 会发现冲突:

```
cool@cool:/workspace/git_test$ git pull origin master
From https://github.com/fodcool/my_git_test
 * branch            master     -> FETCH_HEAD
Auto-merging apple.rb
CONFLICT (content): Merge conflict in apple.rb
Automatic merge failed; fix conflicts and then commit the result.

```

小王这个时候, pull的操作,只是执行了 `fetch`, 还没有merge:


```
cool@cool:/workspace/git_test$ git log
commit 2320e1eec1bad5f8e1da23278a6a9582b4278aba
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:39:34 2016 +0800

    第二行加上了 作者信息

commit 099525404b46a0203f82acb3c183923ba13558a0
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:33:01 2016 +0800

    第一次提交
```

我们输入 status:

```
cool@cool:/workspace/git_test$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   apple.rb
```

编辑 apple.rb

```

<<<<<<< HEAD
# 这是一个Apple类
# 作者小王
=======
#
# 我是小李
>>>>>>> e994603b36330cdafb81482aa1a67de8834b4ade
class Apple

  def color
    return "red"
  end
end
```

上面的 HEAD 和 === 之间,表示的是 本地的commit的内容.
==== 和 e9946 之间, 表示的是 远程的 commit 的内容.

我们就可以人肉修改了:

```

# 这是一个Apple类
# 作者: 小王和小李
class Apple

  def color
    return "red"
  end
end
```

修改完之后, 我们还要告诉 git, 我们修改好了这个文件, 否则 git会一直把这个文件标记成 `both modified`.

```

cool@cool:/workspace/git_test$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   apple.rb
```
我们要告诉git, 已经修改好了这个文件(就是使用最普通的 git add /commit)
```
no changes added to commit (use "git add" and/or "git commit -a")
cool@cool:/workspace/git_test$ git add apple.rb
cool@cool:/workspace/git_test$ git status
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	modified:   apple.rb
```

```
cool@cool:/workspace/git_test$ git commit -m "人肉做了合并"
[master fb957de] 人肉做了合并
```

$ git status 可以查看下当前的状态.
```
cool@cool:/workspace/git_test$ git log
commit fb957de2cef371bb18e262f783db712343fac0e3
Merge: 2320e1e e994603
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:51:48 2016 +0800

    人肉做了合并

commit e994603b36330cdafb81482aa1a67de8834b4ade
Author: 小李 <xiao_li@qq.com>
Date:   Fri Oct 28 10:42:18 2016 +0800

    增加了作者信息: 小李

commit 2320e1eec1bad5f8e1da23278a6a9582b4278aba
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:39:34 2016 +0800

    第二行加上了 作者信息

commit 099525404b46a0203f82acb3c183923ba13558a0
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:33:01 2016 +0800

    第一次提交
```


小李在它那边就可以 git pull了:

```
cool@cool:/workspace/git_test_for_xiao_li$ git pull
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), done.
From https://github.com/fodcool/my_git_test
   e994603..fb957de  master     -> origin/master
Updating e994603..fb957de
Fast-forward
 apple.rb | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)
cool@cool:/workspace/git_test_for_xiao_li$ git log
commit fb957de2cef371bb18e262f783db712343fac0e3
Merge: 2320e1e e994603
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:51:48 2016 +0800

    人肉做了合并

commit e994603b36330cdafb81482aa1a67de8834b4ade
Author: 小李 <xiao_li@qq.com>
Date:   Fri Oct 28 10:42:18 2016 +0800

    增加了作者信息: 小李

commit 2320e1eec1bad5f8e1da23278a6a9582b4278aba
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:39:34 2016 +0800

    第二行加上了 作者信息

commit 099525404b46a0203f82acb3c183923ba13558a0
Author: yuancheng <330469102@qq.com>
Date:   Fri Oct 28 10:33:01 2016 +0800

    第一次提交
```

## 提交代码 git add/commit/push

## 查看历史 git log/show

## 做代码比较 git diff

### 查看所有的GIT变量：

```bash
$ git var -l
```

### 可以载入变量:
```bash
git config -f = ~/.gitconfig
```

## 提交时需要：（代码review的小要求）

- 有正确的 用户名和电子邮件.
- comments 中就不要有"由XX操作"了。重复。

```
$ git -config user.name "Jike Song"
$ git -config user.email [email]albcamus@gmail.com[/email]
```

注意，这样会在当前repository目录下的. git /config中写入配置信息。 如果 git -config加了--global
选项，配置信息就会写入到~/. git config文件中。 因为你可能用不同的身份参与不同的项目，而多个
项目都用 git 管理，所以建议不用global配置。

## 生成本地修改的所有patch（多少次提交就多少个.path文件）:

```bash
$ git format-patch origin
```

## 生成单个patch文件（例子中是将最近5次提交的内容合并到一个文件中）：

```bash
$ git format-patch -5 --stdout  > patch_by_siwei.txt
```

## 往远程服务器上提交分支：

```bash
$ git push origin [本地分支名]:[远程分知名(push之后就存在了)]
```

例如：(理论上)

```bash
git push origin my_local_branch:lily
```

提交之后远程就会出现了一个"lily"分支。

git push new_source master:lily
Total 0 (delta 0), reused 0 (delta 0)
To https://git.coding.net/sweetysoft/happy_book_test.git
 * [new branch]      master -> lily
192:happy_book_operations Andrew$ git branch -a
* master
  remotes/new_source/lily
  remotes/new_source/master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master


## apply patch

最好在linux环境下。如果出现的诡异的 /dev/null问题，
十有八九是因为dos文件格式造成的。试试 dos2unix 。
如果还不行的话。。。哎，手工吧！
血的教训： 每天时不时的 update 一下，
绝对不要等最后push的时候再合并。。。痛苦啊。

每天最好更新一下远程服务器中的代码：

```bash
$ git pull . master    (把远程的master更新到当前的本地分支）
```

## git patch/apply 某一个commit

如何在另外一个机器上 apply某个 path？

我们遇到一个情况， 在某个机器上，需要一个紧急部署，但是又不希望手工去修改（因为已经有了一个commit 了）。这样的情况下，可以使用git path 专门为某个commit 生成patch, 然后在这个远程机器上apply it.  ( apply a patch generated from a specific commit ?)

```bash
$ git format-patch -1 <sha>   # =>  0001__.patch
$ git apply <path_file >
# git cherry-pick. 如何把已经提交的commit, 从一个分支放到另一个分支
```

## cherry pick
实际问题：
在本地 master 分支上做了一个commit (
38361a68138140827b31b72f8bbfd88b3705d77a ) ，
如何把它放到 本地 old_cc 分支上？

办法之一： 使用 cherry-pick.  根据git 文档：

就是对已经存在于别的分支的commit 放到当前的分支上。

简单用法：

```bash
git cherry-pick <commit id>
```

例如：
```bash
$ git checkout old_cc
$ git cherry-pick 38361a68     # 这个 38361a68 号码，位于：
```

```bash
$ git log
commit 38361a68138140827b31b72f8bbfd88b3705d77a
Author: Siwei Shen <siwei.shen@focusbeijing.com>
Date:   Sat Dec 10 00:09:44 2011 +0800
```

如果顺利，就会正常提交。结果：

```bash
Finished one cherry-pick.
# On branch old_cc
# Your branch is ahead of 'origin/old_cc' by 3 commits.
```

如果在cherry-pick 的过程中出现了冲突

```
Automatic cherry-pick failed.  After resolving the conflicts,
mark the corrected paths with 'git add <paths>' or 'git rm <paths>'
and commit the result with:

git commit -c 15a2b6c61927e5aed6718de89ad9dafba939a90b
```

那么我们就当它跟普通的冲突一样，手工解决：

2.1看哪些文件出现冲突
```bash
$ git status

both modified:      app/models/user.rb
```

2.2手动解决它。

```bash
$ vim app/models/user.rb
```

2.3

```bash
$ git add app/models/user.rb
```

2.4

```bash
git commit -c <新的commit号码>
```

## 恢复做过的改动

## 取消上一次提交？
$ git reset --soft HEAD~1

`--soft` 表示把当前git的状态（假设是干净的），还原成“产生了改动，保留在待提交区“中。
`HEAD~1` 表示最新的一个提交。

```bash
Changes to be committed:

modified:   .gitignore
```

继续，我们把修改的文件，从“待提交列表”放回到普通状态：

```bash
$ git reset HEAD .gitignore
Unstaged changes after reset:
M .gitignore
Changes not staged for commit:
  modified:   .gitignore
```

我们可以继续把它的改动恢复：

```bash
$ git checkout -- .gitignore
```

## 分支的知识( git branch)

### 查看所有的分支

所有以 "remotes/"开头的分支，都是远程的分支。
`* master`表示 当前的分支是 master.

```bash
$ git branch -a

sg552@youku:/sg552/workspace/m-cms$ git branch -a
  clean_cache
* master
  siwei_branch
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/siwei_branch
```
上面的 `origin/HEAD -> origin/master`表示 默认的分支就是`origin/master`.

显示本地和远程的origin 上的分支情况

( git pull的过程, 例如: $ git pull origin master
表示: 把远程的  `remotes/origin/master` 合并到 本地的 `master`
git pull 分成两个动作:

1. git fetch
2. git merge

)

## git clone, git pull, git fetch 的区别

clone: 本地没有这个代码时,使用.  (第一次获取代码,用clone)
fetch: 只是把代码 从远程弄到本地, 但是不合并(无merge). 这个命令,用的极少.
pull: 本地有代码仓库, 希望获取 远程的某个commit, 才用pull. 相当于 fetch + merge. (用的最多)

### 新建分支

```bash
$ git branch new_branch
$ git branch -a
# 结果是：
* master
  new_branch
```

### 切换分支

```bash
$ git checkout new_branch
```

### 在新的分支上操作代码

跟普通的操作一模一样。

可以认为是两套不同的代码。 他们有相同的基础。

### merge : 在两个不同的分支间操作commit

比如，我们把 branch1 中的 commit1.1 放到branch2 上：

```bash
$ git checkout branch2
$ git merge branch1
Updating e4b924e..df5529b
Fast-forward
 file1 | 3 +++
 1 file changed, 3 insertions(+)
```
然后，我们再

```bash
$ git log
```
就能看到， 原来位于 branch1 中的 commit 1.1 来到了branch2 上。


### 删掉分支

下面的操作，删掉了 "new_branch" 这个分支。
```bash
$ git branch -d new_branch
Deleted branch new_branch (was e4b924e)
```

### 显示git push/fetch/pull等的细节。

```bash
sg552@youku:/sg552/workspace/m-cms$ git remote show origin
* remote origin
  Fetch URL: ssh://shensiwei@gforge.1verge.net:22022/gitroot/m-cms
  Push  URL: ssh://shensiwei@gforge.1verge.net:22022/gitroot/m-cms
  HEAD branch: master
  Remote branches:
    master       tracked
    siwei_branch tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local refs configured for 'git push':
    master       pushes to master       (up to date)
    siwei_branch pushes to siwei_branch (fast-forwardable)
```

## git tag 基本操作

作用: 把可读性不强的commit 号(a70cff) 改成:可读性强的 1.0
### 查看本地所有的tag

```bash
$ git tag -l
```

### 列出远程的所有的tag:

```bash
$ git ls-remote --tags

From ssh://....
a70cffc64e73ae4b5ab7329480999305f11d9c76  refs/tags/1.0.0
6d1ad6c601652442d6d32fce5233ba50a1dd9f56  refs/tags/1.0.0^{}
1acf5e9aabbd353ff45b7aa3fe319d3822acad19  refs/tags/1.0.1
5bba7a637e64267ffad38dd48c30df00da6cc91e  refs/tags/1.0.1^{}
```

### 新增tag

名字叫 1.1.0, 对应的commit 是 "a50fabb":
```bash
$ git tag -a 1.1.0 a50fabb -m '版本1.1.0'
```

### 删掉tag:

例如，删掉名为 '1.0'的tag:
```bash
$ git tag -d 1.0
```

### 显示tag的详情：

```bash
$ git tag -n2   # will show the tags with their messages.

1.0.0           CMS 3.0 的抢先版，....
1.0.1           3月14日下午.修改了若干BUG（大部分都是表单验证）...
```

### 把本地的tag推送到远程git服务器上

```bash
$ git push --tags
```


## git stash

当我们做一个功能，做到一半时, 来了新需求，该怎么办?

我们可以使用 git stash .把当前的改动保存起来，以备日后使用。

### 新建 stash
```bash
$ git stash save "登陆功能做到一半，完成了视图，控制器未完成"
```

### stash 列表

然后我们使用 `git stash list`命令，就可以看到:

```bash
$ git stash list

stash@{0}: On master: "登陆功能做到一半，完成了视图，控制器未完成"
```

### 恢复 代码

当我们做完另外一件事后，要恢复代码，可以：

```bash
$ git stash apply stash@{0}
$ git status

On branch master
Changes not staged for commit:
(use "git add <file>..." to update what will be committed)
(use "git checkout -- <file>..." to discard changes in working directory)

modified:   file1
modified:   file2
```
可以看到 工作区 又回到了被改动的状态。改了一半的文件又出来了。


### 删除 stash

某个 stash 用不到后，可以把它删掉。

```bash
$ git stash drop stash@{0}
```

## git log

查看日志列表
例如：

```bash
$ git log

commit e4b924e3e0c03a6d71effc3bc3075c2596c14cf3
Author: Shen Siwei <shensiwei@sina.com>
Date:   Tue Oct 13 11:30:31 2015 +0800

增加了登陆功能

commit 7c688ea99317b740fe49193a930a42bcc1056fce
Author: Shen Siwei <shensiwei@sina.com>
Date:   Tue Oct 13 09:45:41 2015 +0800

initial commit
```

## git show

显示某个commit，例如, 我希望看到上面的 "增加了登陆功能" 的commit 所对应的
代码， 就可以：

```bash
$ git show e4b9
```

注意： 这里不须要写出完整的40位commit号，在可以准确定位commit的情况下，
给出前4位字符就可以了。

## git diff

如果file1 文件有了改动，那么我们就可以查看它的变动：

```bash
$ git diff file1
```

也可以显示当前工作区的所有文件的改动：

```bash
$ git diff .
$ git diff  # 也可以直接把后面的 . 省略掉
```

也可以显示某两个commit 之间的改动：

```bash
$ git diff <1#提交> <2#提交>
```

## Git rebase 笔记 ( git rebase introduction)

前几天的一个文章，我提到 自己比较偏好 git-merge. 现在看起来，这个问题不该带
有个人偏好。而是需要根据情况，来选择使用 git rebase /merge

### merge 与 rebase的区别

merge: 会保留时间线。 适合需要保持时间线的场合。  (it would keep the commit order for each branch )

rebase: 不会保留时间线。 适合按照 feature 来查看log的场合。比如，我现在同时在3个分支上工作，但是为了方便代码的提交，我会在一个完成之后再提交另外一个分支/功能模块。所以在这样的情况下使用rebase使得代码更加容易回滚，以及查看log.   ( the origin commit would be removed and a new commit would be generated .  if you are working on 3 branches at the same time, and want to merge the code till the whole module/branch is done, just use this 'rebase' )

简单用法：

$ git rebase master <current_branch>

这样就会把 master上的最后一个commit, 做为当前 分支的commit 的基础。

更多详细，参考 man git-rebase.   ( for more details, please refer to 'man git-rebase' )
