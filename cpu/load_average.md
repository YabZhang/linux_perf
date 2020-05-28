## 平均负载



#### 概念

每次发现系统变慢时，第一件事是确定系统 **平均负载** 。

```bash
$ uptime
 05:16:07 up  7:09,  0 users,  load average: 0.02, 0.10, 0.05
 -------- --------- ---------- -------------------------------
  当前时间  运行时间   登录用户数    最近的1、5、15分钟的平均负载
```

平均负载可以理解为系统中的平均活跃进程数（可不中断状态是系统的一种保护措施）。实际计算上不是绝对平均，参见指数衰减平均。

通过平均负载的三个指标，可以评估出系统平均负载变化趋势（增加或减少）；

理想情况，平均负载匹配系统的逻辑CPU数目，实际上高于逻辑CPU数目70%就需要注意了（分析和排查）；



#### 原理

* 关于`平均负载`的文档解释

>System load averages is the average number of processes that are either in a runnable or uninterruptable state.  A process in a runnable state is either using the CPU or waiting to use the CPU.  A process in uninterruptable state is waiting for some I/O access, eg waiting for disk.  The averages are taken over the three time intervals.  Load averages are not normalized for the number of CPUs in  a  system, so a load average of 1 means a single CPU system is loaded all the time while on a 4 CPU system it means it was idle 75% of the time.
>
>平均负载是处于 **可运行状态(runnable state)** 或者 **不可中断状态(uninterruptable state)** 的 **平均进程数**。
>
>进程的可运行状态是指 *正在使用CPU* 或者 *等待使用CPU*；进程的不可中断转态是指 *等待I/O访问（如磁盘）*；
>
>平均负载程度和CPU核心数量的关系：负载为1的单核CPU为满负载，而对四核心CPU的来说就有75%的时间是空闲。



* 进程状态

  https://en.wikipedia.org/wiki/Process_state

* 系统CPU核心数查询：

```bash
$ grep 'model name' /proc/cpuinfo | wc -l
```

* 理解 `/proc/cpuinfo` or `lscpu` 中的cpu信息：

查看物理 cpu 数（socket 插槽数）：

> ```bash
> cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
> ```

查看每个物理 cpu 中 核心数(单cpu core 数)：

> ```bash
> cat /proc/cpuinfo | grep "cpu cores" | uniq
> ```

查看总的逻辑 cpu 数（逻辑处理器 processor 数；超线程技术）：

> ```bash
> cat /proc/cpuinfo| grep "processor"| wc -l
> ```

查看 cpu 型号：

> ```bash
> cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
> ```

判断 cpu 是否 64 位：

> 检查 cpuinfo 中的 flags 区段，看是否有 lm （long mode） 标识



总的逻辑 cpu 数 = 物理 cpu 数 * 每颗物理 cpu 的核心数 * 每个核心的超线程数

https://zhuanlan.zhihu.com/p/86855590



#### 平均负载实验

`stress` 系统压力测试工具；

`mpstat`多核CPU性能分析工具；`pidstat`进程性能分析工具；=> `sysstat`包



高CPU负载的三种可能情况：

 * CPU密集型，大量使用CPU的任务会导致平均负载升高；

   ```bash
   # terminal 1;
   $ stress --cpu 1 --timeout 600
   
   # terminal 2; -d highlight change area
   $ watch -d uptime
   
   # terminal 3; check all cpu load
   $ mpstat -P ALL 5
   ```

 * I/O密集型，等待I/O也会导致平均负载升高，但CPU使用率不一定很高；

   ```bash
   # terminal 1;
   $ stress -i 1 --timeout 600
   
   # terminal 2;
   watch -d uptime
   
   # terminal 3; chekc all cpu load
   mpstat -P ALL 5 1
   ```

 * 大量等待CPU的进程调度会导致平均负载升高，此时CPU使用率也很高；

   ```bash
   # terminal 1;
   $ stress -c 8 --timeout 600
   
   # terminal 2;
   $ watch -d uptime
   
   # terminal 3; check process load
   $ pidstat -u 5 1
   ```

   

#### 总结

1. 平均负载是系统中活跃的进程数；系统负载的评估应该和CPU数量相当，实际中不超过70%为宜；
2. 平均负载升高的常见原因：a. CPU密集任务；b. I/O密集任务，CPU并不会很高；c. 大量等待CPU调度的工作进程，此时CPU使用率也会很高；