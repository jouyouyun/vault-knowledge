---
title: 部署 xiaoya-tvbox
data: 2024-02-07T21:12:11+08:00
categories:
  - note
tags:
  - note
  - deploy
  - tvbox
  - alist
  - xiaoya
  - docker
draft: false
---
小雅 alist是一个云盘资源合集，可通过 docker 部署，xiaoya-tvbox 则是对其功能进行了增强，具体参见 [power721/alist-tvbox](https://github.com/power721/alist-tvbox) 。

<!--more-->

部署 alist-tvbox，可使用 `sudo bash -c "$(curl -fsSL https://d.har01d.cn/update_xiaoya.sh)"` 命令一键完成，但需要提前准备好阿里云盘的 token、refresh_token、folder_id等。

# Token 获取
这里假定使用默认目录 `/etc/xiaoya/` 。
## 阿里 Token
1. 打开 <https://alist.nn.ci/zh/guide/drivers/aliyundrive.html>
2. 点击获取 Token
3. 使用阿里云盘扫描二维码
4. 点击我已扫描，就可得到 Token

保存此 Token 到 `/etc/xiaoya/mytoken.txt` 中。

## 阿里 RefreshToken
也称为 OpenToken，获取方式：
1. 打开 <https://alist.nn.ci/tool/aliyundrive/request.html>
2. 点击扫描二维码
3. 使用阿里云盘扫描二维码
4. 点击我已扫描，就可得到 RefreshToken

保存此 RefreshToken 到 `/etc/xiaoya/myopentoken.txt`

## 阿里 folder id
1. 登录阿里云盘网页版
2. 在备份盘中新建一个目录，用于临时转存播放时的文件，可清理。也可选择其他目录，如资源盘，但建议选备份盘，据说速度会快一些
3. 转存 <https://www.alipan.com/s/rP9gP3h9asE> 到上面新建的目录中
4. 打开刚才新建的目录，URL 的最后一部分就是 folder id

保存此 folder id 到 `/etc/xiaoya/temp_transfer_folder_id.txt`

# 部署
部署时，默认是 HTTP，需要修改下，不导出到公网，如 `-p '127.0.0.1:5678:80'`，然后使用 caddy 提供 HTTPS 访问。

# 参考
- [如何设置xiaoya的docker](https://xiaoyaliu.notion.site/xiaoya-docker-69404af849504fa5bcf9f2dd5ecaa75f)
- [AlistTvBox README](https://github.com/power721/alist-tvbox/blob/master/doc/README_zh.md)
- [小雅Xiaoya TVbox/Jellyfin/EMBY 进阶之单独安装 Docker /群晖 独家保姆级教程](https://zhuanlan.zhihu.com/p/673584505)
- [Alist小雅超集Docker搭建排坑指南](https://399s.com/7565.html)
