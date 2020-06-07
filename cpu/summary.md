## 套路篇



#### CPU性能指标

* CPU使用率：非空闲CPU时间的占比

  1. 用户CPU（用户态`user`和低优先级`nice`），
  2. 系统CPU（内核态`sys`; 不含中断）
  3. I/O CPU（IO百分比`iowait`，系统与硬件设备IO交互）
  4. 软中断`si`和硬中断`hi`
  5. 虚拟环境窃取`steal`和其他虚拟机占用`guest`

* CPU负载

  过去1, 5 和 15 分钟平均负载；

  理想情况是平均负载等于逻辑CPU个数

* CPU上下文切换（进、线程，内核，中断）

  自愿调度(无法获取资源) vs 非自愿调度(系统强制，争抢CPU)

* CPU缓存命中率

  CPU核心单独有L1和L2缓存，共享L3缓存

  缓存命中率越高，性能越好 => 程序绑定CPU核心



#### 性能工具

1. 平均负载：`uptime` `mpstat`和`pidstat`，`stress`

   `uptime` 查看平均负载

   `mpstat` 查看每个CPU使用情况；`pidstat` 查看每个进程CPU使用情况

2. 上下文切换: `vmstat`, `pidstat`, `sysbench`

   `vmstat` 查看系统上下文切换和中断次数

   `pidstat` 查看 *进程* 和 *线程* 自愿和非自愿上下文切换

3. CPU使用率：

   a. `top`, `perf top`;  

   b. `top`, `pidstat`, `perf record` & `perf report`

   `execsnoop` : 短时进程监控

4. 不可中断和僵尸进程(iowait问题)：`top`, `dstat`, `pidstat`, `strace`, `perf`

   `dstat`: 系统资源统计报告

   `strace`: 查看进程系统调用

5. 软中断: `top`, `/proc/softirqs`, `sar`, `tcpdump`

   `/proc/softirqs`: 查看系统软中断

   `sar`: 报告保存系统活动信息(这里是查看网卡的收发情况)

   `tcpdump`: 抓取`tcp`数据包，定位问题数据包



#### 关联工具和指标

> 活学活用，把 **性能指标** 和 **性能工具** 联系起来



* 从性能指标

| 性能指标            | 工具                                 | 说明                                                         |
| ------------------- | ------------------------------------ | ------------------------------------------------------------ |
| 平均负载            | `uptime`, `top`                      | `uptime` 最简洁，`top`最全面                                 |
| 整体CPU使用率       | vmstat, mpstat, top, sar, /proc/stat | top, vmstat, mpstat 动态追踪不同层级变化，sar 还可用于记录历史数据；/proc/stat 是数据源 |
| 进（线）程CPU使用率 | top, pidstat, ps, htop, atop         | top 和 ps 可以按照使用率排序给出；pidstat只显示实际使用进程； |
| 系统上下文切换      | vmstat                               |                                                              |
| 进(线)程上下文切换  | pidstat                              | -w展示自愿和非自愿切换；-t展示线程数据                       |
| 软中断              | top, /proc/softirqs, mpstat          | top提供软中断CPU使用率； /proc/softirqs和mpstat提供了软中断在每个CPU上运行次数 |
| 硬中断              | Vmstat, /proc/interrupts             | vmstat提供硬中断总数，/proc/interrupts提供CPU级别硬中断运行次数 |
| 网络                | dstat, sar, tcpdump                  | dstat 和 sar 提供统计，tcpdump可以精准抓包                   |
| IO                  | dstat, sar                           |                                                              |
| 事件剖析            | perf, execsnoop                      | perf 可以用来分析CPU缓存以及内核调用链，execsnoop监控短时进程 |



* 从工具出发

  man命令；RTFM！



* 快速分析套路

>  通晓每种性能指标的工作原理，弄清楚性能指标的关联性；



1. 缩小排查范围，首先使用top, vmstat, pidstat; 锁定问题类型

2. 细化排查方向：

   ![排查套路](https://static001.geekbang.org/resource/image/7a/17/7a445960a4bc0a58a02e1bc75648aa17.png "CPU性能排查套路")



#### CPU性能优化思路

* 方法论概览

  1. 评估优化效果，优化后提升多少性能？

  2. 性能问题不是孤立的，多个性能问题发生时应该优先优化哪一个？

     并不是所有的问题都值得优化， 28定律；先主后次；

  3. 提升性能的方法不是唯一的，多个方法如何选择？

     性能优化是有成本的 => 分析和思考 => 权衡

* 评估性能优化效果

  1. 确定性能的量化指标；ps. 数值化才可作比较

     不要局限在单一维度，至少从 应用程序（吞吐量，请求延迟）和系统资源（CPU使用率）

  2. 测试优化前的性能指标；

  3. 测试优化后的性能指标；



#### CPU优化套路

> 降低CPU使用率，提高CPU并行处理能力。



* 应用程序

  减少不必要的工作，仅保留核心逻辑；改变实现方案，提高执行效率；

  1. 编译器优化：开启优化选项，代码级别优化；如，`gcc`的`-O2`优化选项
  2. 算法优化：降低算法复杂度，提高执行效率
  3. 异步处理：避免一直阻塞，提升程序并发能力；如，轮询改为事件通知
  4. 多线程代替多进程：降低上下文切换成本
  5. 善用缓存：合理使用缓存，提高读写效率



* 系统优化

  > 提高CPU缓存命中概率；控制进程的CPU使用，降低相互影响

  1. CPU绑定：把进城绑定到CPU后，可以提高CPU的缓存命中率，减少上下文切换成本
  2. CPU独占：和绑定类似
  3. 优先级调整：通过nice调整优先级，提高CPU执行进程的优先级别
  4. 为进城设置资源限制: 通过`cgroup`来设置CPU的使用上限，防止进程相互影响
  5. NUMA(Non-Uniform Memory Access)优化：让CPU尽量访问本地node内存空间，提高访问效率
  6. 中断负载均衡：开启`irqbalance`服务或者`smp_affinity`,把中断处理自动负载均衡到多个CPU，提高中断响应效率

> 千万避免过早优化
>
> 过早优化是万恶之源