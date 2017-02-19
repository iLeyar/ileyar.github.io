---
layout: post
title: "Compile Shadowsocks-libev in CentOS7"
date: 2017-02-13 19:35:27
updated: 2017-2-19 05:20:00
tags:
- shadowsocks
- centos7
---

## Prerequisites

### Build and install with recent mbedTLS and libsodium
```bash
export LIBSODIUM_VER=1.0.11
export MBEDTLS_VER=2.4.0
wget https://github.com/jedisct1/libsodium/releases/download/1.0.11/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS=-fPIC
sudo make DESTDIR=/usr install
popd
```
<!--more-->
### Other
```bash
yum install epel-release -y
yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto udns-devel libev-devel -y
```
### Get the latest source code
```bash
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive
```
## Installation
```bash
./autogen.sh && ./configure && make
sudo make install
```
## Configuration
### Create the configuration file
```bash
mkdir -p /etc/shadowsocks
vi /etc/shadowsocks/config.json
```
Put the following text into the file:
```json
{
 "server":"0.0.0.0",
 "server_port":40002,
 "local_address": "127.0.0.1",
 "local_port":1080,
 "password":"mypassword",
 "timeout":300,
 "method":"chacha20",
}
```
### To run with deamon in CentOS7
Create and edit a file:
```bash
vi /etc/systemd/system/shadowsocks.service
```
Add the following text to the file `shadowsocks.service` :
```
[Unit]
Description=Shadowsocks
After=network.target

[Service]
Type=forking
PIDFile=/run/shadowsocks/ss.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /run/shadowsocks
ExecStartPre=/bin/chown nobody:nobody /run/shadowsocks
ExecStart=/usr/local/bin/ss-server -u -c /etc/shadowsocks/config.json -v -f /var/run/shadowsocks/ss.pid
Restart=on-abort
User=nobody
Group=nobody
UMask=0027

[Install]
WantedBy=multi-user.target
```
To run    
```bash
systemctl start shadowsocks
systemctl enable shadowsocks
```
To stop
```bash
systemctl stop shadowsocks
```
Check the log
```bash
less /var/log/messages
```
You can also use the following command:
```bash
journalctl | grep ss-server 
```
or
```bash
journalctl -u shadowsocks.service
```
More usage about `journalctl`
+ [How Use Systemd journalctl Command To Manage Logs](http://linoxide.com/linux-how-to/systemd-journalctl-command-logs/)

## Firewalld
```bash
vi /etc/firewalld/services/shadowsocks.xml
```
Add the following text:
```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>shadowsocks</short>
  <description>enable shadowsocks.</description>
  <port protocol="tcp" port="39999"/>
  <port protocol="udp" port="39999"/>
</service>
```
Add a firewall policy use the command `firewall-cmd`
```bash
firewall-cmd --permanent --zone=public --add-service=shadowsocks
firewall-cmd --reload
```
## Reference material
+ [Shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)

*Last updated: 2017/02/19*
