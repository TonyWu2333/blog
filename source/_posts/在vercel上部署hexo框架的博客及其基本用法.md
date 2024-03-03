---
title: 在vercel上部署hexo框架的博客
date: 2024-03-03 04:41:23
tags:
  - hexo
  - vercel
categories: 前端
---
# 在vercel上部署hexo框架的博客及其基本

## 本地部署

### 安装Git、Node.js

Git的安装有多种方式，本文不展开讲。我使用的是[GitHub Desktop](https://desktop.github.com/)，在windows和mac上均可下载使用。Node.js可以去[官网](https://nodejs.cn/download/)下载相应安装包

### 使用npm安装hexo

```
npm install -g hexo-cli
```

### 创建网站

```
hexo init [folder]
```

### 生成静态文件

```
hexo generate
```
也可以简写为
```
hexo g
```

### 启动服务器

```
hexo serve
```
也可以简写为
```
hexo s
```

启动后默认地址为http://localhost:4000/

## vercel部署

首先将存有博客代码的github仓库导入[vercel](https://vercel.com/)，在项目的setting中设置指令（如下图）

![vercel部署hexo指令](https://s2.loli.net/2024/03/03/yFnlUOHpDwQYWER.png)

编辑完成并save后create新的deployment即可，此时可以从vercel给的临时链接访问博客

## 绑定域名

由于vercel提供的域名需要科学上网访问，平时使用起来不方便，因此我们需要给我们的博客一个域名和DNS解析服务。我选择了阿里云平台购买了域名。购买前需要先实名认证。

购买成功后，在vercel中setting->domains下添加购买的域名，并根据主机记录、记录类型和记录值在阿里云平台添加新的解析。

## 关于图库

刚开始我使用github存储图片，但国内访问速度感人，因此使用了[SM.MS](https://sm.ms/)作为图床。为了更方便地管理图库，我下载了[PicGo](https://picgo.github.io/PicGo-Doc/zh/)来上传图片。只需要在PicGo中添加token即可配置完成。

SM.MS的token在user->dashboard->API token中可以查看

## 开始写作

### 创建新的文章

```
hexo new [layout] <title>
```

### 发表草稿

```
hexo publish [layout] <filename>
```

### [更多指令](https://hexo.io/zh-cn/docs/commands)