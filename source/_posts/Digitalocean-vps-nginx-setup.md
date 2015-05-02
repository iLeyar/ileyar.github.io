title: "将 Hexo 部署到 Digitalocean vps"
date: 2015-05-01 21:20:53
tags:
- vps
- hexo
- Digitalocean
- Nginx
categories: Hexo

---

## 前言

之前就一直说要将 Hexo 迁移到 vps 上，因没能找到更好的方法，这个计划就被搁置了。但这段时间 GitHub 抽风更严重，只有在通过某科学上网方式时才能恢复。这实在是太不方便，只得赶紧先把这个问题解决了。

------------------------------

## 基本思路

> 1. VPS 上执行 'hexo server'，再配置 Nginx 反向代理，让域名指向 'http://localhost:4000'。
> 2. 本地生成静态文件，再部署到 VPS 上，用 Nginx 直接做 Web 服务器。

毋庸置疑，为了安全起见并且在本地能同时 Deploy 到 VPS 和 Github （用作备份）上，选第二中方法肯定是比较好的。

------------------------------

## 部署方式

[官方文档](http://hexo.io/zh-cn/docs/deployment.html)支持多种部署方式（当然，你也可以每次通过手动上传静态文件进行更新）：

- git：这是 Google 到的大部分人会选择的方法，参考[这里](http://blog.berry10086.com/Tech/deploy-hexo-to-vps/)。
- rsync：这是我选择的方法。
------------------------------

## 开始安装

整理了大概方向与思路，那么现在就开始来实践操作吧，这次我是边实践边记录。

### Nginx 安装与配置

#### 安装

我的 Vps 选择的是 Digitalocean 的 5 美元每月的方案，安装的是 CentOS 7 系统，安装方式与其他 Linux 方式略有不同，大家自行斟酌。

1. SSH 连接 VPS 后，添加 CenOS 7 的 EPEL 软件包：

```bash
sudo yum install epel-release
```
2. 安装 Nginx

```bash
sudo yum install nginx
```
中间如果有出现提示，直接输入 y ，继续安装即可。

3. 启动 Nginx

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
sudo systemctl enable nginx.service
```

现在可以在浏览器中输入 vps 的 ip 检查看 Nginx!是否启动了。
如果出现 "Welcome to Nginx.." 的字样，恭喜！代表你的 Nginx 成功安装并启动。

#### 配置

一些基础的配置可以参考这里：[Basic Nginx Configuration](https://www.linode.com/docs/websites/nginx/basic-nginx-configuration/)

所有的 Nginx 配置文件都在 '/etc/nginx/' 目录下，其中基础配置文件为目录下的 nginx.conf.
关于一些优化配置后期再慢慢研究，现在主要修改一些网站的操作配置。


