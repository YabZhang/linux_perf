## linux 文件系统



#### 基本概念

* 索引节点, 简称`inode`，记录文件元数据，比如inode编号、文件大小、访问权限、修改时间、数据位置等；索引节点和文件一一对应，会被持久化到磁盘中；
* 目录项，简称`dentry`，记录文件名字、索引节点指针以及与其他目录的关联关系。目录项是由内核维护的内存数据结构，通常也被叫做目录项缓存(cache)；

索引节点是每个文件的唯一标识，目录项维护文件系统的树状结构；

一个文件可以有别名，目录项和索引节点是多对一；

磁盘读写最小单位是扇区（512B）；连续的扇区组成逻辑块（最小的管理单元， 4KB）；



<img src="https://static001.geekbang.org/resource/image/32/47/328d942a38230a973f11bae67307be47.png" alt="文件系统基本组成" style="zoom:67%;" />



* 虚拟文件系统（VFS）, 定义了一组所有文件系统都支持的数据结构和标准接口；



<img src="https://static001.geekbang.org/resource/image/72/12/728b7b39252a1e23a7a223cdf4aa1612.png" alt="虚拟文件系统" style="zoom:67%;" />



实际的文件系统（可以位于磁盘、内存、网络），这些文件系统要挂载到VFS目录树的某个子目录（挂载点）；



* 文件系统IO；挂载后就可以通过标准接口（系统调用）来访问；
  1. 根据是否利用标准库缓存，可分为`缓冲IO`和`非缓冲IO`；
  2. 根据是否利用操作系统页缓存，可分为`直接IO`和`非直接IO`;
  3. 根据是否阻塞运行，分为`阻塞IO`和`非阻塞IO`;
  4. 根据是否等待响应结果，分为`同步IO`和`异步IO`;



* 性能指标

  1. 容量,文件系统磁盘空间； 

  ```bash
  # 查看磁盘空间
  $ df /dev/sda1
  $ df -h /dev/sda1
  $ df -i /dev/sda1  # 展示索引节点个数
  ```

  2. 缓存，主要是页缓存cache和buffer；内核使用slab机制管理目录项和索引节点缓存;

  ```bash
  $ cat /proc/meminfo | grep -E 'SReclaimable|Cached'  # 查看缓存
  Cached: 748316 kB   # cache， 页缓存和可回收的Slab缓存
  SwapCached: 0 kB    # 交换分区缓存
  SReclaimable: 179508 kB   # Slab
  
  $ cat /proc/slabinfo | grep -E '^#|dentry|inode'  # 所有的目录项和各种文件系统索引节点缓存; 
  $ slabtop  # 使用slabtop来查看
  ```

  