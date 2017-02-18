title: Archlinux 下踩过的坑（持续更新）
date: 2016-03-19 06:53:52
tags:
- arch
- linux
- gnome
- gdm
- fctix
- terminal
---

## 前言 
今天又给一台台式装上了 Archlinux。发觉每新安装一次 Arch，对这个系统的了解就y更深入一点，解决起问题来也没以前那么茫然了，对它的喜爱又多了一些，甚至有赶超 MAC OSX 的倾向。

这里就记录一些安装过程遇到的坑。以减少以后掉坑的次数。
<!--more-->

### 那些踩过的坑

#### GDM 登出黑屏现象

> 为了屏保，就选择了 GDM + GNOME3 的桌面环境组合。然遇到个问题，在点击 Logout
> 想注销用户时，会出现黑屏现象。无法唤醒，只能强制关机。
> **解决：**编辑`/etc/gdm/custom.conf`文件，将`WaylandEnable=false`前面的注释去掉。原因参考[这里](https://bbs.archlinux.org/viewtopic.php?pid=1582893#p1582893)

#### gnome-terminal 无法启动

> 刚安装完桌面环境，想启动 terminal ，结果点击图标，转了一会圈圈就没了。启动不起来。
> **解决：**这是由于登录用户 - Acconts settings 里面的 language 不正确导致的。编辑 `/etc/locale.gen`时，反注释 `en_US.UTF-8`，执行`locale-gen`，使语言生效。并且执行`echo LANG=en_US.UTF-8 > /etc/locale.conf` 添加配置文件。

#### gnome-terminal 中无法调出 fctix 中文输入

> 这个在[官方文档](https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))有解决方法。照搬过来。
> **解决：**修改 GSettings 配置，执行如下命令：
> ```bash
> gsettings set \
>   org.gnome.settings-daemon.plugins.xsettings overrides \
>   "{'Gtk/IMModule':<'fcitx'>}"
> ```

## 后续
暂时遇到的问题就这么多，后续遇到在慢慢添加。
