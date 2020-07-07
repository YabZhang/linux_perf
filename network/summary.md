## 套路篇



#### 评估网络性能

确定目标网络的协议栈 —— 低层性能决定高层性能；

* 转发性能：网络层，指标PPS，工具`hping3`和`pktgen`

* TCP/UDP 性能：传输层，tcp或udp吞吐量，工具 `iperf` 或 `netperf`

  ```bash
  # -s表示启动服务端，-i表示汇报间隔，-p表示监听端口
  $ iperf3 -s -i 1 -p 10000
  
  # -c表示启动客户端，192.168.0.30为目标服务器的IP
  # -b表示目标带宽(单位是bits/s)
  # -t表示测试时间
  # -P表示并发数，-p表示目标服务器监听端口
  $ iperf3 -c 192.168.0.30 -b 1G -t 15 -P 2 -p 10000
  
  # 结果
  [ ID] Interval           Transfer     Bandwidth
  ...
  [SUM]   0.00-15.04  sec  0.00 Bytes  0.00 bits/sec                  sender
  [SUM]   0.00-15.04  sec  1.51 GBytes   860 Mbits/sec                  receiver
  ```

* HTTP 性能：应用层, 评估指标（每秒请求数、请求延迟、吞吐量以及请求延迟分布），工具`ab`

* 应用负载性能：应用层（用户请求负载），同上，工具`wrk`, `tcpcopy`, `jmeter`或`LoadRunner`



案例 1：DNS 解析失败

nslookup 解析失败 => ping 可达 => nslookup debug 模式 => 定位到DNS服务器配置错误(`/etc/resolv.conf`);

案例 2: DNS 解析不稳定

nslookup 解析延迟过久 => ping 延迟过久 => 更换DNS服务器配置

```bash
$ nslookup -debug {host}  # debug模式展示解析细节
$ dig +trace +nodnssec {host}  # trace解析的中间节点
```



案例 3: tcpdump 和 wireshark 分析网络流量 

理解协议的细节 https://www.rfc-editor.org/rfc-index.html; 《TCP/IP 详解》



<img src="https://static001.geekbang.org/resource/image/85/ff/859d3b5c0071335429620a3fcdde4fff.png" alt="tcpdump选项" style="zoom:33%;" />

<img src="https://static001.geekbang.org/resource/image/48/b3/4870a28c032bdd2a26561604ae2f7cb3.png" alt="tcpdump的使用" style="zoom:33%;" />



tcpdump 抓包例子：https://hackertarget.com/tcpdump-examples/

Wireshark Doc: https://www.wireshark.org/docs/



#### 案例分析



案例1：网络请求延迟分析

网络请求延迟分析：

1. 网络延迟导致传输慢;

   常用指标: RTT(Round-Trip Time) 衡量

   检测工具: `ping` => `ICMP`； 若`ICMP`被关闭（防止攻击）；可以使用`tcp`或者`udp`模式测量网络延迟;

   ```bash
   # -c表示发送3次请求，-S表示设置TCP SYN，-p表示端口号为80
   $ hping3 -c 3 -S -p 80 baidu.com
   HPING baidu.com (eth0 123.125.115.110): S set, 40 headers + 0 data bytes
   len=46 ip=123.125.115.110 ttl=51 id=47908 sport=80 flags=SA seq=0 win=8192 rtt=20.9 ms
   
   --- baidu.com hping statistic ---
   3 packets transmitted, 3 packets received, 0% packet loss
   round-trip min/avg/max = 20.9/20.9/20.9 ms
   
   
   # --tcp表示使用TCP协议，-p表示端口号，-n表示不对结果中的IP地址执行反向域名解析
   $ traceroute --tcp -p 80 -n baidu.com
   traceroute to baidu.com (123.125.115.110), 30 hops max, 60 byte packets
    1  * * *
    2  * * *
    3  * * *
    4  * * *
    5  * * *
    6  * * *
    7  * * *
    8  * * *
    9  * * *
   10  * * *
   11  * * *
   12  * * *
   13  * * *
   14  123.125.115.110  20.684 ms *  20.798 ms
   ```

   

2. Linux 内核协议栈报文处理慢；

   解析和分析协议, 工具: `tcpdump` 和 `wireshark`, 以及`strace`分析系统调用

3. 应用程序数据处理慢，导致延迟；

   单次请求和并发请求测试: `hping3` 和 `wrk`

常用分析工具:

* 使用 hping3 以及 wrk 等工具，确认单次请求和并发请求情况的网络延迟是否正常。
* 使用 traceroute，确认路由是否正确，并查看路由中每一跳网关的延迟。
* 使用 tcpdump 和 Wireshark，确认网络包的收发是否正常。
* 使用 strace 等，观察应用程序对网络套接字的调用情况是否正常。



案例2:  NAT 性能优化

