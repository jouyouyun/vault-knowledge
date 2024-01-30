---
title: 日志型文件系统 - 原理和优化
data: 2024-01-25T17:15:10+08:00
categories:
  - favorite
tags:
  - favorite
  - webclipper
  - filesystem
  - fs
  - journal
  - ext4
  - ext3
  - ext2
origin-link: https://zhuanlan.zhihu.com/p/107558961
draft: false
---
**说明：** 本文示例基本来自[CRASHCONSISTENCY: FSCKANDJOURNALING](https://link.zhihu.com/?target=http%3A//pages.cs.wisc.edu/~remzi/OSTEP/file-journaling.pdf)。

位于磁盘上的[文件系统](https://zhuanlan.zhihu.com/p/106459445)需要面临的一个问题是：当系统crash或者意外掉电的时候，如何维持数据的一致性。比如现在为了完成某项功能，需要同时更新文件系统中的两个数据结构A和B，因为磁盘每次只能响应一次读写请求，势必造成A和B其中一者的更新先被磁盘接收处理。如果正好在其中一个更新完成后发生了掉电或者crash，那么就会造成不一致的状态。

假设一个文件系统在磁盘上的分布是这样的：其中包括一个文件的inode（即I\[v1\]），data block（即Da）和记录分配状态的inode bitmap, data bitmap。

![](https://pic1.zhimg.com/v2-0d1621b2e45ee4a48f2e02cb7f9b3ba8_b.jpg)

现在要在文件末尾追加一部分内容，那么需要分配一个新的data block（即Db），这会导致inode和data bitmap都发生相应的变化。

![](https://pic4.zhimg.com/v2-f26dfcb482ef977fc5ff81480b38266f_b.jpg)

因此需要对磁盘的三个不同位置（3个blocks）分别进行写操作，如果在这三次写操作的间隙发生了crash，那么可能出现若干种情况：只更新了其中一个部分，或者只更新了其中的两个部分。

传统的Unix系统对此的解决办法是使用fsck工具，但是fsck通常需要在发生crash重启后，扫描整个磁盘，繁琐且速度较慢，因此目前更流行的做法是使用日志（**Journaling**）。

## 原理

日志型文件系统的基本思想是这样的：在真正更新磁盘上的数据之前，先往磁盘上写入一些信息，这些信息主要是描述接下来要更新什么，相当于 **wrtie ahead**，因此这种方式又被称为write-ahead logging。

这样，即便发生crash，也可通过记录的日志信息，回溯并恢复crash前正在进行的操作（称为**replay**）。在更新的时候增加一点额外的操作，换来了recovery时所需的工作量的减少。

在Linux中，ext2文件系统将磁盘划分成若干个block groups，每个block group包含一个inode bitmap, data bitmap, inodes和若干个data blocks，它没有使用日志。

![](https://pic3.zhimg.com/v2-2815b51c09d65b8aa50b604409709e7e_b.png)

而在ext3文件系统中，加入了对日志的支持，日志部分单独占据一块磁盘空间。

![](https://pic2.zhimg.com/v2-0341a084fee9daf8237093a781776845_b.png)

还是前面那个例子，现在我们要更新磁盘上的inode（I\[v2\]）, bitmap（B\[v2\]）和data block （Db）。在更新这个数据之前，我们把这个更新操作的步骤（称为一个 **transcation**），加上分别表示这个transcation开始和结束的TxB和TxE，写入磁盘。

Transaction的概念起源于数据库领域，它有助于在操作未能完成的情况下保证数据的一致性。同一transcation的TxB和TxE具有相同的sequence number。

![](https://pic4.zhimg.com/v2-610cee7e2e5d28b1484ffd19c283724b_b.png)

在一个记录步骤信息的transcation完成之后，才真正地更新磁盘上的文件数据（这一步称为**checkpoint**）。

Journal是为了保证数据一致性的，而journal本身也是由多个部分组成，那在写入journal的过程中发生了crash怎么办？由于I/O scheduling等原因，各个部分不一定是按照提交的顺序写入的（out of order），那么可能出现下图所示的这种情况：缺少了Db，但TxB和TxE都存在，系统恢复的时候会误以为这是一个完整的transcation。

![](https://pic4.zhimg.com/v2-e5655c491410121e939d313582cef477_b.png)

解决的办法是使用**write barrier**。写入除TxE以外的部分后（这一步叫做 **journal write**），执行一次barrier操作（对于支持journal checksum的ext4文件系统，此步骤可省略）。如果在此期间crash了，由于没有TxE，这个transcation会被认为是不完整的，重启后就不会试图恢复这个transcation所代表的步骤。

![](https://pic3.zhimg.com/v2-a752af2b794d9a74641d06f9d9c53d42_b.png)

待journal write顺利完成以后，再写入TxE（这一步叫做 **journal commit**），然后再执行一次barrier操作，以保证数据的写入是发生在journal完成之后（write barrier在保证数据一一致性的同时，会不可避免地对性能造成影响，如果能够接受不使用barrier带来的潜在风险，可以在[mount文件系统](https://zhuanlan.zhihu.com/p/144893220)的时候使用"nobarrier"选项）。

![](https://pic2.zhimg.com/v2-bd63060e028328e458000e3f02821a7d_b.png)

因此，在日志型文件系统中，一个完整的数据写操作由"journal write"，"journal commit"和"checkpoint"三部分组成。写操作完成后，就可以释放journal本身所占据的磁盘空间了。

![](https://pic1.zhimg.com/v2-02e661cc552def6708c125fa3c13a344_b.jpg)

至此，涉及多个block的写操作的数据一致性问题算是有了保证，但这基于的是一个block的数据要么完全写了，要么完全没写的前提，那会不会出现一个block的数据只写了一部分的情况呢（half-written）？这其实是一个**原子性**的问题，由磁盘本身提供保障，即对一个block的操作必须是原子的（不可分割的）。

## 优化

journal好是好，不过要多做的工作也是显而易见的，别的不说，磁盘的I/O负载首先就会蹭蹭地上升。那有什么办法可以优化吗？（此处的「优化」是真的优化哈，不是公司裁人的那种所谓“优化”）。

一种比较容易想到的方法是将多个journal的操作进行聚合处理，这种batching的思想在软件设计中也是随处可见的。结合到日志型文件系统自身的过程，还可以将journal中对"Db"的记录移除，即journal中只包含对inode和bitmap更新的记录。

![](https://pic2.zhimg.com/v2-fccba8c55b41fa08d0f581e4b716b325_b.png)

此时，write barrier只能保证对meta data（包括inode和bitmap）的真实写操作发生在journal的写操作之后，由于磁盘写操作的out of order特性，user data的真实写操作则可能发生在此过程中的任何节点，这种模式被称为"**Writeback**"。

允许out of order对性能的提升当然是有裨益的，但如果crash是发生在journal写操作之后，meta data的真实写操作之前（假设也在user data的写操作之前），那么进行文件系统的recover时，meta data的写操作会被replay，但是user data的写操作不会，这将有可能造成同一文件的meta data和user data的不一致（图1左侧部分）。

![](https://pic1.zhimg.com/v2-76c1b29f6675ea1e888018c85c10fd78_b.jpg)

图 1

不过呢，Writeback模式相对完全不用journal的模式，造成不一致的概率降低了，不一致带来的危害也降低了，作为性能和稳健的平衡，还是有相当的可取之处的。如果想把稳健性再提高一点呢，就再多做出一点限制：即保证对user data的真实写操作发生在journal的写操作之前，这就是"**Ordered**"模式（图1中间部分）。

对于Ordered的模式，如果crash发生在user data的写操作之后，journal的写操作之前，那么将造成这一部分user data的丢失，不过不会造成不一致的问题。如果crash发生在journal的写操作之后，meta data的真实写操作之前，那么完全可以通过replay来还原。

可见，从"Writeback"模式，到"Ordered"模式（同为**Metadata Journal**），再到本文最开始介绍的基本模式（**Data Journal**），数据丢失和不一致的风险是依次降低的，而对性能的损耗则是依次升高的。现代的文件系统通常会提供多种模式的选择，供不同场景下的用户使用。

## 参考

-   [日志文件系统是怎样工作的](https://link.zhihu.com/?target=http%3A//linuxperf.com/%3Fp%3D153)
-   [https://en.wikipedia.org/wiki/Ext3](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Ext3)
-   [https://en.wikipedia.org/wiki/Journaling\_file\_system](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Journaling_file_system)
-   [Barriers and journaling filesystems](https://link.zhihu.com/?target=https%3A//lwn.net/Articles/283161/)
