---
title: ""
data: "2024-01-22T13:45:10+08:00"
categories:
  - favorite
tags:
  - favorite
  - link
  - github
  - gfw
draft: false
---
此文档用于收集感兴趣的链接或项目，以备不时之需。

## Tutorial
- [Learn Rust the Dangerous Way](https://cliffle.com/p/dangerust/)  #rust #tutorial #danger
- [RunningCode](https://code.gpthanghai.com)  #algorithm #leetcode
  - summary: 高频算法题集合，可按题目类型、出处、难易程度进行检索，数据基于公开数据人工整理 + GPT 进行打标签。
  - project: <https://github.com/tonyljx/running-leetcode>
- [线性代数：未竟之美_江大学竺可桢学院辅学讲义](https://github.com/yhwu-is/Linear-Algebra-Left-Undone) #数学 #线性代数 #tutorial
  - features
    - 本讲义兼具教材、笔记、复习提纲等多种功能
    - 本讲义提供了丰富的例题和习题，几乎能覆盖到所有重要的概念、定理和方法，同时我们也为这些题目提供了详细的解答
    - 在本讲义的编排过程中，我们摒弃了传统的讲授思路. 首先我们选择《大学数学：代数与几何》以及《线性代数应该这样学》作为参考教材，它们都是从抽象空间出发研究的，相比于一般的线性代数或高等代数教材更能深入本质
- [Linux 内核教学](https://linux-kernel-labs-zh.xyz)
  - features
    - 本文档包含一系列 Linux 内核主题的课程和实验。课程侧重于理论和 Linux 内核探索。
    - 实验侧重于设备驱动程序主题，文档风格类似“howto”。每个主题分两部分：
      - 主题概述，包含概述、主要抽象概念、简单示例和对 API 的指引。
      - 实践部分，包含几个应由学生解决的练习；为了使学生专注于当下的主题，学生会得到一个起始编码框架和深入的解决练习的技巧提示。
    - 本文档内容基于布加勒斯特理工大学自动控制与计算机学院计算机科学与工程系的 [“操作系统 2”](http://ocw.cs.pub.ro/courses/so2) 课程。
    - 你可以在 https://github.com/linux-kernel-labs-zh 获取最新版本。
## Projects
- [OpenGFW](https://github.com/apernet/OpenGFW)  #GFW #firewall #protocol #expr #offloading
  - summary: 一个 Linux 上灵活、易用、开源的 GFW 实现，并且在许多方面比真正的 GFW 更强大。可以部署在家用路由器上的网络主权。
  - features
    - 完整的 IP/TCP 重组，各种协议解析器
      - HTTP, TLS, DNS, SSH, 更多协议正在开发中
      - Shadowsocks 等 "全加密流量" 检测
      - 基于 Trojan-killer 的 Trojan 检测
    - 同等支持 IPv4 和 IPv6
    - 基于流的多核负载均衡
    - 连接 offloading
    - 基于 expr 的强大规则引擎
    - 灵活的协议解析和修改框架
    - 可扩展的 IO 实现 (目前只有 NFQueue)
  - Use cases
    - 广告拦截
    - 家长控制
    - 恶意软件防护
    - VPN/代理服务滥用防护
    - 流量分析 (纯日志模式)
  - note: 本项目仍处于早期开发阶段。测试时自行承担风险。(2024-01-22)
- <https://github.com/louislam/dockge>
  - summary: 简单易用的docker compose管理平台，支持自建，提供 WebUI

## 文学
- [在线古诗词-温故](https://shici.honmaple.com/)  #online #poetry #shici #古诗 #诗词 #温故
  - summary: 一个在线的古诗词网站，可按照诗集、诗词、诗人等唯独展示
  - features
    - 基于 [chinese-poetry](https://github.com/chinese-poetry/chinese-poetry) 实现，一个中华古典文集数据库

## 英语
- [唐库学习](http://www.tanglib.com)  #english #tutorial
  - summary: 世界名著在线阅读学英语，英汉对照，双语逐句朗读，适合影子跟读法学习。

## Resume
- [A LATEX Based Resume Builder](https://ppresume.com)  #resume #latex #pdf #chinese
  - summary: 基于latex的简历生成网站，能够生成高质量的 pdf，并且支持中文！
