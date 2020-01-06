title: 在同一台电脑上管理多个 GitHub 帐号
date: 2018-09-28 14:28:13
tags:
- github
---

很多时候，会发现自己需要在同一台电脑上操作多个 GitHub 帐号，比如我会有有两个博客，均托管在 GitHub Pages 上，此时就需要再注册一个 GitHub 帐号了。

要在同一台电脑中管理多个 GitHub 帐号，就需要用到 ssh-key. 这里记录下我的操作。

## 生成不同的 SSH Keys

```bash
ssh-keygen -t rsa -C "xxx@mail.com" -f "github_rsa"
ssh-keygen -t rsa -C "yyy@mail.com" -f "github_2_rsa"
```
此时会生成两个不同的 keys：

```
~/.ssh/github_rsa
~/.ssh/github_2_rsa
```
同时还会生成同名的 `.pub` 后缀的公用密钥。 将此密钥分别复制到对应的帐号中。

<!--more-->

## 注册新生成的 SSH Keys

* 确保 ssh-agent 已启用`eval "$(ssh-agent -s)`
* 注册
```bash
ssh-add ~/.ssh/github_rsa
ssh-add ~/.ssh/github_2_rsa
```

## 编辑 SSH 配置文件

编辑 `~/.ssh/config` 文件，添加如下内容
```
# github user(xxx@mail.com)
Host xxx.github.com
HostName github.com
IdentityFile ~/.ssh/github_rsa
User git

# github user2(yyy@mail.com)
Host yyy.github.com
HostName github.com
IdentityFile ~/.ssh/github_2_rsa
User git
```

* \# 后面是注释内容，备注两个不同的 GitHub 帐号
* Host 用以区分两个不同的 GitHub 帐号，后面会用到。

## 克隆仓库，及提交本地仓库。

先检查是否使用了正确的用户名及邮箱。

```bash
git config --list
git config [--global] user.name "[name]"  # 配置用户名
git config [--global] user.email "[email address]"  # 配置邮箱
```

### 克隆远程仓库

克隆帐号1的远程仓库
```bash
git clone git@xxx.github.com:xxx/repo_name.git
```
克隆帐号2的远程仓库
```bash
git clone git@yyy.github.com:yyy/repo_name.git
```

* 注意 @ 与 : 之间的内容修改。

### 提交本地已有仓库

检查远程仓库配置
```bash
git remote -v
```

更新对应配置

```bash
git remote set-url origin git@xxx.github.com:xxx/repo_name.git
git remote set-url origin git@yyy.github.com:yyy/repo_name.git
```

### 本地创建新仓库

```bash
git init
git remote add origin git@xxx.github.com:xxx/repo_name.git
git remote add origin git@yyy.github.com:yyy/repo_name.git
```

提交本地仓库至远程仓库

```bash
git add .
git commit -m "new commit"
git push -u origin master
```

可以看到，其实就是将 @ 与: 之间的内容修改为 SSH keys 配置中对应的 HOST.

### Hexo 部署配置

* 安装 `hexo-deployer-git`
	```bash
	npm install hexo-deployer-git --save
	```
* 编辑`_config.yml`文件
	```
	deploy:
	- type: git
	  repo: git@xxx.github.com:xxx/name_repo.git
	  branch: master
	  message:
	```
注意填写上对应的 repo 链接即可。
