---
title: 关于博客一些整改
date: 2016-03-19 06:08:05
tags:
- blog
- hexo
- theme
---

## 前言

其实一直想给博客换一个主题，顺便也将多说评论迁移到 Disqus，原因之一是为了与主博客统一。原因之二是多说的评论提醒功能形同虚设，经常性收不到邮件提示，再加上现在多说几乎没有人在维护了，以防万一，还是早点迁移出来。

> 计划整改内容：
> + ~~主题由 [Yilia](https://github.com/litten/hexo-theme-yilia) 更换为 [Next](https://github.com/iissnan/hexo-theme-next);~~
> + ~~评论从多说迁移至 Disqus;~~
> + 博客添加 SSL;
> + 邮箱服务搭建;

<!--more-->

## 内容

### 主题
看大家对 Next 的评价很高，就心痒痒给这个博客也换上了。不过发现似乎大部分功能自己也用不上，还有点多余。但整个页面看着也还可以，适合用来做生活博客，貌似不太适合用来存放技术笔记？等找到更好的主题，再换着玩。

给 blog 也重新换了一个 favicon，看着感觉还行。这样也好区分我的生活博跟笔记博了。毕竟两个都存了书签，经常性出误点的情况，囧...

### 评论
将多说评论转到 Disqus, Google 到的方法有两个，一个是通过 Python 2.x 来进行转换，一个是通过 php，因为没有装 Python 2，于是我选择了通过 php 转换。由于评论没几条，所以操作也很快。具体方法详见[多说评论迁移至Disqus](http://urouge.github.io/migrate-to-disqus/)

当然也因为用了 Disqus，这个主题的一些特色功能也用不上了。比如多说评论显示 UA，还有 jiathis 之类的。不过自己也不会用到，也就无所谓了。

### SSL && 邮箱
SSL 因为生活博客已经换上，这个博客要添加上也很快。自建邮箱功能还在研究中，预备使用 Postfix。打算给生活博客换域名，到时候会用到群发功能。

暂时，就先这样。
