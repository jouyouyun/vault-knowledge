---
title: Nomao 美国队长7 US 性能测试
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

Mon 25 Mar 2024 04:59:53 PM CST

Basic System Information:
---------------------------------
Uptime     : 0 days, 0 hours, 12 minutes
Processor  : AMD Phenom(tm) II X4 965 Processor
CPU cores  : 2 @ 3400.000 MHz
AES-NI     : ❌ Disabled
VM-x/AMD-V : ✔ Enabled
RAM        : 2.0 GiB
Swap       : 512.0 MiB
Disk       : 9.7 GiB
Distro     : Debian GNU/Linux 11 (bullseye)
Kernel     : 5.15.107-2-pve
VM Type    : LXC
IPv4/IPv6  : ✔ Online / ✔ Online

IPv6 Network Information:
---------------------------------
ISP        : Nocix, LLC
ASN        : AS33387 Nocix, LLC
Host       : Nocix, LLC
Location   : North Kansas City, Missouri (MO)
Country    : United States

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/mapper/vg1-vm--1044--d2KFfNiRcK6CGndH--JC2cRZLFIPJaVPUz):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 237.00 KB/s     (59) | 3.90 MB/s       (61)
Write      | 255.00 KB/s     (63) | 4.18 MB/s       (65)
Total      | 492.00 KB/s    (122) | 8.09 MB/s      (126)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 23.31 MB/s      (45) | 28.00 MB/s      (27)
Write      | 24.55 MB/s      (47) | 30.82 MB/s      (30)
Total      | 47.86 MB/s      (92) | 58.82 MB/s      (57)

Running GB6 benchmark test... *cue elevator music*
---------------------------------
Test            | Value
                |
Single Core     | 419
Multi Core      | 741
Full Test       | https://browser.geekbench.com/v6/cpu/5458886

YABS completed in 21 min 29 sec
```
