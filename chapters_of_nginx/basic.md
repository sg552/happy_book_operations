# 基础

## 安装

Nginx 可以安装多种操作平台上。你可以运行在windows,mac上玩一玩，但是实际要用的话，都是
运行在Linux上的。

- Linux下的安装：

从源代码编译：

```
$ wget http://nginx.org/download/nginx-1.9.11.tar.gz
$ tar zxvf nginx-1.9.11.tar.gz
$ cd nginx-1.9.11
$ ./configure --prefix=/etc/nginx
$ make && sudo make install
```

或者在ubuntu下，直接

```
$ sudo apt-get install nginx
```

一般说来，在源代码编译最好，可以打开很多默认关闭的功能。

- Mac下的安装：

```
$ brew install nginx
```

## 启动/停止

- 启动
```bash
$ nginx
```

- 快速停止
```
$ nginx -s stop
```

- 比较得体的停止
```
$ nginx -s quit
```

- 重新读取配置文件（重启）
```
$ nginx -s reload
```


## 配置一个静态网站：

假设我们有个html页面, 位于 `/opt/app/nginx_basic/hi.html`:

```html
<html>
  <body> hello! </body>
</html>
```

我们需要编辑配置文件:

```
$ sudo vim /etc/nginx/conf/nginx.conf
```

### 配置文件介绍

下面就是 /etc/nginx/conf/nginx.conf 配置文件的内容

```
# 总共有4个worker, 一般来说, worker数与CPU核心数量相同. 4核CPU,就弄4个worker.
worker_processes  4;

# error log 文件.
error_log  logs/error.log;
# pid 文件.
pid        logs/nginx.pid;

events {
    # 每个worker同时最多可以打开多少 连接. 这里是优化的关键.
    worker_connections  1024;
}

http {
    # 包含若干  mime的 类型.
    include       mime.types;
    # 默认的 mime 类型
    default_type  application/octet-stream;
    # 域名长度.
    server_names_hash_bucket_size 64;
    # 日志格式
    log_format my_format '$remote_addr - $remote_user [$time_local]  '
      '"$request" $status $body_bytes_sent '
      '"$http_referer" "$http_user_agent" $upstream_cache_status';
    # access 日志.
    access_log /var/log/nginx/access.log my_format;
    # 出错的日志.
    error_log  /var/log/nginx/error.log;
    # 是否允许客户端上传文件
    sendfile        on;
    # 客户端连接超时的时间
    keepalive_timeout  65;
    # 打开 gzip 传输方式
    gzip  on;
    # 是否包含其他的配置文件
    include       /opt/nginx/sites-enabled/*;
}
```

可以看出,在上面文件中, 还包含了其他的配置文件。

```
    include       /opt/nginx/sites-enabled/*;
```

所以，我们在这个目录中，增加一个文件： hello

```
server {
    # 定义了监听的端口
    listen 3333;

    # 定义了对于 “任意”的请求，都用这个响应规则
    location / {
        # 定义了文档的根目录
        root /opt/app/nginx_basic;
    }
}
```

然后就可以看到效果：

![nginx hello](/images/nginx_hello.png)


## 为不同的ULR 制定不同的响应规则

下面的例子中，对于'/images'请求，nginx会指向到 `/data` 的本地目录，否则，直接指向
`/data/www`:

```
server {
    listen 80;

    location /images/ {
        root /workspace/test_nginx/images;
    }

    location / {
        root /workspace/test_nginx/www;
    }
}
```

## 配置代理服务器

```
server {

    # 把所有的 / 请求，都指向 http://localhost:8080
    location / {
        proxy_pass http://localhost:8080/;
    }

    # 对于 gif, jpg, png 格式的文件，则指向 /data/images
    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

## master-workers process

nginx包含一个master process, 多个 worker process.

master process 用来读取配置文件， 控制 worker 的。

worker就是用来处理 request的。

所以我们可以看到， 服务器的进程列表中，有一个master进程，多个worker进程：

```
$ ps -ef | grep nginx
nobody   17659 24384  0 18:41 ?        00:00:00 nginx: worker process
nobody   17660 24384  0 18:41 ?        00:00:00 nginx: worker process
nobody   17661 24384  0 18:41 ?        00:00:00 nginx: worker process
nobody   17662 24384  0 18:41 ?        00:00:00 nginx: worker process
nobody   17663 24384  0 18:41 ?        00:00:00 nginx: cache manager process
ubuntu   19353 16896  0 19:16 pts/2    00:00:00 grep --color=auto nginx
root     24384     1  0 Feb04 ?        00:00:00 nginx: master process nginx
```

## 日志

日志非常重要，可以让我们了解到nginx到底发生了什么。如果离开了日志，我们没法debug.

一般都放在 /var/logs/nginx目录下的 access.log, error.log  ，具体要看配置文件

### 正常的日志(access log)

```
$ tail /var/log/nginx/access.log -n 1
14.49.36.61 - - [23/Feb/2016:19:00:36 +0800]  "GET /forums/zuo-pin-zhan-shi HTTP/1.1" 200
8499 "http://tidev.in/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/537.36
 (KHTML, like Gecko) Chrome/47.0.2526.106 Safari/537.36" MISS
```

上面的日志，给它做个解析，如下：

- 客户端的ip: 14.49.36.61
- 访问的时间：[23/Feb/2016:19:00:36 +0800]
- 以GET形式的请求，访问哪个URL： "GET /forums/zuo-pin-zhan-shi HTTP/1.1"
- response的代码：200
- response的内容长度： 8499
- 我方网站的域名： "http://tidev.in/"
- 客户端的User Agent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/537.36
 (KHTML, like Gecko) Chrome/47.0.2526.106 Safari/537.36"
- 是否触发了缓存：MISS

### 报错的日志(error log)

```
$ tail /var/log/nginx/error.log -n 1
2016/02/23 19:16:46 [error] 17659#0: *46688 open()
"/opt/app/doc.tidev.in/part1_ti_premier/setup_titanium_on_mac.html" failed
(2: No such file or directory), client: 66.249.64.21, server: doc.tidev.in,
request: "GET /part1_ti_premier/setup_titanium_on_mac.html HTTP/1.1",
host: "119.254.103.134"
```

对上面的日志，我们可以做解析：

- 错误发生的时间：2016/02/23 19:16:46 [error]
- 出错的进程的pid:  17659#0:
- 做open()操作时出错： *46688 open()
- 客户端请求的URL："/opt/app/doc.tidev.in/part1_ti_premier/setup_titanium_on_mac.html"
- 该请求的结果：failed
- 该请求失败的具体原因：没找到文件。(2: No such file or directory),
- 客户端的IP：client: 66.249.64.21,
- 服务器端：server: doc.tidev.in,
- 客户端发起的原始请求： request: "GET /part1_ti_premier/setup_titanium_on_mac.html HTTP/1.1",
- 服务器的IP：host: "119.254.103.134"

## nginx 处理 request 的过程

在nginx中，一个 server 可以由两个维度来确定：

- 端口, 如  80
- server name ， 如  uubpay.com

在同一个服务器上，可以在同一个端口上定义多个不同name的server.
同理，在同一个server name上，也可以定义不同端口的server。

### 配置域名

会根据request header中的 server name来配对，例如 下面的配置文件定义了三个不同的virtual host:

```nginx
server {
    listen      80;
    server_name example.org;
    ...
}
server {
    listen      80;
    server_name example.net;
    ...
}
server {
    listen      80;
    server_name example.com;
    ...
}
```

- 如果请求 example.come, 那么，就会被匹配到第三个server上。
- 如果请求 example.net, 就会被匹配到第二个server上。
- 如果什么都没指定，直接 localhost:80, 那么，就会被匹配到 default site上（第一个）。

### 配置default_server

使用 `default_server`:
```
server {
    listen      80 default_server;
    server_name example.net www.example.net;
    ...
}
```

## 配置 中文域名

在国内，有时候要用到中文域名，

其实中文域名就是一个经过编码的英文域名（中文域名-->punycode编码-->英文域名）
例如：当在浏览器中敲入`www.优优宝.com` 时，浏览器会转为: `xn--4oqa927g.com`

在线转换地址 http://www.cnkuai.cn/zhuanma.asp?act=save

配置：

```
listen 80;
server_name www.xn--9kRq6Qw2Iu2A.com;
index index.html;
```

同时，有时候要指定`server_names_hash_bucket_size 64`
