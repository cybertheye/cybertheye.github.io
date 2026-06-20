---
title: 🌳 使用certbot来申请Let's Encrypt SSL证书
layout: post
author: cyven
tags: certbot ssl nginx docker
categories: CS CS::Tech
---


## Let's Encrypt

Let's Encrypt 是一个证书颁发机构（CA，Certificate Authority），它的职责是：

验证你对域名的控制权
签发被浏览器信任的 SSL 证书
维护证书的吊销列表

它本身只是一个服务，你无法直接「操作」它，只能通过协议和它通信。

## Certbot

Certbot 是一个客户端工具，它的职责是：

替你和 Let's Encrypt 服务器通信
自动完成域名验证流程（写 challenge 文件、调 DNS API 等）
把申请回来的证书保存到本地
到期前自动续期

Certbot和Let's Encrypt之间通过ACME协议通信

## 步骤

### 域名解析A记录

需要在你的域名服务商这里,添加一个A记录,记录值是域名IP,证明对域名有控制权


### nginx配置

主要的就是 acme-challenge的路径

```
server {
    listen 80;
    server_name domain-name.com www.domain-name.com

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    # location
    # ...
}

```


### docker启动服务

使用docker是为了方便安装 certbot

示例:

```
services:
  nginx:
    image: nginx:alpine
    container_name: gateway
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - certbot-certs:/etc/nginx/ssl:ro
      - certbot-webroot:/var/www/certbot:ro
    restart: unless-stopped

  certbot:
    image: certbot/certbot:latest
    container_name: certbot
    volumes:
      - certbot-certs:/etc/letsencrypt
      - certbot-webroot:/var/www/certbot
    entrypoint: ["/bin/sh", "-c"]
    command:
      - |
        while true; do
          certbot renew --quiet --webroot -w /var/www/certbot
          sleep 12h
        done
    restart: unless-stopped

volumes:
  certbot-certs:
  certbot-webroot:

```

启动服务

### 使用certbot申请域名

`docker exec certbot certbot certonly --webroot -w /var/www/certbot -d domain-name.com -d www.domain-name.com --email abc@xyz.com --agree-tos --no-eff-email`

`-d` 后面添加你需要的申请ssl证书的域名,但必须有对应的域名解析A记录

### 切换回HTTPS

申请成功后,就可以切换到正常的https nginx配置


```
# domain-name.com
server {
    listen       443 ssl http2;
    server_name  domain-name.com www.domain-name.com;

    ssl_certificate     /etc/nginx/ssl/live/domain-name.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/domain-name.com/privkey.pem;
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # location
    # ...
}

```

## 这里解释一下 SSL证书链的路径

Certbot 申请证书时，用 -d 后面第一个域名作为目录名：

```
certbot certonly --webroot \
  -d domain-name.com \        ← 第一个，决定目录名
  -d www.domain-name.com \
  -d ai.domain-name.com \
  ...
```

所以无论这张证书包含多少个域名，都存在：

`/etc/letsencrypt/live/domain-name.com/fullchain.pem`


那我这里使用 `/etc/nginx/ssl`路径是因为前面docker挂载了volume

` - certbot-certs:/etc/nginx/ssl:ro #nginx`
` - certbot-certs:/etc/letsencrypt  #certbot`
