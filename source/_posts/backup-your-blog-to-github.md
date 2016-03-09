title: 备份 Hexo 源文件至 GitHub
date: 2016-03-09 18:34:08
tags:
- git
- hexo
- backup
- github
---

## 前言

这个博客一开始是部署到 GitHub 的，但是每次 Deploy 只是将生成的 html 文件部署进去。根目录文件还是在本机子，这样在换机子之后还得把整个 hexo 目录打包到新电脑，比较麻烦。我前面的操作方式是将真个根目录 push 到同仓库的一个 blog 分支下，当时没有记录操作方法。恰逢自己的生活博客也迁移到了 hexo 上，计划也将他备份到 GitHub，这里将过程在这里记录一下。

## 操作

### 前提

已创建有 GitHub 仓库，并且已使用 hexo-deployer-git 部署到 master 分支。
如果不满足请自行 google hexo 部署到 GitHub 的操作方法。

<!--more-->

### 过程

首先在官网新建一个仓库，比如我新建的仓库名为`ileyar.com`

在本地 hexo *根目录*下，初始化 git 仓库
```
git init
```
创建并切换到名为“hexo” 的分支
```
git checkout -b hexo
```
添加 README，并填写相关的说明，此步骤可略过
```
git add README.md
```
创建忽略文件 `.gitignore`
```
vi .gitignore
```
按需添加如下内容：
```
.DS_Store 
Thumbs.db
db.json  
*.log
.deploy*/
node_modules/
.npmignore
public/
```
上面最后一行忽略 public 目录，已经被 hexo 插件同步到 master 分支里，因此不需要再同步，deploy 是 hexo 的 git 配置存放目录，也不需要同步。其他内容可选择忽略也可以选择同步。

添加内容到仓库并提交到远程仓库
```
git add .   # git add -A
git commit -m "first commit"
git remote add origin git@github.com:iLeyar/ileyar.com.git	 # 后面仓库目录改成自己新建的。
git push -u origin hexo
```

## 写在后面
按照以上的步骤就进行了 hexo 源文件的初次备份。
以后每次修改了内容之后，都可通过一下几条命令实现同步。

```
git add .
git commit -m "..."	 # 双引号内填写更新内容
git push origin hexo	# 或者 git push
```

另外刚在 [stackoverflow](http://stackoverflow.com/questions/572549/difference-between-git-add-a-and-git-add) 上看到一个关于 `git add .` , `git add -u` 以及 `git add -A` 的区别。

> git add -A stages *ALL*
> git add .	stages new and modified, **without deleted**
> git add -u stages modified and deleted, **without new**

