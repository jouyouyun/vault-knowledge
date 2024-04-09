---
title: Lolipa 枣庄联通 NAT 性能测试
data: 2024-04-09T13:13:11+08:00
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
#                     v2024-03-05                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Tue Apr  9 01:20:05 PM CST 2024

Basic System Information:
---------------------------------
Uptime     : 0 days, 3 hours, 6 minutes
Processor  : Intel(R) Xeon(R) CPU E5-2680 v4 @ 2.40GHz
CPU cores  : 2 @ 2394.652 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 969.4 MiB
Swap       : 0.0 KiB
Disk       : 19.5 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.6.25-x64v3-xanmod1
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ❌ Offline

IPv4 Network Information:
---------------------------------
ISP        : CNC Group CHINA169 Shandong Province Network
ASN        : AS4837 CHINA UNICOM China169 Backbone
Location   : Weifang, Shandong (SD)
Country    : China

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/vda1):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 77.98 MB/s   (19.4k) | 87.57 MB/s    (1.3k)
Write      | 78.18 MB/s   (19.5k) | 88.03 MB/s    (1.3k)
Total      | 156.16 MB/s  (39.0k) | 175.61 MB/s   (2.7k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 155.36 MB/s    (303) | 192.92 MB/s    (188)
Write      | 163.61 MB/s    (319) | 205.77 MB/s    (200)
Total      | 318.98 MB/s    (622) | 398.69 MB/s    (388)

Geekbench 4 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 1780
Multi Core      | 3434
Full Test       | https://browser.geekbench.com/v4/cpu/17471553

YABS completed in 8 min 22 sec
```
