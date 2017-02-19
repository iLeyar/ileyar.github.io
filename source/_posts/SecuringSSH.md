---
title: "Vps 的一些安全设置"
date: 2015-04-20 11:22:38
tags:
- vps
- centos
- ssh
---

## 前言:

之前在 [Digitalocean](https://www.digitalocean.com/?refcode=85be3418a147)(PS：这是我的邀请链接) 上买了个 vps，然后架设了个 ss 服务端就一直放着了。昨天晚上登陆 vps 出现这么个提示：
```bash
ssh qstars
Last failed login: Sun Apr 19 09:13:30 EDT 2015 from 182.100.67.113 on ssh:notty
There were 185356 failed login attempts since the last successful login.
Last login: Sun Apr 12 16:20:37 2015 from 222.79.15.190
```
有人在试图暴力破解我的 vps 登陆帐号密码。
到 `/var/log/secure` 下面查看了下日志，各种尝试登陆失败的信息开始刷屏。这人是有多无聊~ 幸好开了 vps 之后我设置了通过 ssh key 来登陆。不过为了把安全性提高一点，还是先把端口改了。并且取消密码登陆。

下面记录下我的操作过程。

> vps 配置： Digitalocean  512MB Ram  20GB SSD Disk 
> 系统环境： CentOS 7 x64
> 本地系统环境： Ubuntu 14.04 LTS
<!--more-->

## 使用 SSH Keys

之前的文章《[关于在 Ubuntu 上部署 Hexo 到 GitHub](http://www.leyar.me/create-a-blog-with-hexo-in-ubuntu/)》的文章也有过配置使用 ssh key 登陆 github 的介绍。这里简单写下步骤。
具体配置方法可以参考官方教程：[点这里](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-digitalocean-droplets)

#### 第一步：本地创建 ssh key。
```bash
ssh-keygen -t rsa
```
#### 第二步：
```bash
Enter file in which to save the key(/leyar/.ssh/id_rsa):     # 这里输入需要修改的 sshkey 名称及路径。默认就直接 Enter

Enter passphrase(empty for no passphrase):     #输入密码，回车即留空。
Enter same passphrase again:     # 确认密码输入

```
接下来正常情况就是配置成功的提示：
```bash
Your identification has been saved in /leyar/.ssh/id_rsa.
Your public key has been saved in /leyar/.ssh/id_rsa.pub.
The key fingerprint is:
4a:dd:0a:c6:35:4e:3f:ed:27:38:8c:74:44:4d:93:67 demo@a
The key's randomart image is:
+--[ RSA 2048]----+
|          .oo.   |
|         .  o.E  |
|        + .  o   |
|     . = = .     |
|      = S = .    |
|     o + = +     |
|      . o + o .  |
|           . o   |
|                 |
+-----------------+

```
提示密钥已经保存在 `/leyar/.ssh` 目录下，公钥为 `id_rsa.pub`, 私钥为 `id_rsa`.

因为使用多个 SSH keys，还需要在 config 配置里面还需要添加一下。
```bash
touch ~/.ssh/config
vim ~/.ssh/config
```
这里贴下我的配置：
```conf
# digitalocean vps
Host qstars
HostName 107.170.110.110       # 服务器 ip
port 1688                      # ssh 登陆端口
IdentityFile ~/.ssh/id_rsa     # 验证密钥路径
User root                      # ssh 用户名
```
#### 第三步： vps 官网添加 SSH Keys

打开公钥文件
```bash
cat ~/.ssh/id_rsa.pub
```
复制里面的内容，粘帖到 [这个页面](https://cloud.digitalocean.com/settings/security) SSH Keys 栏目下。

#### 第四步： 将公钥上传到已创建的 vps 中。
```bash
cat ~/.ssh/id_rsa.pub | ssh root@[107.170.110.110] "cat >> ~/.ssh/authorized_keys"
## ip 为你服务器 ip，这里将本地的公钥传到服务器并重命名为 authorized_keys
```
如上都正常操作之后，即可通过 `ssh root@107.170.110.110`  或者是 `ssh qstars` （这里qstars 就是你在 config 里面填写的 host）直接免密码登陆。

## SSH 相关设置

登陆 vps，修改 `/etc/ssh/sshd_config` 文件
```bash
vi /etc/ssh/sshd_config
```
部分配置改成如下：

```conf
Port 1688       #修改 ssh 端口，取消 # 号，22 端口改成其他。
PermitRootLogin without-password      #禁止 root 密码登陆
AuthorizedKeysFile      .ssh/authorized_keys      #验证文件路径
RSAAuthentication yes          #RSA 认证 （可不改）
PubkeyAuthentication yes       #开启公钥认证 （可不改）
PermitEmptyPasswords no        #禁止空密码
PasswordAuthentication no      #禁止密码认证
UsePAM no                      #禁用PAM
```
修改了配置后，":wq" 保存返回，然后重启 ssh 服务
```bash
systemctl  restart sshd.service
```
## 更多相关安全设置

进行了这些操作，第二天登陆就没再出现前言里的提醒了，瞬间 vps 清爽了。
当然啦，如果你还觉得不够安全，
centOS 官方还给出了更详细的安全相关操作配置，具体可以参考：
- [Securing OpenSSH](http://wiki.centos.org/HowTos/Network/SecuringSSH)
- 对应的中文版：[保卫 OpenSSH](http://wiki.centos.org/zh/HowTos/Network/SecuringSSH)

你还可以给 vps 安装上这个：
- [DenyHosts](http://denyhosts.sourceforge.net/)

