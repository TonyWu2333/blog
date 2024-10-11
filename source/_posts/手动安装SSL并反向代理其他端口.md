---
title: 手动安装SSL并反向代理其他端口
date: 2024-10-12 02:47:50
tags: web
---

# 手动添加SSL证书并反向代理其他端口

## 下载证书

这里以我在阿里云上申请的免费证书为例

![20241012021332](https://s2.loli.net/2024/10/12/DpxaVQwHX1Oz5NG.png)

因为我用的服务器是nginx，所以下载对应的压缩包，解压后有两个文件。key是密钥，pem是证书

![20241012021558](https://s2.loli.net/2024/10/12/gSkCrKAV7aE4dnY.png)

## 上传文件至服务器

接下来把证书和密钥上传到服务器，路径随意（配置里可指定），一般默认如下：

证书文件：/etc/ssl/certs/

私钥文件：/etc/ssl/private/

## 更改nginx配置文件

```
sudo vim /etc/nginx/sites-available/default
```

按 i 进入vim的编辑模式

在配置里指定证书和密钥的路径，我的alist运行在5244端口，所以我通过下面配置直接把443端口反向代理到5244端口。

```
server {
    listen 443 ssl;  # 监听443端口并启用SSL
    server_name your_domain_or_ip;  # 替换为你的域名或 IP

    ssl_certificate /etc/ssl/certs/your_certificate.crt;  # 证书路径
    ssl_certificate_key /etc/ssl/private/your_private.key;  # 私钥路径

    location / {
        proxy_pass http://localhost:5244;  # 反向代理到5244端口
        proxy_set_header Host $host;  # 转发主机头
        proxy_set_header X-Real-IP $remote_addr;  # 转发客户端真实IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  # 转发代理链中的IP
        proxy_set_header X-Forwarded-Proto $scheme;  # 转发协议
    }
}

```

证书安装完成，重启下nginx，即可用https访问网站

```
sudo nginx -t
sudo systemctl reload nginx
```