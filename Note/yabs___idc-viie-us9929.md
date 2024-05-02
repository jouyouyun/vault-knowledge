---
title: Viie Cloud LAX 9929 性能测试
data: 2024-05-02T22:17:11+08:00
categories:
  - note
tags:
  - note
  - yabs
  - fio
  - geekbench
  - viie
  - lax
  - 9929
draft: false
---
## YABS

``` shell
$ bash ./yabs.sh
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-04-22                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Thu May  2 07:20:13 PM CST 2024

Basic System Information:
---------------------------------
Uptime     : 0 days, 11 hours, 18 minutes
Processor  : Intel Xeon Processor (Skylake, IBRS)
CPU cores  : 4 @ 2499.998 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ❌ Disabled
RAM        : 7.7 GiB
Swap       : 0.0 KiB
Disk       : 78.2 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.6.29-x64v4-xanmod1
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ❌ Offline

IPv4 Network Information:
---------------------------------
ISP        : Cogent Communications
ASN        : AS8796 FASTNET DATA INC
Host       : Kurun Cloud Inc
Location   : Los Angeles, California (CA)
Country    : United States

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/vda1):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 80.10 MB/s   (20.0k) | 310.19 MB/s   (4.8k)
Write      | 80.31 MB/s   (20.0k) | 311.82 MB/s   (4.8k)
Total      | 160.41 MB/s  (40.1k) | 622.02 MB/s   (9.7k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 295.89 MB/s    (577) | 292.23 MB/s    (285)
Write      | 311.61 MB/s    (608) | 311.69 MB/s    (304)
Total      | 607.51 MB/s   (1.1k) | 603.93 MB/s    (589)

iperf3 Network Speed Tests (IPv4):
---------------------------------
Provider        | Location (Link)           | Send Speed      | Recv Speed      | Ping
-----           | -----                     | ----            | ----            | ----
Clouvider       | London, UK (10G)          | 258 Mbits/sec   | 296 Mbits/sec   | 138 ms
Eranium         | Amsterdam, NL (100G)      | 255 Mbits/sec   | 296 Mbits/sec   | 155 ms
Telia           | Helsinki, FI (10G)        | busy            | busy            | 172 ms
Uztelecom       | Tashkent, UZ (10G)        | 234 Mbits/sec   | 284 Mbits/sec   | 254 ms
Leaseweb        | Singapore, SG (10G)       | 52.9 Mbits/sec  | 281 Mbits/sec   | 169 ms
Clouvider       | Los Angeles, CA, US (10G) | 287 Mbits/sec   | 326 Mbits/sec   | 0.423 ms
Leaseweb        | NYC, NY, US (10G)         | 272 Mbits/sec   | busy            | 66.7 ms
Edgoo           | Sao Paulo, BR (1G)        | 230 Mbits/sec   | 288 Mbits/sec   | 217 ms

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 692
Multi Core      | 2140
Full Test       | https://browser.geekbench.com/v6/cpu/5937110

YABS completed in 16 min 16 sec
```
