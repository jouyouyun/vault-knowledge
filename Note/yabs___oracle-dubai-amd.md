---
title: Oracle Dubai AMD 性能测试
data: 2024-04-20T20:13:11+08:00
categories:
  - note
tags:
  - note
  - yabs
  - fio
  - geekbench
  - oracle
  - dubai
draft: false
---
## YABS

``` shell
$ bash ./yabs.sh -i -b -4
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-03-05                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Sat Apr 20 12:16:59 UTC 2024

Basic System Information:
---------------------------------
Uptime     : 0 days, 0 hours, 5 minutes
Processor  : AMD EPYC 7551 32-Core Processor
CPU cores  : 2 @ 1996.249 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 964.4 MiB
Swap       : 0.0 KiB
Disk       : 45.1 GiB
Distro     : Ubuntu 22.04.3 LTS
Kernel     : 6.6.28-x64v3-xanmod1
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ❌ Offline

IPv4 Network Information:
---------------------------------
ISP        : Oracle Corporation
ASN        : AS31898 Oracle Corporation
Host       : Oracle Cloud Infrastructure (me-dubai-1)
Location   : Dubai, Dubai (DU)
Country    : United Arab Emirates

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/sda1):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 6.31 MB/s     (1.5k) | 26.26 MB/s     (410)
Write      | 6.31 MB/s     (1.5k) | 26.72 MB/s     (417)
Total      | 12.62 MB/s    (3.1k) | 52.98 MB/s     (827)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 24.64 MB/s      (48) | 24.18 MB/s      (23)
Write      | 26.00 MB/s      (50) | 26.69 MB/s      (26)
Total      | 50.65 MB/s      (98) | 50.87 MB/s      (49)

Geekbench 4 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 1618
Multi Core      | 1508
Full Test       | https://browser.geekbench.com/v4/cpu/17504028

YABS completed in 10 min 13 sec
```
