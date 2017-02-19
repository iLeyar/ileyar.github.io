---
title: Archlinux 下为 wndr4300 编译 OpenWrt trunk 版固件
date: 2016-02-24 12:46
updated: 2016-12-15 12:00
tags: 
- Arch
- wndr4300
- openwrt
- trunk
- compile
---

## 前言

前段时间一直想给我的 wndr4300 接个 USB 移动硬盘实现网络共享，然而一直未能成功，针对遇见的问题 Google 了许久并且也曾试图在[V2EX](https://v2ex.com/t/258234#reply19)上寻求过帮助，未果。后来在 [openwrt](https://forum.openwrt.org/viewtopic.php?id=52504) 论坛上搜到个跟我出现问题一模一样的人，他的解决办法是重新刷固件，如此也让我有了重新刷固件的念头。

> [官方](https://wiki.openwrt.org/doc/howto/obtain.firmware)获取固件的 5 种方式：
> 1. 从官网下载服务器下载预编译好的固件；
> 2. 使用[Image Generator](https://wiki.openwrt.org/doc/howto/obtain.firmware.generate) 生成固件镜像文件再自行定制编译；
> 3. 使用 OpenWrt 的 [SDK](https://wiki.openwrt.org/doc/howto/obtain.firmware.sdk) 工具交叉编译包；
> 4. 通过[OpenWrt Buildroot](https://wiki.openwrt.org/about/toolchain)从源代码进行编译；
> 5. 通过已做好的[Docker Image](https://wiki.openwrt.org/doc/howto/obtain.firmware.docker)定制自己的固件

在[Ariane](http://aiyou.im)的鼓励下，我决定从源文件编译一个固件。那么先把[官方文档](https://wiki.openwrt.org/about/toolchain)啃几遍。

## 具体步骤

条件：
+ Linux 系统（我用的 Arch Linux）；
+ 至少 5G 的硬盘空间;
+ 联网;
+ 设置环境变量
<!--more-->

### 预编译

Arch Linux 下安装 OpenWrt 编译环境相关依赖；

```bash
sudo pacman -Su && sudo pacman -Syy
sudo pacman -S --needed subversion asciidoc bash bc binutils bzip2 fastjar flex git gcc util-linux gawk intltool zlib make cdrkit ncurses openssl patch perl-extutils-makemaker rsync sdcc unzip wget gettext libxslt boost libusb bin86 sharutils b43-fwcutter findutils
```
安装 `subversion` 以及 `mercurial`:

```bash
sudo pacman -S install subversion mercurial
```

下载 OpenWrt trunk 版本源码，更多版本参考[Download Sources](https://wiki.openwrt.org/doc/howto/buildroot.exigence#downloading_sources)

```bash
mkdir openwrt
cd openwrt
git clone git://git.openwrt.org/openwrt.git trunk/
```

更新下载并安装所有可用的 `feeds`

```bash
cd trunk
./scripts/feeds update -a
./scripts/feeds install -a
```

### 构建基础环境

通过图形界面选择路由器型号，检测缺失包添加自己想要的包并构建基础环境。

```bash
make menuconfig
```
如图：

![](http://www.leyar.me/images/Screenshot_2016-02-24_14-42-35.png)

+ `Target System` 选中'(Atheros AR7xxx/AR9xxx)'
+ `Subtarget` 选中 '(Genaric devices with NAND flsh)'
+ `Target Profile` 选中'(NETGEAR WNDR3700v4/WNDR4300)'

简单操作指令：
【Enter】进入下级菜单；【Y】编译进固件包；【M】编译成单独的安装文件；【N】取消选择；按两次【Esc】退出或者返回上级菜单；【？】帮助；【/】搜索

标记说明：
【*】表示编译进固件包，【M】表示编译成安装文件，【】为不做操作。

选择好路由器型号之后，则可以执行
```bash
make defconfig
```
### 可选配置

再次执行：
```bash
make menuconfig
```
选择自己所需要安装的包。

这里列一些我的选择：

**Base system -->**
 * block-mount  # 挂载设备需要
 * 取消 dnsmasq，选择 dnsmasq-full

**Kernel modulesi -->**
 *	Block Devices --> kmod-scsi-core  # 内核对 SCSI 设备的支持
 *	Filesystems --> kmod-fs-ext4	# 我的移动硬盘是 ext4, 额外可选 ntfs,exfat等
 *	LED modules --> kmod-ledtrig-usbdev		# USB 设备闪灯
 *	Native Language Support --> kmod-nls-utf8	# utf8 编码格式的支持
 *	USB Support --> kmod-usb-core kmod-usb-storage kmod-usb-storage-extras kmod-usb-uhci kmod-usb2 

**Libraries -->**
 *	SSL --> libopenssl

**LuCI -->**
 *	Collections --> luci
 *	Applications --> luci-adblock luci-app-aria2 luci-ddns luci-app-mwan3 luci-app-samba luci-app-shadowsocks

**Network -->**
 * ChinaDNS
 * Download Manager --> webui-aria2 yaaw
 * Fire Transfer --> curl wget
 * shadowsocks-libev (官方源版本较低，可以自行编译一个，后面附方法)

**Utilities -->**
 * compression --> bzip2 unrar unzip zip	#压缩解压缩工具
 * Editors --> vim
 * Filesystem --> e2fsprogs	# 支持ext2、ext3、ext4 等分区工具(可选ntfs-3g ntfs-3g-utils)
 * disc --> blkid fdisk lsblk	
 * zoneinfo --> zoninfo-asia
 * dmesg	# 可使用 dmesg 命令
 * file	# 可使用 file 命令
 * mount-utils		# 挂载助手

执行`scripts/diffconfig.sh > diffconfig` 保存修改内容至`diffconfig`文件。

### 移除保留空间

修改文件将 wndr4300 被 Openwrt 留的 96M 空间完全利用

```bash
cp ./target/linux/ar71xx/image/Makefile ./target/linux/ar71xx/image/Makefile.bak	# 备份文件
vi ./target/linux/ar71xx/image/Makefile
```
搜索 wndr4300，找到其后面的 `(ubi)`以及`(firmware)` ，

将
```
23552k(ubi),25600k@0x6c0000(firmware)
```
修改为
```
121856k(ubi),123904k@0x6c0000(firmware)
```
使整个语段变为
```
wndr4300_mtdlayout=mtdparts=ar934x-nfc:256k(u-boot)ro,256k(u-boot-env)ro,256k(caldata),512k(pot),2048k(language),512k(config),3072k(traffic_meter),2048k(kernel),121856k(ubi),123904k@0x6c0000(firmware),256k(caldata_backup),-(reserved) 
```
### 编译

ok, 现在开始编译：
```bash
make -j 3 V=s  # j 后面数字改为你的 cpu 数量 +1
```
初次编译，大概需要花一个多小时左右。期间可以好好看看教程，睡个午觉。

编译完成之后，就会生成新的刷机包在`~/openwrt/trunk/bin/ar71xx/`目录下，文件名为`openwrt-ar71xx-nand-wndr4300-squashfs-sysupgrade.tar` ，在路由器中刷进去即可。(需要注意的是，只有已经刷了 Openwrt 之后才能刷这个包，如果你还是路由器出厂界面，请先刷同目录下的 .img 文件后，再刷这个更新包。）

## 编译 shadowsocks-libev
```bash
./scripts/feeds update packages
./scripts/feeds install libpcre
git clone https://github.com/shadowsocks/openwrt-shadowsocks.git package/shadowsocks-libev
make menuconfig
```
编译：
```bash
make V=s -j 3 
```
也可以单独编译包：
```bash
make V=99 package/shadowsocks-libev/openwrt/compile
```
其他需要额外编译的软件：
+ [luci-app-shadowsocks](https://github.com/shadowsocks/luci-app-shadowsocks)
+ [OpenWrt-dist Luci](https://github.com/aa65535/openwrt-dist-luci) - 包含了chinadns，dns-forwarder,redsocks2,shadowvpn 的 Luci 可选包
+ [DNS-Forwarder](https://github.com/aa65535/openwrt-dns-forwarder)
+ [ChinaDNS](https://github.com/aa65535/openwrt-chinadns)

具体编译方法参考链接，基本同上。

其他`make`相关指令
```bash
make clean		# 移除目录下的 /bin 和 /build_dir 文件夹
make dirclean	# 移除目录下的 /bin 和 /build_dir 以及 /staging_dir ，/toolchain，/logs 文件夹
make distclean  # 移除所有产生的配置编译文件及feeds 下载的内容。
make package/luci/clean  # 移除 luci 目录下的内容
```
### 技巧

配置文件可以备份到 `~/openwrt/trunk/files` 目录下，新编译的固件会自动将这些配置打包到固件里，这样你刷完固件就不需要再次配置文件啦！特别要注意路径的放置问题，例如
路由器路径： `/etc/config/network` 
备份路径：   `~/openwrt/trunk/files/etc/config/network`

最好别弄错啦！这里截个图，我目前备份的了的配置文件：

![](http://www.leyar.me/images/Screenshot_2016-02-24_17-05-09.png)

## 写在后面
经过这一番折腾，对于编译这个名词总算不至于特别陌生了。而我的路由器也弄好了，移动硬盘也能挂载上去了。准备研究一下 Samba，实现网络共享就在眼前...

参考文档：
+ [OpenWrt build system - Installation](https://wiki.openwrt.org/doc/howto/buildroot.exigence)
+ [OpenWrt build system -Usage](https://wiki.openwrt.org/doc/howto/build)

*本文最后更新：2016-12-15*
+ 已额外添加功能 samba, aria3, 硬盘休眠（hd-idle), 动态DNS (DDNS), mwan3, SQM Qos, UPNP 等。
+ 科学上网方案 shadowsocks-libev + ChinaDNS 
+ DNS 防污染方案：dnsmasq-full (dnsmasq.d) + ChinaDNS + DNS Forward( TCP 查询）
