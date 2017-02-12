title: 科学上网新姿势（docker + ss）
date: 2016-04-09 00:31:22
tags:
- Docker
- Shadowsocks
- Carina
---

2016-12-15 更新: Cariana 已失效，可以尝试自建服务器上使用 docker

最近折腾的东西很杂很乱，笔记都没来得及整理。折腾内容包括但不限于： Docker, Ruby on Rails，vim 进阶用法...当然折腾的内容也比较浅显，没有系统的学习显得有点乱，基本是仅能满足需求而已。

这里记录下，使用 [Carina by Rackspace](https://getcarina.com/) + Docker 实现科学上网的过程。（其实用 Docker 搭建 ssserver 很早就有，只不过一直没研究，这里当作练手了)

<!--more-->

## 操作步骤

> 操作系统： Arch Linux （Windows 用户参考[八戒的博客](http://www.rendoumi.com/wan-quan-mian-fei-de-shadowsocksfu-wu-qi/)
> 涉及内容： Docker 指令，Linux command

### 注册帐号

链接：https://getcarina.com/ ，注册步骤很简单，邮箱验证，手机验证即可。据说手机号可以重复验证多个帐号。

### 新建 Cluster

登录界面后的界面，点击 `Add Cluster...`，进入 `Create Cluster` 界面，`Cluster Name` 随便填写一个名字，我这里填写的 `ss`，点击`Create Cluster` 按钮，稍等一会，STATUS 变成 `active` 绿色框框后，点击 `Get access` 按钮，`Download file` 下载压缩包到本地。

例如我是下载到 `～/Docker` 文件夹里，那么
```
cd ～/Docker
unzip ss.zip
```
此时会 `Docker` 目录下生成一个 `ss` 文件夹。
### 安装 Docker
Arch 软件仓库有这个软件包的稳定办，可以直接安装`Docker`
```
sudo pacman -S docker
```
### 设置环境变量
```
source ～/Docker/ss/docker.env
```
此时可以直接操作 Docker 了。
使用` docker info` 命令可以检查是否连接成功。
### 创建 ss 
```
docker network create my_network	# 创建网络，名为“my_network”
docker run -d --name ss --net my_network -p 40004:40004 oddrationale/docker-shadowsocks -s 0.0.0.0 -p 40004 -k $YOURPASSWORD -m aes-256-cfb
```
验证启动情况
```
docker port ss	# 查询 ss 所使用端口
docker ps	# 检查进程
```
如果一切正常，那么可以在客户端进行测试了。
## 待实现
准备自己做个能包含`chacha20` 加密协议的`Docker 镜像`，如果能打包配置文件，似乎也不错诶。


