## C10K 和 C1000K 回顾



* C10K 

  C 10k 问题, 1999, Dan Kegel 提出；[C10K 问题 ](http://www.kegel.com/c10k.html); 

  C10K 之前的 时代，网络请求处理多采用多进程（多线程）模型—— 一个请求一个进程(线程)；

1. 如何能够在单个线程中处理多个请求；==> IO模型优化(IO复用)
2. 怎么更节省资源地处理客户请求；==> 工作模型优化(线程复用)



IO事件通知方式:

1. 水平触发: fd可以非阻塞地执行IO, 就会触发通知；应用程序可随时检查fd状态，然后根据状态操作IO;
2. 边缘触发: 仅在fd状态改变时(如IO请求达到), 才发送一次通知; 因为仅通知一次，要在单次通知完成全部IO；



IO 模型优化:

1. 非阻塞IO + 水平触发通知, 如select 或 poll

   select 和 poll 从监听的fd列表中，遍历查找可执行的IO，然后进行非阻塞的IO操作; 

   不足: select 有fd列表长度限制(32位时默认1024)，并采用轮询方式查询状态; poll 替换为没有固定长度的数组，移除了长度限制，但是凸显了轮询的效率低的问题；此外select 和 poll 需要频繁系统调用传递fd集合；

2. 非阻塞IO + 边缘触发通知, 如epoll

   epoll 使用红黑树在内核管理fd集合，不需要频繁系统调用；epoll 使用事件驱动机制，只关注有IO事件发生的fd，无需遍历整个fd集合；Linux 2.6以后才支持epoll;

   不足：边缘触发通知只在fd的状态转换时发送一次，因此要在单次通知触发完成IO操作并要处理更多事件；

3. 异步IO(Async IO, 简称AIO)

   异步IO允许应用程序发起大量IO操作，而不用等待操作完成；待IO完成后，系统会用事件通知(如信号或回调函数)的方式，通知应用程序。此时，应用程序才回去查询IO操作结果；Linux 2.6以后支持，设计较为繁琐；



工作模型优化：

1. 主进程 + 多个worker子进程

   主进程bind + listen, 创建多个子进程；然后在每个子进程中通过accept或epoll_wait来处理相同套接字;

   <img src="https://static001.geekbang.org/resource/image/45/7e/451a24fb8f096729ed6822b1615b097e.png" alt="主进程+多worker子进程模型" style="zoom:50%;" />

不足: accept 和 epoll_wait 惊群问题。网络IO事件发生时，多个进程同时被唤醒，但实际只需一个进程来响应事件，其他重新休眠；accept 在 2.6版本内核已解决, epoll 在 4.5以后才解决;

2. 监听到相同端口的多进程模型

   所有进程都监听相同端口，并开启`SO_REUSEPORT` (需要Linux 内核 3.9 以上)选项，由内核将请求负载均衡到监听进程中。

<img src="https://static001.geekbang.org/resource/image/90/bd/90df0945f6ce5c910ae361bf2b135bbd.png" alt="多进程监听相同端口" style="zoom:50%;" />



​	Nginx 1.9.1 中已支持此模式。官方Benchmarks 性能可提升 x2 - 3 倍; 阻塞情况可能会影响链接的后续请求；

​	https://www.nginx.com/blog/socket-sharding-nginx-release-1-9-1/#upgrade



* C1000K（百万）

  通过上述的IO模型和工作模型的优化，C10K问题很容易就可以解决；

  随之而来的 C100K 问题也可在 C10K的理论上，epoll 配合线程池，以及更强的硬件资源自然达到；

  C1000K的问题要求大量的系统资源，如至少几十G的内存，万兆或更大的吞吐量；此外，大量的连接也会占用大量资源，如fd数量、连接状态跟踪、网络协议栈缓存（套接字读写缓存、TCP读写缓存）等; 而且大量中断处理的成本也会非常高，可能需要多队列网卡、中断负载均衡、CPU绑定、RPS/RFS、以及硬件解包处理等各种硬件和软件优化；

  C1000K 的解决方法，本质上还是构建在 epoll 的非阻塞IO模型上，但还需要从应用程序到Linux内核、再到CPU、内存和网络等各个层次的深度优化，特别是需要借助硬件来卸载原有的软件功能；



* C10M（千万）

  在C1000K问题中，各种软、硬件的性能基本已经发挥做到了极致。通过优化应用程序和内核来满足千万并发已经是及其困难的。有关C10M问题的探讨, http://c10m.robertgraham.com/p/blog-page.html.

  其中主要是内核协议栈开销巨大。从网卡硬中断处理、到软中断的各层协议处理、最后再到应用程序的整个处理链路过长。那么跳过内核协议栈把网络包直接送到应用程序将有机会达到C10M的目标。两种常见的机制 DPDK 和 XDP。

1. DPDK(Intel Data Plane Development Kit), 用户态网络标准; 

   DPDK 跳过内核协议栈，直接由用户态进程通过轮询方式来处理接收到的网络包。

   相关介绍参见: https://blog.selectel.com/introduction-dpdk-architecture-principles/

   <img src="https://static001.geekbang.org/resource/image/99/3a/998fd2f52f0a48a910517ada9f2bb23a.png" alt="DPDK" style="zoom: 33%;" />

2. XDP(eXpress Data Path), 是Linux内核提供的一种高性能数据路径，允许网络包在进入内核协议站前进行处理以实现高性能。XDP的底层和bcc-tools一样，基于Linux内核的eBPF机制实现。

   [XDP 要求内核 4.8 以上](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md#xdp)，不提供缓存队列，通常是专用网络应用，如IDS(入侵检测系统)、DDoS防御、[cilium 容器网络插件](https://github.com/cilium/cilium)等. 相关介绍参见: https://www.iovisor.org/technology/xdp

   <img src="https://static001.geekbang.org/resource/image/06/be/067ef9df4212cd4ede3cffcdac7001be.png" alt="XDP原理图" style="zoom: 50%;" />

* 总结

  首先回顾了经典的C10K问题，其根源一方面是系统资源限制；另一方面是同步阻塞的IO模型以及轮询socket限制了网络处理效率。Linux 2.6 中引入了 epoll 完美解决C10K问题, 目前的高性能网络方案都基于epoll；

  从C10K 到 C100K，可能增加系统的物理资源就可以满足；但到了C1000K 就不仅仅是物理资源的问题，这事需要的是多方面和全方位的优化工作，从硬件中断处理和网络解包、到网络协议栈的fd数量、连接状态跟踪、缓存队列等内核优化，再到应用程序的工作模型优化等，都是涉及点。

  再进一步，若要实现C10M，就不能简单通过优化软硬件来实现了，而必须从网络包处理流程上进行调整。使用DPDK 跳过笨重的内核网络协议栈，或 XDP 的方式在网络协议栈之前处理包；大多数场景并不需要单机C10M，通过系统架构把请求分发到多台服务器处理是更简单和易拓展的方案。