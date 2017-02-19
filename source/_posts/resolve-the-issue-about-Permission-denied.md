---
title: CentOS 7 新建用户密钥验证问题
date: 2015-8-1 23:57
tags:
- CentOS
- SSH
- linux
---

> 之前一直使用的 root 帐号来登录管理我的其中一个 vps，虽然开启了仅密钥验证，但这也只是一道锁，不知道哪天密钥就被破了。遂决定把这个安全隐患先解决掉。
> VPS 系统：CentOS 7 64bit

新建用户并加进管理组
-------------------

```bash
adduser *username*
passwd *username*
gpasswd -a *username* wheel
```
<!--more-->

拷贝公钥到个人目录下
-------------------

```bash
cd /home/*username*
mkdir .ssh
cd .ssh
cp /root/.ssh/authorized_keys .
```

尝试登录
---------

保留当前终端，在新终端登录：

```bash
ssh *username*@*server-ip* -p *port*
```

出现登录失败提示：

```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

尝试修改`.ssh`文件夹的用户及用户组

```bash
cd /home/*username*/
chown *username* .ssh
chgrp *username* .ssh
```

再次尝试登录，登录成功。

禁止 root 登录
------------------

此时可以取消 root 登录了。

```bash
sudo vi /etc/ssh/sshd_config
```
查找并修改为如下参数：
```
PermitRootLogin no
```

之前已经编辑过一些内容，具体查看[Vps的一些安全设置](http://www.leyar.me/SecuringSSH/)


