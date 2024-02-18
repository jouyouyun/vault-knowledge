---
title: 修改字体名称
data: 2024-02-18T14:13:44+08:00
categories:
  - note
tags:
  - node
  - tool
  - fonttools
  - noto
  - font
draft: false
---
在 deepin/ubuntu 中使用 wps office 时，经常遇到字体缺失的情况。一般有以下两种处理方式：
- 从 windows 中复制缺少的字体到 deepin/ubuntu ，但仅限个人使用，商业使用可能存在版权问题
- 修改开源的 noto fonts，将其中的字体名称修改为wps office可识别的名称

下面将主要介绍第二种方法。

<!--more-->

## 字体选择
noto fonts 提供了大量的字体，这里参考 windows 中字体的名称，选择修改以下字体的名称：
- Noto Sans CJK SC --> SimHei 黑体
  - <https://github.com/notofonts/noto-cjk/releases>
- Noto Serif CJK SC --> SimSun 宋体，NSimSun 新宋体
  - <https://github.com/notofonts/noto-cjk/releases>
- Noto Sans Kaithi --> SimKai，KaiTi 楷体
  - [Full - NotoSansKaithi-Regular.otf](https://notofonts.github.io/kaithi/)
- Noto Fangsong KSS Vertical --> SimFang，FangSong 仿宋
  - [Full - NotoFangsongKSSVertical-Regular.ttf](https://notofonts.github.io/khitan-small-script/)
- Noto Sans Lisu --> Lisu 隶书
  - [Full - NotoSansLisu-Regular.ttf](https://notofonts.github.io/lisu/)
## 字体修改
### fonttools
fonttools 是一个可对 opentype、truetype 等格式的字体进行修改的工具，可通过 pip 安装：
```shell
pip install fonttools
pip3 install fonttools
```
在 deepin v20 上，上述两条命令需要都执行。

将 ttf/otf 转换为可编辑文件：
```shell
fonttools ttx -v -o <output file>.ttx <input file>
```
将编辑后的文件转换为字体文件：
```shell
fonttools ttx -v -o <output file> <input file>.ttx
```
### 修改名称
编辑 ttx 文件，找到 langID 所在位置，将其下的
```xml
<namerecord nameID="1" platformID="3" platEncID="1" langID="0x409">
``` 
的值，修改为需要的英文名称，多个英文名称的添加多条 namerecord 记录。

修改
```xml
<namerecord nameID="4" platformID="3" platEncID="1" langID="0x409">
```
的值为英文全名。

修改
```xml
<namerecord nameID="1" platformID="3" platEncID="1" langID="0x804">
```
的值为中文名。

修改
```xml
<namerecord nameID="4" platformID="3" platEncID="1" langID="0x804">
```
的值为中文全名。
