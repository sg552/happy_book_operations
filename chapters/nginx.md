# nginx 基础 (nginx beginners guide)

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

## 配置一个静态网站：

可以看出， 对于 '/images'请求，会指向到 /data 的本地目录，否则，直接指向 /data/www

```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
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

TODO： 配置rails,  php, 查看nginx日志


# nginx 对文件(动态页面) 进行缓存 ( nginx content caching)

nginx可以对某个请求进行缓存，

例子：

```
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m;

    server {
        proxy_cache one;
        location / {
            proxy_pass http://localhost:8000;
        }
    }
}
```

设置好了允许缓存后，进一步可以设置它的过期时间： （iteration 如何解释。。需要动手弄一下）

## 参与cache 过程 的，有两个角色， cache manager 和 cache loader:

1. cache manager 会循环的检查 cache的状态。当它发现 缓存的文件超过了 max_size 这个数目后，就会删掉最少访问的cache page.

2. cache loader： 仅仅在nginx启动后 随之启动一次。它把之前的cache 信息加载到 shared memory中去。这在nginx启动的前几分钟会拖累nginx的速度。

以上的iteration, 比 loader_threshold（默认是200ms) 要少。 每次加载的文件数目小于 loader_files(默认是100），每个iteration 间隔 loader_sleeps (默认50ms）。

下面是个例子：

```nginx
proxy_cache_path /data/nginx/cache keys_zone=one:10m
                 loader_threshold=300 loader_files=200;
```

## 指定某个URL 要缓存

如果某个response来自 proxy_server, 并且request是 GET/HEAD 方法，则nginx 默认会把它做缓存.

而且默认使用的key就是 url ，你也可以指定这个key, 例如：

```
proxy_cache_key "$host$request_uri$cookie_user";
```

如果我们希望某个 url 至少被请求5次之后才被缓存，就这样：

```
proxy_cache_min_uses 5;
```

如果希望对POST和 DELETE进行缓存：

```
proxy_cache_methods GET HEAD POST；
```

下面的例子：对于 200 , 302的response, 缓存 10分钟，

```
proxy_cache_valid 200 302 10m;  # 对于 200， 302，缓存10分钟
proxy_cache_valid 404 1m;       # 缓存1分钟
proxy_cache_valid any 10m;      # 对于所有的响应，都缓存10分钟。
```

也可以根据条件来判断是否使用cache: （ cookie 中的变量：nocache, parameter中的变量：nocache 或者 comment, 只要有一个 不是空，也不是 0, 那么这个request就不会使用cache)

```
proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
```

对于下面的例子：压根就不使用cache:

```
proxy_no_cache $http_pragma $http_authorization;
```

下面是一个更大的例子：

```nginx
http {
    ...
    # 定义了一个 proxy_cache_path, :
    proxy_cache_path /data/nginx/cache keys_zone=one:10m
                     loader_threshold=300 loader_files=200
                     max_size=200m;

    # 这个server中有两个backend, 对应两种不同的cache策略
    server {
        listen 8080;
        # cache的名字叫做 one （注意上面的 keys_zone=one:10m)
        proxy_cache one;

        # 对所有的 / 请求，都尽可能长久的缓存，不存在过期
        location / {
            proxy_pass http://backend1;
        }


        location /some/path {
            proxy_cache_valid any   1m;  # 任何内容，都缓存1分钟
            proxy_cache_min_uses 3;      # 访问3次后，触发缓存
            proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;  # 设置好不使用缓存的规则
            proxy_pass http://backend2;
        }
    }
}
```

## 注意: 如何调试呢?

1. 要设置log format, 把日志打印出来. 例如,配置文件为: (注意其中的 $upstream_cache_status, 这个变量最重要, 从它我们可以知道, 是HIT 还是MISS )

```
    log_format my_format '$remote_addr - $remote_user [$time_local]  '
      '"$request" $status $body_bytes_sent '
      '"$http_referer" "$http_user_agent" $upstream_cache_status';

    access_log logs/my_access.log my_format;
```

2. 要有对应的 ignore headers, 如果后端返回的结果中,增加了 cache-control (也有一说是 set-cookie) 或者 啥的,就不行了.

```
server{
            proxy_ignore_headers "cache-control";
            proxy_hide_header "cache-control";
}
```

下面是一个完整的 nginx.conf例子;

```
http{
    # 其他内容
    proxy_cache_path /tmp/nginx_cache keys_zone=cache_one:10m
                     loader_threshold=300 loader_files=200
                     max_size=200m;

    log_format my_format '$remote_addr - $remote_user [$time_local]  '
      '"$request" $status $body_bytes_sent '
      '"$http_referer" "$http_user_agent" $upstream_cache_status';

    access_log logs/my_access.log my_format;

    server {
        listen       80;

        location / {
            proxy_ignore_headers "cache-control";
            proxy_hide_header "cache-control";
            proxy_cache cache_one;
            proxy_cache_valid any   10s;  # 任何内容，都缓存10秒钟
            proxy_pass http://rails_api;
        }

    }
    upstream rails_api{
        server localhost:3000;
    }
```

## 缓存用的哪些文件?

我们可以在 proxy_cache_path中设置, 例如:

```
    proxy_cache_path /tmp/nginx_cache keys_zone=cache_one:10m
                     loader_threshold=300 loader_files=200
                     max_size=200m;
```

然后, 找到 /tmp/nginx_cache 目录, 如果某个 cache被命中过, 就会看到出现一个以md5 结果命名的文件:

```bash
/tmp/nginx_cache$ ll
total 20
drwxrwxrwx  2 nobody sg552    4096 Sep 10 11:39 ./
drwxrwxrwt 10 root   root    12288 Sep 10 11:38 ../
-rw-------  1 nobody nogroup   594 Sep 10 11:39 f8924891f34a941a8342ccd19c4cf290
```

上面中, 这个文件 "f89..." 就是缓存文件. 它的内容如下.

```
���U���������Ud����0""b4945c5f2d4b62faae53f44f44a5e946"
KEY: http://rails_api/prices/say_hi
HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8
ETag: "b4945c5f2d4b62faae53f44f44a5e946"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: 90fbc91b-e5a4-4279-8832-5d484cba7ba8
X-Runtime: 0.007120
Connection: close
Server: thin 1.6.2 codename Doc Brown

time is: 2015-09-10 11:39:58 +0800
```

可以看出, 该静态文件, 以文本的形式缓存了 所有的response信息.


# 使用nginx 做代理, 使用 无法备案的域名

其实很简单.

一个例子, 我们的一个新款域名, 无法备案, 无法在国内的主机供应商上使用.

所以一种解决方案,是在香港上个主机, 然后把  special.domain 的A 记录指上去.

然后在 香港主机上, 弄个nginx,配置代理,把所有的request 都打到国内的服务器上去.

也就是:   用户 ->  special.domain -> 香港服务器 ->  大陆服务器(接收到了request, 返回response )  -> 香港服务器 -> 用户

那么, 香港的nginx配置如下:

```nginx
  server {
    listen       80;
   server_name  special.domain;
    charset utf-8;
    location / {
      expires 30m;
      access_log on;
      add_header Cache-Control "public";
      proxy_pass          http://normal.domain.com ;  #   这里就是 http://<国内服务器ip>;
      proxy_redirect      default;
      proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header    X-Real-IP $remote_addr;
      proxy_set_header    Host $http_host;
      proxy_next_upstream http_502 http_504 error timeout invalid_header;
    }
  }
```


# nginx的debug级的日志 (nginx debugging log)

refer to:  http://nginx.org/en/docs/debugging_log.html

1. 默认是不带这个特性的。需要重新编译安装：

./configure --with-debug ...
然后在 error_log directive 中，使用：

error_log /path/to/log debug;
注意，如果在下层配置文件中，重新定义了 error_log, 那么会覆盖掉上级的debug配置，例如：

```nginx
error_log /path/to/log debug;

http {
    server {
        error_log /path/to/log; # 这里会使上面的debug输出失效。
        error_log /path/to/log debug;  # 应该这样。
        ...
```

# nginx 处理 request 的过程 (how nginx processes a request)

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

# nginx跟安全访问相关的几个模块(nginx security related modules)

1. ngx_http_limit_conn_module

用来定义某个ip或者（key)访问的次数.

```
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    ...
    server {
        ...
        location /download/ {
            limit_conn addr 1;
        }
```

2. ngx_http_addition_module

在某个request之前或者之后加上参数，

```
location / {
    add_before_body /before_action;
    add_after_body  /after_action;
}
```

3. ngx_http_access_module

对于具体的IP地址段进行控制，（使用 deny, allow 等关键字）

```
location / {
    deny  192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny  all;
}
```

4.  ngx_http_auth_request_module

对访问进行控制，使用到了 sub-request（就是下面的 auth_request中所定义的 /auth )，如果sub-request返回2xx,那么整个 url就是可以访问的。否则就是不可以访问。

```
location /private/ {
    auth_request /auth;
    ...
}
location = /auth {
    proxy_pass ...
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
    proxy_set_header X-Original-URI $request_uri;
}
```

5. ngx_http_auth_basic_module

通过用户名-密码 文件来控制访问，比如：

```
location / {
    auth_basic           "closed site";
    auth_basic_user_file conf/htpasswd;
}
```

而 conf/htpasswd 的内容应该是：

```
# comment
name1:password1
name2:password2:comment
name3:password3
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

# nginx worker process is shutting down

2014-06-20 18:13
分类： 技术
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

# nginx的限速相关(nginx limit download rate)

指令名称： limit_rate、limit_rate_after \\limit_rate_after用于设置http请求传输多少字节后开始限速
使用环境： http, server, location, if in location
示例：

```
location /download {
    limit_rate_after 4m;
    limit_rate 512k;
}
```

限制单个IP最大连接数（线程数）
指令名称： limit_conn_zone
使用环境： http
示例：

```
http {
    limit_conn_zone $binary_remote_addr zone=client_addr:10m; \\这是内存的开销
}
```

指令名称： limit_conn
使用环境： http, server, location
示例：
```
server {
    location /download {
        limit_conn client_addr 1;
    }
}
```

隐藏Nginx版本信息：

```
# curl --head http://www.tianyun.com //查看主机的响应头信息
http{
    server_tokens off;
}
```

# 在nginx转发时保留原始域名( keep the Host header via nginx proxy_pass )

最近的子项目越来越多，但是只有一个域名， 在做了nginx的跳转之后， 发现原来的域名会丢失，取代出现的是IP。

(with the growth of the sub-systems of my current project, there is a problem occurred: the origin domain name disappeared and it is replaced by the ip addresses , which is not liked by my workmates.  )

所以我们要保留原始域名，这货也叫 Host header.  （so the solution is to keep the origin domain name . aka host Header.  ）

solution is : proxy_set_header Host $http_host;

```
    server {
        listen 80;
        # this is the key !!!!!
        proxy_set_header Host $host;
        location /client/pids {
            proxy_pass http://10.103.13.103:3200/client/pids;
        }
        location /interface/client/pids {
            proxy_pass http://10.103.13.103:3200/interface/client/pids;
        }
        ......
```bash

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

# 使用 nginx + thin 的配置启动 rails server. ( using nginx + thin to serve rails app)

2014-04-01 09:18
分类： 技术
1. nginx 中做如下配置：
```
     server {
         listen 80;
         charset utf-8;
         location / {
             proxy_pass          http://rails_servers;
             proxy_redirect      default;
             proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header    X-Real-IP $remote_addr;
             proxy_set_header    Host $http_host;
             proxy_next_upstream http_502 http_504 error timeout invalid_header;
        }
     }
     upstream rails_servers{
            server 127.0.0.1:6661;
            server 127.0.0.1:6662;
            server 127.0.0.1:6663;
            server 127.0.0.1:6664;
     }
```

重启 nginx:

```bash
  $ nginx -t  （测试一下配置文件）
  $ nginx -s reload
```

2. 使用 配置文件来启动 thin:

2.1. 建立 /config/thin.yml , 内容如下：

```yaml
chdir: '/opt/app/ruby/m-cms-for-tudou/current'  #  这里需要修改。
environment: production
address: 0.0.0.0
port: 6661
timeout: 30
log: log/thin.log
pid: tmp/pids/thin.pid
max_conns: 1024
max_persistent_conns: 512
require: []
wait: 30
servers: 4
daemonize: true
```

2.2. 启动thin: (记得Gemfile 中要有 gem 'thin' )

```bash
$ bundle exec thin start -C config/thin.yml
```

2.3. 记得在 `config/environments/production.rb`文件中： (以后可以使用nginx来 配置，处理静态文件。现在先这样弄着）

```ruby
Cms::Application.configure do
    config.serve_static_assets = true
end
```

就可以了。

# nginx rewrite/try_file tips(nginx rewrite/try_files 小提示）

refer to: http://stackoverflow.com/questions/22032751/how-to-process-dynamic-urls-as-static-pages-using-nginx

最近，一个项目的请求让我们的Rails不足以负担（300 req /seconds是我们的极限），所以我打算把它做成静态化的页面，这样在我的 2核CPU上都可以轻松跑到15000 req/s.  ( recently I am considering migrate our dynamic rails pages to static files served by Nginx which is much powerful than rails- 15k req/s v.s. 300 req/s. )

于是我们的nginx 配置文件是： ( so our nginx config file looks like: )

```
  server {
    listen       100;
    charset utf-8;
    root /workspace/test_static_files;
    index index.html index.htm;
    location /popup_pages {
      log_subrequest on;
      try_files /platform-$arg_platform-product-$arg_product.json /default.json;
#  DON'T use like below, nginx could not process underscore filenames mixing up with $arg_parameter_name:
#      try_files /platform_$arg_platform_product_$arg_product.json /default.json?q=$uri;
#      don't use rewrite ...  use try_files instead.
#      rewrite ^/popup_pages /platform-$arg_platform-product-$arg_product.json;
    }
  }
```

几个小tips: (some tips)

1.不要使用下划线。 因为nginx无法正确判断 $arg_param 中的变量。 要使用横线'-' (use '-' instead of underscore '_' in your static file names)

```
# DON'T use like below, nginx could not process underscore filenames mixing up with $arg_parameter_name:
#      try_files /platform_$arg_platform_product_$arg_product.json /default.json?q=$uri;
```

2.尽量不要使用rewrite ，因为if is Evil ( If is evil, see: http://wiki.nginx.org/IfIsEvil )

```
#      Don't use rewrite ...  use try_files instead.
#      rewrite ^/popup_pages /platform-$arg_platform-product-$arg_product.json;
```

3.调试时，可以先使用 rewrite 来调试，调试完毕后，把它改写成 try_files. 调试rewrite 时，时刻关注access.log/error.log

# nginx built-in variables (nginx 内置的变量)

Nginx 这些变量非常有用， refer to: http://wiki.nginx.org/HttpCoreModule#.24args

The core module supports built-in variables, whose names correspond with the names of variables in Apache.

First of all, there are variables which represent header lines in the client request, for example, $http_user_agent, $http_cookie, and so forth. Note that because these correspond to what the client actually sends, they are not guaranteed to exist and their meaning is defined elsewhere (e.g. in relevant standards).

Furthermore, there are other variables:

$arg_PARAMETER

This variable contains the value of the GET request variable PARAMETER if present in the query string

$args
This variable is the GET parameters in request line, e.g. foo=123&bar=blahblah; This variable could be changed.

$binary_remote_addr
The address of the client in binary form;

$body_bytes_sent
The amount of bytes sent as part of the body of the response. Is accurate even when connections are aborted or interrupted.

$content_length
This variable is equal to line Content-Length in the header of request;

$content_type
This variable is equal to line Content-Type in the header of request;

$cookie_COOKIE
The value of the cookie COOKIE;

$document_root
This variable is equal to the value of directive root for the current request;

$document_uri
The same as $uri.

$host
This variable is equal to line Host in the header of request or name of the server processing the request if the Host header is not available.

This variable may have a different value from $http_host in such cases: 1) when the Host input header is absent or has an empty value, $host equals to the value of server_name directive; 2)when the value of Host contains port number, $host doesn't include that port number. $host's value is always lowercase since 0.8.17.

$hostname
Set to the machine's hostname as returned by gethostname

$http_HEADER
The value of the HTTP request header HEADER when converted to lowercase and with 'dashes' converted to 'underscores', e.g. $http_user_agent, $http_referer...;

$is_args
Evaluates to "?" if $args is set, "" otherwise.

$limit_rate
This variable allows limiting the connection rate.

$nginx_version
The version of Nginx that the server is currently running;

$query_string
The same as $args except that this variable is readonly.

$remote_addr
The address of the client.

$remote_port
The port of the client;

$remote_user
This variable is equal to the name of user, authenticated by the Auth Basic Module;

$request_filename
This variable is equal to path to the file for the current request, formed from directives root or alias and URI request;

$request_body
This variable(0.7.58+) contains the body of the request. The significance of this variable appears in locations with directives proxy_pass or fastcgi_pass.

$request_body_file
Client request body temporary filename;

$request_completion
Set to "OK" if request was completed successfully. Empty if request was not completed or if request was not the last part of a series of range requests.

$request_method
This variable is equal to the method of request, usually GET or POST.

This variable always evaluates to the method name of the main request, not the current request, when the current request is a subrequest.

$request_uri
This variable is equal to the *original* request URI as received from the client including the args. It cannot be modified. Look at $uri for the post-rewrite/altered URI. Does not include host name. Example: "/foo/bar.php?arg=baz"

$scheme
The HTTP scheme (i.e. http, https). Evaluated only on demand, for example:

rewrite ^ $scheme://example.com$uri redirect;
$sent_http_HEADER
The value of the HTTP response header HEADER when converted to lowercase and with 'dashes' converted to 'underscores', e.g. $sent_http_cache_control, $sent_http_content_type... .

$server_addr
The server address. Generally nginx makes a system call to obtain this value. To improve efficiency and avoid this system call, specify an address with the listen directive and to use the bind parameter.

$server_name
The name of the server.

$server_port
This variable is equal to the port of the server, to which the request arrived;

$server_protocol
This variable is the protocol of the request. Common examples are: HTTP/1.0 or HTTP/1.1

$uri
This variable is the current request URI, without any arguments (see $args for those). This variable will reflect any modifications done so far by internal redirects or the index module. Note this may be different from $request_uri, as $request_uri is what was originally sent by the browser before any such modifications. Does not include the protocol or host name. Example: /foo/bar.html

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

# nginx sub uri 转发

nginx的转发非常重要，它可以让我们实现复杂的 集群。

下面是个例子：

```
server {
  listen 5000;
  location /client {
      proxy_pass http://10.103.13.103:5100/client;
  }
  location /interface/client {
      proxy_pass http://10.103.13.103:5100/interface/client;
  }
  location / {
      proxy_pass http://10.103.13.103:5118;
  }
}
```

5100, 5118分别是两个端口，跑着CLIENT 和主CMS。

（注：后来我们就把这两个app放在了不同的服务器上，大大降低了系统负载，提高了app的稳定性)

# 使用logrotate 为nginx日志 分卷 (rotate the nginx logs )

see:  http://serverfault.com/questions/284729/nginx-log-rotation

1.locate your Nginx pid file:   ( normally it's at:  /opt/nginx/logs/nginx.pid  in my Ubuntu)

2.you should know this key process:

```bash
# this just simply make nginx start a new log file, but NOT restart.  ( quite fast)
kill -USR1 `cat /opt/nginx/logs/nginx.pid`
```

3. create a new file (/etc/logrotate.d/nginx)  containing this :

```bash
# nginx SIGUSR1: Re-opens the log files.
"/opt/nginx/logs/access.log" "/opt/nginx/logs/error.log"{
  daily
  rotate 7
  dateext
  copytruncate
  missingok
  notifempty
  delaycompress
  sharedscripts
  postrotate
    test ! -f /opt/nginx/logs/nginx.pid || kill -USR1 `cat /opt/nginx/logs/nginx.pid`
  endscript
}
```

4. 记住： logrotate 运行时，是有一个自己的状态记录文件的。( /var/lib/logrotate.status ) 它会在这个文件中记录好每个文件的最早时间。然后每次运行时把目标文件跟自己的表做对比。如果发现目标文件的日期超过了这个表的日期，比如一个星期，那么 logrotate 才会对它分卷。  ( keep in mind that logrotate has its own status file which is used to remember when the target file is created and determine if the target file should be rotated.  )

5. 典型命令：

```bash
$logrotate -v /etc/logrotate.d/nginx   (如果你使用了 -vd 那么就仅仅是一个dry-run 预演，不会对 /var/lib/logrotate.status 生效 ) ( also remember that don't use the debug mode since it's just lead to a dry-run result and won't take effect on /var/lib/logrotate.status file )
```

6. 把logrotate增加到你的 crontab中去。  (make it affect via crontab -e)

例如：

```crontab
# in crontab editor:
0 0 * * * logrotate -v /etc/logrotate.conf
```

p.s. 所以，想要你的改动立即生效的话，

1.先运行logrotate :

```bash
$ logrotate -v /etc/logrotate.d/nginx
```

2.编辑 vim /var/lib/logrotate.status 把里面对应的内容，改成昨天。

例如：

```
"/usr/local/nginx/logs/access.log" 2013-12-22
```

要改成：

```
"/usr/local/nginx/logs/access.log" 2013-12-21
```

3. 再运行：

```
$ logrotate -v /etc/logrotate.d/nginx
```
