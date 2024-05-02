---
title: Oracle Dubai ARM 性能测试
data: 2024-05-02T20:13:11+08:00
categories:
  - note
tags:
  - note
  - yabs
  - fio
  - geekbench
  - oracle
  - dubai
  - arm
draft: false
---
## YABS

``` shell
$ bash ./yabs.sh -b -f -6
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-04-22                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Thu May  2 12:17:37 UTC 2024

ARM compatibility is considered *experimental*

Basic System Information:
---------------------------------
Uptime     : 1 days, 3 hours, 26 minutes
Processor  : Neoverse-N1
CPU cores  : 4 @ ??? MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ❌ Disabled
RAM        : 23.4 GiB
Swap       : 0.0 KiB
Disk       : 116.2 GiB
Distro     : Ubuntu 22.04.4 LTS
Kernel     : 6.5.0-1019-oracle
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
Read       | 15.03 MB/s    (3.7k) | 30.98 MB/s     (484)
Write      | 15.04 MB/s    (3.7k) | 31.90 MB/s     (498)
Total      | 30.08 MB/s    (7.5k) | 62.89 MB/s     (982)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 28.47 MB/s      (55) | 28.07 MB/s      (27)
Write      | 30.90 MB/s      (60) | 31.32 MB/s      (30)
Total      | 59.37 MB/s     (115) | 59.39 MB/s      (57)

iperf3 Network Speed Tests (IPv4):
---------------------------------
Provider        | Location (Link)           | Send Speed      | Recv Speed      | Ping
-----           | -----                     | ----            | ----            | ----
Clouvider       | London, UK (10G)          | 1.28 Gbits/sec  | 721 Mbits/sec   | 124 ms
Eranium         | Amsterdam, NL (100G)      | 1.32 Gbits/sec  | 1.51 Gbits/sec  | 123 ms
Telia           | Helsinki, FI (10G)        | busy            | busy            | 137 ms
Uztelecom       | Tashkent, UZ (10G)        | 712 Mbits/sec   | 737 Mbits/sec   | 211 ms
Leaseweb        | Singapore, SG (10G)       | 1.37 Gbits/sec  | 1.61 Gbits/sec  | 225 ms
Clouvider       | Los Angeles, CA, US (10G) | 496 Mbits/sec   | 429 Mbits/sec   | 251 ms
Leaseweb        | NYC, NY, US (10G)         | 788 Mbits/sec   | 909 Mbits/sec   | 189 ms
Edgoo           | Sao Paulo, BR (1G)        | 413 Mbits/sec   | 430 Mbits/sec   | 324 ms

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 1094
Multi Core      | 2542
Full Test       | https://browser.geekbench.com/v6/cpu/5937611

YABS completed in 15 min 31 sec
```
