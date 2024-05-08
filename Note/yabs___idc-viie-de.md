---
title: Viie Cloud DE 性能测试
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
$ bash ./yabs.sh -b -i -6
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-03-05                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Mon Mar 25 05:39:19 AM GMT 2024

Basic System Information:
---------------------------------
Uptime     : 2 days, 4 hours, 24 minutes
Processor  : AMD Ryzen 9 7950X3D 16-Core Processor
CPU cores  : 8 @ 4192.100 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ❌ Disabled
RAM        : 7.7 GiB
Swap       : 0.0 KiB
Disk       : 99.9 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.6.22-x64v4-xanmod1
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv4 Network Information:
---------------------------------
ISP        : Cloudflare, Inc.
ASN        : AS13335 Cloudflare, Inc.
Host       : Cloudflare WARP
Location   : Bielefeld, North Rhine-Westphalia (NW)
Country    : Germany

Preparing system for disk tests...
---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 655.52 MB/s (163.8k) | 5.10 GB/s    (79.7k)
Write      | 657.25 MB/s (164.3k) | 5.12 GB/s    (80.1k)
Total      | 1.31 GB/s   (328.1k) | 10.23 GB/s  (159.8k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 5.85 GB/s    (11.4k) | 5.74 GB/s     (5.6k)
Write      | 6.16 GB/s    (12.0k) | 6.13 GB/s     (5.9k)
Total      | 12.01 GB/s   (23.4k) | 11.88 GB/s   (11.6k)

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 2492
Multi Core      | 11026
Full Test       | https://browser.geekbench.com/v6/cpu/5457127

YABS completed in 5 min 4 sec
```

---

重开后，12C-22G 的性能测试：

``` shell
$ bash ./yabs.sh -b -i -6
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2024-04-22                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Wed May  8 02:23:27 AM BST 2024

Basic System Information:
---------------------------------
Uptime     : 0 days, 1 hours, 4 minutes
Processor  : AMD EPYC-Milan Processor
CPU cores  : 12 @ 4192.096 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ❌ Disabled
RAM        : 21.9 GiB
Swap       : 0.0 KiB
Disk       : 180.9 GiB
Distro     : Debian GNU/Linux 12 (bookworm)
Kernel     : 6.1.0-9-amd64
VM Type    : KVM
IPv4/IPv6  : ✔ Online / ✔ Online

IPv6 Network Information:
---------------------------------
ISP        : Hetzner Online GmbH
ASN        : AS24940 Hetzner Online GmbH
Location   : Falkenstein, Saxony (SN)
Country    : Germany

---------------------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 567.73 MB/s (141.9k) | 2.95 GB/s    (46.0k)
Write      | 569.23 MB/s (142.3k) | 2.96 GB/s    (46.3k)
Total      | 1.13 GB/s   (284.2k) | 5.91 GB/s    (92.4k)
           |                      |
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ----
Read       | 3.14 GB/s     (6.1k) | 3.21 GB/s     (3.1k)
Write      | 3.30 GB/s     (6.4k) | 3.43 GB/s     (3.3k)
Total      | 6.45 GB/s    (12.6k) | 6.64 GB/s     (6.4k)

Geekbench 6 Benchmark Test:
---------------------------------
Test            | Value
                |
Single Core     | 2737
Multi Core      | 13368
Full Test       | https://browser.geekbench.com/v6/cpu/6004164

YABS completed in 4 min 56 sec
```
