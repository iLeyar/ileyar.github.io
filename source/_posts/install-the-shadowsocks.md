title: "Shadowsocks 之安装篇"
date: 2015-8-01 14:00
tag: 
- shadowsocks
- linux
---

> 版本： Python版
> 加密方式： Chacha20

#Install
## 安装 shadowsocks
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
## 加密
使用 chacha20 加密，必须安装 M2Crypto 以及libsodium
### Installing M2Crypto
Debian/Ubuntu:
```
apt-get install python-m2crypto
```
CentOS:
```
yum install m2crypto
```
### Installing libsodium
安装编译工具包 build-essential

Debian/Ubuntu:
```
sudo apt-get install build-essential
```
CentOS 7:
```
sudo yum groupinstall "Development Tools"
#sudo yum install kernel-devel kernel-headers
```
编译安装 [libsodium](https://github.com/jedisct1/libsodium)（版本需>= 1.0.0）
```
wget https://github.com/jedisct1/libsodium/releases/download/1.0.8/libsodium-1.0.8.tar.gz
tar xf libsodium-1.0.8.tar.gz && cd libsodium-1.0.8
./configure && make -j2
sudo make install
sudo ldconfig
```
# 配置
Create a config file `/etc/shadowsocks.json` .
```
{
	"server":"0.0.0.0",
	"server_port":39999,
	"password":"mypassword",
	"timeout":300,
	"method":"chacha20",
	"fast_open":false
}
```
多用户配置
```
{
		"server":"0.0.0.0",
		"port_password": {
			"38888":"password1",
			"38889":"password2",
			"38890":"password3"
		},
		"timeout":300,
		"method":"chacha20",
		"fast_open":false
}
```
## 服务端启动
### 常规启动
+ To run the config file in the foreground:
```
ssserver -c /etc/shadowsocks.json
```
+ or to run in the background:
```
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop	# 停止 shadowsocks
```
### 查看日志
+ To check the log:
```
sudo less /var/log/shadowsocks.log
```
### 守护程序
+ To run with deamon in CentOS7

create file `sudo vi /etc/systemd/system/shadowsocks.service`
```
[Unit]
Description=Shadowsocks
After=network.target

[Service]
Type=forking
PIDFile=/run/shadowsocks/server.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /run/shadowsocks
ExecStartPre=/bin/chown root:root /run/shadowsocks
ExecStart=/usr/bin/ssserver --pid-file /var/run/shadowsocks/server.pid -c /etc/shadowsocks/config.json -d start
Restart=on-abort
User=root
Group=root
UMask=0027

[Install]
WantedBy=multi-user.target
```
give the file permission:
```
# cd /etc/systemd/system
# chmod 755 shadowsocks.service
```
to run
```
sudo systemctl start shadowsocks
sudo systemctl enable shadowsocks
```
### Firewalld
```
touch /etc/firewalld/services/shadowsocks.xml
sudo vi /etc/firewalld/services/shadowsocks.xml
```
加入如下内容：
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>shadowsocks</short>
  <description>enable shadowsocks.</description>
  <port protocol="tcp" port="39999"/>
  <port protocol="udp" port="39999"/>
</service>
```
添加防火墙策略
```
firewall-cmd --permanent --zone=public --add-service=shadowsocks
firewall-cmd --permanent --add-masquerade
firewall-cmd --reload
```
## 客户端启动

```
sslocal -c /etc/shadowsocks.json
```
Optimize
-----------

[Shadowsocks 之优化篇](http://www.leyar.me/optimize-the-shadowsocks/)
[Optimizing-Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)
[CentOS 7 Shadowsocks优化](https://www.ifshow.com/centos-7-shadowsocks-optimization/)
