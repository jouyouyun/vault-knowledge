---
title: Lolipa Cloud US 性能测试
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
$ curl -sL yabs.sh | bash -s -- -i -5
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-01-01                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Sun Feb  4 02:26:21 PM CST 2024

Basic System Information:
---------------------------------
Uptime     : 20 days, 4 hours, 28 minutes
Processor  : Intel(R) Xeon(R) Gold 6133 CPU @ 2.50GHz
CPU cores  : 2 @ 2494.138 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 1.9 GiB
Swap       : 0.0 KiB
Disk       : 99.9 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.1.68-x64v3-xanmod1
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv4 Network Information:
---------------------------------
ISP        : Arosscloud Inc.
ASN        : AS400619 AROSSCLOUD INC.
Host       : PEG Tech Inc
Location   : San Jose, California (CA)
Country    : United States

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/vda1):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 44.98 MB/s   (11.2k) | 185.56 MB/s   (2.8k)
Write      | 45.04 MB/s   (11.2k) | 186.53 MB/s   (2.9k)
Total      | 90.02 MB/s   (22.5k) | 372.09 MB/s   (5.8k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 242.19 MB/s    (473) | 286.33 MB/s    (279)
Write      | 255.05 MB/s    (498) | 305.40 MB/s    (298)
Total      | 497.24 MB/s    (971) | 591.74 MB/s    (577)

Running GB5 benchmark test... *cue elevator music*
Geekbench 5 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 293
Multi Core      | 565
Full Test       | https://browser.geekbench.com/v5/cpu/22198331

YABS completed in 7 min 52 sec
```
