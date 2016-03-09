title: hexo 部署到 github 出错
date: 2016-02-15 15:36:08
tags:
- hexo
- github
---

我在新的笔记本上安装了 Arch，并且将个人文件夹备份了过来，其中包含 ssh 的配置文件以及 hexo 的文件夹。

简易安装
----

经过安装 Git，通过`nvm` 安装 Node.js，然后安装 Hexo，并且配置 Git ，简略步骤如下：

```
$ sudo pacman -S git
$ git config --global user.name "leyar"
$ git config --global user.email "leyar.me@gmail.com"
$ git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout `git describe --abbrev=0 --tags`
. ~/.nvm/nvm.sh
```
将以下内容添加进`~/.zshrc`
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```
<!--more-->

(可选备份操作）
手动更新 `nvm`:
```
cd "$NVM_DIR" && git pull origin master && git checkout `git describe --abbrev=0 --tags`
. "$NVM_DIR/nvm.sh"
```
重启 Terminal, 之后可通过 nvm 来控制管理 nodejs 版本。

```
$ nvm install 4
$ nvm use 4
```
安装 hexo
```
npm install -g hexo-cli
cd hexo
npm install
```
经过如上操作，理论上是可以直接使用`hexo g -d` 命令来提交并部署更新了。

git 部署错误
----

然而，在 deploy 到 git 时，这里出现了错误，提示如下：

```
INFO  Files loaded in 898 ms
INFO  0 files generated in 957 ms
INFO  Deploying: git
INFO  Clearing .deploy folder...
INFO  Copying files from public folder...
On branch master
nothing to commit, working directory clean
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (/home/leyar/hexo/node_modules/hexo-deployer-git/node_modules/hexo-util/lib/spawn.js:42:17)
    at emitTwo (events.js:87:13)
    at ChildProcess.emit (events.js:172:7)
    at maybeClose (internal/child_process.js:821:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:211:5)
FATAL Error: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (/home/leyar/hexo/node_modules/hexo-deployer-git/node_modules/hexo-util/lib/spawn.js:42:17)
    at emitTwo (events.js:87:13)
    at ChildProcess.emit (events.js:172:7)
    at maybeClose (internal/child_process.js:821:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:211:5)

```

错误重点就在 `Permission denied (publickey)` 这里，git 本身我配置了非默认名的 key，即`github_rsa`和`github_rsa.pub`并且指定了验证文件。如下：

```
Host ileyar.github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_rsa
User git
```

解决
---

通过 google 搜索到官方的解决办法：[Error: Permission denied (publickey)](https://help.github.com/articles/error-permission-denied-publickey/)

在尝试了如下操作之后问题解决：
```
ssh-add ~/.ssh/github_rsa
```


