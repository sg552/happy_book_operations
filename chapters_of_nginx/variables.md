TODO:

介绍nginx 中脚本的情况

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

