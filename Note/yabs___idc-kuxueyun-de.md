---
title: Kuxuyun DE 性能测试
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
$ sudo bash ./yabs.sh  -b -i -6
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-01-01                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Sun Mar  3 09:37:10 AM EST 2024

Basic System Information:
---------------------------------
Uptime     : 0 days, 0 hours, 11 minutes
Processor  : AMD Ryzen 9 3900 12-Core Processor
CPU cores  : 4 @ 3099.998 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 3.8 GiB
Swap       : 0.0 KiB
Disk       : 24.5 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.1.0-9-amd64
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv6 Network Information:
---------------------------------
ISP        : Hetzner Online GmbH
ASN        : AS24940 Hetzner Online GmbH
Location   : Ehingen, Baden-Wurttemberg (BW)
Country    : Germany

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/sda1):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 335.96 MB/s  (83.9k) | 774.68 MB/s  (12.1k)
Write      | 336.84 MB/s  (84.2k) | 778.76 MB/s  (12.1k)
Total      | 672.81 MB/s (168.2k) | 1.55 GB/s    (24.2k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 618.86 MB/s   (1.2k) | 593.61 MB/s    (579)
Write      | 651.74 MB/s   (1.2k) | 633.14 MB/s    (618)
Total      | 1.27 GB/s     (2.4k) | 1.22 GB/s     (1.1k)

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 1747
Multi Core      | 5374
Full Test       | https://browser.geekbench.com/v6/cpu/5158035

YABS completed in 6 min 4 sec
```
