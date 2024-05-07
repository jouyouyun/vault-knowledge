---
title: Diylink HK 观塘性能测试
data: 2024-05-07T11:17:11+08:00
categories:
  - note
tags:
  - note
  - yabs
  - fio
  - geekbench
  - diylink
  - hk
draft: false
---
## YABS

``` shell
$ bash ./yabs.sh -b -i -5
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-04-22                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Tue May  7 11:03:34 CST 2024

Basic System Information:
---------------------------------
Uptime     : 0 days, 14 hours, 16 minutes
Processor  : Intel Core Processor (Haswell, no TSX, IBRS)
CPU cores  : 1 @ 2499.990 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 969.7 MiB
Swap       : 0.0 KiB
Disk       : 24.6 GiB
Distro     : Debian GNU/Linux 11 (bullseye)
Kernel     : 6.6.30-x64v3-xanmod1
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv6 Network Information:
---------------------------------
ISP        : Cloudflare, Inc.
ASN        : AS13335 Cloudflare, Inc.
Host       : Cloudflare WARP
Location   : Hong Kong, Central and Western District (HCW)
Country    : Hong Kong

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/vda1):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 48.72 MB/s   (12.1k) | 107.05 MB/s   (1.6k)
Write      | 48.80 MB/s   (12.2k) | 107.61 MB/s   (1.6k)
Total      | 97.53 MB/s   (24.3k) | 214.67 MB/s   (3.3k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 99.35 MB/s     (194) | 99.04 MB/s      (96)
Write      | 104.63 MB/s    (204) | 105.63 MB/s    (103)
Total      | 203.98 MB/s    (398) | 204.67 MB/s    (199)

Geekbench 5 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 649
Multi Core      | 653
Full Test       | https://browser.geekbench.com/v5/cpu/22459466

YABS completed in 12 min 18 sec
```
