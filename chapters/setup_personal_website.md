# 带完善 搭建个人博客的方法 (from foot to head setup your blog )

2014-12-19 13:53
分类： 技术
1. 需要买个域名。 例如： siwei.me  或者 shensiwei.com

2. 需要有个 独立的服务器 . 比如 aliyun, linode ,

3. 选取个人的博客框架， 是 ruby ( publify, refinery cms) ，还是 PHP ， 还是java .  下面以  ruby : publify 为准

4. 登陆到你的服务器（一般来说是个linux 系统， SSH登陆） 。

5. 安装 nginx, 作为

6. 安装 Rails, ruby, mysql 环境

7. 下载publify

8. 配置。 并且运行 blog.

9. 进入到 godaddy.com 或者 www.net.cn 或者你的域名供应商的后台， 把你域名的 www, @ 的A记录的地址 指向 你的服务器ip.

如果你用的是大陆之外的域名供应商， 则需要使用dnspod 这样的域名解析商（免费的不用担心），这样才能保证大陆用户可以百分百访问。

10.  配置nginx,  实现可以对传入的domain进行解析，然后把请求分发到你的blog上。
