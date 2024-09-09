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



## 内存泄漏定位
