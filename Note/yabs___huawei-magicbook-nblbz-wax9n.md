---
title: Huawei MagicBook NBLBZ-WAX9N 性能测试
data: 2024-03-25T10:13:11+08:00
categories:
  - note
tags:
  - note
  - yabs
  - fio
  - geekbench
draft: false
---
## YABS

``` shell
$ sudo bash ./yabs.sh -b -i -6
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-03-05                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

2024年 03月 25日 星期一 13:16:37 CST

Basic System Information:
---------------------------------
Uptime     : 0 days, 0 hours, 10 minutes
Processor  : Intel(R) Core(TM) i5-10210U CPU @ 1.60GHz
CPU cores  : 8 @ 3097.445 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 7.5 GiB
Swap       : 7.8 GiB
Disk       : 186.2 GiB
Distro     : UOS Desktop 20 Pro
Kernel     : 5.10.0-amd64-desktop
VM Type    : NONE
IPv4/IPv6  : ✔ Online / ❌ Offline

IPv6 Network Information:
---------------------------------
ISP        : Cloudflare, Inc.
ASN        : AS13335 Cloudflare, Inc.
Host       : Cloudflare WARP
Location   : Kassel, Hesse (HE)
Country    : Germany

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/nvme0n1p5):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 252.30 MB/s  (63.0k) | 185.01 MB/s   (2.8k)
Write      | 252.97 MB/s  (63.2k) | 185.99 MB/s   (2.9k)
Total      | 505.27 MB/s (126.3k) | 371.01 MB/s   (5.7k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 212.26 MB/s    (414) | 237.32 MB/s    (231)
Write      | 223.54 MB/s    (436) | 253.12 MB/s    (247)
Total      | 435.81 MB/s    (850) | 490.44 MB/s    (478)

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 1410
Multi Core      | 4046
Full Test       | https://browser.geekbench.com/v6/cpu/5457006

YABS completed in 9 min 50 sec
```
