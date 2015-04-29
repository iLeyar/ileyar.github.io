title: "XP 系统下硬盘安装 Ubuntu 14.04"
date: 2015-04-22 12:26:53
tags: Ubuntu
categories: Ubuntu
---

## 前言

这几天某人翻出来一台 N 年前淘汰下来的方正笔记本，因其某些情怀在作祟而木有扔掉，于是本着物尽其用的美好品德，我决定给他装上 Ubuntu 系统以作他用。但这个笔记本光驱已坏，无法通过光盘安装，而尝试使用 U 盘引导，bios 却不认。幸好其 xp 系统还能正常使用，遂决定通过硬盘使用 grub4dos 引导安装。

准备工具：

- grub4dos 下载地址： [点此链接](http://download.gna.org/grub4dos/)
- Ubuntu 14.04.2 LTS 32位镜像： [点此链接](http://www.ubuntu.com/download/desktop)
- 参考文档： [Ubuntu 硬盘安装](http://linux-wiki.cn/wiki/zh-hans/Ubuntu%E7%A1%AC%E7%9B%98%E5%AE%89%E8%A3%85)

>笔记本配置：
>  1G DDR2 内存, 120GB 硬盘容量

<!--more-->



操作步骤如下：

1. 将grub4dos 压缩包内的 grldr、menu.lst 两个文件解压到 C 盘根目录；
2. Ubuntu 镜像文件 ubuntu-14.04-desktop-i386.iso 放到 C 盘；
3. 解压 ubuntu-14.04-desktop-i386.iso 镜像文件中 casper 目录下的 vmlinuz 和 initrd.lz 文件到 C 盘；
4. 修改 menu.lst 文件，末尾处添加下列内容：

    ```
  title Install Ubuntu
  root (hd0,0)
  kernel (hd0,0)/vmlinuz boot=casper iso-scan/filename=/ubuntu-14.04-desktop-i386.iso ro quiet splash locale=zh_CN.UTF-8
  initrd (hd0,0)/initrd.lz
    ```

5. C 盘文件夹选项设置显示所有隐藏文件和文件夹，并且隐藏受保护的操作系统文件去勾，然后编辑 C:\boot.ini 文件，在末尾处添加如下内容，然后保存即可，如不能保存，可去掉文件的只读属性，或保存到其他路径再覆盖。

    ```
    C:\grldr="install ubuntu"
    ```

6. 重启，选择 install unbuntu 即可进入 ubuntu 试用安装界面。
7. Ubuntu 的安装及分区这里就不细说。我主要分了 /boot、/、/home、swap 四个区。

- 注意：这里主要是说下我安装 Ubuntu 过程中出现的一个问题，就是在开始安装时进度条一直卡在 “正在探测文件系统...”这一步，我等了大概十几分钟没动静就随手 Google 了下，在上面的参考文档里找到了答案。 即在安装之前需要 Ctrl + Alt + T 在终端中输入如下命令：

    ```
    sudo umount -l /isodevice
    ```

  关于 umount 命令，文档里有详细介绍。

<hr>

- 后记1：这个笔记本起初装的是国产麒麟版，装完后发现自带了一堆软件实在不爽，索性重装一次换成国际版，分区的时候手抖了下把引导项所在分区也格掉了，于是悲剧，重启无引导。只能将硬盘拆下来接到台式机通过 U 盘安装完成再接回笔记本。

- 后记2：使用 U 盘启动盘安装过程中，发现 Ubuntu 自带的“启动盘创建器”程序创建的 U 盘启动盘在引导进入之后会一直跳出 “gfxboot.c32: not a COM32R image” 的提示，经 Google 及本人实践，解决办法就是，按下 Tab 键，会出现 live live-install check memtext hd mainmenu help ，此时输入 live 就会进入试用界面，输入 live-install 则进入安装界面。 


> 总结：其实 XP 系统我许久未曾接触了，我开始学会安装系统的时候也是从 win7 开始的。突然遇到这么个古董级别的东西也算是让我长了些知识。这里就当做个记录也做个纪念，以后估计也不会遇到这么奇葩的情况，嗯。
>
