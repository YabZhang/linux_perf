## 套路总结



内存性能指标：

1. 系统内存：已用内存、剩余内存、共享内存、可用内存、缓存和缓冲区
2. 进程内存：虚拟内存（已申请 ）、常驻内存、共享内存、swap内存
3. 缺页异常：次缺页异常(可直接分配)，主缺页异常(需要磁盘IO介入如swap, 性能损耗较大)
4. swap内存：已用空间、剩余空间、换入速度、换出速度



<img src="https://static001.geekbang.org/resource/image/e2/36/e28cf90f0b137574bca170984d1e6736.png" alt="内存性能指标" style="zoom:40%;" />

内存性能工具：

<img src="https://static001.geekbang.org/resource/image/8f/ed/8f477035fc4348a1f80bde3117a7dfed.png" alt="性能指标工具" style="zoom:30%;" />



工具反查指标：



<img src="https://static001.geekbang.org/resource/image/52/9b/52bb55fba133401889206d02c224769b.png" alt="工具反查指标" style="zoom:30%;" />



快速分析套路：

* 先用 free 和 top 查看系统整体内存使用
* 再用 vmstat 和 pidstat 查看一段时间的变化趋势，判断问题类型
* 最后详细分析，比如内存分配分析、缓存/缓冲区分析、具体进程的使用分析



<img src="https://static001.geekbang.org/resource/image/d7/fe/d79cd017f0c90b84a36e70a3c5dccffe.png" alt="内存问题分析套路" style="zoom:80%;" />



常见优化思路：

1. 最好禁止swap, 如开启swap, 降低swappiness值，减少内存回收时swap的使用倾向；
2. 减少内存的动态分配，比如可以使用内存池，大页等；
3. 尽量使用缓存和缓冲区来访问数据，比如可以使用栈空间来存储需要缓存的数据；或者使用redis这样的外部缓存；
4. 使用cgroups等方式限制进程的内存使用情况，确保系统不会被异常进程耗尽；
5. 通过/proc/pid/oom_adj, 调整核心应用的 oom_score，保障核心应用不会被OOM杀死；