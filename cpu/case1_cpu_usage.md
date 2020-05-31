## 案例篇：系统CPU很高，却找不到应用



### 操作与分析



1. 启动应用

   ```bash
   $ docker run --name nginx -p 10000:80 -itd feisky/nginx:sp
   $ docker run --name phpfpm -itd --network container:nginx feisky/php-fpm:sp
   ```

   

   确认启动成功

   ```bash
   # 192.168.0.10是第一台虚拟机的IP地址
   $ curl http://192.168.0.10:10000/
   It works!
   ```

   

2. 性能测试

   ```bash
   # 并发100个请求测试Nginx性能，总共测试1000个请求
   $ ab -c 100 -n 1000 http://192.168.0.10:10000/
   This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
   Copyright 1996 Adam Twiss, Zeus Technology Ltd, 
   ...
   Requests per second:    87.86 [#/sec] (mean)
   Time per request:       1138.229 [ms] (mean)
   ...
   ```

   QPS过低！

   

3. 问题跟踪

   ```bash
   # 继续测试
   $ ab -c 5 -t 600 http://192.168.0.10:10000/
   
   # 使用top查看系统数据
   $ top
   ...
   %Cpu(s): 80.8 us, 15.1 sy,  0.0 ni,  2.8 id,  0.0 wa,  0.0 hi,  1.3 si,  0.0 st
   ...
   
     PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
    6882 root      20   0    8456   5052   3884 S   2.7  0.1   0:04.78 docker-containe
    6947 systemd+  20   0   33104   3716   2340 S   2.7  0.0   0:04.92 nginx
    7494 daemon    20   0  336696  15012   7332 S   2.0  0.2   0:03.55 php-fpm
    7495 daemon    20   0  336696  15160   7480 S   2.0  0.2   0:03.55 php-fpm
   10547 daemon    20   0  336696  16200   8520 S   2.0  0.2   0:03.13 php-fpm
   10155 daemon    20   0  336696  16200   8520 S   1.7  0.2   0:03.12 php-fpm
   10552 daemon    20   0  336696  16200   8520 S   1.7  0.2   0:03.12 php-fpm
   15006 root      20   0 1168608  66264  37536 S   1.0  0.8   9:39.51 dockerd
    4323 root      20   0       0      0      0 I   0.3  0.0   0:00.87 kworker/u4:1
   ...
   ```

   发现系统CPU过高，但各个进程的的CPU都比较低？似乎没有可以进程，那么系统资源怎么消耗了呢？

   ```bash
   
   # 间隔1秒输出一组数据（按Ctrl+C结束）
   $ pidstat 1
   ...
   04:36:24      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
   04:36:25        0      6882    1.00    3.00    0.00    0.00    4.00     0  docker-containe
   04:36:25      101      6947    1.00    2.00    0.00    1.00    3.00     1  nginx
   04:36:25        1     14834    1.00    1.00    0.00    1.00    2.00     0  php-fpm
   04:36:25        1     14835    1.00    1.00    0.00    1.00    2.00     0  php-fpm
   04:36:25        1     14845    0.00    2.00    0.00    2.00    2.00     1  php-fpm
   04:36:25        1     14855    0.00    1.00    0.00    1.00    1.00     1  php-fpm
   04:36:25        1     14857    1.00    2.00    0.00    1.00    3.00     0  php-fpm
   04:36:25        0     15006    0.00    1.00    0.00    0.00    1.00     0  dockerd
   04:36:25        0     15801    0.00    1.00    0.00    0.00    1.00     1  pidstat
   04:36:25        1     17084    1.00    0.00    0.00    2.00    1.00     0  stress
   04:36:25        0     31116    0.00    1.00    0.00    0.00    1.00     0  atopacctd
   ...
   ```

   `pidstat`再次检查各个进程，也没有发现可以进程。再次仔细观察下`top`。

   ```bash
   
   $ top
   top - 04:58:24 up 14 days, 15:47,  1 user,  load average: 3.39, 3.82, 2.74
   Tasks: 149 total,   6 running,  93 sleeping,   0 stopped,   0 zombie
   %Cpu(s): 77.7 us, 19.3 sy,  0.0 ni,  2.0 id,  0.0 wa,  0.0 hi,  1.0 si,  0.0 st
   KiB Mem :  8169348 total,  2543916 free,   457976 used,  5167456 buff/cache
   KiB Swap:        0 total,        0 free,        0 used.  7363908 avail Mem
   
     PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
    6947 systemd+  20   0   33104   3764   2340 S   4.0  0.0   0:32.69 nginx
    6882 root      20   0   12108   8360   3884 S   2.0  0.1   0:31.40 docker-containe
   15465 daemon    20   0  336696  15256   7576 S   2.0  0.2   0:00.62 php-fpm
   15466 daemon    20   0  336696  15196   7516 S   2.0  0.2   0:00.62 php-fpm
   15489 daemon    20   0  336696  16200   8520 S   2.0  0.2   0:00.62 php-fpm
    6948 systemd+  20   0   33104   3764   2340 S   1.0  0.0   0:00.95 nginx
   15006 root      20   0 1168608  65632  37536 S   1.0  0.8   9:51.09 dockerd
   15476 daemon    20   0  336696  16200   8520 S   1.0  0.2   0:00.61 php-fpm
   15477 daemon    20   0  336696  16200   8520 S   1.0  0.2   0:00.61 php-fpm
   24340 daemon    20   0    8184   1616    536 R   1.0  0.0   0:00.01 stress
   24342 daemon    20   0    8196   1580    492 R   1.0  0.0   0:00.01 stress
   24344 daemon    20   0    8188   1056    492 R   1.0  0.0   0:00.01 stress
   24347 daemon    20   0    8184   1356    540 R   1.0  0.0   0:00.01 stress
   ...
   ```

   发现疑点：

   1. 负载和CPU使用率仍然过高；
   2. 就绪队列的任务有点过高，和下图`stress`对应不上；

   进一步使用`pidstat`和`ps`交叉确认，发现`stress`进程有较快更新现象！

   推测可能原因：

   1. 进程在不停地崩溃重启，比如因为段错误、配置错误等等，这时，进程在退出后可能又被监控系统自动重启了。
   2. 这些进程都是短时进程，也就是在其他应用内部通过 exec 调用的外面命令。这些命令一般都只运行很短的时间就会结束，你很难用 top 这种间隔时间比较长的工具发现（上面的案例，我们碰巧发现了）。

   

   找到对应的进程关系：

   ```bash
   $ pstree | grep stress
           |-docker-containe-+-php-fpm-+-php-fpm---sh---stress
           |         |-3*[php-fpm---sh---stress---stress]
   ```

   

   果然和`app`有关（有调用）! 

   ```bash
   # 拷贝源码到本地
   $ docker cp phpfpm:/app .
   
   # grep 查找看看是不是有代码在调用stress命令
   $ grep stress -r app
   app/index.php:// fake I/O with stress (via write()/unlink()).
   app/index.php:$result = exec("/usr/local/bin/stress -t 1 -d 1 2>&1", $output, $status);
   
   
   # 在代码中找到了
   $ cat app/index.php
   <?php
   // fake I/O with stress (via write()/unlink()).
   $result = exec("/usr/local/bin/stress -t 1 -d 1 2>&1", $output, $status);
   if (isset($_GET["verbose"]) && $_GET["verbose"]==1 && $status != 0) {
     echo "Server internal error: ";
     print_r($output);
   } else {
     echo "It works!";
   }
   ?>
   ```

   

   确认到`app`中调用`stree`! 负载`verbose`参数查看详细原因。

   ```bash
   
   $ curl http://192.168.0.10:10000?verbose=1
   Server internal error: Array
   (
       [0] => stress: info: [19607] dispatching hogs: 0 cpu, 0 io, 0 vm, 1 hdd
       [1] => stress: FAIL: [19608] (563) mkstemp failed: Permission denied
       [2] => stress: FAIL: [19607] (394) <-- worker 19608 returned error 1
       [3] => stress: WARN: [19607] (396) now reaping child worker processes
       [4] => stress: FAIL: [19607] (400) kill error: No such process
       [5] => stress: FAIL: [19607] (451) failed run completed in 0s
   )
   ```

   发现`stree`执行错误，印证猜想。使用`perf`再次确认猜想:

   ```bash
   # 记录性能事件，等待大约15秒后按 Ctrl+C 退出
   $ perf record -g
   
   # 查看报告
   $ perf report
   ```

   

4. 问题总结

   再对应用的测试中发现系统资源异常，但是没有找到对应的高消耗应用。后进一步分析，原来是由于命令执行错误，导致很多很快死亡的子进程产生，所以传统的`top`, `ps`和`pidstat`三板斧无法轻易发现。

   在通过`pstree` 确认猜想后，经由`perf`工具进一步验证后，确认问题的根源是对`stress`的频繁调用导致。

   

   对于短时进程分析，新工具`execsnoop`(常用的动态追踪技术):

   ```bash
   
   # 按 Ctrl+C 结束
   $ execsnoop
   PCOMM            PID    PPID   RET ARGS
   sh               30394  30393    0
   stress           30396  30394    0 /usr/local/bin/stress -t 1 -d 1
   sh               30398  30393    0
   stress           30399  30398    0 /usr/local/bin/stress -t 1 -d 1
   sh               30402  30400    0
   stress           30403  30402    0 /usr/local/bin/stress -t 1 -d 1
   sh               30405  30393    0
   stress           30407  30405    0 /usr/local/bin/stress -t 1 -d 1
   ...
   ```

   



#### 总结

对于发现“三板斧”无法解释的CPU异常问题，有可能是短时进程导致：

1. 应用里直接调用了其他二进制程序，这些程序通常运行时间比较短，通过 top 等工具也不容易发现。

2. 应用本身在不停地崩溃重启，而启动过程的资源初始化，很可能会占用相当多的 CPU。

对于此类问题，可以使用`pstree`和`execsnoop`来帮助分析。