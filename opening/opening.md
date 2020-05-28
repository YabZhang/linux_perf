## 开篇词



1. 把观察到的性能问题跟系统原理关联起来，特别是把系统从应用程序、库函数、系统调用、再到内核和硬件等不同的层级贯穿起来。
2. 性能优化是软件系统最有挑战的工作之一，它也是最考验综合能力的工作之一。
3. 最好的学习方式一定是带着问题学习——“做中学”。
4. 学习要会抓重点：
   * 理解少数几个系统组件原理
   * 掌握基本的性能指标和工具
   * 学会常用的性能优化常用技巧
   * 分析和研究实际性能问题
   * 研读经典

5. 专栏内容结构：

   五个模块（CPU, 磁盘 I/O, 内存, 网络, 实战） &   四个篇章（原理基础、案例分析、套路总结、答疑）



总结：性能优化涉及到计算机系统的各个方面，所以分析和处理起来比较复杂。学习性能优化的做好方式是通过问题入手。首先掌握基本原理，然后通过指标和工具分析和定位，再结合常用的优化技巧，就可以解决大多数问题。通过持续的实战练习和理论知识的积累，以点带面，再通过阅读经典书籍，逐步建立起系统优化的全局观念。



## 如何学习



困难点：

1. “系统”、“底层”此类过于晦涩和深奥，无法深入学习和没有全局观；=> 战略错误
2. 性能问题太复杂，不懂分析的思路和方法，被迫“拿来主义”和生搬硬套，日后问题依旧；=> 战术错误



> 性能问题并没有你想象的那么难，**只要你理解了应用程序和系统的少数几个基本原理，在进行大量的实战练习，建立起整体性能的全局观**，大多数性能问题的优化就会水到渠成。----《Linux性能优化实战》



学习要领：

1. 了解*性能指标*

   * 应用角度(吞吐、延时) vs 系统资源(资源使用率、饱和度)；

   * 性能问题的本质：系统资源瓶颈但无法满足需求；找出瓶颈，避免或者缓解；

2. 性能分析步骤：

   * 选择指标评估应用程序和系统性能
   * 为应用程序和系统设置性能目标
   * 进行性能基准测试
   * 性能分析定位瓶颈
   * 优化系统和应用程序
   * 性能监控和告警

3. 建立整体系统性能分析的全局观

   * 理解最基本的几个系统知识原理（操作系统和计算机基本原理）=> 道
   * 掌握必要的性能工具（整理常用的调试工具集）=> 术
   * 通过实际的场景演练，贯穿不同的组件 （训练分析问题和解决问题的头脑）=> 内化知识和技巧

4. 选好工具，但工具不是全部

   Brendan Gregg 的 [Linux 性能工具图谱](https://static001.geekbang.org/resource/image/9e/7a/9ee6c1c5d88b0468af1a3280865a6b7a.png)

   [性能优化知识点思维导图 ](https://static001.geekbang.org/resource/image/0f/ba/0faf56cd9521e665f739b03dd04470ba.png)



高效建议：

* 虽然系统的原理很重要，但在刚开始一定不要试图抓住所有的实现细节。
  * 关注性能指标
  * 寻找指标观察工具
  * 导致指标变化的因素等
* 边学边实践，通过大量的案例演习掌握 Linux 性能分析和优化。
* 勤思考，多反思，善总结，多为为什么。



总结：

1. 性能优化困难的原因：涉及知识量多又广，要求正确的分析方法和手段；
2. 掌握性能优化应该循序渐进，由浅入深；掌握好原理，用好分析工具，多练多想；
