title: Archlinux 安装笔记
date: 2015-07-28 11:28:46
tags:
- archlinux
- linux
---

前言
-----

之前就打算将 Ubuntu 换成 Archlinux，但这个计划一直被搁置，直到把双系统中的
Windows 换成了 Windows 10，才决定顺便一起将 Ubuntu 也换了。

安装方法参考的官方 [Wiki](https://wiki.archlinux.org/index.php/Main_page)，
完全在文字界面下安装，以便更深入的认识 Linux。这里简单记录下我的安装过程。

<!--more-->

安装准备
-----

### 制作 U 盘启动盘

在官方 Archlinux [Download Page](https://www.archlinux.org/download/)下载最新的镜像。

使用 `dd` 制作 U 盘启动盘

先通过 `lsblk` 查看 U 盘所在驱动器，并确保其处于未挂载状态，然后通过以下命令开始制作启动 U 盘。

```
# dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx && sync
```

`if` 后面的参数为 iso 所在的路径，`of` 后面的参数为 u 盘所在驱动器名，例如`/dev/sdf`

注意：以上方式会格式化 U 盘，需注意做好备份工作。

其他方法参照官方 Wiki：
[USB flash installation media](https://wiki.archlinux.org/index.php/USB_flash_installation_media)

U 盘制作完成后，重启并启动到 BIOS 模式（也可启动到 UEFI 模式），选择Archlinux 进入系统。之后会看到一个 `zsh`命令提示符。

### 网络连接

正常情况下有线网络会自动连接上。需要设置静态 IP 或者无线网络可以参考官方 Wiki

### 硬盘分区

以下分区步骤是通过命令行操作，可以提前用 PE 等其他工具在图形界面下先进行分区。我这里分为 /，/boot,/home,swap 四个分区。

如果已在图形界面分好区，可以直接跳到格式化分区步骤。

以 BIOS/MBR 为例进行分区(双系统请注意填写正确的分区编号)：

```
# lsblk		# 查看设备及分区情况
# parted /dev/sdx print		# 查看磁盘分区表类型
# parted /dev/sdx	# 打开需要新建分区表的设备,并进入 parted 交互模式
(parted) mkpart primary ext3 1M 300M		# 创建 /boot 分区
(parted) set 1 boot on		# 设置 /boot 为启动目录
(parted) mkpart primary ext3 300M 20G		# 20 G `/` 分区
(parted) mkpart primary linux-swap 20G 24G	# swap 分区
(parted) mkpart primary ext3 24G 100%		# 剩下的空间给 /home
```

*图形分区工具可以选择 [GParted](https://wiki.archlinux.org/index.php/GParted)*

### 创建文件系统

分区之后需要将其格式化为指定的文件系统。

```
# lsblk /dev/sdx	# 查看所有分区
# mkswap /dev/sdax	# 格式化 swap 区
# swapon /dev/sdax	# 启用 swap 区
# mkfs.ext4 /dev/sdaxy	# 格式化其他分区为 ext4 文件系统
```

### 挂载分区

```
# mount /dev/sdxR /mnt	# 挂载 `/`（root）分区
# mkdir /mnt/boot
# mount /dev/sdxy /mnt/boot	# 挂载 `/boot` 分区
# mkdir /mnt/home
# mount /dev/sdxy /mnt/home	# 挂载 `/home` 分区
```

### 选择安装镜像

编辑 `/etc/pacman.d/mirrorlist`，设置安装源。

```
# vi /etc/pacman.d/mirrorlist
```

通过 `/cn` 搜索中国的镜像源，去掉前面的注释启用。

这里贴一个使用[镜像列表生成器](https://www.archlinux.org/mirrorlist/)生成的距离我这边最近的镜像列表。

```
##
## Arch Linux repository mirrorlist
## Generated on 2015-07-31
##

## China
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
Server = http://mirror.bjtu.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.hust.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.hustunique.com/archlinux/$repo/os/$arch
Server = http://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.opencas.cn/archlinux/$repo/os/$arch
Server = http://run.hit.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.zju.edu.cn/archlinux/$repo/os/$arch
```

更改镜像源后，使用`pacman -Syy`强制刷新。

安装系统
----------------

```
# pacstrap -i /mnt base base-devel
```

生成 fstab (file system table)
----------------

```
# genfstab -U -p /mnt >> /mnt/etc/fstab
# vi /mnt/etc/fstabi	# 检查生成 fstab 是否正确
```

切换新系统
---------------

```
# arch-chroot /mnt /bin/bash
```

配置新系统
---------------

### Locale

```
# vi /etc/locale.gen
```

去掉以下几项前面的注释“#”

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
zh_HK.UTF-8 UTF-8
```

```
# locale-gen		# 生成 locale 讯息
# echo LANG=en_US.UTF-8 > /etc/locale.conf		# 提交本地化选项
```


### 时区

```
# ls /usr/share/zoneinfo	#查看可用时区
# ls /usr/share/Asia		#查看亚洲可用时区
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime	#设置时区
```

注意： 后期当出现时间不准确的问题，可通过强制执行上面的最后一项命令来修正。

```
sudo ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 硬件时间

为了便于统一机器上所有系统的硬件时间模式，可将操作系统时间设置成 UTC时间。

```
# hwclock --systohc --utc	# 自动生成 /etc/adjtime
```

关于时间问题，还可以参考[Time](https://wiki.archlinux.org/index.php/Time)

### 设置主机名

```
# echo *ileyar* > /etc/hostname		# myhostname 为 ileyar
```

修改 `/etc/hosts` 添加新建的主机名

```
#<ip-address>	<hostname.domain.org>	<hostname>
127.0.0.1	localhost.localdomain	localhost	ileyar
::1		localhost.localdomain	localhost	ileyar
```

### 配置网络

我这里使用的有线，直接使用 dhcpcd 服务即可。

```
# ip link	#查看网络接口名
# systemctl enable dhcpcd@*interface_name*.service	#interface_name 为网络接口名
```

其他也可以使用 netctl 或者 systemd-networkd 来配置并管理网络。详细操作方法参考官方 Wiki。

### 设置 Root 密码

```
# passwd
```

### 安装并配置 bootloader

这里针对 BIOS 主板使用 GRUB 来进行引导。

```
# pacman -S grub	# 安装 grub 包
# grub-install --target=i386-pc --recheck /dev/sda
# pacman -S os-prober		# 检测硬盘上其他已安装系统的程序
# grub-mkconfig -o /boot/grub/grub.cfg		# 自动生成引导项配置
```

### 卸载分区并重启

```
# exit
# reboot
```

拔除 U 盘，通过引导项进入新系统即可。至此基本系统安装完毕。

安装系统后必备操作
------------------

#### 安装 zsh：

```
# pacman -S zsh
```

#### 创建新用户并设置用户组

```
# useradd -m -g users -G audio,video,floppy,network,rfkill,scanner,storage,optical,power,wheel,uucp -s /usr/bin/zsh leyar	# 新建用户 leyar
# passwd leyar		# 设置用户密码
```

此时可以切换到新建的用户进行余下的操作了。

#### 安装图形界面

##### 安装 X 图形服务器：

```
$ sudo pacman -S xorg-server xorg-xinit     
```

##### 安装显卡驱动：

```
$ lspci | grep VGA	# 查看显卡
$ sudo pacman -Ss xf86-video-ati	# 查找并安装 ATI 开源驱动
```

常用开源驱动：

* NVIDIA: xf86-video-nouveau
* Intel: xf86-video-intel
* ATI: xf86-video-ati

至此可以使用 `startx` 命令手动启动 X图形界面了。

##### 安装桌面环境

###### GNOME 
由于 Xorg 只提供了图形环境的基本框架，用户体验有点差强人意。为了使界面更友好，此时需要安装桌面环境，我这里使用的是 GNOME

```
$ sudo pacman -S gnome
```

将以下命令添加到`~/.xinitrc`，即可从控制台下启动 GNOME.

```
exec gnome-session
```

至此，可通过 `startx`来启动 GNOME.

###### GDM

* 如果想通过图形化登陆，则可以使用登陆管理器来操作，比如 GDM

```
$ sudo pacman -S gdm		# 安装 gdm
$ systemctl enable gdm		# 开机启动 gdm.service
$ systemctl -f enable graphical.target		# 当无法进入图形界面时则经此实现
$ systemctl enable NetworkManager.service	# 启动网络管理服务
```

未完待续
--------

至此，基本的系统及桌面环境都安装完成，更多的有关程序安装配置等问题，也可以在图形界面浏览器中自行 Google，后续会更新一些我在配置过程中遇到的问题及解决办法。




