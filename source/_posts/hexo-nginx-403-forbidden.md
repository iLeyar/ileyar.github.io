title: hexo 出现 nginx 403 错误
date: 2016-02-15 20:45:34
tags:
- hexo
- nginx
- linux
---

问题
---

2.14 号情人节那天正式将生活博客迁移到 Hexo 上。根据官方文档操作一路都很顺畅没有出现大的问题。现在就差将评论也迁移到 Disqus 上了。

将域名解析过来并配置好了 Ngix 之后，新博客也正常启动起来。但问题来了，这个博客却无法访问了。提示 403 错误。

分析
----

Google 了下出现这个错误的可能原因主要有两点：

+ 索引文件（例如 index.html）缺失
+ 权限配置问题
<!--more-->

解决
----

检查博客文件存放目录，索引文件并没有缺少。域名目录下 public_html 文件夹权限用户与组为 leyar。

检查配置文件`/etc/nginx/nginx.conf`, 发现第一行 user 为 nginx，尝试将其改为 leyar, 重启 nginx `sudo systemctl restart nginx`，依然 403。

尝试改为 root 并重启 nginx, 再次刷新网页，问题解决。
猜想原因应该是使用 sudo 安装的 nginx，故 user 为 root.
