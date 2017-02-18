title: Shadowsocks 之优化篇
date: 2015-08-01 14:40
tags:
- shadowsocks
- linux

---

前言
---------

优化 Shadowsocks server 的前提条件是内核版本在 3.5 以上，可通过`uname -ir`查看当前内核版本，如果版本过低，需要升级。

以下我的 VPS 内核版本升级的简单步骤：

>	VPS ： Digitalocean
>	系统版本： CentOS 7 64bit

<!--more-->

更新内核
----------

查看当前内核版本信息

```    
uname -ir
3.10.0-123.8.1.el7.x86_64 x86_64
```
Digitalocean 关于 Kernels 的更新操作可参考[这里](https://www.digitalocean.com/community/tutorials/how-to-update-a-digitalocean-server-s-kernel)

更新并安装最新版本

```
yum update
yum list --showduplicates kernel	# 列出可用的 kernel 版本
--------------------------------
yum install kernel-*(3.10.0-229.el7)*		# 安装所需版本
```

验证安装及列出服务器上所有已安装的 kernels：

```
cd /boot
ls vmlinuz*
```

记住版本信息,进入 [Digitalocean 控制面板](https://www.digitalocean.com/community/tutorials/how-to-update-a-digitalocean-server-s-kernel-using-the-control-panel#changing-the-kernel-in-the-digitalocean-control-panel) ,修改想要使用的内核版本。

Poweroff then poweron.

[Optimize the shadowsocks server](http://shadowsocks.org/en/config/advanced.html)
---------------------------------

增加 TCP 连接数量
	
```
echo '* soft nofile 51200
* hard nofile 51200' >> /etc/security/limits.conf
```

编辑 `/etc/sysctl.conf` 文件，优化 TCP 参数
	
```
vi /etc/sysctl.conf
```

加入以下内容：
```
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1
```
刷新配置文件使之生效： 
```
sysctl --system
```
修改服务器中 shadowsocks 的 json 文件以开启 fast open:

```
fast_open: true
```

SSH 到路由器，进行如下操作:
 
```
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
```

参考链接：

+ [Optimize the shadowsocks server on Linux](https://shadowsocks.org/en/config/advanced.html)
+ [Optimizing Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)
+ [TCP Fast Open](https://github.com/shadowsocks/shadowsocks/wiki/TCP-Fast-Open)
+ [CentOS 7 Shadowsocks 优化](https://www.ifshow.com/centos-7-shadowsocks-optimization/)
--------

更新　
------

2016-1-12: 不要启用 `net.ipv4.tcp_tw_reuse` ,上文的配置文件已修正．具体原因可参考:

+ [Coping with the TCP TIME-WAIT state on busy Linux servers](http://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux.html)
+ [不要在linux上启用net.ipv4.tcp_tw_recycle参数](http://www.cnxct.com/coping-with-the-tcp-time_wait-state-on-busy-linux-servers-in-chinese-and-dont-enable-tcp_tw_recycle/)

2017-2-14: 修改`sysctl.conf`配置文件
