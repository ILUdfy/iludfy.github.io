---
title: ubuntu安装mysql8.0流程记录
date: 2023-05-08 13:56:05
tags:
  - mysql
  - ubuntu
categories: mysql
---
# ubuntu安装mysql8.0流程记录
先切换到root用户下：
```
sudo su
```

本文档中使用的操作系统是ubuntu20.04，默认使用apt下载mysql默认版本可能是mysql5，所以先去官网<https://dev.mysql.com/downloads/mysql/>下载mysql8的源：
![img](./mysql8安装1.png)
![img](./mysql8安装2.png)
我们将下载到的```mysql-apt-config_0.8.24-1_all.deb```文件放到用户主目录下，执行下面的命令：
```
dpkg -i mysql-apt-config_0.8.24-1_all.deb
```
会出现下面的界面，选择```mysql server -> mysql-8.0 -> ok```
![img](./mysql8安装3.png)
之后我们使用下面的命令安装mysql：s
```ss
apt-get upgrade
apt-get install mysql-server
```
安装过程中会让你设置初始密码(如果没有这一步，看下面)：
![img](./mysql8安装4.png)
若是没出错就已经安装好了，可以尝试登录一下：
```
mysql -u root -p
```
输入刚才的初始密码即可。
![img](./mysql8安装5.png)

## 没有初始密码的情况
这种情况我遇见很多次了，不知道原因是什么
但是如果没有让你设置初始密码，那么理论上第一次登录你输入什么密码都会登录成功，这也是给你修改密码的机会。
## 修改密码
修改密码操作如下：
```
mysql> use mysql;
mysql> alter user 'root'@'localhost' identified by '你的新密码';
mysql> flush privileges;
```
然后退出数据库重新登录，输入你刚才设置的密码，即可登录成功。


**登录失败有如下可能性：**
> 1. 你的密码输入错误，谁是小丑我不说
> 2. 如果你登录时没有加sudo，试着使用sudo进行登录。如果使用sudo才可以登录成功，那么其他连接数据库的接口，比如jdbc，大概率是无法连接数据库的，这种情况看下一节：**远程访问**

如果你比上面说的还要小丑：密码记不住了，可以用如下两种方法之一来登录mysql：
![小丑竟在我身边](./小丑.png)
### 1.默认账户登录
mysql会创建一个默认账户，账户密码存放在 **/etc/mysql/debian.cnf**文件内：
```
sudo cat /etc/mysql/debian.cnf
# 输出如下：
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint  #用户名
password = UXuglBQfbMeF4aEu  #密码，这个密码每次安装都不一样，不要直接复制我的
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = UXuglBQfbMeF4aEu
socket   = /var/run/mysqld/mysqld.sock
```
按照这个用户名密码登录：
```
mysql -u debian-sys-maint -p UXuglBQfbMeF4aEu
```

之后按照上面的方法改密码即可。

### 2. 免密登录
在```/etc/mysql/mysql.conf.d/mysqld.cnf```文件的最后一行添加：```skip-grant-tables```
重启数据库：
```
systemctl restart mysql
```
再次登录数据库，随便输入一个密码就可。
修改完密码之后，把```/etc/mysql/mysql.conf.d/mysqld.cnf```还原，再次重启数据库即可。

## 远程访问
防火墙这里先不提，需要提前配置好防火墙，开放端口。
远程访问mysql至少需要满足两个条件：
### 1.配置用户允许访问的ip地址
mysql8默认只有本机可以访问，如果需要从其他机器远程访问数据库，需要进行配置。
登录数据库，查看权限表：
```
mysql -u root -p
mysql> use mysql;
mysql> select user,host from user where user='root';
```
会查询到如下结果：
```
+------+-----------------+
| user | host            |
+------+-----------------+
|root  | localhost       |
+------+-----------------+
1 row in set (0.00sec)
```
这里可以看到root用户的host字段是localhost，代表root用户只有本机可以访问。把它改成%（代表所有ip地址）：
```
update user set host = '%' where user ='root';
flush privileges;
```
### 2.若只有sudo才可以登录成功
需要修改一下用户的认证方式为```mysql_native_password```
运行如下命令：
```
mysql> use mysql;
mysql> alter user 'root'@'%' identified with mysql_native_password
mysql> flush privileges;
```