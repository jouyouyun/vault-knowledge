---
title: YABS 介绍
data: 2024-03-25T10:13:11+08:00
categories:
  - note
tags:
  - note
  - yabs
draft: false
---
[YABS](https://github.com/masonr/yet-another-bench-script) 是一个测试 Disk、Network、CPU 等性能的脚本集合。使用：
- fio 测试 Disk 性能
- iperf3 测试 Network 性能
- Geekbench 测试 CPU 性能

<!--more-->

执行 `wget yabs.sh -O yabs.sh` 获取测试脚本，通过 `-h` 参数查看使用方法。如：
- 测试所有项: `sudo bash yabs.sh -b`
- 测试 Disk 和 CPU(Geekbench 6): `sudo bash yabs.sh -b -i -6`

# 测试记录
## Huawei MagicBook NBLBZ-WAX9N
![yabs___huawei-magicbook-nblbz-wax9n](yabs___huawei-magicbook-nblbz-wax9n.md)
## HONOR MagicBook NBLK-WAX9X
![yabs___honor-mgicbook-nblk-wax9x](yabs___honor-mgicbook-nblk-wax9x.md)
## Viie Cloud DE
![yabs___idc-viie-de](yabs___idc-viie-de.md)
## Kuxueyun DE
![yabs___idc-kuxueyun-de](yabs___idc-kuxueyun-de.md)
## TuanCloud LAX GIA
![yabs___idc-tuancloud-lax](yabs___idc-tuancloud-lax.md)
## Lolipa Cloud US
![yabs___idc-lolipa-us](yabs___idc-lolipa-us.md)
## Hostyun HK MG
![yabs___idc-hostyun-hk-mg](yabs___idc-hostyun-hk-mg.md)
## Nomao 美国队长7 US
![yabs___idc-nomao-us7](yabs___idc-nomao-us.md)
