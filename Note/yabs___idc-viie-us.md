---
title: Viie Cloud US 性能测试
data: 2024-04-05T22:13:11+08:00
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
                                                    
Fri Apr  5 23:28:40 CST 2024                        

Basic System Information:
---------------------------------
Uptime     : 0 days, 0 hours, 17 minutes
Processor  : Intel Core Processor (Broadwell, IBRS)
CPU cores  : 16 @ 1995.379 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ❌ Disabled
RAM        : 31.3 GiB
Swap       : 0.0 KiB
Disk       : 129.9 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.1.0-9-amd64
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv6 Network Information:
---------------------------------
ISP        : System In Place
ASN        : AS398493 System In Place
Host       : System In Place
Location   : Brentwood, California (CA)
Country    : United States

fio Disk Speed Tests (Mixed R/W 50/50) (Partition /dev/vda3):
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ---- 
Read       | 178.94 MB/s  (44.7k) | 1.76 GB/s    (27.5k)
Write      | 179.41 MB/s  (44.8k) | 1.77 GB/s    (27.7k)
Total      | 358.36 MB/s  (89.5k) | 3.53 GB/s    (55.3k)
           |                      |                      
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ---- 
Read       | 1.61 GB/s     (3.1k) | 424.50 MB/s    (414)
Write      | 1.69 GB/s     (3.3k) | 452.77 MB/s    (442)
Total      | 3.31 GB/s     (6.4k) | 877.28 MB/s    (856)

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value                         
                |
Single Core     | 765                           
Multi Core      | 5249                          
Full Test       | https://browser.geekbench.com/v6/cpu/5607071

YABS completed in 9 min 20 sec
```
