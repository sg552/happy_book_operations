# 使用nginx 做代理

## 使用 无法备案的域名

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




## nginx sub uri 转发

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

