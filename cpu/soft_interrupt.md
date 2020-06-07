## 软中断



#### 概念

> 中断是系统用来响应硬件设备请求的一种机制，它会打断进程正常的调度和执行，然后调用内核的中断处理程序来响应设备的请求。



为避免中断程序执行过长和丢失其他中断，Linux将中断处理分为两部分：

1. 上半部分（硬中断，硬件触发）：**快速处理**硬件相关或时间敏感工作，直接调度CPU来执行内核处理程序

2. 下半部分（软中断，内核触发）：**延迟处理**后续中断程序，通过内核线程执行（`ksoftirqd/CPU 编号`）



查看中断：

`/proc/interrupts` 硬件中断；

`/proc/softirqs` 软件中断；



#### 原理

Linux 中断：https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html#

多核系统中断处理架构：`local APIC`处理CPU关联设备中断，`IO APIC`分发外部中断到多核上（通过总线）



IDT(` interrupt descriptor table`)：关联每个`exception`或者`interrupt`和对应的处理handler

1. `interrupt gate`, 保存`exception`或`interrupt`处理器（跳转到处理器`disables maskable`中断）
2. `trap gate`,类似`interrupt gate`, 但不做`disable`处理
3. `task gate`(not used in linux)



中断处理（主要是上下文切换）：https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html#handling-an-interrupt-request

一般中断处理流程：https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html#generic-interrupt-handling-in-linux



延迟执行任务类型：

1. `softIRQ`: 运行在*中断上下文*, 静态分配资源, 处理方法可多核并行
2. `tasklet`: 运行在*中断上下文*, 可动态分配, 处理方法顺序运行
3. `workqueues`: 运行在进程上下文环境



#### 实验步骤

工具:

​	`nginx`, `hping3`, `top`, `tcpdump`, `/proc/softirqs`

1. 在机器A启动nginx服务器，在内网机器B上测试服务可达，然后使用`hping3`在机器B模拟`SYN FLOOD`攻击
2. 机器A的软中断`si`CPU消耗相对较高，通过查看`/proc/softirqs`发现`NET_TX`软中断事件格外高，怀疑原因可能是异常网络包导致的软中断升高；
3. 使用`sar` 展示全部网络的数据帧数和字节数，然后判断是有大量的小数据包到`eth0`，导致的软中断消耗过高；
4. 通过`tcpdump`抓包分析，确定是来自特定IP的大量`SYNC`包的`SYN FLOOD`攻击；



涉及到的命令:

```bash
# -S参数表示设置TCP协议的SYN（同步序列号），-p表示目的端口为80
# -i u100表示每隔100微秒发送一个网络帧
hping3 -S -p 80 -i u100 192.168.0.30
# 查看软中断变化数
watch -d cat /proc/softirqs
# -n DEV 表示显示网络收发的报告，间隔1秒输出一组数据
sar -n DEV 1
# -i eth0 只抓取eth0网卡，-n不解析协议名和主机名；
# tcp port 80表示只抓取tcp协议并且端口号为80的网络帧
tcpdump -i eth0 -n tcp port 80
```

#### 分析总结

1. 中断是系统处理异步请求的一种机制，包含两个阶段硬中断（立即执行）和软中断（延迟执行），以提高系统中断处理效率；
2. 中断包含多种类型，可通过`/proc/softirqs`查看不同类型的软中断触发次数（`/proc/interrupts`用于查看硬中断）；
3. 通过模拟一次`SYN FLOOD`模拟攻击，调用各种分析工具来定位频繁的网络小包导致的中断消耗较多CPU资源问题；

