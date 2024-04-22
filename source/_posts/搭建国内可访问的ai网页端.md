---
title: 搭建国内可访问的ai网页端
date: 2024-04-03 12:10:17
tags: vercel aicate
categories: AI
---

# 搭建国内可访问的ai网页端

各个大厂的AI大家想必应该都已经体验过了。但当我们的亲朋好友想使用ai时，就比较麻烦了。所以这里提供一个快速自建ai网页端的方案，能让朋友不用科学上网也能体验到国外的大模型。

## Vercel部署网页

如果你有openai的api额度，可以创建一个api key。没有openai的话，google的gemini或其他模型的api key也可以。

部署非常简单，创建仓库后填一下OPENAI_API_KEY和CODE（安全密码，可选）即可。

[点我跳转部署](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2FChatGPTNextWeb%2FChatGPT-Next-Web&env=OPENAI_API_KEY&env=CODE&project-name=nextchat&repository-name=NextChat)

![](https://s2.loli.net/2024/04/03/SQ2ueCkZ97WGTib.png)

点击deploy，等待几分钟后，网页便部署好了

## 添加GOOGLE API KEY

在setting中的environment variables中添加key（密钥名称GOOGLE_API_KEY）和value（你的密钥），点击save。

![](https://s2.loli.net/2024/04/03/Leyl84fGHIKBUYT.png)

## 域名绑定

在settings->domain中添加你的域名，然后在你购买域名的平台cname解析到vercel给你显示的地址。
在deployment中点击redeploy重新部署网页

## 开始使用

在登陆输入刚刚设置的code

![](https://s2.loli.net/2024/04/03/UhR3l4pF5LADJWY.png)

登陆后可以在设置中将模型改为google的gemini

![](https://s2.loli.net/2024/04/03/T6ocMVp57g9yUbz.png)

之后就可用开始愉快使用啦！