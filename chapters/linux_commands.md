# Linux的文件系统
# Linux的用户和用户组
# Linux下的进程, cpu, 内存, 硬盘, 网络

# linux下的常用命令 ( widely used linux commands)

## 入门
linux下的命令，都可以通过 -h 或者 --help 或者 man 来查看细节，例如：

```bash
$ git -h
$ git --help
$ man git
```

一般说来， `-h` 与 `--help`都是比较简洁的帮助信息，而`man <command>`给出
的都是非常系统的文档（很厚的一本书）

所以，大家遇到有趣、有用的命令时，最好自己看一眼它的文档。这样就能不断地提高。

另外，linux下的命令组成都是：

```
$ <命令> <参数> <目标名>
```

这样的模式。 而参数一般都是 `-h` 或者 `--help`，可以是一个横杠，也可以是两个，在Mac
下面<参数>必须在<目标>前面，在Linux中则前后都无所谓。

所谓的命令行，也叫 shell, bash, zsh , terminal  等等，基本认为是一样的。详细的说，
就是：TODO

说明：Mac也是Unix, 所以大多数的命令与Linux相同。

## shell 下的特殊的符号

- ~ 表示当前用户的home目录, 例如 `cd ~`
- - 表示前一个目录, 例如 `cd -`
- . 表示当前目录, 例如 `ls .`
- .. 表示上一层目录
- `$ ls` 表示在命令行下执行 "ls" 这个命令
- | 表示 pipe， 例如： `ls | grep a` ，先执行 ls, 然后在`ls`的结果中搜索"a"
- > 表示 写入信息，例如： `echo hihihi > file3` 把 "hihihi"这个本应由"echo"
命令输出的字符串写入到 "file3"中。

## ls

列出当前目录下的文件。
```
$ ls
file1 file2
```

查看当前目录下所有的文件，并且给出比较详细的信息：

```
$ ls -al
total 16
drwxr-xr-x   5 sg552  wheel  170 10 13 15:13 .
drwxr-xr-x  10 sg552  wheel  340 10 13 15:15 ..
drwxr-xr-x  14 sg552  wheel  476 10 13 15:13 .git
-rw-r--r--   1 sg552  wheel   28 10 13 15:13 file1
-rw-r--r--   1 sg552  wheel    8 10 13 11:30 file2
```

在当前目录下，从旧到新的列出所有的文件： (注意参数： trh)
```
$ ls -altrh
total 16
-rw-r--r--   1 sg552  wheel     8B 10 13 11:30 file2
-rw-r--r--   1 sg552  wheel    28B 10 13 15:13 file1
drwxr-xr-x  14 sg552  wheel   476B 10 13 15:13 .git
drwxr-xr-x   5 sg552  wheel   170B 10 13 15:13 .
drwxr-xr-x  10 sg552  wheel   340B 10 13 15:15 ..
```

```

## mkdir

新建一个目录：
```bash
$ mkdir my_folder
```

也可以一股脑建立多层文件夹：

```bash
$ mkdir a/b/c/d/e/f -p
```

## top
查看当前的系统（CPU, 内存，进程）状态。

```bash
$ top
```

进入界面后，默认按照CPU，内存排序。

按“1”就能看到CPU中的每个core的负载，
按"c" 就能看到每个进程所对应的具体的命令。
按回车可以立刻刷新状态。
这个命令用好了相当有用。可以找到当前最消耗CPU的进程。然后看它是否正常。

## ps

查看系统的进程。也是绝对重要的命令，可以非常直观地了解到当前服务器的内存使用状态。

列出系统中的所有名字中带有 "thin"的进程：
```bash
$ ps -ef | grep thin
```

显示出系统中的所有进程，并且以消耗的内存来从低到高的排序。
```bash
$ ps aux --sort rss
```

## df
查看当前服务器的所有分区，并且以用户

```bash
$ df -kh
```

## du
查看当前文件夹的大小

```bash
$ du . -kh  查看文件夹大小
```

## ln -s
新建一个soft link (软连接)。可以认为它就是windows中的快捷方式

## kill
终止某个进程。
例如： 某个进程的id是3366

```bash
$ kill -9 3366
```

这里务必使用 "-9" 参数。表示终止。还有其他的参数形式（在kill命令中叫signal），
具体请自行查看文档。

## crontab

crontab 是定时执行任务的工具。 我们需要使用下面命令进入到编辑页面：

```bash
$ crontab -e
```

TODO crontab的格式。

## tail

查看某个文件的尾部。 例如：

```bash
$ tail /var/log/system.log
```

默认显示文件的最后10行。

可以跟踪显示某个文件的尾部：
```bash
$ tail -f /var/log/system.log
```

可以指定显示该文件的尾部100行：
```bash
$ tail -n 100 /var/log/system.log
```

最常见的用法：跟grep 共同使用, 例如，实时跟踪显示某个文件，只过滤出内容中带有ERROR：

```bash
$ tail -f /var/log/system.log | grep ERROR
```

## head
## grep *** 重中之重啊。 一定要知道各种形式的参数， 以及各种变种。 比如：   $ grep -F 'fixed string' -R --include=*rb
## zgrep 搜索 压缩文件的内容
## find .
## rsync  传送文件
## sftp
## ssh
## wget
## curl HTTP 请求
## vim ~/.bashrc
## chmod
## chown
## passwd
## netstat
## ifconfig 查看网卡信息
## netrafic 查看流量的（似乎）
## top 命令 （进入之后的各种参数， 按c,1 )
## 各种服务器的命令： nginx -s reload|stop,  rails server...
## tar zxvf
## unzip
## pwd 获知当前目录
## cp -r
## mv

删除大文件： $ echo '' > big_file


知道各个系统文件夹的基本用处。  /var/logs,  /etc/

高级的：
## pipe     find . | grep '.rb'
## logrotate
## split
## awk
## seed
## bash script
## expect

## Linux 的几个好用的命令 ( sweet commands in linux)

这是一个循环，把 /tmp/test.lst 文件中的 每一行都读出来，然后循环运行rsync 命令 . (注意其中的 exclude ) ( a loop that reads all the lines of contents in a file named /tmp/test.lst and then run them one by one)

```bash
for i in `cat /tmp/test.lst`;do
rsync -avzPl --exclude="2013*" --exclude="201404*" --exclude='*.log*' --exclude='*.git' --exclude='*.sql' ./$i 10.103.13.37::cmscode
done
```

在当前目录下，找到所有的文件中包含的 10.103.13.74 ，并且把它替换成： 10.103.13.37 ( find all the '10.103.13.74' and then replace them with '10.103.13.37') 这个功能跟 VIM plugin: Greplace 是一样的。

```bash
$ sed -i "s/10.103.13.74/10.103.13.37/g" `grep "10.103.13.74" -rl ./`
```

## Linux 控制台神器：搜索历史命令 Ctrl + R

(press ctl + r ) 输入任意字符，例如： "mig"  就会出现 $  rake db:migrate    ( press ctrl + r, then input the content you want to search)

如果我想找另一个命令呢？  输入完 'mig' 多按几次 ctrl + r ，就可以继续向前搜索 “mig" 的内容了。  ( if you want to search the last but n result, just press n times 'ctrl + r'.)

如果找到了，按 -> 或者直接回车，就是执行  ( press Enter to execute the command , or 'right arrow (->)' to go to the end of the command.

