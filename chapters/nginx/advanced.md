# nginx的限速相关

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
