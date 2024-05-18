---
title: 通过部署alist挂载各大网盘资源
date: 2024-05-16 06:10:14
tags: alist
categories: web
---

# 通过部署alist挂载各大网盘资源

## 安装alist

```
sudo apt update
sudo apt install curl
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s install
```

```
启动: systemctl start alist
关闭: systemctl stop alist
状态: systemctl status alist
重启: systemctl restart alist
```

放行5244端口

```
sudo ufw allow 5244/tcp
```

## 配置管理员密码

alist默认安装在 /opt/alist 中。进入其目录后执行：
```
./alist admin set NEW_PASSWORD
```
NEW_PASSWORD即为admin用户的密码。

## 挂载网盘

我这里以天翼云盘为例

驱动选择天翼云盘客户端

挂载路径必填，相当于目录位置

用户和密码必填，这就是天翼云盘的账号密码

根文件夹ID可以根据下图我选中的部分来查看

![文件夹ID](https://s2.loli.net/2024/05/16/4yUvGHBO7Yhbntf.png)

其他选项根据需要填写

之后保存即可

[这是我的alist网页](http://s.tonywu.top:5244)

可以使用下面这个账号登陆

用户名：friend

密码：123123123

有查看和下载的权限

欢迎来白嫖哦~

## 离线下载功能

离线下载功能需要安装aria2，安装方法可查看[这篇文章](https://zhuanlan.zhihu.com/p/658156257)

安装完成后打开alist右下角的按钮并展开，选择离线下载，并选择aria2即可。