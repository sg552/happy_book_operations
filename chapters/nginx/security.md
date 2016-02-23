# 跟安全访问相关的几个模块

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
