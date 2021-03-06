# 域名

## 基本概念

网络上，不同的服务器， IP地址是不同的。
114.114.114.114  (真实的IP， 专门查询域名的IP）

但是大家记不住 IP地址。
所以，域名就出现了。

例如：  百度域名： 220.181.57.217
谁也记不住。 baidu.com 大家就都能记得住。

p.s. 我们很多时候，发现家里上不了网，但是可以用QQ。

实际上不是QQ牛，而是 QQ 在客户端访问服务器端的时候， 使用了IP地址
而不是  api.touring.com.cn

这就在： 家里上不了网（往往是DNS（域名解析服务器） 设置的不对造成的），QQ就可以上。
因为QQ 不需要域名解析。

## 一级域名，二级域名，三级域名。。。

baidu.com  一级域名
pan.baidu.com  , tieba.baidu.com :   pan 和 tieba 就是 二级域名
mp.weixin.qq.com 三级域名。（用的不多）不是特别规范的域名。

域名供应商，实际支持的域名，只到二级。

特殊的域名：

@: 代表： 一级域名。 例如： baidu.com
www : 是个二级域名，  往往跟 @ 一个意思。
www.baidu.com  与 baidu.com 是一个IP地址。

*: 是个通配符，表示所有的二级域名。(只在域名配置后台中使用）
例如  *.baidu.com  会表示， 你所有的  abc.baidu.com    bbc.baidu.com 都由  *.baidu.com 来
表示。

## 国际上的域名分级：

.com 最流行，最初的，使用最广的
.net : 同.com ，但是不如.com流行
.org: 组织使用
.gov: 政府机关
.edu: 教育机构。
.cn/.us/.jp/.uk   不同国家的。
.中文  .公司    ： 中文域名, 华而不实。 不要用。(本质上，是把 .中文 转换成了 a1b2c3d4...z100 这样的长域名 ）
.com.cn, .com.jp, .com.hk ： 带有区域的二级后缀的域名。
.cc/.video/.xxx/.tech/.shop: 好多好多好多。都是新型域名。

## 在国内，用域名， 要备案。

所有的域名都要备案。
所有的服务器，都要备案。

所以，要备案两次。

早期的备案： 1. 你要有本人照片。  2. 到机房，拍照。
现在： 阿里云来推动：  没那么麻烦了。

但是，备案还是比较麻烦。所以催生出了一条灰色产业： 备案。
大家可以到淘宝上，百度上，搜索备案。 2，300块就可以快速备案。（一天，3天搞定）
但是缺点： 在你不知情的情况下，你的备案就被人撤销了。

所以，要备案，还是要自己拿出时间， 到阿里云上备案。 到阿里云买主机。
原因之一：  方便备案。

国外主机不用备案。 所以，如果你不希望备案的话，选择：

1. 在 国外域名供应商，购买域名（  godaddy )
2. 在 国外购买云主机（aws, linode 。。。）

缺点：
1. 国外主机，对于国内用户，很慢。
2. 国外的域名，被国内屏蔽的几率很大（ 50% + )

所以，早期，我很多湖南， 广东的朋友跟我反应：  思维你的网站我上不去。



大部分的传统域名，都可以备案。 ( .com/cn/org/cc/co/info ... )
新域名都不能备案：  .tech/.house/.cake/.software/.space


## 对于域名的设置。

购买域名很简单。 如何设置呢？

在设置的时候，注意两个特殊的符号：

@  表示一级域名    (就是  siwei.tech)
*   域名的通配符。

## 1. A记录。
最最常用

A地址，规定了， 你的域名，直接指向 哪个IP地址。

siwei.tech   A 记录： 112.126.91.145

## 2. C记录

也叫 别名地址。

qqmail31017ca8   的C地址，要指向的是一个域名。 往往用在： 企业邮箱的配置上。

## NS 记录

域名解析服务器。  一般来说不用动。
我在阿里云购买的域名， NS地址不用动。
我在godaddy购买的域名（国外服务器，容易被屏蔽）， 那么我就用国内的 DNS供应商。
这个时候，我就可以在godaddy 后台，把  siwei.tech 的NS 地址，改成 国内的dns供应商要求的地址，
然后就可以了。

域名所在地   域名解析服务器   国内的各级dns服务器   明创软件办公室
godaddy       dnspod.cn       a.bj.china.com        动态IP

## MX 记录

用来设置邮箱。 例如： qq邮箱： 就要你的@的MX记录，指向mxdomain.qq.com.

## TXT 记录  （很罕见）

在国内，只见到 163企业邮箱，会要求设置这个。 按照163的说明，照葫芦画瓢就可以了。
