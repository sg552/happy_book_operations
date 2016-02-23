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
