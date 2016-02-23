# Nginx

nginx (读作 [engine x]是 HTTP 服务器, 也是多种代理(reverse proxy, mail, TCP等) 服务器。)
作者是 Igor Sysoev, 一位俄罗斯人。最初 nginx 被广泛用于俄罗斯的各种大型网站。截止
到2016年1月，16%的网站使用nginx. （另外几个大户分别是 Apache: 34%, Microsoft IIS: 29%,
nginx: 16%, 其他: 20%)

在国内，几乎所有的公司都在用nginx. 百度阿里腾讯优酷都是这样。

官方网站：http://nginx.org

在本章， "HTTP服务器"也被简称为“服务器”或者“server“, 截止2016年2月，最新版本是1.9.11.

## 与其他HTTP 服务器的对比

IIS 是C#系列的服务器。不能运行在Linux上，所以我们不考虑它，只考虑Apache.

Apache: Linux上最古老的HTTP 服务器，诞生于1995年。目前也是Linux上用的最广泛的服务器。
它是开源的，Apache module 也是开源的。例如 mod_php(php的module) , mod_jk(java +
tomcat的module)

但是Apache的缺点：大并发访问下，性能会下降的厉害。

- Apache中，每个http request, 都会消耗apache的进程。并发数越大，apache的负担就越重，
最终apache是有一个处理并发请求的上限的。
- nginx则使用了固定的内存数，事件驱动。一般情况下，不需要多少配置就能很好的工作。

所以，我们用nginx的原因是：

- 速度快, 处理静态内容的速度是apache的4倍。
- 消耗内存,CPU少。
- 还有第三点：配置简单。

引用[nginx官方网站的文章](https://www.nginx.com/resources/wiki/community/why_use_it/)：
:

 | nginx | Apache
:---:|:---:|:---:
消耗的CPU | 15% | 30%
消耗的内存 | 1MB | 17MB
每秒并发| 11,500 | 6,500

nginx的配置比apache的容易很多。 apache的代码风格明显是对机器友好。 而nginx则对人友好，
下面是apache 配置一个server的例子, 把 /my/html_files 作为根目录，本地运行个server,
跑在80端口：

Ubuntu下,Apache的配置步骤：

- 新建站点的配置文件： /etc/apache2/sites-enabled/my_site.conf
```xml
<VirtualHost *:80>
  ServerName localhost
  DocumentRoot "/my/html_files"
</VirtualHost>
```
- 新建对端口的监听文件： /etc/apache2/ports.conf
```xml
NameVirtualHost *:80
Listen 80
```

在Ubuntu下，nginx的配置步骤：
- 新建站点的配置文件：  /etc/nginx/sites-enabled/my_site.conf
```nginx
server {
  listen 80;
  server_name localhost;
  root "/my/html_files";
}
```

可以看出， apache中对80端口的监听文件，完全就没有必要存在。
这种叫做“废代码(boilerplate code)”, 在apache中，同样的“废代码”随处可见。

nginx的废代码更少，表达更加直观，这也是我们使用nginx的原因之一。
