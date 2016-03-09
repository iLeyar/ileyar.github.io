title: NETGEAR WNDR4300 折腾之 OpenWrt + dnsmasq + SS  
date: 2015-08-01 00:14
tag: 
- OpenWrt
- SS
- NETGEAR
- dnsmasq 
- linux
---

前言
-----

一直想入手一个可以刷 [OpenWrt](http://wiki.openwrt.org/toh/netgear/wndr4300) 的路由器，国内的不考虑，国外的看中了两款，NETGEAR 的 WNDR4300 以及 linksys 的 E4200，E4200 貌似不能刷 OpenWrt，而且国内也没有地方可以买到。于是 WNDR4300 成了不二之选。

这里记录下我的刷机并且实现自动科学上网的过程.

> 本机系统环境： Archlinux
> [防 DNS 劫持](https://sourceforge.net/p/openwrt-dist/wiki/DNS/)方案: dnsmasq 搭配 ss-tunnel 
> 针对 IP 封锁：搭配 ignore.list 列表
<!--more-->

刷机
-----

### 下载 OpenWrt 固件

下载 OpenWrt 官方安装镜像及升级包，

```
wget http://downloads.openwrt.org/barrier_breaker/14.07/ar71xx/nand/openwrt-ar71xx-nand-wndr4300-ubi-factory.img
wget http://downloads.openwrt.org/barrier_breaker/14.07/ar71xx/nand/openwrt-ar71xx-nand-wndr4300-squashfs-sysupgrade.tar
```

### 刷入 OpenWrt 固件

登录路由器管理界面 (http://www.routerlogin.net) 进入「固件升级」页面，选择下载的安装镜像`openwrt-ar71xx-nand-wndr4300-ubi-factory.img`，点「确定」。

等待镜像刷入成功。

初次登入 OpenWrt 是没有密码的，随机键入一个字符即可进入管理界面。

### 升级 OpenWrt 固件

「System」-「Backup/Flash Firmware」-「Flash new firmware image」，选中下载的`openwrt-ar71xx-nand-wndr4300-squashfs-sysupgrade.tar`升级包，「Flash image...」

等待升级成功。

### 重启路由

当固件刷入成功后，为了确保 5GHz 的 WiFi 功能能正常使用，必须先关闭路由电源等待 5s 再开启电源重新启动。

至此，刷机步骤完成。

配置 OpenWrt
------------

### 配置网络

「Network」-「Interface」-`WAN`后面的「Edit」，`Protocol`选择`PPPoE`，「Switch Protocol」，输入帐号密码，「Save&Apply」，配置正确的话，此时有线网络可以正常连接上。

### 更改界面语言

*PS: 此步骤可选，我为了尽量节省空间，就没有做修改.*

「System」-「Software」-「Update list」-`filter`后面输入`chinese`搜索语言包，切换到「Available Packages」，点击搜索到的软件包后面的「install」安装中文包。

「System」-「System」-「Language&style」选择`chinese`-「Save&Apply」，刷新即可甚至界面为中文。

安装及配置 Dnsmasq 和 SS
----------

> 这里罗列一下使用到的开源项目包括版本(截至2015-11-9)：

> + [shadowsocks-libev-spec](https://github.com/shadowsocks/openwrt-shadowsocks) v2.4.1-1 
> + [dnsmasq-full](https://github.com/aa65535/openwrt-dnsmasq) v2.72-4

### 安装

`ssh` 到 openwrt

```
$ ssh root@192.168.1.1
```

更新软件包列表并安装依赖包

```
# opkg update
# opkg install ip libc libgcc libgmp libnettle libopenssl zlib
```

添加第三方源`OpenWrt-dist`源到`etc/opkg.conf`:

```
# echo 'src/gz openwrt_dist http://openwrt-dist.sourceforge.net/releases/ar71xx/packages
src/gz openwrt_dist_luci http://openwrt-dist.sourceforge.net/releases/luci/packages' >> /etc/opkg.conf
```

卸载原本自带的 dnsmasq，安装 dnsmasq-full 和 spec 版 ss 

```
# opkg update
# opkg remove dnsmasq
# opkg install dnsmasq-full
# opkg install shadowsocks-libev-spec
# opkg install luci-app-shadowsocks-spec
```

> 有时候会发现安装速度很慢，此时也可通过 PC 端下载并传至路由器 `/tmp`目录再执行安装．
> 上传文件到路由器：
> 
 	```
 	scp luci-app-shadowsocks-spec_1.3.3-1_all.ipk shadowsocks-libev-spec_2.2.3-1_ar71xx.ipk root@192.168.1.1:/tmp/
 	```
> 
> ssh 到路由器执行安装:
> 
 	```
 	# cd /tmp
 	# opkg install shadowsocks-libev-spec_2.2.3-1_ar71xx.ipk
 	# opkg install luci-app-shadowsocks-spec_1.3.3-1_all.ipk
 	```

下载忽略列表文件到`/etc/shadowsocks/ignore.list`，之后更新该文件也是使用此命令。

```
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/shadowsocks/ignore.list
```

### 配置

#### SS 的配置

登录到路由器的 WEB 管理界面，

「Services」-「ShadowSocks」- 「Global Setting」 里 「Enable」打勾并在下面输入 ss 服务器信息，「Proxy Setting」下的 「Proxy Method」选择`Ignore List`，「Proxy Protocol」选择`TCP only`，「UDP Forward」下勾选 「Enable」，然后点击「Save & Apply」即可．

这里配置启用了UDP 端口转发，默认会把本地的 5300 端口内容转发到 `8.8.4.4#53`．

#### DNS 的配置

「Network」-「DHCP and DNS」-「Resolv and Hosts Files」- 勾选「Ignore Hosts files」防止上级 DNS 的污染.

打开 SSH 窗口，在`/etc/dnsmasq.conf`中加入如下内容:

```
server=127.0.0.1#5300
```

通过如上配置，可以有效防止DNS污染，使所有的网址都通过 GOOGLE 公共DNS 服务器来解析，但同时会使某些情况下访问国内网站会出现国际版的情况，此时就需要进行针对性的优化了．

基本思路是，国内的站点使用 114 来进行解析，国外的站点继续走 GOOGLE 的公共 DNS．这可以通过对 dnsmasq 进行配置来实现．

> 相关的配置文件如下:
> [dnsmasq.conf] (https://github.com/aa65535/openwrt-dnsmasq/tree/master/etc)
> [dnsmasq-china-list] (https://github.com/felixonmars/dnsmasq-china-list)

操作步骤:

SSH 到openwrt

修改 `/etc/dnsmasq.conf` 文件，对应的配置范例可点击上面链接，主要修改的几个地方:

```
no-poll
no-resolv
all-servers

server=127.0.0.1#5300
conf-dir=/etc/dnsmasq.d
```

创建 `dnsmasq.d` 配置目录

```
mkdir /etc/dnsmasq.d
```

切换到 PC 端 terminal 操作(Linux 系统下)
```
$ cd ~/Download
$ mkdir dnsmasq.d
$ cd dnsmasq.d
$ git clone git://github.com/felixonmars/dnsmasq-china-list .
$ scp accelerated-domains.china.conf google.china.conf root@192.168.1.1:/etc/dnsmasq.d/		# 这里将国内网址列表这个文件上传到路由器.
```

### 重启 

```
/etc/init.d/shadowsocks restart
/etc/init.d/dnsmasq restart
```

此时可以测试操作是否成功了。测试相关 ip 网址如下:

+ 国内 http://ip.cn
+ 国外 http://www.whatsmyip.org/
+ 查询所使用的 DNS 服务 https://www.whatsmydns.net/

添加计划任务实现自动更新
-----

备注: 此更新脚本可以自动更新已安装的软件脚本以及 `ignore list`文件. 关于国内网址列表文件，我是在本机客户端自做了一个小脚本自动更新 git 仓库并上传到路由器覆盖.

ssh 到 openwrt，新建文件 `/root/update`，添加如下内容:

```
#!/bin/sh

# 更新 ignore 文件

set -e -o pipefail

wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | \
    awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > \ 
    /tmp/ignore.list

mv /tmp/ignore.list /etc/shadowsocks/

if pidof ss-redir>/dev/null; then
    /etc/init.d/shadowsocks rules
fi

# 更新软件

opkg update

for ipk in $(opkg list-upgradable | awk '$1!~/^kmod|^Multiple/{print $1}'); do
    opkg upgrade $ipk
done
```

使用 `chmod +x /root/update` 给脚本添加可执行权限

打开路由器 WEB 管理界面:

「System」-「Scheduled Tasks」填写如下内容(每天 04:30 执行):

```
30    4     *     *     *     /root/update>/dev/null 2>&1
```

后续补充
---

还有一些涉及到的例如 SS 服务器的安装与优化，及其他的一些改善内容，后续再慢慢研究。

> 参考资料:

> + [1](http://undownding.me/2015/02/10/use-shadowsocks-on-openwrt/)
