title: 5.5 每日折腾记
tag:
- powerline
- chrome
- lantern
- solarized
categories: Ubuntu
--------------------------

前言：
--------------------------

> 悲催，今天手贱直接来了一个 `rm -rv Documents`， 导致前面的所有文档记录资料都没了。Google 了下感觉恢复起来好麻烦，T T~ 
> 吸取教训，以后一定要加个 `i` ，趁有点记忆，先将今天的折腾记录下。

概括：
--------------------------

- 编辑了文件 `~/.vimrc`，添加了一些关于 [vim-airline](https://github.com/bling/vim-airline) 的配置；
- Powerline 字符显示异常，已修复，安装并配置对应的字体补充包；（下面附操作步骤）
- 安装 chrome dev版，出现问题及解决办法；（附操作步骤）
- 安装使用 [Lantern](https://getlantern.org/)；
- 使用 [`oh-my-zsh`](https://github.com/robbyrussell/oh-my-zsh)，搭配 [agnoster](https://github.com/robbyrussell/oh-my-zsh/wiki/themes#agnoster) 主题；
- 终端 ls 添加配色，使用 [`dircolors-solarized`](https://github.com/seebi/dircolors-solarized)
- 终端配色，使用 [`gnome-terminal-colors-solarized`](https://github.com/Anthony25/gnome-terminal-colors-solarized)

<!--more-->

操作：
--------------------------

### chrome dev 安装及问题解决

for 64 bit

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

出现问题：

```bash
$ sudo dpkg -i google-chrome-stable_current_i386.deb
Selecting previously unselected package google-chrome-stable.
(正在读取数据库 … 系统当前共安装有 165551 个文件和目录。)
Preparing to unpack google-chrome-stable_current_i386.deb …
Unpacking google-chrome-stable (36.0.1985.125-1) …
dpkg: dependency problems prevent configuration of google-chrome-stable:
google-chrome-stable 依赖于 libappindicator1；然而：
未安装软件包 libappindicator1。

dpkg: error processing package google-chrome-stable (–install):
依赖关系问题 – 仍未被配置
Processing triggers for man-db (2.6.7.1-1) …
Processing triggers for gnome-menus (3.10.1-0ubuntu2) …
Processing triggers for desktop-file-utils (0.22-1ubuntu1) …
Processing triggers for bamfdaemon (0.5.1+14.04.20140409-0ubuntu1) …
Rebuilding /usr/share/applications/bamf-2.index…
Processing triggers for mime-support (3.54ubuntu1) …
在处理时有错误发生：
google-chrome-stable
```
解决办法：

```bash
sudo dpkg -i google-chrome-stable_current_amd64.deb -force    # 强制安装
sudo apt-get install -f                                       # 补齐依赖
```
---------------------------------------------------------------------------

### Powerline fonts

```bash
git clone https://github.com/powerline/fonts.git ~/powerlinefonts

cd powerlinefonts
source install.sh

cp UbuntuMono/fonts.dir ~/.fonts/
cp UbuntuMono/fonts.scale ~/.fonts/
```

```bash
xset q           # check valid X font path

xset +fp ~/.fonts   # set path
```

```bash
fc-cache -vf ~/.fonts/   # update font cache for the path the font was moved to
```
终端配置使用 Ubuntu Mono derivative Powerline 字体即可。
其他参考：[fonts installation documentation](https://powerline.readthedocs.org/en/latest/installation/linux.html#fonts-installation)

--------------------------------------------------------------------------

### Dircolors-solarized

```bash
git clone https://github.com/seebi/dircolors-solarized.git ~/.dir_colors
cd .dir_colors
cp dircolors.256dark dircolors
```

新建终端配置。

```bash
eval `dircolors ~/.dir_colors/dircolors`
```
将以上命令添加至 `~/.zshrc` 文件

添加 256-color 终端支持：

```
if [[ ($COLORTERM == gnome-terminal || $(cat /proc/$PPID/cmdline) == *gnome-terminal* )                                                                 
       && $TERM != screen* ]]; then
       export TERM=xterm-256color
fi
```

将以上配置添加至 `~/.zshrc` 文件

-------------------------------------------------------------------------

### gnome-terminal-colors-solarized

```bash
git clone https://github.com/Anthony25/gnome-terminal-colors-solarized.git ~/.solarized

sudo apt-get install dconf-cli

cd .solarized
source set_dark.sh
```

---------------------------------------------------------------------

最后：
--------------------------------------------

还有一些记录，暂时想不起来了。删掉的文档只能到时候遇到了再重新 GOOGLE 摸索了。无比怨念中，罪恶的 `rm -rf` 之流啊~~~以后只用 `rm -ri`...
 
**生命不息，折腾不止**


