## 套路篇



#### 评估网络性能

确定目标网络的协议栈 —— 低层性能决定高层性能；

* 转发性能：网络层，指标PPS，工具`hping3`和`pktgen`

* TCP/UDP 性能：传输层，tcp或udp吞吐量，工具`iperf`或`netperf`

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