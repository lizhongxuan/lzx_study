# Linux 内核 IO

## 全局概览
VFS 层： 数据是“文件流” (Stream of bytes)。

Page Cache 层： 数据是“内存页” (4KB Pages)。

Block Layer 层： 数据是“块 I/O 请求” (BIOs / Requests)。

NVMe Driver 层： 数据是“NVMe 命令” (NVMe Commands / PRPs)。

## 路径图解

用户态: write(data) -> VFS (找到 inode)。

内核态: Page Cache (数据拷贝到 RAM，标记 Dirty，write 调用返回)。

后台线程: 触发回写 -> Filesystem (计算 LBA)。

块层: 生成 BIO -> 进入 blk-mq (软队列) -> 合并请求 -> 映射到 硬队列。

驱动层: NVMe Driver 生成 Command -> 放入 SQ -> 敲响 Doorbell。

硬件层: SSD 控制器 DMA 读取数据 -> 落盘 -> 写 CQ -> 发 中断。

回调: 驱动处理中断 -> 释放 BIO -> 标记 Page Clean。



## 第一站：VFS
当你的程序调用 write(fd, buffer, size) 时，它并不关心底层是 NVMe SSD、机械硬盘还是网络文件系统。VFS 的作用就是屏蔽这些差异。

1. 系统调用 (System Call): 用户态发生 write 系统调用，陷入内核态，调用 sys_write。

2. 定位对象: VFS 根据文件描述符 (fd) 找到对应的 file 结构体，进而找到 dentry (目录项) 和 inode (索引节点)。inode 记录了文件属于哪个具体的文件系统（如 ext4, xfs）。

3. 通用接口: VFS 调用具体文件系统提供的 file_operations->write_iter 接口。

>  核心点： VFS 层只处理逻辑上的“写文件”，不知道也不关心物理磁盘扇区。

## 第二站：Page Cache (页缓存)
这是 Linux I/O 性能的关键。磁盘（即使是 NVMe）比起内存还是太慢了。

1. Buffered I/O (默认模式):

- VFS 将数据交给 Page Cache。

- 内核在内存中查找该文件对应的页（Page）。

- 命中: 直接修改内存中的页。

- 未命中: 分配新页，将数据写入。

- 标记脏页 (Dirty): 写入完成后，该页被标记为 Dirty。

- 立即返回: 注意！ 此时数据通常还没有真正写入 NVMe SSD。内核欺骗应用程序说“写完了”，然后让程序继续运行。这是为了性能。

2. 回写 (Writeback):

- 脏页会在内存中停留一段时间，直到：

  - 内存紧张。

  - 定时回写任务触发（kworker 线程）。

  - 用户手动调用 fsync()。

- 此时，数据才开始真正流向下一层。

> 例外：Direct I/O (O_DIRECT) 如果程序打开文件时使用了 O_DIRECT 标志，数据将绕过 Page Cache，直接进入 Block Layer。这常用于数据库（如 MySQL、PostgreSQL），因为它们更喜欢自己管理缓存。

## 第三站：File System
当 Page Cache 决定刷盘（或者 Direct I/O）时，数据还在“文件偏移量 (File Offset)”的概念里。 文件系统（如 ext4/xfs）介入，通过 iomap 或 buffer_head 将 (文件 ID, 偏移量) 翻译成 (逻辑块地址 LBA)。

## 第四站：Block Layer (块设备层) —— 调度与合并
这是 Linux I/O 栈中最复杂的区域之一，也是针对 NVMe 优化最大的地方。

1. 生成 BIO (Block I/O)
   文件系统将数据封装成一个 struct bio 结构。这是 Block Layer 的基本流通货币。一个 BIO 包含了：

- 目标设备。

- 物理扇区地址 (LBA)。

- 内存地址 (Page)。

- 数据大小。

2. blk-mq (Block Multi-Queue, 多队列机制)
> 在 HDD 时代，Block Layer 只有一个全局请求队列（Single Queue），还要加锁，这对高性能 NVMe 来说是巨大的瓶颈。 现代 Linux 针对 NVMe 使用 blk-mq：

- 软件队列 (Software Queues): 每个 CPU 核心（或一组核心）对应一个软件队列。提交 I/O 时不需要竞争全局锁，极大提高了并发度。

- 合并 (Merging): 如果 CPU 1 连续发出了几个对相邻磁盘扇区的写请求，Block Layer 会在这里把它们合并成一个大的请求。这减少了与硬件交互的次数。

- 硬件队列 (Hardware Queues): 软件队列的数据最终会映射到 NVMe 驱动提供的硬件队列中。

> 总结： bio 在这里被转换成 request，经过合并和简单的调度，放入分发队列。

## 第五站：NVMe Driver — 硬件交互
NVMe 协议本质上是建立在 PCIe 之上的，它完全摒弃了旧的 AHCI/SATA 交互方式。

1. 映射 (Mapping): NVMe 驱动从 blk-mq 的硬件队列中取出 request。

2. 构建命令: 驱动将 request 转换成 NVMe 硬件能看懂的 NVMe Command (64字节大小)。

3. DMA 准备: 驱动不需要把数据拷贝给 SSD（那太慢了）。它构建 PRP (Physical Region Page) 或 SGL (Scatter Gather List) 列表。这实际上是告诉 SSD：“数据在内存物理地址 0xABC...，你去那里自己拿。”

4. 入队 (Submission Queue - SQ): 驱动将 NVMe Command 写入内存中的 提交队列 (SQ)。

5. 敲门 (Doorbell): 驱动写一个特殊的 PCIe 寄存器（Doorbell Register），相当于按门铃告诉 SSD 控制器：“SQ 尾部有新命令了，快去干活！”

## 终点：NVMe SSD 硬件
1. DMA 读取: SSD 控制器收到 Doorbell 通知，通过 PCIe 总线，利用 DMA (Direct Memory Access) 直接从主存（RAM）中读取数据，不经过 CPU。

2. NAND 写入: SSD 控制器将数据写入闪存颗粒。

3. 完成通知:

- SSD 将结果写入内存中的 完成队列 (Completion Queue - CQ)。

- SSD 发送一个 MSI-X 中断 给 CPU。

4. 回调: CPU 收到中断，驱动处理 CQ，向上层层回调，最终告诉应用程序（如果是异步 I/O）或唤醒阻塞的进程：“写入完成了！”