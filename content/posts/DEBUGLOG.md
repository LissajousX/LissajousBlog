---
date: 2024-09-05T11:13:06+08:00
title: 故障定位记录
description: 记录一些难以定位的故障定位经历
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - operation
categories: Tutorials
---
## CPU指令集缺失导致的BUG
### 故障描述

在某次部署调试中，发现我们的软件运行在工控机平台时会崩溃，通过GDB的`disassemble $pc` 命令看到如下信息：

![GDB调试信息](1.png)

### 定位过程

vmovdqu64指令出错，查intelSDM文档定位到相关指令集为AVX512，如图所示：  

![GDB调试信息](2.PNG)

之所以出现此问题就是在编译机器上，cpu支持AVX512指令集，然而在部署机器上cpu不支持此指令集。具体可以通过`lscpu`查看cpu信息中的`flags`中是否包含`avx512`指令集。

> AVX-512（Advanced Vector Extensions 512）是Intel推出的一组指令集扩展，用于提供更高的并行计算能力和更大的数据处理带宽。它是AVX（Advanced Vector Extensions）系列的扩展，支持512位宽的矢量操作。AVX-512旨在提高高性能计算（HPC）、数据分析、机器学习和科学计算等领域的处理效率。
> 
> **AVX-512的主要特性**：
> 
> 1. **512位矢量寄存器**：
>     
>     - 引入了64个512位ZMM寄存器（ZMM0-ZMM31），提供比之前的256位YMM寄存器更大的数据宽度，可以在单次操作中处理更多的数据。
> 2. **掩码寄存器**：
>     
>     - 提供了8个掩码寄存器（k0-k7），用于控制每个矢量元素的操作。这使得处理器能够对矢量中的每个元素进行选择性操作，提高了计算的灵活性和效率。
> 3. **改进的指令集**：
>     
>     - 增强了对浮点运算、整数运算和逻辑运算的支持，包括更复杂的计算操作和新的指令集，如聚合、条件操作等。
> 4. **更高的并行性**：
>     
>     - 支持更高水平的并行计算，能够同时处理更多的数据元素，从而提升计算密集型应用的性能。
> 
> **AVX-512的子集**
> 
> AVX-512指令集包含多个子集，每个子集提供特定的功能和优化。例如：
> 
> - **AVX-512F（Foundation）**：AVX-512的基础指令集，所有支持AVX-512的处理器必须支持。
> - **AVX-512VL（Vector Length Extensions）**：扩展AVX-512支持256位和128位操作。
> - **AVX-512DQ（Doubleword and Quadword）**：增加对双字（64位）和四字（128位）整数的支持。
> - **AVX-512BW（Byte and Word）**：增强对字节和字（16位）整数的支持。
> - **AVX-512IFMA（Integer Fused Multiply-Add）**：支持整数融合乘加操作。
> - **AVX-512VBMI（Vector Bit Manipulation Instructions）**：提供矢量位操作指令。
> 
> **应用领域**
> 
> AVX-512主要用于需要大规模并行计算的应用场景，如：
> 
> - 科学计算
> - 数据分析
> - 高性能计算（HPC）
> - 机器学习和人工智能
> - 图形处理
> 
> 总的来说，AVX-512通过提供更宽的数据路径和更高的并行处理能力，显著提升了计算密集型任务的性能。

### 解决方法

因此解决方案就是在编译过程中禁用此指令集，本项目中则是通过增加`-mno-avx512f`编译参数来解决。

[参考链接：x86 linux AVX512指令引起的进程crash](https://cloud.tencent.com/developer/article/1356935)

[参考链接：Intel SDM 文档](https://cdrdv2.intel.com/v1/dl/getContent/671200)

---
## DPDK遇到的网卡驱动BUG

### 故障描述

在对E810型号的网卡绑定DPDK驱动后执行程序会报如下错误：

```
ice_program_hw_rx_queue(): currently package doesn'tsupport RXDID (22)
ice_rx_queue_start(): fail to program RX queue 0
ice_dev_start(): fail to start Rx queue 0
EAL: Error - exiting with code: 1
Cause: rte_eth_dev_start:err -5, port 0
```

### 定位过程

通过万能的Google找到了如下链接，似乎是同样的问题。

[https://inbox.dpdk.org/users/3725F2FC-40C5-41E9-A00F-3EC3EADCEBB0@intel.com/](https://inbox.dpdk.org/users/3725F2FC-40C5-41E9-A00F-3EC3EADCEBB0@intel.com/)

### 解决方法

更新网卡的DDP版本即可。

DDP文件链接：[https://www.intel.com/content/www/us/en/download/19660/29889/intel-ethernet-800-series-dynamic-device-personalization-ddp-for-telecommunication-comms-package.html](https://www.intel.com/content/www/us/en/download/19660/29889/intel-ethernet-800-series-dynamic-device-personalization-ddp-for-telecommunication-comms-package.html)

---
## 云平台CPU调度故障BUG

### 故障现象

原本的高可用部署的软件部署在虚机集群上，基础硬件是两台物理机，高可用部署用到了zookeeper组成三节点。发现虚机重启软件表现正常，可以主备倒换，但是一旦物理机掉电，主备倒换就不正常了。

### 定位过程

起初怀疑网络发生断链，集群对网络的管理出现问题，后面发现是乌龙。

然后观察到zookeeper的日志的打印了写文件延时过高的警告日志，怀疑是存储管理的问题，还写了个脚本不断的写入文件来测试，结果也没发现异常。

最后通过top命令观察cpu的调度才发现物理机掉电时，对侧正常机器上的多个cpu的idle占比都马上变为0了，据此推测是平台对cpu的管理问题，导致我们的软件长时间无法获得cpu时间，因此出现问题。

后面进一步确认是两台物理机在存储资源上存在依赖关系。当一台物理机掉电时，另一台物理机上的虚拟机可能会尝试访问掉电物理机上的存储资源。在这种情况下，系统会进行 I/O 切换，但底层存在一个检测机制，导致 I/O 资源不能立即切换到可用的存储。这个过程可能需要长达 30-40 秒的时间，期间 I/O 操作暂停，导致虚拟机上的软件运行受到影响，甚至出现业务中断。


> `top` 命令是 Linux 系统中一个非常常用的实时任务监控工具。它能够显示系统中各个进程的状态，包括 CPU 和内存的使用情况。在 `top` 命令的输出中，有几行是与 CPU 使用相关的。以下是这些输出的详细解释：
> 
> **1. CPU 状态概要**
> 
> 当你运行 `top` 命令时，屏幕上方有一行关于 CPU 使用情况的概要信息，类似于：
> 
> scss
> 
> 复制代码
> 
> `%Cpu(s):  1.0 us,  0.5 sy,  0.0 ni, 98.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st`
> 
> 每个字段的含义如下：
> 
> - **us (user)**: 用户空间的 CPU 使用百分比。表示用户进程（不包括 nice 优先级调整的进程）所占用的 CPU 时间百分比。
>     
> - **sy (system)**: 内核空间的 CPU 使用百分比。表示系统内核进程占用的 CPU 时间百分比。
>     
> - **ni (nice)**: 具有正 nice 值的用户进程的 CPU 使用百分比。表示通过 nice 值调低优先级的进程所占用的 CPU 时间百分比。
>     
> - **id (idle)**: CPU 空闲百分比。表示 CPU 处于空闲状态的时间百分比。
>     
> - **wa (iowait)**: 等待 I/O 操作的 CPU 时间百分比。表示 CPU 等待磁盘或网络 I/O 操作完成所占的时间百分比。
>     
> - **hi (hardware interrupts)**: 处理硬件中断的 CPU 时间百分比。表示 CPU 处理硬件中断请求所占用的时间百分比。
>     
> - **si (software interrupts)**: 处理软件中断的 CPU 时间百分比。表示 CPU 处理软件中断请求所占用的时间百分比。
>     
> - **st (steal time)**: 虚拟机“偷取”时间的百分比。表示虚拟机监控程序在其他虚拟机上花费的时间，这段时间对当前虚拟机来说是被“偷走”的。
>     
> 
> **2. 每个进程的 CPU 使用情况**
> 
> 在 `top` 命令输出的下方，有一个表格显示系统中每个进程的详细信息。其中与 CPU 使用相关的字段包括：
> 
> - **PID**: 进程的 ID。
>     
> - **%CPU**: 该进程占用的 CPU 使用百分比。这个值表示该进程占用的 CPU 资源占总体 CPU 资源的百分比。
>     
> - **TIME+**: 该进程在启动后累计的 CPU 时间（单位为 1/100 秒）。这是进程使用的总 CPU 时间。

### 解决方法

修改虚机只用所在物理机的存储，在线的虚机不会去读取关机那台物理机上面的磁盘即可。

## 内存泄漏定位

jemalloc 工具定位内存泄漏问题。

[Heap-Profiling](https://github.com/jemalloc/jemalloc/wiki/Use-Case%3A-Heap-Profiling?open_in_browser=true)

[C/C++ Jemalloc + Jeprof内存泄漏分析](https://blog.csdn.net/bandaoyu/article/details/137574343)