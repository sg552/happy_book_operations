# 大并发下的nginx

# nginx worker process is shutting down

今天修改完nginx 的配置，重启之后，发现有 3个nginx进程是这样，其他5个左右都是正常的nginx; 看起来跟下图一样： ( today I met a strange case that some of my nginx processes become "is shutting down" status. )

```
nobody 6246 6241 0 10:51 ? 00:00:00 nginx: worker process
nobody 6247 6241 0 10:51 ? 00:00:00 nginx: worker process
nobody 6247 6241 0 10:51 ? 00:00:00 nginx: worker process
nobody 6248 6241 0 10:51 ? 00:00:00 nginx: worker process
nobody 6249 6241 0 10:51 ? 00:00:00 nginx: worker process
nobody 7995 10419 0 Jan12 ? 00:20:37 nginx: worker process is shutting down
nobody 7995 10419 0 Jan12 ? 00:20:37 nginx: worker process is shutting down
nobody 7996 10419 0 Jan12 ? 00:20:11 nginx: worker process is shutting down
```

经过google, 才知道这是 nginx -s reload之后， nginx 正在平滑的重启。(graceful reboot) 。 等调用对应nginx 的进程结束之后，这个process就会重启了。 (after googling, I found that the root cause is nginx's graceful reboot. the nginx process which shoud be shut down is still responding a request, and it will restart once the current request is done )

果然，大约20分钟后，这些进程都变成了 'worker process' 了。( 20 minutes later, all the nginx worker are rebooted ) o


# 使用 goaccess分析nginx日志（analyze Nginx log using GoAccess)

2014-04-14 09:41
分类： 技术
nginx 日志无法用rails-request-analyzer 来分析。 需要使用 goaccess 。 因为前者一分析就死机，后者速度更快，分析1.2G的日志大约20秒。 ( if you want to analyze nginx log, use GoAccess)

用法非常简单：  (quite simple to use)

```bash
$ goaccess -f <your_log_file> -a > result.html
```

但是goaccess的一个缺点是：  分析大日志时，如果你的机器内存太小，就会报错退出。例如，你的机器是4G内存，但是要分析的内容是7G大小，这时候就会 在机器运行2，3分钟，接近死机是，出现 Killed 的结果（还好GoAccess会自动 干掉这个进程 )  ( but it's a weakpoint that goaccess can't analyze big file, e.g. 7G size.  )

所以解决办法是： 1. 把大日志切成小文件。  2. 分析小文件。

```bash
$ split -b 2G <your_log_file>
```

# 动态web页面 的速度跟nginx服务下的静态页面的支持速度，天壤之别啊。 (dynamic pages is so slow comparing to static pages served by nginx)

今天心血来潮，比较了一下静态服务器和动态WEB服务器，在同样并发下的相应速度。 前者是后者速度的50倍。 在我的机器上轻松支撑到 15K req/s, 而 使用了cache的 rails : 300 req/s. 哎。。。 (in short: nginx serving static page is 50 faster then rails server using cache)

thin:

```
Concurrency Level:      1000
Time taken for tests:   3.068 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      574000 bytes
HTML transferred:       311000 bytes
Requests per second:    325.98 [#/sec] (mean)
Time per request:       3067.652 [ms] (mean)
Time per request:       3.068 [ms] (mean, across all concurrent requests)
Transfer rate:          182.73 [Kbytes/sec] received
```

nginx:

```
Concurrency Level:      1000
Time taken for tests:   0.068 seconds
Complete requests:      1000
Failed requests:        1000
   (Connect: 0, Receive: 0, Length: 766, Exceptions: 234)
Write errors:           0
Non-2xx responses:      481
Total transferred:      415286 bytes
HTML transferred:       271101 bytes
Requests per second:    14735.35 [#/sec] (mean)
Time per request:       67.864 [ms] (mean)
Time per request:       0.068 [ms] (mean, across all concurrent requests)
Transfer rate:          5975.96 [Kbytes/sec] received
```

