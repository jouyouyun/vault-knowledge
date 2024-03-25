---
title: 不可变系统调研
data: 2024-03-25T10:12:11+08:00
categories:
  - note
tags:
  - note
  - research
  - macos
  - android
  - immutable
  - concept
draft: false
---
# 概念
对于下一代操作系统，不可变系统通常是其中的重要特性，那不可变系统到底是什么了？

- 最小(Minimal): 组成系统的组件是完成系统所需功能的最小集合
- 整体(Monolithic): 系统由一个单体组成，其更新、备份、回滚等通过整体替换完成。如 image-based
- 不可变(Immutable): 系统的核心基础是只读的，用户无法篡改。日志、配置、数据和用户目录除外
- 分层(Layered): 将系统与应用、用户等分层，不可变的为系统底层，上层可构建可变的应用和用户层

<!--more-->

不可变系统让系统的发布、更新、回滚、备份等变得简单，但由于其只读，需要应用遵循规范，获将配置和数据与系统分离，才能保证应用的功能正常。

同时对兼容性的要求有了更高的要求，因为应用是基于不可变系统的，所以不同版本的系统需要保持兼容。

# 实现
## MacOS
APFS 是一种新的文件系统，实现了写时复制，可用来实现快照。MacOS 基于此特性将磁盘分为了系统卷和数据卷，将不可变数据和可变数据进行隔离。

MacOS 默认将系统卷挂载为只读，并通过 SIP 来实现系统卷在更新时可写。并与安全启动结合，使用密封机制验证系统卷的完整性。

系统在更新时，首先卸载数据卷，以避免受到更新失败的影响，然后禁用 SIP 并挂载系统卷为读写，进行系统更新，完成后，启用 SIP 并挂载数据卷。
系统的更新实质是创建新的快照，生成密封签名，并修改引导，下次启动时从新的快照中引导。

MacOS 使用 firmlinked 将数据卷和系统卷的文件拼接在一起，但具体技术不明。

## Android
Android 将系统的启动镜像存放在一个单独的只读文件系统上，如 EROFS ，以此实现系统的只读，对于应用则存放在分离的数据分区上。

更新时采用动态分区和AB分区的方式完成更新，首先创建B分区，将更新数据写入B分区，然后修改引导，下次启动从B分区启动。

但 Android 和 MacOS 类似，都是针对特定设备的，一般不需要额外安装驱动等，并提供了统一的 SDK 用于应用访问系统资源。

# 参考链接
- [The future is minimal and immutable - a new generation of operating systems](https://sonalake.com/latest/the-future-is-minimal-and-immutable-a-new-generation-of-operating-systems/)
- [How macOS is more reliable, and doesn’t need reinstalling](https://eclecticlight.co/2021/10/29/how-macos-is-more-reliable-and-doesnt-need-reinstalling/)
- [How macOS depends on firmlinks](https://eclecticlight.co/2023/07/22/how-macos-depends-on-firmlinks/)
- [Android 分区](https://source.android.com/docs/core/architecture/partitions?hl=zh-cn)
- [Android 动态分区](https://source.android.google.cn/devices/tech/ota/dynamic_partitions/implement?hl=zh-cn#adb-remount)
- [Understanding Immutable Linux OS: Benefits, Architecture, and Challenges](https://kairos.io/blog/2023/03/22/understanding-immutable-linux-os-benefits-architecture-and-challenges/)
- [MacOS Security & Privacy Guide](https://github.com/drduh/macOS-Security-and-Privacy-Guide)
