---
title: 使用Cloudflare SaaS加速网站访问速度
data: 2024-02-13T21:12:11+08:00
categories:
  - note
tags:
  - note
  - deploy
  - cloudflare
  - saas
  - cf
  - site
draft: false
---
因为性能、国内访问速度、配置、流量都好的小鸡，价格都是比较昂贵的，又没有盈利的业务需要部署，所以会选择一些国内访问速度较差的小鸡来部署个人业务。

但也需要国内能够正常访问，就考虑了 CDN 的方案，进而又了解到了 Cloudflare SaaS 回源的方案。就是将个人的网站托管到Cloudflare,并启用代理，然后解析接入了Cloudflare的、国内访问速度好的域名或IP上，实现加速访问。

<!--more-->

# 准备
- 两个域名
  - 一个托管在Cloudflare，假定为 cf.xyz
  - 一个托管到只能智能解析的DNS服务商，如阿里云解析，假定为 cf.top
- 一个小鸡

# 部署
## 设置回退源
- 在Cloudflare中添加指向实际业务的域名，并启用代理，假定为 xxx.cf.xyz，解析到小鸡的 IP
- 在小鸡中部署此域名，这里以 caddy 和 nextcloud 为例，需要先获取 cloudflare dns 的 token

  ``` ini
  https://xxx.cf.xyz {
      encode gzip
      tls {
          dns cloudflare <token>
      }

      redir /.well-known/carddav /remote.php/dav 301
      redir /.well-known/caldav /remote.php/dav 301

      reverse_proxy * localhost:10010 {
              header_up X-Forwarded-Ssl on
              header_up HOST {host}
              header_up X-Real-IP {remote}
              header_up X-Forwarded-For {remote}
              header_up X-Forwarded-Port {server_port}
              header_up X-Forwarded-Proto {scheme}
              header_up X-Url-Scheme {scheme}
              header_up X-Forwarded-Host {host}
              header_up Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
      }
  }
  ```
- 配置域名的 SSL/TLS
  - 在概述中将加密模式改为 Full 及其之上，不然可能会出现重定向过多的问题
  - 在自定义主机名中
    - 添加回退源：xxx.cf.xyz
    - 添加自定义主机名，这个域名是对外公开用来访问业务的域名，是另一个域名，假定为 xxx.cf.top。然后，按照 cloudflare 中出现的信息，去阿里云解析中添加对应的 TXT 记录，以验证所有权
## 设置分流
- 使用一个中间域名来设置分流，假定为 xxx-cdn.cf.top
  - 将其海外查询解析到 1.0.0.5，将其其他地区的查询解析到优选域名或IP，即默认，这里解析到 csgo.com
- 添加对外域名记录，即 xxx.cf.top，解析到 xxx-cdn.cf.top
- 在小鸡中使用 caddy 为 xxx.cf.top 申请证书，不然会出现证书无效的错误，配置如下：

  ``` ini
  https://xxx.cf.xyz {
      encode gzip
      tls {
          protocols tls1.2 tls1.3
          ciphers TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
          curves x25519
      }

      redir /.well-known/carddav /remote.php/dav 301
      redir /.well-known/caldav /remote.php/dav 301

      reverse_proxy * localhost:10010 {
              header_up X-Forwarded-Ssl on
              header_up HOST {host}
              header_up X-Real-IP {remote}
              header_up X-Forwarded-For {remote}
              header_up X-Forwarded-Port {server_port}
              header_up X-Forwarded-Proto {scheme}
              header_up X-Url-Scheme {scheme}
              header_up X-Forwarded-Host {host}
              header_up Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
      }
  }
  ```

# 优先域名

``` ini
Qatar2022(SAN JOSE IP) :
www.qatar2022.qa

IP查询 :
ip.skk.moe
ping.pe

政府网站:
乌克兰政府
www.gov.ua

泰国政府
www.thaigov.go.th

卡塔尔政府
www.gco.gov.qa

瑞典政府
www.gov.se

美国FBI:
fbi.gov

商业网站：
CSGO官方网站
(Cloudflare LONDON ASN)：
csgo.com

DigitalOcean
digitalocean.com

VISA官方网站：
(格式： VISA.国家顶级域名)
VISA.COM
VISA.CN
VISA.FI
VISA.HK

伪AMEX:
AMEX.COM

SHOPIFY(推荐):
SHOPIFY.COM

域名注册商：
Dynadot.COM

其它网站：
SINGAPORE.COM
JAPAN.COM
BRAZIL.COM
MALAYSIA.COM
RUSSIA.COM
```

# 参考文档
- [通过CloudFlare+SaaS回源优选IP使国内用户高速访问网站](https://dooo.ng/archives/1701171631107)
- [CF-IP优选](https://github.com/EzXxY/CF-IP)
