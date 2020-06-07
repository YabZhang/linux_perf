## 答疑篇笔记



> 性能分析的原理和思路，性能工具只是我们的路径和手段



1. 先熟练使用工具，然后再返过来学习原理；=> 经典的what, how, why模式
2. 使用docker环境可以限制资源来更好的模拟问题
3. 使用perf工具时，看到的时16进制地址而不是函数名
   * 在容器内查看结果`perf report`

4. 一开始不要陷入单一语言

5. 推荐书籍和资料

   * 《Systems Performance: Enterprise and the Cloud》中文名《性能之巅：洞悉系统、企业与云计算》
   * http://www.brendangregg.com/， 尤其是http://www.brendangregg.com/linuxperf.html

   ![Linux性能观测工具](http://www.brendangregg.com/Perf/linux_observability_tools.png)