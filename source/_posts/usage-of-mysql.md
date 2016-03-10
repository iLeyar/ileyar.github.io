title: MariaDB 一些使用命令
tags:
- mysql
- mariadb
- linux
date: 2016-03-11 01:04

---

### 前言

涉及到数据库，偶尔会用到一些命令。这里记录一下用作备忘。

### 初始操作
初始化数据库服务程序
```
mysql_secure_installation
```
防火墙允许策略
```
firewall-cmd --permanent --add-service=mysql
firewall-cmd --reload
```
登陆操作
```
mysql -u root -p	# 登陆 root 账号
show databases;	# 查看已有数据库
set password = password('123456')	# 修改当前用户在数据库中的密码为 123456
exit	# 登出
```
<!--more-->
### 管理数据库与表单
```
create user name@localhost IDENTIFIED BY 'password';	# name 用户名 localhost 主机名 password 密码
use mysql;	# 进入 mysql 数据库
select host,user,password from user where user="name";	# 查看新建的用户、主机、姓名与密码信息
```
#### 授权与取消授权
```
GRANT 权限 ON 数据库.表单名称 TO 用户名@主机名	# 授权特定数据库表单
GRANT 权限 ON 数据库.* TO 用户名@主机名		# 授权特定数据库所有表单
GRANT 权限 ON *.* TO 用户名@主机名	 # 授权所有数据库及表单
GRANT 权限1,权限2 ON 数据库.* TO 用户名@主机名	# 多个授权特定数据库所有表单
GRANT ALL PRIVILEGES ON *。* TO 用户名@主机名	# 全部授权所有数据库表单
GRANT SELECT,UPDATE,DELETE,INSERT on mysql.user to name@localhost;	# 给name用户对 user 表单的查询、更新、删除、插入权限
show grants for name@localhost;		# 查看 name用户的权限
revoke SELECT,UPDATE,DELETE,INSERT on mysql.user from name@localhost;	# 取消 name 用户对 user 表单的查询、更新、删除、插入权限
```
#### 表单
```
show tables;	# 查看表单
```

创建表单示例
```
INSERT INTO books(name,price,pages) VALUES('secrets','60',518);
select * from books;
update books set price=55 ;
select name,price from books;
delete from books;
```
where 查找
```
select * from books where price! =90
```
关于表单的操作，可以查看[这里](http://www.linuxprobe.com/chapter-18/#182_ma)

### 备份与恢复
```
mysqldump -u root -p password mysql > /root/mysqlDB.dump	# 导出数据库， mysql 数据库名
```
```
drop database mysql;	# 删除数据库 mysql
create database mysql1;		# 创建数据库 mysql1
```
```
mysql -u root -p password < /root/mysqlDB.dump		# 导入数据库
```

