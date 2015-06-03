title: "修复 Ubuntu 上无法自动挂载 U 盘的问题"
date: 2015-05-15 22:38:05
tags: ubuntu
categories: ubuntu

---

前言：
--------------

今天打开电脑想把某些文件拷进 U 盘，结果插入 U 盘没反应。Gnome 没有自动挂载上去。通过前后对比 `cat /proc/partitions` 的结果发现有多出来新的分区 sdf 但使用 `fdisk -l /dev/sdf` 检查新的 U 盘提示无法打开。

<!--more-->

解决：
--------------

检查全部分区：

```bash
sudo fdisk -l
```

发现最后出现一个提示：

```
Partition table entries are not in disk order 
```

于是决定从这里入手，先修复这个错误。

```bash
sudo fdisk /dev/sda
```

可以根据提示命令 m ，然后依次选择输入：

```
x f r p w
```

x --extra functionality (experts only)
f --fix partition order
r --return to main menu
p --print the partition table
w --write table to disk and exit

PS：显示结果没有及时保存下来就重启了，这里就只记录输入过程。

通过 w 写入内容保存并退出后，重启系统。

```bash
sudo reboot
```

重启之后发现 U 盘能识别并读取了。

总结：
-----------

虽然还不是很清楚具体问题出在哪里，不过问题解决了就好。下次遇到同样的问题，再试试看其他的方法。 

其他[参考方案](http://blog.csdn.net/csfreebird/article/details/7669815)
