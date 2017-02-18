title: "使用 powerline 美化插件"
date: 2015-05-04 22:40:32
tags:
- powerline
- ubuntu
- bash

---

关于 Powerline
==============

Powerline 是一款 vim 状态栏的美化插件，它还可以美化其他的比如 zsh,bash,tmux,iPython,Awesome and Qtile 程序。

> 参考 [Github](https://github.com/powerline/powerline)
> 参考 [Powerline 官方手册](https://powerline.readthedocs.org/en/latest/)

这里主要记录我在 bash 上的安装及使用方法， vim 的插件我使用的是 [vim-airline](http://www.leyar.me/Something-about-vim/)搭配 powerline 的字体符号补充包。

<!--more-->

安装及使用
===============
安装插件
---------------
需要先安装 `python-pip` 及 `git`，方法自行 Google
安装 powerline

```bash
pip install --user git+git://github.com/powerline/powerline
```
修改 `~/.profile` 文件，
```bash
sudo vim ~/.profile
```
添加以下内容到后面：
```bash
if [ -d "$HOME/.local/bin" ]; then
    PATH="$HOME/.local/bin:$PATH"
fi
```
安装字体
------------------
我使用的是 Gnome Terminal，因此使用下面的方法安装。
```bash
wget https://github.com/Lokaltog/powerline/raw/develop/font/PowerlineSymbols.otf https://github.com/Lokaltog/powerline/raw/develop/font/10-powerline-symbols.conf
mkdir -p ~/.fonts/ && mv PowerlineSymbols.otf ~/.fonts/
fc-cache -vf ~/.fonts
mkdir -p ~/.config/fontconfig/conf.d/ && mv 10-powerline-symbols.conf ~/.config/fontconfig/conf.d/
```
使用插件美化 Bash
-------------------------
```bash
source ~/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh
```
为了让他自行启动，可以编辑 `~/.bashrc` 文件
```bash
vim ~/.bashrc
```
添加以下内容：

```bash
if [ -f ~/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh ]; then
    source ~/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh
fi
```
另 Gnome Terminal 支持 256 色：

添加以下内容到 `~/.bashrc` 文件中：
```bash
if [[ ($COLORTERM == gnome-terminal || $(cat /proc/$PPID/cmdline) == *gnome-terminal* )
   && $TERM != screen* ]]; then
   export TERM=xterm-256color
fi
```
此配置使 256 色仅在 Gnome Terminal 中显示且不在 screen/tmux 会话中。

总结
========================
更详细的方法参考： [How can i install and user powerline plugin](http://askubuntu.com/questions/283908/how-can-i-install-and-use-powerline-plugin)
经此一番设置，你的 bash 应该已经美化过啦，如果没有，就重启下~ enjoy~


