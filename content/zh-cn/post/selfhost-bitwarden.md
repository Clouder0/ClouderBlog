---
title: "自建 Bitwarden"
date: 2022-09-26T10:09:04+08:00
slug: "selfhost-bitwarden"
type: post
tags:
categories: 
  - Tech
---

闲置的腾讯云机子，由于没有备案域名啥都干不了，于是我打算废物利用一下，建一个 Bitwarden self host instance.

官方的 self host 对配置要求比较高，可以使用第三方的 [vaultwarden](https://github.com/dani-garcia/vaultwarden) 项目搭建。当然第三方可能没那么靠谱，相对而言。

个人使用完全足够。

vaultwarden 需要启用 https，那我们就换个端口吧，，，

## 安装 vaultwarden

在 Docker 环境下非常简单。

```bash
docker pull vaultwarden/server:latest
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 8080:80 vaultwarden/server:latest
```

这样就暴露到 8080 端口上了，用 `http://ip:8080/` 就可以访问。但应当是注册不了的。

## Caddy 反代

caddy 配置 ssl 比较方便，所以选用。

caddy 实在是太好用啦，一键配置，简单无脑。

ubuntu/debian 系的安装：

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

然后先手动停掉服务：

```bash
sudo systemctl stop caddy
```

手动开启反向代理：

```bash
sudo caddy reverse-proxy --from bitwarden.codein.icu:2016 --to 127.0.0.1:8080
```

Done!

## 配置

简单配置一下。

主要可能是关闭用户注册，根据 repo 中的[ .env template](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template) 稍作修改。把 `SIGNUPS_ALLOWED=true` 改成 `false` 就行了。

还有设置 `DOMAIN` 也比较重要。

再通过 `docker run --env-file <env-file> ...` 加载环境变量。

## 客户端配置

先在网页拓展上登出，在登录页面有设置选项，进去写 server url 就可以了。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220918145809-lzm7c8l.png)​

## 最后

其实跟用官方的也没有太大差别？不过倒是不用担心被墙。。。

记得定期备份！
