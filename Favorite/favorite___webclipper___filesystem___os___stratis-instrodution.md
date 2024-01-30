---
title: Stratis 简介
data: 2024-01-25T16:15:10+08:00
categories:
  - favorite
tags:
  - favorite
  - webclipper
  - filesystem
  - fs
  - stratis
  - btrfs
  - zfs
  - lvm
origin-link: https://www.cnblogs.com/luxiaodai/p/14296883.html
draft: false
---
-   [在 Linux 中使用 Stratis 配置本地存储](https://www.cnblogs.com/luxiaodai/p/14296883.html#_label0)
-   [Stratis 从 ZFS、Btrfs 和 LVM 学到哪些](https://www.cnblogs.com/luxiaodai/p/14296883.html#_label1)

 [原文](https://linux.cn/article-9736-1.html)

## 在 Linux 中使用 Stratis 配置本地存储

___

### 概述

> 关注于易用性，Stratis 为桌面用户提供了一套强力的高级存储功能。
>
> 对桌面 Linux 用户而言，极少或仅在安装系统时配置本地存储。Linux 存储技术进展比较慢，以至于 20 年前的很多存储工具仍在今天广泛使用。但从那之后，存储技术已经提升了不少，我们为何不享受新特性带来的好处呢？
>
> 本文介绍 Startis，这是一个新项目，试图让所有 Linux 用户从存储技术进步中受益，适用场景可以是仅有一块 SSD 的单台笔记本，也可以是包含上百块硬盘的存储阵列。Linux 支持新特性，但由于缺乏易于使用的解决方案，使其没有被广泛采用。Stratis 的目标就是让 Linux 的高级存储特性更加可用。

### 简单可靠地使用高级存储特性

> Stratis 希望让如下三件事变得更加容易：存储初始化配置；后续变更；使用高级存储特性，包括快照snapshots、精简配置thin provisioning，甚至分层tiering。

### 一个卷管理文件系统

> Stratis 是一个卷管理文件系统volume-managing filesystem（VMF），类似于 [ZFS](https://en.wikipedia.org/wiki/ZFS) 和 [Btrfs](https://en.wikipedia.org/wiki/Btrfs)。它使用了存储“池”的核心思想，该思想被各种 VMF 和 形如 [LVM](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) 的独立卷管理器采用。使用一个或多个硬盘（或分区）创建存储池，然后在存储池中创建卷volume。与使用 [fdisk](https://en.wikipedia.org/wiki/Fdisk) 或 [GParted](https://gparted.org/) 执行的传统硬盘分区不同，存储池中的卷分布无需用户指定。
>
> VMF 更进一步与文件系统层结合起来。用户无需在卷上部署选取的文件系统，因为文件系统和卷已经被合并在一起，成为一个概念上的文件树（ZFS 称之为数据集dataset，Brtfs 称之为子卷subvolume，Stratis 称之为文件系统），文件数据位于存储池中，但文件大小仅受存储池整体容量限制。
>
> 换一个角度来看：正如文件系统对其中单个文件的真实存储块的实际位置做了一层抽象abstract，而 VMF 对存储池中单个文件系统的真实存储块的实际位置做了一层抽象。
>
> 基于存储池，我们可以启用其它有用的特性。特性中的一部分理所当然地来自典型的 VMF 实现implementation，例如文件系统快照，毕竟存储池中的多个文件系统可以共享物理数据块physical data block；冗余redundancy，分层，完整性integrity等其它特性也很符合逻辑，因为存储池是操作系统中管理所有文件系统上述特性的重要场所。
>
> 上述结果表明，相比独立的卷管理器和文件系统层，VMF 的搭建和管理更简单，启用高级存储特性也更容易。

### Stratis与 ZFS 和 Btrfs 有哪些不同

> 作为新项目，Stratis 可以从已有项目中吸取经验，我们将在[第二部分](https://opensource.com/article/18/4/stratis-lessons-learned)深入介绍 Stratis 采用了 ZFS、Brtfs 和 LVM 的哪些设计。总结一下，Stratis 与其不同之处来自于对功能特性支持的观察，来自于个人使用及计算机自动化运行方式的改变，以及来自于底层硬件的改变。
>
> 首先，Stratis 强调易用性和安全性。对个人用户而言，这很重要，毕竟他们与 Stratis 交互的时间间隔可能很长。如果交互不那么友好，尤其是有丢数据的可能性，大部分人宁愿放弃使用新特性，继续使用功能比较基础的文件系统。
>
> 第二，当前 API 和 DevOps 式Devops-style自动化的重要性远高于早些年。Stratis 提供了支持自动化的一流 API，这样人们可以直接通过自动化工具使用 Stratis。
>
> 第三，SSD 的容量和市场份额都已经显著提升。早期的文件系统中很多代码用于优化机械介质访问速度慢的问题，但对于基于闪存的介质，这些优化变得不那么重要。即使当存储池过大而不适合使用 SSD 的情况，仍可以考虑使用 SSD 充当缓存层caching tier，可以提供不错的性能提升。考虑到 SSD 的优良性能，Stratis 主要聚焦存储池设计方面的灵活性flexibility和可靠性reliability。
>
> 最后，与 ZFS 和 Btrfs 相比，Stratis 具有明显不一样的实现模型implementation model（我会在[第二部分](https://opensource.com/article/18/4/stratis-lessons-learned)进一步分析）。这意味着对 Stratis 而言，虽然一些功能较难实现，但一些功能较容易实现。这也加快了 Stratis 的开发进度。

## Stratis 从 ZFS、Btrfs 和 LVM 学到哪些

___

### 为何不使用已有解决方案

> 理由千差万别。先说说 [ZFS](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/ZFS)，它最初由 Sun Microsystems 为 Solaris （目前为 Oracle 所有）开发，后移植到 Linux。但 [CDDL](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Common_Development_and_Distribution_License) 协议授权的代码无法合并到 [GPL](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/GNU_General_Public_License) 协议授权的 Linux 源码树中。CDDL 与 GPLv2 是否真的不兼容有待讨论，但这种不确定性足以打消企业级 Linux 供应商采用并支持 ZFS 的积极性。
>
> [Btrfs](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Btrfs) 发展也很好，没有授权问题。它已经多年被很多用户列为“最佳文件系统”，但在稳定性和功能特性方面仍有待提高。
>
> 我们希望打破现状，解决已有方案的种种问题，这种渴望促成了 Stratis。

### Stratis如何与众不同

> ZFS 和 Btrfs 让我们知道一件事情，即编写一个内核支持的 VMF 文件系统需要花费极大的时间和精力，才能消除漏洞、增强稳定性。涉及核心数据时，提供正确性保证是必要的。如果 Stratis 也采用这种方案并从零开始的话，开发工作也需要十数年，这是无法接受的。
>
> 相反地，Stratis 采用 Linux 内核的其它一些已有特性：[device mapper](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Device_mapper) 子系统以及久经考验的高性能文件系统 [XFS](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/XFS)，其中前者被 LVM 用于提供 RAID、精简配置和其它块设备特性而广为人知。Stratis 将已有技术作为（技术架构中的）层来创建存储池，目标是通过集成为用户提供一个看似无缝的整体。

### Stratis从ZFS学到哪些

> 对很多用户而言，ZFS 影响了他们对下一代文件系统的预期。通过查看人们在互联网上关于 ZFS 的讨论，我们设定了 Stratis 的最初开发目标。ZFS 的设计思路也潜在地为我们指明应该避免哪些东西。例如，当挂载一个在其它主机上创建的存储池时，ZFS 需要一个“ 导入(import)”步骤。这样做出于某些原因，但每一种原因都似乎是 Stratis 需要解决的问题，无论是否采用同样的实现方式。
>
> 对于增加新硬盘或将已有硬盘替换为更大容量的硬盘，ZFS 有一些限制，尤其是存储池做了冗余配置的时候，这一点让我们不太满意。当然，这么设计也是有其原因的，但我们更愿意将其视为可以改进的空间。
>
> 最后，一旦掌握了 ZFS 的命令行工具，用户体验很好。我们希望让 Stratis 的命令行工具能够保持这种体验；同时，我们也很喜欢 ZFS 命令行工具的发展趋势，包括使用 位置参数(positional parameters)和控制每个命令需要的键盘输入量。
>
> （LCTT 译注：位置参数来自脚本，$n 代表第 n 个参数）

### Stratis从Btrfs学到哪些

> Btrfs 让我们满意的一点是，有单一的包含位置子命令的命令行工具。Btrfs 也将冗余（选择对应的 Btrfs profiles）视为存储池的特性之一。而且和 ZFS 相比实现方式更好理解，也允许增加甚至移除硬盘。
>
> （LCTT 译注：Btrfs profiles 包括 single/DUP 和 各种 RAID 等类型）
>
> 最后，通过了解 ZFS 和 Btrfs 共有的特性，例如快照的实现、对发送/接收的支持，让我们更好的抉择 Stratis 应该包括的特性。

### Stratis从LVM学到哪些

> 在 Stratis 设计阶段早期，我们仔细研究了 LVM。LVM 目前是 Linux device mapper (DM) 最主要的使用者；事实上，DM 就是由 LVM 的核心开发团队维护的。我们研究了将 LVM 真的作为 Stratis 其中一层的可能性，也使用 DM 做了实验，其中 Stratis 可以作为 对等角色(peer)直接与 LVM 打交道。我们参考了 LVM 的 磁盘元数据格式(on-disk metadata format)（也结合 ZFS 和 XFS 的相应格式），获取灵感并定义了 Stratis 的磁盘元数据格式。
>
> 在提到的项目中，LVM 与 Stratis 内在地有最多的共性，毕竟它们都使用 DM。不过从使用的角度来看，LVM 内在工作更加透明，为专业用户提供相当多的控制和选项，使其可以精确配置 卷组(volume group)（存储池）的 布局(layout)；但 Stratis 不采用这种方式。

### 多种多样的解决方案

> 基于自由和开源软件工作的明显好处在于，没有什么组件是不可替代的。包括内核在内的每个组成部分都是开源的，可以查看修改源代码，如果当前的软件不能满足用户需求可以用其它软件替换。新项目产生不一定意味着旧项目的终结，只要都有足够的（社区）支持，两者可以并行存在。
>
> 对于寻找一个不存在争议、简单易用、强大的本地存储管理解决方案的人而言，Stratis 是更好满足其需求的一种尝试。这意味着一种设计思路所做的抉择不一定对所有用户适用。考虑到用户的其它需求，另一种设计思路可能需要艰难的做出抉择。所有用户可以选择最适合其的工作的工具并从这种自由选择中受益。
