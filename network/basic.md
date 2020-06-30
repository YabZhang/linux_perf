## Linux 网络基础



#### 基本概念

* 网络模型

  7层 OSI 网络模型 (开放式系统互联通信参考模型) ；过于复杂

  4层 TCP/IP 网络模型

  <img src="https://static001.geekbang.org/resource/image/f2/bd/f2dbfb5500c2aa7c47de6216ee7098bd.png" alt="网络模型" style="zoom:50%;" />

* Linux 网络栈

<img src="https://static001.geekbang.org/resource/image/c8/79/c8dfe80acc44ba1aa9df327c54349e79.png" alt="Linux网络栈" style="zoom:50%;" />

网络数据包传输最大单元(MTU) 默认 1500 Byte (含协议帧头)，大小超过后就会再网络层分包；

<img src="https://static001.geekbang.org/resource/image/c7/ac/c7b5b16539f90caabb537362ee7c27ac.png" alt="通用IP网络栈" style="zoom:50%;" />



* Linux 网络收发流程



​	<img src="https://static001.geekbang.org/resource/image/3a/65/3af644b6d463869ece19786a4634f765.png" alt="网络包接收流程" style="zoom:50%;" />



网路帧到达后，网卡通过DMA(直接内存访问)把数据放到收包队列中；然后通过硬中断告诉中断处理程序；

然后，网卡中断程序(注册在CPU上；唤起内核线程执行)会为网络帧分布内核数据结构(sk_buff)，并将其拷贝到sk_buff缓冲区；然后再通过软中断，通知内核收到新的网络帧；

接下来，内核协议栈从缓冲区取出网络帧，并通过网络协议栈校验和解析内容:

* 链路层检查报文合法性，确定上层协议；无误后把内容交给网络层处理；
* 网络层取出IP头，确认网络包下一步走向，如交给协议栈上层或转发出去；
* 当传输层读取到TCP或UDP头后，根据<源IP, 源端口, 目标IP, 目标端口>作为标识找到对应socket，并发数据拷贝的socket的接收缓存中；
* 最后应用程序通过socket的接口读取到传输数据；



网络数据包的发包流程是收包的逆过程；

socket API --(系统调用|缓冲区)-->  网络协议栈(构建数据包) ----> 发包队列 (软中断通知)  ---> 网卡发送(驱动程序DMA读取和发送) 



* 性能指标

  1. 带宽，表示链路的最大传输速率，单位通常为 b/s (比特/秒)；
  2. 吞吐量，表示单位时间成功传输的数据量，单位为 b/s （比特/秒）或者 B/s (字节/秒)；受带宽限制；
  3. 延时，从请求发出到接收远端响应的时间延迟，和具体过程有关。常用的有连接建立时间（TCP握手时间），一个数据包往返时间(RTT)；
  4. PPS，包/秒；表示网络包的传输速率；通常用来评估网络转发能力；

  此外还有，网络可用性，并发连接数(TCP连接数),丢包率(丢包百分比),重传率(重新传输的网络包比例)等常用指标;

  

* 网络配置

  > 常用`ifconfig`或`ip`命令查看网络配置;
  >
  > `ifconfig`和`ip`分别属于`net-tools`和`iproute2`包，`iproute2`是新一代工具包；

  ```bash
  $ ipconfig eth0
  eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
          ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
          RX packets 13492  bytes 18351744 (18.3 MB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 6201  bytes 340087 (340.0 KB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
          
  $ ip -s addr show dev eth0
  10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
      link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
      inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
         valid_lft forever preferred_lft forever
      RX: bytes  packets  errors  dropped overrun mcast
      42972296   31947    0       0       0       0
      TX: bytes  packets  errors  dropped carrier collsns
      826144     14952    0       0       0       0
  ```



	1. 网络接口状态, `RUNNING`(ifconfig) 或者 `UP`(ip), 表示物理网络是连通的，即网卡接入路由或者交换机
 	2. MTU大小，默认是1500，根据网络架构不同，可能需要重新调大或者调小MTU的数值
 	3. 网络接口的IP地址、子网以及MAC地址
 	4. 网络收发字节数、包数、错误数以及丢包情况，特别是TX和RX的errors、dropped、 overruns、carrier 以及 collisions 等指标不为0时，通常表示出现了网络IO问题;

其中 errors 表示发生错误的包数，比如校验错误、帧同步错误等；

dropped 表示丢弃的包数，即数据包放到了 Ring Buffer中，但因为内存不足等原因丢包;

overruns 表示超限数据包数，即网络IO速度过快，导致Ring Buffer中数据来不及处理(队列满)而导致的丢包;

carrier 表示发生 carrier 错误的数据包数，比如双工模式不匹配、物理电缆出现问题等；

collisions 表示碰撞数据包数;

* 套接字信息

协议栈中的统计信息，可以用netstat或者ss来查看，如套接字、网络栈、网络接口以及路由表信息等；

```bash

# head -n 3 表示只显示前面3行
# -l 表示只显示监听套接字
# -n 表示显示数字地址和端口(而不是名字)
# -p 表示显示进程信息
$ netstat -nlp | head -n 3
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      840/systemd-resolve

# -l 表示只显示监听套接字
# -t 表示只显示 TCP 套接字
# -n 表示显示数字地址和端口(而不是名字)
# -p 表示显示进程信息
$ ss -ltnp | head -n 3
State    Recv-Q    Send-Q        Local Address:Port        Peer Address:Port
LISTEN   0         128           127.0.0.53%lo:53               0.0.0.0:*        users:(("systemd-resolve",pid=840,fd=13))
LISTEN   0         128                 0.0.0.0:22               0.0.0.0:*        users:(("sshd",pid=1459,fd=3))
```



接收队列(Recv-Q)和发送队列(Send-Q) ，通常是0； 当不是0时说明有网络包堆积；

当套接字出于连接状态时（Established）:

1. Recv-Q 标识套接字缓冲还没有被应用内程序取走的字节数(接收队列长度)

2. Send-Q 表示还没有被远程主机确认的字节数(即发送队列)

当套接字出于监听状态时(Listening)：

1. Recv-Q 表示全连接队列的长度

2. Send-Q 标识全连接队列的最大长度

全连接是指完成了TCP三次握手的连接; 这些全连接的套接字还需要被`accept()`系统调用取走，服务器才开始真正处理客户端请求；半连接队列，是还没有完成TCP三次握手的连接队列；

* 协议栈统计信息

```bash
# 查看协议栈统计信息
$ netstat -s
$ ss -s
```

* 网络吞吐 和 PPS

```bash

# 数字1表示每隔1秒输出一组数据
$ sar -n DEV 1
Linux 4.15.0-1035-azure (ubuntu)   01/06/19   _x86_64_  (2 CPU)

13:21:40        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
13:21:41         eth0     18.00     20.00      5.79      4.25      0.00      0.00      0.00      0.00
13:21:41      docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
13:21:41           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

给 sar 增加 -n 参数就可以查看网络的统计信息，比如网络接口（DEV）、网络接口错误（EDEV）、TCP、UDP、ICMP 等等. 

rxpck/s 和 txpck/s 分别是接收和发送的 PPS，单位为包 / 秒。

rxkB/s 和 txkB/s 分别是接收和发送的吞吐量，单位是 KB/ 秒。

rxcmp/s 和 txcmp/s 分别是接收和发送的压缩数据包数，单位是包 / 秒。

%ifutil 是网络接口的使用率，即半双工模式下为 (rxkB/s+txkB/s)/Bandwidth，而全双工模式下为 max(rxkB/s, txkB/s)/Bandwidth。

查询网卡带宽：

```bash
$ ethtool eth0 | grep Speed
Speed: 1000Mb/s
```



* 连通性和延迟

使用`ping`来测试远程主机的连通性和延时(基于ICMP协议)