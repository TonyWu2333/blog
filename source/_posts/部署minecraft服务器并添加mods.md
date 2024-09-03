---
title: 部署minecraft服务器并添加mods
date: 2024-04-21 22:24:37
tags: mc
categories: minecraft
---

# 搭建minecraft服务器并添加mods

## 下载[forge](https://files.minecraftforge.net/net/minecraftforge/forge/index_1.20.1.html)

找到你要的forge版本
![](https://s2.loli.net/2024/04/21/vO21fKTjPQUXR4A.png)
选择intall server
![](https://s2.loli.net/2024/04/21/MACfe2IRPgFDG3E.png)

## 部署forge服务器

根据你的操作系统打开run.sh（linux）或run.bat（windows），在此之前要先安装java，并确认你的java版本与forge版本兼容。
![](https://s2.loli.net/2024/04/21/91DhbGNkB2av8qE.png)
此时会生成一个elua.txt，打开并将elua的值改为true，再次启动run.sh（run.bat）
![](https://s2.loli.net/2024/04/21/fGFoIdql5OAxL7U.png)
mod可以放入mods文件夹内，成功启动后界面如下
![](https://s2.loli.net/2024/04/21/Dld5x6PoiEVXgJt.png)
打开服务器的25565端口即可远程连接mc服务器

## 其他MC服务器核心

除了forge其他还有很多服务器核心，如paper，mohist，arclight等，可以根据是否要添加插件或模组进行选择。其他服务器的部署与使用与forge大同小异，具体可以参考该服务器核心的文档。

[这里是服务器核心镜像网站](https://www.fastmirror.net/?coreVersion=1.20.1#/download/Paper?coreVersion=1.21.1)。