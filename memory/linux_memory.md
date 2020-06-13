## Linux  内存



#### 基本概念



Linux 内核为独立进程提供一个连续的独立虚拟地址空间，便于进程的虚拟访问内存。虚拟地址分为用户空间和内核空间，进程用户态只能访问用户空间，只有进入内核态才能访问内核地址空间；并且所有进程的内核态都映射到相同的物理地址，便于内核空间的访问，但用户空间对应的物理内存可以映射到不同位置。

<img src="https://static001.geekbang.org/resource/image/ed/7b/ed8824c7a2e4020e2fdd2a104c70ab7b.png" alt="Linux 虚拟地址空间" title="Linux 虚拟地址空间" style="zoom:67%;" />



Linux 为每个进程维护一张页表，用于把**虚拟内存**映射到**物理内存地址**。页表存储在CPU的内存管理单元MMU中，TLB（Translation Lookaside Buffer，转译后备缓冲器；进程上下文切换有涉及）就是页面映射内容的高速缓存。MMU 中最小内存单元为页，大小为4KB， 通过 **多级页表** 和 **大页** 来提高页表映射效率。

<img src="https://static001.geekbang.org/resource/image/fc/b6/fcfbe2f8eb7c6090d82bf93ecdc1f0b6.png" alt="内存映射" style="zoom:75%;" />



当进程访问的虚拟地址在页表中找不到映射，系统会产生**缺页异常**，进程到内核空间分配物理内存、更新进程表，最后再回到用户空间，回复程序运行。



进程(32位系统)虚拟内存空间分布图：

![虚拟内存空间分布](https://static001.geekbang.org/resource/image/71/5d/71a754523386cc75f4456a5eabc93c5d.png)

* 只读段：包含代码和常量等
* 数据段：包括全局变量等
* 堆：包括动态分配的内存，从低地址开始向上增长
* 文件映射段，包括动态库、共享内存等，从高地址开始向下增长
* 栈，包括局部变量和函数调用的上下文等。栈的大小是固定的，一般是 8 MB



C 语言的内存分配函数`malloc`有两种实现：`brk` 和 `mmap`; 

1. `brk` 用于分配 **堆** 上的小内存块（小于 128K）；这些小内存块 `free` 释放后不会立刻会换系统，而是管理系统缓存起来重复利用；以减少小内存块的利用效率，减轻系统压力；

2. `mmap` 在 **文件映射段** 分配大内存块（大于128K）；并且在 `unmap` 释放后立刻归还给系统；

以上两种系统调用完成后，并没有真正分配物理内存；只在首次访问时，通过 **缺页异常** 进入内核来完成实际内存分配。



当系统内存紧张时，通过以下三种机制强制回收：

1. 回收 **缓存(cache & buffer)**，如通过LRU，回收最近最少使用的文件内存页
2. 回收不常用内存，把不常用的匿名内存页通过交换分区直接写到磁盘中；swap分区可能导致性能问题
3. 杀死进程，内存紧张时系统还会通过 OOM 杀死进程；通过`oom_score`决定



常用的内存查看工具: `free`, `top` 和 `ps` 等



`buffers`: 内核缓冲区用到的内存，数值来自`/proc/meminfo`中的`Buffers`; 是对原始磁盘块的临时存储，用来缓存磁盘的数据；

`cache`: 内核页缓存和 Slab 用到的内存, 数值对应 `/proc/meminfo` 中的 `Cached` 与 `SReclaimable` 之和。其中`Cached` 是从磁盘读取文件的页缓存，也就是用来缓存从文件读取的数据。SReclaimable 是 Slab 的一部分。Slab 包括两部分，其中的可回收部分，用 SReclaimable 记录；而不可回收部分，用 SUnreclaim 记录。

`buffers` 和 `cache` 分别是对磁盘数据和文件数据的缓存，读写请求均会使用！



SWAP 交换空间: 磁盘或者本地文件用作内存

1. 换出(page out): memory => swap
2. 换入(page in): swap => memory



回收场景：

1. 直接内存回收：系统自行回收缓存； 

2. Kswapd0 内核线程定期回收，根据评估结果

![kswapd0 内核线程回收量级](https://static001.geekbang.org/resource/image/c1/20/c1054f1e71037795c6f290e670b29120.png)

`free`大于`pages_low`以上时，不会触发内存回收；小于`pages_low`才会触发`kswapd0`执行回收，知道高于`pages_high`;

`pages_low`配置可在`/proc/sys/vm/min_free_kbytes`间接配置；



NUMA(Non-Uniform Memory Access) 架构

有时剩余内存还很多，但`swap`升高；可能是由于处理器的`NUMA`架构导致；



`NUMA`架构把多个处理器分配到不同Node上，每个Node都有本地内存空间；

每个Node内部又可分为不同的区域：

![NUMA Node](https://static001.geekbang.org/resource/image/be/d9/be6cabdecc2ec98893f67ebd5b9aead9.png)



```bash
# 查看node内存情况
$ numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1
node 0 size: 7977 MB
node 0 free: 4416 MB
...
```

```bash
# 查看内存域阀值（pages_high, pages_low, pages_min）
$ cat /proc/zoneinfo
...
Node 0, zone   Normal
 pages free     227894
       min      14896
       low      18620
       high     22344
...
     nr_free_pages 227894
     nr_zone_inactive_anon 11082  # 非活跃匿名内存页
     nr_zone_active_anon 14024    # 活跃匿名内存页
     nr_zone_inactive_file 539024  # 非活跃文件数
     nr_zone_active_file 923986    # 活跃文件数
...
```



默认当某个Node内存不足，可以从其他Node找空闲，也可以从本地回收;

通过`/proc/sys/vm/zone_reclaim_mode` 0#默认；1,2,4#只回收本地内存；2#回写脏数据，4#swap方式回收



内存页回收：文件页回收（直接回收，或脏页回写并回收）；匿名页回收(swap)

配置`/proc/sys/vm/swappiness`调整swap回收积极性：0-100，越大越swap积极,越小越倾向于文件页回收

