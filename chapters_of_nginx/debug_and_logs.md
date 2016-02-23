# nginx的debug级的日志

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

