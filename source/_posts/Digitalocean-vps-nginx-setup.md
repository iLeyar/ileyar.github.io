title: "通过 rsync 将 Hexo 部署到 Digitalocean vps"
date: 2015-05-01 21:20:53
tags:
- vps
- hexo
- digitalocean
- nginx
- rsync

---

前言
=================================

之前就一直说要将 Hexo 迁移到 vps 上，因没能找到更好的方法，这个计划就被搁置了。但这段时间 GitHub 抽风更严重，只有在通过某科学上网方式时才能恢复。这实在是太不方便，只得赶紧先把这个问题解决了。

> 2016-01-09 更新: 服务器上 rsync 的启动与修改
> 2016-01-09 更新: nginx 具体配置

方案
================================

1. VPS 上执行 `hexo server`，再配置 Nginx 反向代理，让域名指向 `http://localhost:4000`。
2. 本地生成静态文件，再部署到 VPS 上，用 Nginx 直接做 Web 服务器。

来源参考：[在 VPS 上部署 hexo](http://blog.berry10086.com/Tech/deploy-hexo-to-vps/)。 

毋庸置疑，为了安全起见并且在本地能同时 Deploy 到 VPS 和 Github （用作备份）上，选第二中方法肯定是比较好的。

<!--more-->

部署方式
================================

[官方文档](http://hexo.io/zh-cn/docs/deployment.html)支持多种部署方式（当然，你也可以每次通过手动上传静态文件进行更新）：

- git：这是 Google 到的大部分人会选择的方式
- rsync：这是我选择的方式

开始安装
================================

整理了大概方向与思路，那么现在就开始来实践操作吧，这次我是边实践边记录。

Nginx 安装与配置
--------------------------------

### 安装

我的 Vps 选择的是 Digitalocean 的 5 美元每月的方案，安装的是 CentOS 7 系统，安装方式与其他 Linux 方式略有不同，大家自行斟酌。

SSH 连接 VPS 后，添加 CenOS 7 的 EPEL 软件包：
```bash
sudo yum install epel-release
```
安装 Nginx
```bash
sudo yum install nginx
```
中间如果有出现提示，直接输入 y ，继续安装即可。

启动 Nginx
```bash
sudo systemctl start nginx.service
```
如果开启了防火墙，需要添加规则允许 HTTP 以及 HTTPS：
```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```
设置 Nginx 自动跟随系统启动
```bash
sudo systemctl enable nginx
```
现在可以在浏览器中输入 vps 的 ip 检查看 Nginx!是否启动了。
如果出现 "Welcome to Nginx.." 的字样，恭喜！代表你的 Nginx 成功安装并启动。

### 配置

创建配置文件存放目录
```
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```
编辑默认配置文件
```
sudo vi /etc/nginx/nginx.conf
```
将以下内容添加到 `http{}` 里,
```
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
```
保存退出.

为新的网站创建配置,以我的博客 leyar.me 为例:
```
sudo vi /etc/nginx/sites-available/leyar.me.conf     #  名字可自定义,以 .conf 结尾
```
添加一下配置内容,这里附上我的配置,请自行修改.
```
server {
        listen       80;
        server_name  leyar.me www.leyar.me;
        location / {
                root   /var/www/leyar.me/public_html;
                index  index.html index.htm;
                try_files $uri $uri/ =404;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
        root   html;
        }
}
```
创建软链接:
```
sudo ln -s /etc/nginx/sites-available/leyar.me.conf /etc/nginx/sites-enabled/leyar.me.conf
```
重启 Nginx:
```
sudo systemctl restart nginx          ## service nginx restart
```
参考链接:

+ [NGINX DOCUMENTATION](http://nginx.org/en/docs/)
+ [HOW TO INSTALL AND CONFIGURE NGINX ON CENTOS 7](https://garage.godaddy.com/tech/config/how-to-install-and-configure-nginx-on-centos-7/)
+ [Basic Nginx Configuration](https://www.linode.com/docs/websites/nginx/basic-nginx-configuration/)

rsync 部署
-----------------------------------

### 安装 rsync

远程 vps 与本地 vps 都需要安装 rsync 

vps （centOS 7）安装 rsync：

```bash
sudo yum -y install rsync
```

启动
```
sudo systemctl start rsyncd
sudo systemctl enable rsyncd
```

防火墙添加权限:
```
sudo firewall-cmd --permanent --add-port=873/tcp
sudo firewall-cmd --permanent --add-port=873/udp
```

本地安装 rsync (Ubuntu 系统为例)
```
sudo apt-get install rsync
```

hexo 目录下安装 rsync

```bash
$ npm install hexo-deployer-rsync --save
```

### 遇见问题

发现使用 rsync 的部署方式还是无法成功。每次发布总是出现如下错误，暂时不知道什么原因。

	```
INFO  Deploying: rsync
events.js:85
      throw er; // Unhandled 'error' event
                  ^
Error: spawn rsync ENOENT
    at exports._errnoException (util.js:746:11)
    at Process.ChildProcess._handle.onexit (child_process.js:1053:32)
    at child_process.js:1144:20
    at process._tickCallback (node.js:355:11)
	```

静待解决...

### 解决问题

Update:

感谢 [Willin Wang](http://willin.wang/) 的热心帮助，问题得以解决。

本地需要安装 rsync

```bash

sudo apt-get install rsync

```

目测问题就可能是出现在这...

-------------------------------

记录下我其中的操作步骤。（可忽略）

```bash

vim node_modules/hexo-deployer-rsync/lib/deployer.js

```

在 `return` 前面添加 `console.log(params.join(' '));`（！PS：括号引号内是一个空格） 然后 `:wq` 保存退出。

```bash
hexo -g d
```
然后出现一列参数如 `--delete -v -az public/ -e ssh -p 1688 root@107.170.216.51:/var/www/leyar.me`

执行这个命令，我因为前面添加时没有加上空格，在这个步骤上出现各种错误，正好到饭点，就暂时搁置了。

回来之后设置里添加上空格，再执行 `hexo -g d` 结果竟然 deploy 成功！

```bash
sent 353,570 bytes  received 2,012 bytes  30,920.17 bytes/sec
total size is 870,006  speedup is 2.45
INFO  Deploy done: rsync
```

纠结了我许久的问题总算解决。再次感谢帮助过我的 Willin Wang！

<br/>

总结
=====================================

这个问题困扰了我大概也好几周了，自己也是够倔强，此路不通本来可以换一条路，比如换 git 部署也不错，但心里作祟就是不愿意 vps 上安装太多东西，囧！不过有付出总是有收获的，至少以后不会让自己再遇到类似的问题了。


