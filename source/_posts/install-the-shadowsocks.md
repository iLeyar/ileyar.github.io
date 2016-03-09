title: "Shadowsocks 之安装篇"
date: 2015-8-01 14:00
tag: 
- shadowsocks
- linux
---

Install
-------

Debian/Ubuntu:

```
$ sudo apt-get install python-pip
$ sudo pip install shadowsocks
```

CentOS 7(于2016-2-5更新)

```
$ sudo yum -y update
$ sudo yum -y install python-pip
$ pip install --upgrade pip
$ sudo pip install shadowsocks
```
<!--more-->

Configuration
----------------

Create a config file `/etc/shadowsocks.json` .

```
{
	"server":"0.0.0.0",
	"server_port":8388,
	"local_address":"127.0.0.1",
	"local_port":1080,
	"password":"mypassword",
	"timeout":300,
	"method":"rc4-md5",
	"fast_open":false
}
```

### server

To run the config file in the foreground:

```
ssserver -c /etc/shadowsocks.json
```

or to run in the background:

```
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```

> you can also run it using command
> 
> To run in the background:
> 
	```
	$ sudo ssserver -p *port* -k *password* -m *method* --user nobody -d start 
	$ sudo ssserver -d stop
 	```

To check the log:

```
sudo less /var/log/shadowsocks.log
```

---

To run with deamon (Archlinux)

```
$ sudo systemctl start shadowsocks-server@shadowsocks
$ sudo systemctl enable shadowsocks-server@shadowsocks
```

#### iptable.firewall.rules

```
sudo vi /etc/iptables.firewall.rules
```

add the rules 

```
-A INPUT -p tcp --dport 443 -j ACCEPT
```

#### Daemon Auto-Restart

CentOS 7:

```
vi /etc/systemd/system/shadowsocks-server.service
```

write into the configuration:

```
[Unit]
Description=Shadowsocks Server

After=network.target

[Service]
Type=forking
PIDFile=/var/run/shadowsocks/server.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /var/run/shadowsocks
ExecStartPre=/bin/chown root:root /var/run/shadowsocks
ExecStart=/usr/bin/ssserver --pid-file /var/run/shadowsocks/server.pid -c /etc/shadowsocks/config.json -d start
Restart=always
RestartSec=24h
User=root
Group=root
UMask=0027

[Install]
WantedBy=multi-user.target
```

give the file permission:

```
# cd /etc/systemd/system
# chmod 755 shadowsocks-server.service
```

To run:

```
# systemctl start shadowsocks-server.service
# systemctl enable shadowsocks-server.service 
```

### Client

```
sslocal -c /etc/shadowsocks.json
```

or for deamon (Archlinux)

```
systemctl start shadowsocks@shadowsocks	   #run
systemctl enable shadowsocks@shadowsocks    #run when system startup
journalctl -u shadowsocks@shadowsocks	#check the log
```

Optimize
-----------

[Shadowsocks 之优化篇](http://www.leyar.me/optimize-the-shadowsocks/)
[Optimizing-Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)
[CentOS 7 Shadowsocks优化](https://www.ifshow.com/centos-7-shadowsocks-optimization/)
