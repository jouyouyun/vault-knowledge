---
title: Horor MagicBook NBLK-WAX9X 性能测试
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

2024年 03月 25日 星期一 13:45:09 CST

Basic System Information:
---------------------------------
Uptime     : 0 days, 3 hours, 14 minutes
Processor  : AMD Ryzen 7 3700U with Radeon Vega Mobile Gfx
CPU cores  : 8 @ 3621.385 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 14.6 GiB
Swap       : 8.0 GiB
Disk       : 419.7 GiB
Distro     : Deepin 23
Kernel     : 6.1.32-amd64-desktop-hwe
VM Type    : NONE
IPv4/IPv6  : ✔ Online / ❌ Offline

IPv4 Network Information:
---------------------------------
ISP        : Chinanet
ASN        : AS4134 CHINANET-BACKBONE
Host       : Chinanet HB-WH
Location   : Wuhan, Hubei (HB)
Country    : China

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/nvme0n1p7):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ---
Read       | 542.44 MB/s (135.6k) | 933.35 MB/s  (14.5k)
Write      | 543.87 MB/s (135.9k) | 938.26 MB/s  (14.6k)
Total      | 1.08 GB/s   (271.5k) | 1.87 GB/s    (29.2k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 738.83 MB/s   (1.4k) | 790.33 MB/s    (771)
Write      | 778.09 MB/s   (1.5k) | 842.96 MB/s    (823)
Total      | 1.51 GB/s     (2.9k) | 1.63 GB/s     (1.5k)

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 1088
Multi Core      | 3190
Full Test       | https://browser.geekbench.com/v6/cpu/5457219

YABS completed in 11 min 19 sec
```
