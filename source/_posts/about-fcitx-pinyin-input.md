title: 关于 Archlinux 下使用 fcitx 拼音输入法失效的解决办法
date: 2015-11-09 00:55
tag:
- fcitx
- pinyin
- input
---

### 前言

在 Arch 上通过 fcitx 使用拼音输入法有时候会出现快捷键"Ctrl + space" 切换不到拼音输入的情况,或者是有时候修改了下配置,输入法就又无法正常工作.重启也无法解决.

当`locale`为英文的时候,这种情况会比较普遍. 下面针对几种情况记录一下我的解决办法.

+ GTK2 程序中(例如 Firefox, Chromium 等)拼音输入法无法正常启用:
    
   > 解决办法就是安装 `fcitx-gtk2` 并且设置 `GTK_IM_MODULE`
   ```
   sudo pacman install fcitx-gtk2
   export GTK_IM_MODULE=fcitx
   ```

+ gnome-terminal 中拼音输入法无法正常启用:

   > 修改 GSetting 配置
   ```
   gsettings set \
   	org.gnome.settings-daemon.plugins.xsettings overrides \
	"{'Gtk/IMModule':<'fcitx'>}"
   ```

通过如上的设置,麻麻再也不用担心我打不出中文拉:)

> 参考资料:
> [Arch Wiki](https://wiki.archlinux.org/index.php/Fcitx_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29#.E5.9C.A8_gnome-terminal.E4.B8.AD_Ctrl_.2B_Space_.E4.B8.8D.E8.83.BD.E8.B0.83.E5.87.BA.E8.BE.93.E5.85.A5.E6.B3.95)
   
