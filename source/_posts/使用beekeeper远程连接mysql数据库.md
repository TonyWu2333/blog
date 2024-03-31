---
title: 使用beekeeper远程连接mysql数据库
date: 2024-04-01 07:10:12
tags: mysql
---

# 使用beekeeper远程连接mysql数据库

首先更新一下软件包列表
```
sudo apt update
```
出现了几个无法下载
```
命中:1 http://security.ubuntu.com/ubuntu xenial-security InRelease
获取:2 http://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial InRelease [10.3 kB]
错误:2 http://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial InRelease
  明文签署文件不可用，结果为‘NOSPLIT’（您的网络需要认证吗？）
获取:3 http://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-updates InRelease [10.3 kB]
错误:3 http://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-updates InRelease
  明文签署文件不可用，结果为‘NOSPLIT’（您的网络需要认证吗？）
获取:4 http://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-backports InRelease [10.3 kB]
错误:4 http://mirrors.tuna.tsinghua.edu.cn/ubuntu xenial-backports InRelease
  明文签署文件不可用，结果为‘NOSPLIT’（您的网络需要认证吗？）
已下载 31.0 kB，耗时 1秒 (26.2 kB/s)
正在读取软件包列表... 完成
E: 无法下载 http://cn.archive.ubuntu.com/ubuntu/dists/xenial/InRelease  明文签署文件不可用，结果为‘NOSPLIT’（您的网络需要认证吗？）
E: 无法下载 http://cn.archive.ubuntu.com/ubuntu/dists/xenial-updates/InRelease  明文签署文件不可用，结果为‘NOSPLIT’（您的网络需要认证吗？）
E: 无法下载 http://cn.archive.ubuntu.com/ubuntu/dists/xenial-backports/InRelease  明文签署文件不可用，结果为‘NOSPLIT’（您的网络需要认证吗？）
E: 部分索引文件下载失败。如果忽略它们，那将转而使用旧的索引文件。
```
因为是刚装的虚拟机，所以还没有配置过apt源
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
cd /etc/apt
vim sources.list
```
把源替换成阿里源、清华源或中科大源。这里我选择了阿里源
```
#  阿里源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```
安装mysql-server
```
sudo apt install mysql-server
```
启动mysql
```
sudo /etc/init.d/mysql start
[ ok ] Starting mysql (via systemctl): mysql.service.
```
查看mysql版本验证是否安装成功
```
mysql --version
mysql  Ver 14.14 Distrib 5.7.42, for Linux (x86_64) using  EditLine wrapper
```
进入mysql
```
sudo mysql
```
创建一个新的用户并配置密码
```
CREATE USER 'tony'@'%' IDENTIFIED BY 'yourpasswd';
```
查看用户表
```
SELECT User, Host, authentication_string FROM mysql.user;
+------------------+-----------+-------------------------------------------+
| User             | Host      | authentication_string                     |
+------------------+-----------+-------------------------------------------+
| root             | localhost |                                           |
| mysql.session    | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys        | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| debian-sys-maint | localhost | *F88FAB99DA931815794B206EF76153157BB46708 |
| tony             | %         | *71C4791904DFA9FBABAE8F06D9CD9EF4F84544E4 |
+------------------+-----------+-------------------------------------------+
5 rows in set (0.00 sec)
```
给予新用户权限(具体权限可以根据自己的需求更改)
```
GRANT ALL PRIVILEGES ON *.* TO 'tony'@'%';
FLUSH PRIVILEGES;
```
配置防火墙，允许3306端口
```
sudo ufw allow 3306/tcp
```
修改`/etc/mysql/my.cnf`或`/etc/mysql/mysql.conf.d/mysqld.cnf`文件
```
[mysqld]
bind-address = 0.0.0.0
```
重启mysql
```
sudo systemctl restart mysql
```
下载社区版的[beekeeper](https://github.com/beekeeper-studio/beekeeper-studio/releases)

安装完成后打开，点击new connection

![image](https://s2.loli.net/2024/04/01/jTJO8cdCuRv3PB2.png)

connection type选择mysql

接下来依次填写你服务器的ip（域名），要连接的端口（mysql默认3306），用户名及用户密码（刚刚创建的）

![image-1](https://s2.loli.net/2024/04/01/8lvArm9iFsS4H3q.png)

点击connect连接成功

![20240401070534](https://s2.loli.net/2024/04/01/K6EXShsaJ7WilUp.png)