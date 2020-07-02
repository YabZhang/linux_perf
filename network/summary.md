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