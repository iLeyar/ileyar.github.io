
---
layout: post
title: "Hexo 之后续篇"
date: 2015-04-17 02:58:29
tags: 
- hexo
- dns
---

## 前言：
上一篇文记录了我部署 Hexo 的过程，然后某人跟我说还可以把 Hexo 部署到 gitcafe 或者自己的 vps 上，这几天也发现在 github 打开页面实在是太慢（穿墙会快一些），看来得找个时间把他迁出去了（真折腾...）。



现在记录下继上篇文之后未完的操作,大概就是这几件事：

> * Hexo 常用操作及配置。
> * 解析域名。
> * 某些问题及解决办法。

<!--more-->
## Hexo 
PS：前面的操作完成之后，需要安装一个插件（hexo-deployer-git）才能正常发布文章。
```
$ npm install hexo-deployer-git --save
```

    
### 配置
大部分的配置都在根目录的 _config.yml 文件中修改。
配置文件里面的相关参数注解可以查看 [**官方手册**](http://hexo.io/zh-cn/docs/configuration.html)。
官方手册写得足够详细，全面，而且还支持中文简体、繁体以及英文版。

这里就不做介绍，列几个需要注意的地方：

```
# URL
## 如果存放在子目录中，例如 http://yoursite.com/blog , 则需要将下面的 url 设置为 http://yoursite.com/blog 并把 root 设为 /blog .
## 因为我的放置在根目录里，因此 url 以及 root 直接如下设置，permalink 参数是设置文章的固定链接，我的设置是直接用标题。

url: www.leyar.me
root: /
permalink: :title/  
permalink_defaults:

```
关于固定链接这块可以根据自己的需要设置其他的变量。具体请参考这里：[永久链接](http://hexo.io/zh-cn/docs/permalinks.html)

```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
## repo 地址可以在 github 进入你的库里面右下角的 SSH clone URL 复制过来。

deploy:
  type: git
  repo: git@github.com:iLeyar/ileyar.github.io.git
  branch: master
  message:

```
Hexo 允许同时设置多个 deployer，要设置其他的 deployer ,参照这里：[部署](http://hexo.io/zh-cn/docs/deployment.html) （PS：等迁移的时候我也该用到了。）

<hr>

### 命令

列几个常用到的，更多的命令参考这里[指令](http://hexo.io/zh-cn/docs/commands.html)

```
$ hexo new <title>               # 新建文章，文章标题如果有空格，需要引号括起来。
$ hexo g                         # 生成静态文件 "g" 即 "generate"
$ hexo d                         # 发布文件到网站，需先生成文件
$ hexo g -d                      # 生成文件并发布到网站。 "d" 即 "deploy"
$ hexo g -w                      # 生成文件并查看文件变动 "w" 即 "watch"
$ hexo clean                     # 清除缓存文件（db.json）和已生成的静态文件（public）

```
### 主题

Hexo 的主题都比较简洁，官方有给出几个[主题](http://hexo.io/themes/)，根据需要自行提取。
我这里使用的是 [Yilia](https://github.com/litten/hexo-theme-yilia) 主题，有些地方还需要改进，有空我慢慢给它改改，喜欢这个主题你们可以点链接去作者的 github 页面查看详细的介绍及安装配置方法。

小节：关于 Hexo 的就先记录这么多，其他官方文档里面介绍的更多功能还有待慢慢去操作了。

<hr>

## DNS

要将自己域名解析到 github，可以很简单也可以很复杂。

- 关于给 GitHub Pages 自定义域名的介绍 --- [About custom domains for GitHub Pages sites](https://help.github.com/articles/about-custom-domains-for-github-pages-sites/)
- 为 GitHub Pages 设置自定义域操作方法 --- [setting up a custom domain with Github Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/)
- 关于 A，CNAME，ALIAS 和 URL转发之间的区别 --- [Differents between the A, CNAME, ALIAS and URL records](http://support.dnsimple.com/articles/differences-between-a-cname-alias-url/)

我暂时为了方便，外加这个域名还没想到其他的用处，就只做了个 CNAME 解析。

- 操作方法如下：
1. 添加一个 CNAME 记录指向 ileyar.github.io 
2. 然后终端里操作：
  ```
$ touch ~/hexo/source/CNAME   ## 在 source 目录下创建一个命名为 CNAME 的文件
$ vim ~/hexo/source/CNAME     ## 编辑 CNAME 文件
  ```
3. 将自己的域名填写上，比如我的 www.leyar.me，然后 :wq 保存退出。
  ```
$ cd ~/hexo
& hexo g -d              ## 生成并发布文件
  ```

4. 接着就坐等解析生效了，查询配置是否生效，可以使用 dig 命令:
  ```
$ dig www.leyar.me +nostats +nocomments +nocmd  ## www.leyar.me 改成你设置的域名
  ```

小节：DNS 暂时就到这，到时候迁移了位置，再回头来研究下设置二级域名~

<hr>

## 其他问题及解决办法：

遇到个比较奇怪的问题，重启开机之后，有时候 hexo 命令提交会出现如下错误：
```
leyar@iLeyar:~/hexo$ hexo vision
/usr/bin/env: node: 没有那个文件或目录
```
一开始我的操作就是，重装 nvm，在重装 nodejs，然后重装 hexo 来解决。导致后来 hexo 的文件都给我弄得乱七八糟。

郁闷的这并没有彻底解决这个问题。当它第Ｎ次再出现的时候，不得已只能先好好检查下具体哪里出问题了。
先试试 nvm 命令：
```
leyar@iLeyar:~/hexo$ nvm

Node Version Manager

Usage:
  nvm help                              Show this message
  nvm --version                         Print out the latest released version of nvm
  nvm install [-s] <version>            Download and install a <version>, [-s] from source. Uses .nvmrc if available
  nvm uninstall <version>               Uninstall a version
  nvm use <version>                     Modify PATH to use <version>. Uses .nvmrc if available
  nvm run <version> [<args>]            Run <version> with <args> as arguments. Uses .nvmrc if available for <version>
  nvm current                           Display currently activated version
  nvm ls                                List installed versions
  nvm ls <version>                      List versions matching a given description
  nvm ls-remote                         List remote versions available for install
  nvm deactivate                        Undo effects of `nvm` on current shell
  nvm alias [<pattern>]                 Show all aliases beginning with <pattern>
  nvm alias <name> <version>            Set an alias named <name> pointing to <version>
  nvm unalias <name>                    Deletes the alias named <name>
  nvm reinstall-packages <version>      Reinstall global `npm` packages contained in <version> to current version
  nvm unload                            Unload `nvm` from shell
  nvm which [<version>]                 Display path to installed node version. Uses .nvmrc if available

Example:
  nvm install v0.10.32                  Install a specific version number
  nvm use 0.10                          Use the latest available 0.10.x release
  nvm run 0.10.32 app.js                Run app.js using node v0.10.32
  nvm exec 0.10.32 node app.js          Run `node app.js` with the PATH pointing to node v0.10.32
  nvm alias default 0.10.32             Set default node version on a shell

Note:
  to remove, delete, or uninstall nvm - just remove ~/.nvm, ~/.npm, and ~/.bower folders

```
nvm 命令可用的，那再看看 nodejs 是否还存在：
```
leyar@iLeyar:~/hexo$ nvm ls
        v0.12.2
node -> stable (-> v0.12.2) (default)
stable -> 0.12 (-> v0.12.2) (default)
iojs -> iojs- (-> N/A) (default)
```
问题在这，没有指定 nodejs 版本
```
leyar@iLeyar:~/hexo$ nvm use 0.12
Now using node v0.12.2
```
再查看下，
```
leyar@iLeyar:~/hexo$ nvm ls
->      v0.12.2
node -> stable (-> v0.12.2) (default)
stable -> 0.12 (-> v0.12.2) (default)
iojs -> iojs- (-> N/A) (default)

```
接下来，hexo 命令应该可以用了。
```
leyar@iLeyar:~/hexo$ hexo version
hexo: 3.0.1
os: Linux 3.13.0-49-generic linux x64
http_parser: 2.3
node: 0.12.2
v8: 3.28.73
uv: 1.4.2-node1
zlib: 1.2.8
modules: 14
openssl: 1.0.1m

```
问题解决~
这个问题不知道还会不会出现，先酱紫 。

另外，在通过 Ｇoogle 中，还顺带了解了某些命令，这里作下记录：
```
ls -a | grep .nvm      # 这个命令是查找是否有 .nvm 
```

其他的相关问题，以后可能派上用场：

- 为多用户安装 nvm & nodejs 的方法推荐：[参考链接](http://stackoverflow.com/questions/11542846/nvm-node-js-recommended-install-for-all-users)
- 卸载 nvm 方法：[参考链接](https://github.com/creationix/nvm/issues/298)
  移除添加到 ~/.bashrc 里面的命令: source ~/.nvm/nvm.sh
  接着移除文件：

  ```
  rm -rf $NVM_DIR ~/.npm ~/.bower
  ```
  
