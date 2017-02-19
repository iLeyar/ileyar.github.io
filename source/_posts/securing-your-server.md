---
title: "VPS 安全设置"
date: 2015-06-10 11:59:59
tags: 
- linux
- vps
- ssh
---


当给一台 VPS 安装好系统之后，最首要的任务，就是进行一些安全设置了。这里记录一下我的操作过程。

> 这里 VPS 系统以 Debian7.0 32位为例子，正常情况下，Debian/Ubuntu 系统适用。

大致要设置的内容如下：
 
* 添加一个新用户
* 使用 SSH Key 密钥登陆
* 修改 SSH 端口，禁止 SSH 密码验证和 Root 登陆
* 创建防火墙
* 安装配置 Fail2Ban

<!--more-->

下面说下具体的实现步骤。

添加新用户
----------

首先使用 SSH 登陆 vps

添加用户, leyar 改为其他想要设置的用户名
```bash
 adduser leyar
```

添加用户到系统管理员组
```bash
 usermod -a -G sudo leyar
```

退出 root 用户登陆
```bash
 logout
```

使用新用户登陆 Vps
```bash
$ ssh leyar@162.254.0.100
leyar@162.254.0.100's password:                  #这里输入新用户密码
```

使用 SSH Key 密钥
-------------------

首先在**本机系统**（我的是 Ubuntu系统）生成 SSH keys
```bash
$ ssh-keygen
```

默认情况下会在本机 `~/.ssh/`目录下产生两个文件`id_rsa`和`id_rsa.pub`，我这里因生成的过程中修改了文件名为`witlayer_rsa`，因此我的公钥为`witlayer_rsa.pub`

从本机上传公钥到服务器
```
$ scp ~/.ssh/witlayer_rsa.pub leyar@162.254.0.100:
```

在 Vps 服务器的`/home/leyar`目录下创建`.ssh`文件夹，并将上传的文件移动到该目录下并重命名为`authorized_keys`
```bash
mkdir .ssh
mv witlayer_rsa.pub .ssh/authorized_keys
```

给`authorized_keys`文件设置权限
```bash
chown -R leyar:leyar .ssh
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```

到此，SSH Key 设置完毕。

修改端口,禁止 SSH 密码登陆及 Root 登陆。
-----------------------------

打开配置文件：
```bash
$ sudo vi /etc/ssh/sshd_config
```

修改配置：
```
Port 1688

PermitRootLogin no

PasswordAuthentication no
```

重启 SSH 服务：
```bash
$ sudo service ssh restart
```

如果出现如下错误：
 ```
Could not load host key: /etc/ssh/ssh_host_ecdsa_key
[....] Restarting OpenBSD Secure Shell server: sshdCould not load host key: /etc/ssh/ssh_host_ecdsa_key
. ok 
```

此时通过执行重新生成 hosts keys 操作
```bash
sudo rm /etc/ssh/ssh_host_*
dpkg-reconfigure openssh-server
```

然后再重启 ssh 服务即可。

创建防火墙
------------

查看默认防火墙规则：
```bash
sudo iptables -L
```

如果出现如下提示，则代表还没有配置过任何防火墙规则。
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination  
```

创建规则配置文件：
```bash
sudo vi /etc/iptables.firewall.rules
```

这里提供一个基础配置，可复制粘帖进去。

```
*filter

#  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
-A INPUT -i lo -j ACCEPT
-A INPUT -d 127.0.0.0/8 -j REJECT

#  Accept all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#  Allow all outbound traffic - you can modify this to only allow certain traffic
-A OUTPUT -j ACCEPT

#  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT

#  Allow SSH connections
#
#  The -dport number should be the same port number you set in sshd_config
#
-A INPUT -p tcp -m state --state NEW --dport 1688 -j ACCEPT

#  Allow ping
-A INPUT -p icmp --icmp-type echo-request -j ACCEPT

#  Log iptables denied calls
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

#  Drop all other inbound - default deny unless explicitly allowed policy
-A INPUT -j DROP
-A FORWARD -j DROP

COMMIT

```

`:wq`保存退出。

这个基础配置目前只允许了如下服务和端口：HTTP(80),HTTPS(443),SSH(1688) 和 ping 。

启用规则：
```bash
sudo iptables-restore < /etc/iptables.firewall.rules
```

检查是否启用成功：
```bash
sudo iptables -L
```

新规则如果设置成功会出现如下输出提示：

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
REJECT     all  --  anywhere             loopback/8           reject-with icmp-port-unreachable
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:https
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:1688
ACCEPT     icmp --  anywhere             anywhere             icmp echo-request
LOG        all  --  anywhere             anywhere             limit: avg 5/min burst 5 LOG level debug prefix "iptables denied: "
DROP       all  --  anywhere             anywhere            

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
DROP       all  --  anywhere             anywhere            

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
```

创建脚本使其随机启动：
```bash
sudo vi /etc/network/if-pre-up.d/firewall
```

粘帖如下内容到打开的界面：
```bash
#!/bin/sh
/sbin/iptables-restore < /etc/iptables.firewall.rules
```

保存并退出。

赋予脚本可执行权限：
```bash
sudo chmod +x /etc/network/if-pre-up.d/firewall
```

至此防火墙规则配置开始生效并运作。

安装配置 Fail2Ban
---------------------

安装：
```bash
sudo apt-get install fail2ban
```

复制并创建个人配置文件：
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

通过`jail.local`文件修改配置：
```bash
sudo vi /etc/fail2ban/jail.local
```

修改配置文件里的 ssh 端口为自己设置的端口
```
[ssh]

enabled  = true
port     = 1688
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 6
```

其他的按需配置。

重启 Fail2Ban 使配置生效：

```bash
sudo service fail2ban restart
```

可以通过`sudo iptables -L`命令查看 Fail2Ban 的某些规则是否添加进去了。

至此，关于 VPS 的安全配置暂且告一段落。

> 参考资料：
> 
> [Securing-your-server](https://www.linode.com/docs/security/securing-your-server)
> 
> [An-Introduction-to-Securing-your-Linux-VPS](https://www.digitalocean.com/community/tutorials/an-introduction-to-securing-your-linux-vps)
> 

