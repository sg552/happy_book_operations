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
$ ./configure --prefix=/opt/app/nginx
$ make && make install
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

## 使用

## 配置一个静态网站：

需要加上 listen, location, root等属性。

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



配置一个 Proxy:

```
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

# 基本功能

## 作为server

nginx 是一个HTML服务器，用来返回HTML所用的内容。

## master-workers process

nginx包含一个master process, 多个 worker process.

master process 用来读取配置文件， 控制 worker 的。

worker就是用来处理 request的。

nginx 由于使用了 event-based model, 所以处理request速度很快。

## 日志

一般都放在 /var/logs/nginx目录下的 access.log, error.log  ，具体要看配置文件

## 启动，停止等命令

```bash
$ nginx -s stop # 快速停止

$ nginx -s quit # 比较得体的停止

$ nginx -s reload # 重新读取配置文件（ 重启）

$ nginx -s reopen # 重新打开日志
```


# 处理 request 的过程

refer  to http://nginx.org/en/docs/http/request_processing.html#simple_php_site_configuration

##  Name based virtual servers

会根据request header中的 server name来配对，例如， 给定配置文件：

```nginx
server {
    listen      80;
    server_name example.org www.example.org;
    ...
}

server {
    listen      80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      80;
    server_name example.com www.example.com;
    ...
}
```

如果我们请求   example.come/lala , 那么，就会被匹配到第三个配置项目上。

如果啥也没指定，直接 localhost:80, 那么，就会被匹配到 default site上（通常是第一个）。

这个是配置default site的例子：

```
server {
    listen      80 default_server;
    server_name example.net www.example.net;
    ...
}
```

如果希望阻止用户访问 localhost:80呢？  按照下面的方式配置： （就是配置一个 server_name = ''的server)

```
server {
    listen      80;
    server_name "";
    return      444;
}
```

再来个高级些，或者说 ”奇怪些“的功能：如何监听从 192.168.1.2 请求过来的，访问80端口的request呢？

需要按照下面的方式配置：

```
server {
    listen      192.168.1.1:80;  # 监听 192.168.1.1 过来的，访问80端口的request
    server_name example.org www.example.org;
    ...
}

server {
    listen      192.168.1.1:80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      192.168.1.2:80;
    server_name example.com www.example.com;
    ...
}
```


# nginx中 配置 中文域名 ( configure chinese domain name in nginx)

refer to: http://www.92csz.com/57/853.html

开始配置直接写的中文域名，但是解析不到正确的server，后来google了一把终于找到原因了，当在浏览器中敲入www.明月博客.com时，浏览器会转为www.xn--9kRq6Qw2Iu2A.com
其实中文域名就是一个经过编码的英文域名（中文域名-->punycode编码-->英文域名）
在线转换地址http://www.cnnic.net.cn/html/Dir/2003/10/29/1112.htm


配置：

```
listen 80;
server_name www.xn--9kRq6Qw2Iu2A.com;
index index.html;
```

p.s. 中文域名好难看
