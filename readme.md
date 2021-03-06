## linux_perf

---

这里整理了本人学习倪鹏飞的《Linux 性能优化实战》专栏的学习笔记和相关资料汇总。

此外，特别感谢`TalkGo`读书会发起的学习活动。



> ### 相关链接
>
> [《Linux 性能优化专栏》倪鹏飞 ](https://time.geekbang.org/column/article/68728)
>
> [TalkGo 读书会网站 ](https://talkgo.org/)
>
> [TalkGo 读书会第一期活动资料](https://shimo.im/sheets/1lq7MgXnBphdeWAe/MODOC)



### 学习笔记

---



* 开篇词

1. [开篇词](https://github.com/YabZhang/linux_perf/blob/master/opening/opening.md)



* CPU 性能篇

1. [平均负载](https://github.com/YabZhang/linux_perf/blob/master/cpu/load_average.md)

2. [上下文切换](https://github.com/YabZhang/linux_perf/blob/master/cpu/load_average.md)

3. [CPU使用率](https://github.com/YabZhang/linux_perf/blob/master/cpu/cpu_usage.md)

4. [案例1：系统CPU很高，却找不到应用](https://github.com/YabZhang/linux_perf/blob/master/cpu/case1_cpu_usage.md)

5. [案例2：系统中出现大量的不可中断进程和僵尸进程](https://github.com/YabZhang/linux_perf/blob/master/cpu/case2_iowait.md)
6. [软中断](https://github.com/YabZhang/linux_perf/blob/master/cpu/soft_interrupt.md)

7. [CPU分析套路篇](https://github.com/YabZhang/linux_perf/blob/master/cpu/soft_interrupt.md)
8. [CPU性能答疑](https://github.com/YabZhang/linux_perf/blob/master/cpu/qa.md)



* 内存性能篇

1. [Linux 内存基础](https://github.com/YabZhang/linux_perf/blob/master/memory/linux_memory.md)
2. [内存分析套路](https://github.com/YabZhang/linux_perf/blob/master/memory/summary.md)



* I/O性能篇

1. [Linux 文件系统](https://github.com/YabZhang/linux_perf/blob/master/io/filesys.md)
2. [磁盘 IO 原理](https://github.com/YabZhang/linux_perf/blob/master/io/disk_io.md)



* 网络性能篇

1. [Linux 网络基础](https://github.com/YabZhang/linux_perf/blob/master/network/basic.md)
2. [从C10K到C10M](https://github.com/YabZhang/linux_perf/blob/master/network/c10k_and_more.md)



* 综合实战篇

