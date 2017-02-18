---
layout: post
title: 关于在 Ubuntu 上部署 Hexo 到 Github
date: 2015-04-13 14:57:30
updated: 2017-2-19 3:43:00
tags:
- hexo
- github
- ubuntu
---

## 前言：
之前在 Mac OX S 系统上有部署过一次 Hexo，不过都是新手上路处于懵懂茫然的状态，最后怎么搭建成功了也不甚清楚。这一次电脑系统从 Windows8.1 换到 Ubuntu14.04，打算完全参照官方文档，自己从头到尾操作一次，也顺便熟悉一下 Ubuntu 系统，因为是完全自己摸索，其中参照了大量的文档，数次Google，其中的艰辛自不必说，满满都是泪。（我是小白T^T）

> - 系统环境：Ubuntu 14.04 LTS （如果是其他系统环境请自行 Google，这里围绕 Ubuntu 系统操作。）
> - 涉及内容：Git、GitHub SSH KEYS、node.js、npm等
> - 此教程亦适用于其他 Linux 发行版
<!--more-->

## Git And SSH-Key

### 安装 Git 
Ctrl + Alt + T 打开 终端，输入如下命令：
``` bash
sudo apt-get install git
```
### 生成 SSH keys
到这一步泪啊，我本身已经有一个 VPS 并且已经配置了 SSH keys，如果按照 [官方文档](https://help.github.com/categories/ssh/) 的教程操作，就会把我之前生成的与 VPS 之间的 SSH Keys 覆盖掉，于是经过多次 Google，找到了解决方法，此方法稍作修改也适用于解决多个 Github 账号的 SSH keys 配对问题。具体可以参考这里第二条：
 - [同个客户端使用多个密钥的解决办法](http://stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client)
 - [多个 GitHub 账号 & SSH 配置](http://stackoverflow.com/questions/3225862/multiple-github-accounts-ssh-config?rq=1)

下面列出步骤：
先查看下我已经存在的 SSH keys：
```bash
$ ls -al ~/.ssh 
    # Lists the files in your .ssh directory, if they exist
```

这是我这边的显示结果：
```text
id_rsa				# 这是私钥
id_rsa.pub			# 这是公钥
github_rsa			# 这是给 github 生成的私钥，下面会讲到创建方法
github_rsa.pub 
known_hosts
config				# 这个文件就是重点，下面会讲到。
```
下面我们来开始给创建 SSH key 吧。
```bash
ssh-keygen -t rsa -C ' "your_email@example.com" ' 
# 这里引号内改成你在 Github 上设置的邮箱，比如我的是 leyar.me@gmail.com
```
回车后会提示：
```bash
Generating public/private rsa key pair.
# 提示生成密钥配对
```
继续回车，提示：
```bash
Enter file in which to save the key (/home/leyar/.ssh/id_rsa): /home/leyar/.ssh/github_rsa 
#注意！冒号后面这里官方帮助文档里是建议你直接回车，如果涉及多个 SSH keys，这个方法是不可行的。因此后面输入你想要创建的文件及其路径再回车。比如上面我输入的。 github_rsa 这个文件名你可以换别的。
```
然后出现提示：
```bash
Enter passphrase (empty 'for' no passphrase):		# 这里输入你想设置的密码，也可以直接回车
Enter same passphrase again:			# 再次输入密码，或者直接回车
```
官方文档强烈推荐设置密码,具体原因参考：[Working with SSH key passphrases](https://help.github.com/articles/working-with-ssh-key-passphrases/)。

界面会提示成功：
```bash
Your identification has been saved in /home/leyar/.ssh/github_rsa.
Your public key has been saved in /home/leyar/.ssh/github_rsa.pub.
The key fingerprint is:
01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db leyar.me@gmail.com
```
接下来也是至关重要的一步，它保证了 GitHub 能否正确读取到你新建的密钥。
编辑 ssh 配置文件：
```bash
vi ~/.ssh/config
```
在文件里面输入如下配置信息：
```
    # GitHub user(leyar.me@gmail.com)
    Host ileyar.github.com				#这里修改成你自己的 host
    HostName github.com
    PreferredAuthentications publickey	# 这个貌似可以不需要的，暂时还没弄清楚。
    IdentityFile ~/.ssh/github_rsa		# 这里填入你前面新建的密钥的路径
    User git 
```
输入完成后，保存，关闭。回到终端。

接下来看看你添加的 SSH key 是否在运行了。
```bash
ssh-add -l
2048 d9:1b:af:b4:8e:8d:28:bb:2e:b3:7c:ce:eb:83:c6:52 leyar@iLeyar (RSA)
2048 3e:22:25:65:0b:24:f5:51:43:27:8b:fd:91:3f:7f:85 leyar.me@gmail.com (RSA)
```
可以看到第二个就是我新建的，已经在运行了。如果没有出现，可以通过如下操作：
```bash
ssh-add ~/.ssh/github_rsa
```
接着就是把本地生成的 SSH key 拷贝到 [GitHub](https://github.com/settings/ssh) 网页里面了。
打开 `~/.ssh/github_rsa.pub` 文件，把里面的内容（公钥）复制出来。
```bash
vi ~/.ssh/github_rsa.pub
```
登陆 [Github](https://github.com/settings/ssh) ，点击 Add SSH key，”Title“ 随便填写，然后把你复制的内容，粘贴到 ”Key“ 里面。点击 Add key，OK！
这里有官方的图文教程：[点我](https://help.github.com/articles/generating-ssh-keys/#step-4-add-your-ssh-key-to-your-account)

检验结果的时刻到来：
```bash
ssh -T git@github.com
Hi iLeyar! You’ve successfully authenticated, but GitHub does not provide shell access.
```
这里代表已经配对成功了～ 

 - 更多 SSH 相关的帮助信息可以查阅 [GitHub Help](https://github.com/settings/ssh) .

---------------------------------------------------------------

### 安装Node.js
三种安装方法，前两种是我安装过的，后一种是 Google 到的。

#### 从软件仓库安装（不推荐）
在 终端 输入 `nodejs` 或者 `npm` ，如果没有安装会提示你进行安装，命令如下：
```bash
sudo apt-get update
sudo apt-get nodejs
sudo apt-get install npm
```
直接安装完会遇到一个问题，即在后续安装完 hexo 使用 npm install 命令安装相关依赖时会出现这个错误提示：
```bash
/usr/bin/env: node: 不是目录
```
原因是用包管理器安装的话，二进制表文件被叫做 nodejs，但 hexo 用的是 node，解决办法就是通过软链接的形式将 nodejs 链接到 node：
```bash
ln -s /usr/bin/nodejs /usr/bin/node
```

 - 参考链接： [点这里](http://chyoo.me/2014/10/26/create-a-blog-with-hexo/)
这个安装方法我只进行了一半，出错了就把 nodejs 卸载了然后使用 Hexo 官方推荐的方法安装（即接下来的方法二）。解决办法是后面偶然 Google 到的。

#### 通过 nvm 安装（推荐）
运行如下命令：
```bash
export NVM_DIR="$HOME/.nvm" && (
  git clone https://github.com/creationix/nvm.git "$NVM_DIR"
  cd "$NVM_DIR"
  git checkout `git describe --abbrev=0 --tags --match "v[0-9]*" origin`
) && . "$NVM_DIR/nvm.sh"
```
等待完成，
为了使 nvm 能开机自动启动，需要添加如下内容到你的`~/.bashrc`,`~/.profile`,或者`~/.zshrc` 文件中：
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" 
```
##### nvm 的部分使用说明
列出可安装包信息
```bash
nvm ls-remote
```
安装最新稳定版：
```bash
nvm install stable
```
启用安装好的版本
```bash
nvm use node
```
你也可以这样启用它
```bash
nvm run node --version
```
设置默认版本：
```bash
nvm alias default node
```
安装完成～
##### 手动更新 nvm
```bash
cd "$NVM_DIR"
  git fetch origin
  git checkout `git describe --abbrev=0 --tags --match "v[0-9]*" origin`
) && . "$NVM_DIR/nvm.sh"
```

 参考链接：
 - [Hexo 官方教程](http://hexo.io/docs/#Install_Node-js)
 - [nvm](https://github.com/creationix/nvm)

-----------------------------------------------------------------------

### 安装 Hexo
如果前面的步骤都正确没问题，那么现在可以通过 npm 安装 hexo 了：
```bash
npm install -g hexo-cli
```
安装完成后，进行初始化操作：
```bash
mkdir blog
hexo init blog      #初始化，创建一个你专门存放博客文件的文件夹，我这里把文件夹命名为 blog，你可以改成你想要的名字
cd blog                #进入 blog 目录
npm install          #安装相关依赖
```
#### 一些使用说明
hexo 更新
```bash
npm update -g
```
安装插件
```bash
npm install <plugin-name> --save
```
卸载插件
```bash
npm uninstall <plugin-name>
```
hexo 简单使用命令参考 [Hello World!](http://www.leyar.me/hello-world/)

Markdown 代码语法高亮支持语言
+ [Posting Source Code](https://en.support.wordpress.com/code/posting-source-code/)

## 写在后面
至此，本地安装 Hexo 完成。其他的配置，命令，发布等可以查阅官方文档：
- 英文版：http://hexo.io/docs/index.html
- 中文版：http://hexo.io/zh-cn/docs/index.html

相关资源：
+ [Hexo 之后续篇](http://www.leyar.me/After-installing-Hexo/)
+ [通过 rsync 将 Hexo 部署到 Digitalocean vps](http://www.leyar.me/Digitalocean-vps-nginx-setup/)
+ [Hexo 部署到 GitHub 出错](http://www.leyar.me/hexo-deploy-to-git-error/)
+ [Hexo 出现 Nginx 403 错误](http://www.leyar.me/hexo-nginx-403-forbidden/)
+ [备份 Hexo 源文件至 GitHub](http://www.leyar.me/backup-your-blog-to-github/)
+ [为 Hexo 博客添加 404 页面](http://www.leyar.me/create-404-page/)

*本文最后更新：2017-2-19*
