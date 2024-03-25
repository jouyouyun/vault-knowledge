---
title: Tuancloud LAX 性能测试
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

Sat Feb  3 10:24:41 PM PST 2024

Basic System Information:
---------------------------------
Uptime     : 2 days, 23 hours, 30 minutes
Processor  : Intel Xeon Processor (Skylake, IBRS)
CPU cores  : 1 @ 2500.000 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 926.1 MiB
Swap       : 0.0 KiB
Disk       : 9.9 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.1.0-9-amd64
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv6 Network Information:
---------------------------------
ISP        : Kurun Cloud Inc
ASN        : Unknown
Host       : Kurun Cloud Inc
Location   : Los Angeles, California (CA)
Country    : United States

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/vda3):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 194.73 MB/s  (48.6k) | 1.10 GB/s    (17.2k)
Write      | 195.25 MB/s  (48.8k) | 1.11 GB/s    (17.3k)
Total      | 389.98 MB/s  (97.4k) | 2.21 GB/s    (34.6k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 1.19 GB/s     (2.3k) | 1.24 GB/s     (1.2k)
Write      | 1.26 GB/s     (2.4k) | 1.33 GB/s     (1.2k)
Total      | 2.45 GB/s     (4.8k) | 2.57 GB/s     (2.5k)

Geekbench 4 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 2695
Multi Core      | 2530
Full Test       | https://browser.geekbench.com/v4/cpu/17230042

YABS completed in 3 min 38 sec
```
