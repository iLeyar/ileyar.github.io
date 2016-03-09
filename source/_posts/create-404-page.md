title: 为 Hexo 博客添加 404 页面
tags:
- 404
- hexo
date: 2016-3-10 00:14

---

## 前言

一直想为自己的 hexo 博客添加 404 页面来着。 Google 搜到的很多方法，貌似都不是特别完美。这里记录下自己的创建方法。

> 大致思路如下
> + 创建`404.md`页面
> + 通过编辑`Front-matter`来定义页面的布局，固定链接； 
> + 编辑 nginx 配置文件
<!--more-->

## 操作

### 创建页面

首先，创建一个`404.md` 文件放到 `source` 目录下

```
touch ~/hexo/source/404.md
```
接下来，编辑 404 页面的内容，你可以自己修改成自己喜欢的样式嵌套在主题下面，也可以使用公益 404 页面，或者 google 的 404 小游戏，也不错。

我这里选择了 404 公益。有两个选择：

+ [腾讯404](http://www.qq.com/404/)
+ [益播](http://yibo.iyiyun.com/Home/Index/web404)

我用的是腾讯 404。 编辑 `404.md` 文件

```
vi ~/hexo/source/404.md
```
编辑 Front-matter ，并添加腾讯 404 页面代码。

```
layout: false 
comments: false
permalink: /404.html

---

<html>
<head>
</head>
<body>
<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="http://leyar.me" homePageName="返回主页"></script>
</body>
</html>
```
注意：

+ 将`homePageUrl=""` 引号内的内容改成你的网址；
+ 如果你是自定义 404，并且搭配主题，则需要去掉 `layout: false` 这行。

### 编辑 NGINX 配置文件

在 server 区块内添加

``` 
error_page   404   /404.html;
```

现在 deploy 再刷新网页随便打开一个不存在的页面看看你的 404 页面是不是添加成功啦？

你可以试试我的404：http://www.leyar.me/none


