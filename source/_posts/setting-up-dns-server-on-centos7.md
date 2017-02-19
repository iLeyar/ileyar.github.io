---
title: "CentOS7 上使用 pdnsd 搭建 DNS 服务器"
date: 2015-07-30 01:10
tags:
- pdnsd
- dns
- centos
- linux
---

最近在折腾路由器（后续再把自己的步骤记录一下）其中涉及到防 DNS 污染的问
题，优化方案我选择了在自己的 VPS 上搭建 DNS 服务器。这里记录下我的搭建过
程。

系统： CentOS 7 64bit

安装 pdnsd
----

到官方[pdnsd Download Page](http://members.home.nl/p.a.rombouts/pdnsd/dl.html)下载最新版本 rpm 包，截至今日（2015-7-30）最新版本为 1.2.9：
```bash
wget http://members.home.nl/p.a.rombouts/pdnsd/releases/pdnsd-1.2.9a-par_sl6.x86_64.rpm
```
然后通过 yum 安装
```bash
yum localinstall pdnsd-1.2.9a-par_sl6.x86_64.rpm
```
如此安装完成。
<!--more-->

配置 pdnsd
----

拷贝配置样本 `/etc/pdnsd.conf.sample`，并重命名为`/etc/pdnsd.conf`

```bash
cp /etc/pdnsd.conf.sample /etc/pdnsd.conf
```
编辑配置文件 `pdnsd.conf`

```bash
vi /etc/pdnsd.conf
```
参考我的配置如下：
```conf
// Sample pdnsd configuration file. Must be customized to obtain a working pdnsd setup!
// Read the pdnsd.conf(5) manpage for an explanation of the options.
// Add or remove '#' in front of options you want to disable or enable, respectively.
// Remove '/*' and '*/' to enable complete sections.

global {
	perm_cache=4096;
	cache_dir="/var/cache/pdnsd";
#	pid_file = /var/run/pdnsd.pid;
	run_as="pdnsd";
	server_ip = 0.0.0.0;  # Use eth0 here if you want to allow other
				# machines on your network to query pdnsd.
	server_port = 9909;
	status_ctl = on;
#	paranoid=on;       # This option reduces the chance of cache poisoning
	                   # but may make pdnsd less efficient, unfortunately.
	query_method=udp_tcp;
	min_ttl=1d;       # Retain cached entries at least 15 minutes.
	max_ttl=1w;        # One week.
	timeout=5;        # Global timeout option (10 seconds).
	neg_domain_pol=on;
	udpbufsize=1024;   # Upper limit on the size of UDP messages.
}

# The following section is most appropriate if you have a fixed connection to
# the Internet and an ISP which provides good DNS servers.
server {
	label= "googledns";
	ip = 8.8.8.8
	, 8.8.4.4
	;		   # Put your ISP's DNS-server address(es) here.
#	proxy_only=on;     # Do not query any name servers beside your ISP's.
	                   # This may be necessary if you are behind some
	                   # kind of firewall and cannot receive replies
	                   # from outside name servers.
	timeout=3;         # Server timeout; this may be much shorter
			   # that the global timeout option.
	uptest=none;       # Test if the network interface is active.
	interface=eth0;    # The name of the interface to check.
	interval=10m;      # Check every 10 minutes.
	purge_cache=off;   # Keep stale cache entries in case the ISP's
			   # DNS servers go offline.
	edns_query=yes;    # Use EDNS for outgoing queries to allow UDP messages
			   # larger than 512 bytes. May cause trouble with some
			   # legacy systems.
#	exclude=.localdomain;  # If your ISP censors certain names, you may
#			   # want to exclude them here, and provide an
#			    # alternative server section below that will
#			    # successfully resolve the names.
}

/*
# The following section is more appropriate for dial-up connections.
# Read about how to use pdnsd-ctl for dynamic configuration in the documentation.
server {
	label= "dialup";
	file = "/etc/ppp/resolv.conf";  # Preferably do not use /etc/resolv.conf
	proxy_only=on;
	timeout=4;
	uptest=if;
	interface = ppp0;
	interval=10;       # Check the interface every 10 seconds.
	purge_cache=off;
	preset=off;
}
*/

/*
# The servers provided by OpenDNS are fast, but they do not reply with
# NXDOMAIN for non-existant domains, instead they supply you with an
# address of one of their search engines. They also lie about the addresses of 
# of the search engines of google, microsoft and yahoo.
# If you do not like this behaviour the "reject" option may be useful.
server {
	label = "opendns";
	ip = 208.67.222.222, 208.67.220.220;
	reject = 208.69.32.0/24,  # You may need to add additional address ranges
	         208.69.34.0/24,  # here if the addresses of their search engines
	         208.67.219.0/24; # change.
	reject_policy = fail;     # If you do not provide any alternative server
	                          # sections, like the following root-server
	                          # example, "negate" may be more appropriate here.
	timeout = 4;
	uptest = ping;            # Test availability using ICMP echo requests.
        ping_timeout = 100;       # ping test will time out after 10 seconds.
	interval = 15m;           # Test every 15 minutes.
	preset = off;
}
*/

/*
# This section is meant for resolving from root servers.
server {
	label = "root-servers";
	root_server = discover; # Query the name servers listed below
				# to obtain a full list of root servers.
	randomize_servers = on; # Give every root server an equal chance
	                        # of being queried.
	ip = 	198.41.0.4,     # This list will be expanded to the full
		192.228.79.201; # list on start up.
	timeout = 5;
	uptest = query;         # Test availability using empty DNS queries.
#	query_test_name = .;    # To be used if remote servers ignore empty queries.
	interval = 30m;         # Test every half hour.
	ping_timeout = 300;     # Test should time out after 30 seconds.
	purge_cache = off;
#	edns_query = yes;	# Use EDNS for outgoing queries to allow UDP messages
			   	# larger than 512 bytes. May cause trouble with some
			   	# legacy systems.
	exclude = .localdomain;
	policy = included;
	preset = off;
}
*/

source {
	owner=localhost;
#	serve_aliases=on;
	file="/etc/hosts";
}

/*
include {file="/etc/pdnsd.include";}	# Read additional definitions from /etc/pdnsd.include.
*/

rr {
	name=localhost;
	reverse=on;
	a=127.0.0.1;
	owner=localhost;
	soa=localhost,root.localhost,42,86400,900,86400,86400;
}

/*
neg {
	name=doubleclick.net;
	types=domain;   # This will also block xxx.doubleclick.net, etc.
}
*/

/*
neg {
	name=bad.server.com;   # Badly behaved server you don't want to connect to.
	types=A,AAAA;
}
*/
```

更多配置详情参考：[pdnsd Documents](http://members.home.nl/p.a.rombouts/pdnsd/doc.html)

启动 pdnsd
-----
```bash
service pdnsd start
```
开机启动：
```bash
chkcongif pdnsd on
```
添加防火墙规则
----
将 pdnsd 端口（我的是 9909）加入防火墙规则列表里：
```bash
/sbin/iptables -l INPUT -p tcp --dport 9909 -j ACCEPT
/etc/init.d/iptables save
service iptables restart
```
测试 pdnsd
----
本地使用自建 DNS 测试
```bash
dig @x.x.x.x -p 9909 youtube.com	# x.x.x.x 改为 DNS 服务器 IP
```
如果设置正确，则会出现如下查询结果：
```
; <<>> DiG 9.10.2-P2 <<>> @104.131.144.145 -p 9909 youtube.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11382
;; flags: qr rd ra; QUERY: 1, ANSWER: 11, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1024
;; QUESTION SECTION:
;youtube.com.			IN	A

;; ANSWER SECTION:
youtube.com.		82182	IN	A	74.125.239.98
youtube.com.		82182	IN	A	74.125.239.96
youtube.com.		82182	IN	A	74.125.239.102
youtube.com.		82182	IN	A	74.125.239.97
youtube.com.		82182	IN	A	74.125.239.105
youtube.com.		82182	IN	A	74.125.239.100
youtube.com.		82182	IN	A	74.125.239.110
youtube.com.		82182	IN	A	74.125.239.99
youtube.com.		82182	IN	A	74.125.239.104
youtube.com.		82182	IN	A	74.125.239.103
youtube.com.		82182	IN	A	74.125.239.101

;; Query time: 190 msec
;; SERVER: 104.131.144.145#9909(104.131.144.145)
;; WHEN: Thu Jul 30 01:50:40 CST 2015
;; MSG SIZE  rcvd: 216
```
返回的 ip 可参考：[Whatsmydns](https://www.whatsmydns.net)

参考来源：[Centos自建DNS](http://blog.tshine.me/centos%E8%87%AA%E5%BB%BAdns%E6%9C%8D%E5%8A%A1%E5%99%A8.html) 
