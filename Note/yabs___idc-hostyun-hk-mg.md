---
title: Hostyun HK MG 性能测试
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
$ curl -sL yabs.sh | bash -s -- -i -4
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-01-01                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Sunday, February 04, 2024 PM02:34:32 HKT

Basic System Information:
---------------------------------
Uptime     : 65 days, 16 hours, 25 minutes
Processor  : Common KVM processor
CPU cores  : 1 @ 2894.562 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ❌ Disabled
RAM        : 961.0 MiB
Swap       : 0.0 KiB
Disk       : 9.8 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.1.0-10-amd64
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv4 Network Information:
---------------------------------
ISP        : Xnnet LLC
ASN        : AS932 XNNET LLC
Location   : Hong Kong, Central and Western District (HCW)
Country    : Hong Kong

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/vda1):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 38.84 MB/s    (9.7k) | 51.20 MB/s     (800)
Write      | 38.92 MB/s    (9.7k) | 51.52 MB/s     (805)
Total      | 77.76 MB/s   (19.4k) | 102.72 MB/s   (1.6k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 49.88 MB/s      (97) | 49.66 MB/s      (48)
Write      | 52.83 MB/s     (103) | 53.06 MB/s      (51)
Total      | 102.72 MB/s    (200) | 102.73 MB/s     (99)

Geekbench 4 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 4272
Multi Core      | 4005
Full Test       | https://browser.geekbench.com/v4/cpu/17230084

YABS completed in 5 min 24 sec
```
