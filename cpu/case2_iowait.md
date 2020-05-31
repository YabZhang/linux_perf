## 案例篇：系统中出现大量的不可中断进程和僵尸进程



#### 概念

1. 进程状态（top或ps工具中查看）：

   * R: Running 或 Runnable , 表示在CPU的就绪队列中，正在运行或正在等待运行；
   * D: Disk Sleep, 不可中断睡眠，一边表示在和硬件交互；
   * Z: Zombie, 僵尸进程，表示进程已经结束，但父进程仍未回收；
   * I: Idle, 空闲状态，没有任何任务负载；
   * T或者(t): Stopped 或 Traced，出于暂停或者跟踪状态；如后台进程

   * X: Dead, 表示已消亡，一般在工具中看不到;

2. 进程组 ， 进程组表示一组相互关联的进程，比如每个子进程都是父进程所在组的成员；

3. 会话是指共享同一个控制终端的一个或多个进程组。



#### 发现问题

首先检查系统整体情况

```bash
# 按下数字 1 切换到所有 CPU 的使用情况，观察一会儿按 Ctrl+C 结束
$ top
top - 05:56:23 up 17 days, 16:45,  2 users,  load average: 2.00, 1.68, 1.39
Tasks: 247 total,   1 running,  79 sleeping,   0 stopped, 115 zombie
%Cpu0  :  0.0 us,  0.7 sy,  0.0 ni, 38.9 id, 60.5 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.7 sy,  0.0 ni,  4.7 id, 94.6 wa,  0.0 hi,  0.0 si,  0.0 st
...

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 4340 root      20   0   44676   4048   3432 R   0.3  0.0   0:00.05 top
 4345 root      20   0   37280  33624    860 D   0.3  0.0   0:00.01 app
 4344 root      20   0   37280  33624    860 D   0.3  0.4   0:00.01 app
    1 root      20   0  160072   9416   6752 S   0.0  0.1   0:38.59 systemd
...

```

发现疑点：

1. 负载逐渐升高，可能有性能问题；
2. 1个进程正在运行，但是较多僵尸进程；
3. CPU使用率最高70%, iowait相对高，最高为94.6%;
4. 查看各个进程情况，发现CPU使用率都较低；



问题归纳：

	1. iowait 太高了，导致系统的平均负载升高，甚至达到了系统 CPU 的个数。
 	2. 僵尸进程在不断增多，说明有程序没能正确清理子进程的资源。



#### 定位和分析问题

1. 首先来看`iowait`过高问题

   ```bash
   # 间隔1秒输出10组数据
   $ dstat 1 10
   You did not select any stats, using -cdngy by default.
   --total-cpu-usage-- -dsk/total- -net/total- ---paging-- ---system--
   usr sys idl wai stl| read  writ| recv  send|  in   out | int   csw
     0   0  96   4   0|1219k  408k|   0     0 |   0     0 |  42   885
     0   0   2  98   0|  34M    0 | 198B  790B|   0     0 |  42   138
     0   0   0 100   0|  34M    0 |  66B  342B|   0     0 |  42   135
     0   0  84  16   0|5633k    0 |  66B  342B|   0     0 |  52   177
     0   3  39  58   0|  22M    0 |  66B  342B|   0     0 |  43   144
     0   0   0 100   0|  34M    0 | 200B  450B|   0     0 |  46   147
     0   0   2  98   0|  34M    0 |  66B  342B|   0     0 |  45   134
     0   0   0 100   0|  34M    0 |  66B  342B|   0     0 |  39   131
     0   0  83  17   0|5633k    0 |  66B  342B|   0     0 |  46   168
     0   3  39  59   0|  22M    0 |  66B  342B|   0     0 |  37   134
   ```

   使用`dstat`工具发现io问题和磁盘读有关。

   ```bash
   # 间隔 1 秒输出多组数据 (这里是 20 组); -d 展示 I/O 统计数据
   $ pidstat -d 1 20
   ...
   06:48:46      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
   06:48:47        0      4615      0.00      0.00      0.00       1  kworker/u4:1
   06:48:47        0      6080  32768.00      0.00      0.00     170  app
   06:48:47        0      6081  32768.00      0.00      0.00     184  app
   
   06:48:47      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
   06:48:48        0      6080      0.00      0.00      0.00     110  app
   
   06:48:48      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
   06:48:49        0      6081      0.00      0.00      0.00     191  app
   
   06:48:49      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
   
   06:48:50      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
   06:48:51        0      6082  32768.00      0.00      0.00       0  app
   06:48:51        0      6083  32768.00      0.00      0.00       0  app
   
   06:48:51      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
   06:48:52        0      6082  32768.00      0.00      0.00     184  app
   06:48:52        0      6083  32768.00      0.00      0.00     175  app
   
   06:48:52      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
   06:48:53        0      6083      0.00      0.00      0.00     105  app
   ...
   ```

   定位到`app`有较大的IO读操作，约`32M`；而且发现app反复被挂起称为僵尸进程。

   

   ```bash
   $ perf record -g
   $ perf report
   ```

   使用`perf`定位到应用有直接读磁盘操作，这可能就是导致IO问题的根源。

2. 僵尸进程

   找出父进程，然后在父进程里解决。

   ```bash
   # -a 表示输出命令行选项
   # p表PID
   # s表示指定进程的父进程
   $ pstree -aps 3084
   systemd,1
     └─dockerd,15006 -H fd://
         └─docker-containe,15024 --config /var/run/docker/containerd/containerd.toml
             └─docker-containe,3991 -namespace moby -workdir...
                 └─app,4009
                     └─(app,3084)
   ```

   定位到代码里的子进程资源回收bug。

#### 总结

1. 碰到iowait升高，首先用工具(`dstat`或`pidstat`)确认是否是磁盘IO问题，然后再去找对应的进程。因为有时候IO高并不是问题，比如对于IO密集型应用。
2. 对于进程会快速死去，没法用动态检测的技术(如`strace`)时，可以通过`perf`分析系统的CPU时钟时间来 定位问题代码。
3. 对于僵尸进程问题，使用`pstree`找到父进程后，检查 wait() / waitpid() 的调用，或是 SIGCHLD 信号处理函数的注册就很容易处理了。