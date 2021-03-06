---
title: 使用密钥登陆 OpenWrt
date: 2016-02-19 17:05:43
tags:
- openwrt
- ssh
- key
- dropbear
---

## 前言

最近 ss 线路总是不稳定，时不时需要 ssh 到路由器检查系统日志，这就增加了密码输入的次数，这对于享受惯了 ssh-key 便利性的我来说简直不要太麻烦。遂决定路由器上也使用 ssh-key。

> 使用 key 验证登陆的优点：（这里引用 [OpenWrt官方](https://wiki.openwrt.org/oldwiki/dropbearpublickeyauthenticationhowto)的描述）
> + you no longer have to type the password,
> + less effort to log in,
> + less times for it to be seen on your fingers by others,
> + easier to automate things like SCP or remote commands,
> + the password is no longer sent encrypted to OpenWrt,
> + less likely for an eavesdropper to capture it,
> + allows you to turn off password authentication,
> + impossible for an attacker to guess your password on OpenWrt.

优点这么多，赶紧来试试吧～

<!--more-->

## 操作

首先需要注意的是，不用于其他版本的 linux,OpenWrt 所使用的 SSH server 是 Dropbear，它可以支持 RSA Keys，但是验证路径位于`/etc/dropbear/authorized_keys` 操作时只需要注意公钥的放置路径即可。

这里记录下我的操作步骤：

在本机 linux 下：

生成密钥：
```bash
ssh-keygen
```
接下来需要输入存放路径时，输入
```bash
~/.ssh/openwrt_rsa		# 随自己喜好命名
```
配置`~/.ssh/config`文件，添加如下内容
```config
# route
Host ss
HostName 192.168.1.1
IdentityFile ~/.ssh/openwrt_rsa
User root
```
将公钥上传到 openwrt 相应目录下
```bash
cat ~/.ssh/openwrt_rsa.pub \ | ssh root@192.168.1.1 "cat >> /etc/dropbear/authorized_keys"
```
也可以使用如下命令（两者选其一）：
```bash
ssh root@192.168.1.1 "tee -a /etc/dropbear/authorized_keys" < ~/.ssh/openwrt_rsa.pub
```
登陆路由器，设置权限
```bash
ssh ss
chmod 700 /etc/dropbear
chmod 600 /etc/dropbear/authorized_keys
```
此时就可以通过 `ssh ss` 直接无密码直接登陆路由器了。

Enjow～



