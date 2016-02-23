# 对文件(动态页面) 进行缓存

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
