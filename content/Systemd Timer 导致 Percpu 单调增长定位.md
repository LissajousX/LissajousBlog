---
date: 2025-12-29T10:21:56+08:00
title: Systemd Timer 导致 Percpu 单调增长定位
description: Systemd Timer 导致 Percpu 单调增长定位
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - OS
categories: Tutorials
---

## 1. 背景与现象

  

我们在多台 Ubuntu 系统上运行 AiLink5GS 相关服务。由于系统中启用了较多的 `systemd timer`（部分为秒级高频触发），观察到 `/proc/meminfo` 中 `Percpu` 字段**单调升高**：

  

- 启动 timer 之后，`Percpu` 每隔几秒增加几百 KB

- 停止 timer 后，增长明显变慢或停止（已通过实验验证）

  

这里的 `Percpu` 指 **Linux 内核 per-cpu allocator（每 CPU 专用内存）的总量**，不是用户态进程 RSS。

  

> 重要提醒：percpu allocator 经常表现为“只增不减”（高水位/碎片化导致），因此排查与验收应以“增长是否停止/显著放缓”为准，而不是期望 `Percpu` 立即下降。

  

## 2. 环境信息（样例机器）

  

在一台复现机器上采集到如下版本信息：

  

```bash

uname -r

systemd --version

stat -fc %T /sys/fs/cgroup

```

  

输出：

  

```text

6.8.0-88-generic

systemd 255 (255.4-1ubuntu8.8)

...

cgroup2fs

```

  

结论：

  

- Ubuntu 24.04.2 LTS

- Kernel 6.8.0

- systemd 255

- **cgroup v2 (unified hierarchy)**

  

## 3. 初步观测与排除思路

  

### 3.1 确认 cgroup 数量（排除“cgroup 爆炸”）

  

```bash

find /sys/fs/cgroup -type d | wc -l

```

  

样例输出：

  

```text

71

```

  

结论：

  

- 该机器 cgroup 目录数量很小（71），**不是**典型的 transient unit / run-*.scope 无限制累积导致的 cgroup 数量爆炸。

  

### 3.2 观测 /proc/meminfo

  

```bash

grep -E 'Percpu|Slab|SReclaimable|SUnreclaim' /proc/meminfo

```

  

样例输出：

  

```text

Slab:            7021364 kB

SReclaimable:    1001096 kB

SUnreclaim:      6020268 kB

Percpu:         21614240 kB

```

  

说明：

  

- `Percpu` 达到约 21GB，明显异常。

- 同时 `SUnreclaim` 也较大（6GB），属于另外一条线（slab 不可回收增长），本次重点定位 `Percpu` 单调增长。

  

### 3.3 查看 systemd cgroup 树/定时器/服务状态

  

```bash

systemd-cgls --no-pager | head -n 200

systemctl list-timers

systemctl list-units --state=failed

```

  

此处主要用于确认：

  

- 是否存在大量 `run-*.scope` transient unit

- 哪些 timer 为高频触发

- 是否存在失败风暴（失败会额外增加 churn，但后续证明即使服务成功也会增长，因此失败不是根因）

  

## 4. 排除“CPU possible 过大导致 Percpu 天生很大”

  

在某些场景，若系统 `possible CPU` 极大（例如 `0-4095`），percpu 结构预留可能异常大。

  

本次复现机器采集：

  

```bash

nproc

lscpu | egrep 'CPU\(s\)|On-line CPU\(s\) list|Thread\(s\) per core|Core\(s\) per socket|Socket\(s\)|NUMA'

cat /sys/devices/system/cpu/possible

cat /sys/devices/system/cpu/online

dmesg | grep -i 'percpu' | head -n 20

```

  

样例输出：

  

```text

34

NUMA node(s):                         2

NUMA node0 CPU(s):                    0-9,20-29

NUMA node1 CPU(s):                    10-19,30-39

0-39

0-39

[    0.186778] setup_percpu: NR_CPUS:8192 nr_cpumask_bits:40 nr_cpu_ids:40 nr_node_ids:2

[    0.191373] percpu: Embedded 86 pages/cpu s229376 r8192 d114688 u524288

```

  

分析结论：

  

- `possible=0-39`、`online=0-39`，`nr_cpu_ids=40`，CPU 枚举正常。

- 启动阶段的 embedded percpu（86 pages/cpu）量级约 `86 * 4KB * 40 ≈ 13MB`，**不足以解释** `Percpu ≈ 21GB`。

  

因此：

  

- 当前的异常 `Percpu` 主要来自运行过程中不断发生的 **动态 per-cpu 分配**。

  

## 5. 从 vmallocinfo 观察 percpu 动态分配痕迹

  

```bash

grep -E 'Percpu|VmallocUsed|VmallocTotal|VmallocChunk|Slab|SReclaimable|SUnreclaim|BPF' /proc/meminfo

sudo grep -iE 'pcpu|percpu' /proc/vmallocinfo | head -n 80

```

  

样例输出：

  

```text

VmallocTotal:   34359738367 kB

VmallocUsed:     1275348 kB

VmallocChunk:          0 kB

Percpu:         21629600 kB

```

  

`/proc/vmallocinfo` 片段（节选）：

  

```text

... pcpu_mem_zalloc+0x30/0x70 pages=4 vmalloc ...

... pcpu_mem_zalloc+0x30/0x70 pages=5 vmalloc ...

... irq_init_percpu_irqstack+0x114/0x1b0 vmap ...

```

  

解释：

  

- 出现大量 `pcpu_mem_zalloc` 相关条目，说明曾经发生过 percpu allocator 的 chunk 扩容/分配。

- 但 `pcpu_mem_zalloc` 并不会在每一次 `alloc_percpu` 调用时触发（仅在 chunk 扩容/填充等时刻），因此后续用它作为 kprobe 点不一定能稳定观测到。

  

## 6. eBPF 定位：抓 __alloc_percpu_gfp 的触发者

  

### 6.1 为什么最初 kprobe:pcpu_mem_zalloc 看不到输出

  

我们曾尝试：

  

```bash

sudo bpftrace -e 'kprobe:pcpu_mem_zalloc { @[kstack(20)] = count(); }'

```

  

观察 1~2 分钟没有输出，原因通常是：

  

- `pcpu_mem_zalloc()` 不是每次都会走；只有在 percpu allocator 需要扩容/建立新 chunk 才会触发。

- bpftrace 对聚合 map 的输出也常常需要 `Ctrl-C` 才打印（未显式 `print(@map)` 时尤其明显）。

  

因此，本次定位切换到 `__alloc_percpu_gfp`（更底层、更通用的 percpu 分配入口）。

  

### 6.2 按进程名统计：5 秒窗口内谁在触发 alloc_percpu

  

命令：

  

```bash

sudo bpftrace -e '

kprobe:__alloc_percpu_gfp

{

  @cnt[comm] = count();

  @bytes[comm] = sum(arg0);

}

  

interval:s:5

{

  printf("---- 5s window ----\n");

  print(@cnt);

  print(@bytes);

  clear(@cnt);

  clear(@bytes);

}

'

```

  

样例输出（两次 5s 窗口）：

  

```text

---- 5s window ----

@cnt[systemd]: 36

@cnt[bash]: 84

@cnt[watching_up.sh]: 113

...

@bytes[bash]: 1344

@bytes[watching_up.sh]: 1808

@bytes[systemd]: 8508

  

---- 5s window ----

@cnt[systemd]: 36

@cnt[bash]: 84

@cnt[watching_up.sh]: 109

...

@bytes[bash]: 1344

@bytes[watching_up.sh]: 1744

@bytes[systemd]: 8508

```

  

关键解释：

  

- `__alloc_percpu_gfp(arg0)` 的 `arg0` 是“每 CPU 的对象大小（bytes/CPU）”。

- 真实消耗粗略估算：

  

```text

real_bytes ≈ arg0 * nr_cpu_ids

```

  

该机器 `nr_cpu_ids=40`，所以仅 `systemd` 在 5 秒窗口的估算量级：

  

```text

8508 * 40 ≈ 340KB/5s

```

  

再加上 `watching_up.sh`、`bash` 等进程的分配，合计就与“几秒几百 KB 增长”的现象高度吻合。

  

初步结论：

  

- `Percpu` 增长主要由 `systemd`（其次是高频脚本/子进程）驱动。

  

## 7. eBPF 深入：抓 systemd 的内核栈（定位到 cgroup v2 memcg）

  

为了明确 `systemd` 为什么会触发 percpu 分配，进一步抓 `systemd` 的 `__alloc_percpu_gfp` 调用栈：

  

命令：

  

```bash

sudo bpftrace -e '

kprobe:__alloc_percpu_gfp /comm=="systemd"/

{

  @[kstack(20)] = sum(arg0);

}

  

interval:s:10

{

  printf("---- 10s window (systemd stacks) ----\n");

  print(@);

  clear(@);

}

'

```

  

样例输出（节选，10 秒窗口）：

  

```text

@[

    __alloc_percpu_gfp+1

    mem_cgroup_css_alloc+61

    css_create+73

    cgroup_apply_control_enable+428

    cgroup_mkdir+213

    kernfs_iop_mkdir+90

    vfs_mkdir+421

    do_mkdirat+327

    __x64_sys_mkdir+74

    ...

]: 16512

  

@[

    __alloc_percpu_gfp+1

    cgroup_bpf_inherit+91

    cgroup_create+700

    cgroup_mkdir+163

    ...

]: 48

  

@[

    __alloc_percpu_gfp+1

    cgroup_create+147

    cgroup_mkdir+163

    ...

]: 48

  

@[

    __alloc_percpu_gfp+1

    mm_init+767

    dup_mm.constprop.0+81

    copy_process+3034

    ...

]: 96

```

  

关键分析：

  

- **占比最大的栈**是：

  

```text

mem_cgroup_css_alloc -> css_create -> cgroup_apply_control_enable -> cgroup_mkdir -> vfs_mkdir

```

  

这说明：

  

- `systemd` 在执行某些操作时，会触发 `mkdir` 创建 cgroup 目录（cgroup v2）

- 在 cgroup 创建/启用 controller（尤其是 memory controller/memcg）过程中，内核需要为 memcg 分配 per-cpu 统计/会计结构

- 该分配路径最终进入 `__alloc_percpu_gfp`

  

并且该条目显示 `sum(arg0)=16512`，即：

  

- 每次创建相关 cgroup 时会分配 `16512 bytes/CPU`

- 本机 `nr_cpu_ids=40`，粗略估算单次创建成本：

  

```text

16512 * 40 ≈ 660KB/次

```

  

这就能直接解释：

  

- 为什么 timer 触发频率很高时，`Percpu` 会以“几秒几百 KB”的速度增长

  

结论：

  

- 本次 `Percpu` 单调增长的核心根因是：

  

> **高频 systemd timer 触发短命 service** → systemd 频繁创建/切换 unit 的 cgroup（`cgroup_mkdir`） → **cgroup v2 memcg 在 css_create/mem_cgroup_css_alloc 中触发 alloc_percpu** → percpu allocator 高水位被持续推高 → `/proc/meminfo` 的 `Percpu` 表现为单调增长。

  

## 8. 结论汇总

  

- **不是** CPU possible 异常导致的启动即巨大：`possible/online` 都为 `0-39`。

- `Percpu` 的增长与 `systemd` 的 `__alloc_percpu_gfp` 强相关。

- systemd 的主要触发栈落在 `cgroup_mkdir` 并进入 `mem_cgroup_css_alloc`，说明与 **cgroup v2 + memcg controller** 创建/启用过程密切相关。

- 对于“秒级高频触发的短命 service”，即使 service 成功，也会产生同样问题（失败与否不是根因）。

  

## 9. 整改建议（按推荐优先级）

  

### 9.1 最高优先级：用“常驻服务 + 内部 sleep”替代“高频 timer + 短命 service”

  

适用：`1s/5s/10s` 级别的健康检查、依赖检测、采集脚本等。

  

思路：

  

- 把 timer 触发的脚本拆成 `xxx_once.sh`（只做一次检查）

- 用一个常驻 service 循环执行：

  

```bash

while true; do

  /path/xxx_once.sh

  sleep 5

done

```

  

收益：

  

- cgroup 只创建一次，避免反复 `cgroup_mkdir`

- `mem_cgroup_css_alloc` 的 alloc_percpu 触发频率大幅降低

  

### 9.2 合并 timers

  

把多个高频 timer 合并为 1 个 timer，由一个 service 内顺序执行多个检查任务，减少 unit/cgroup 生命周期 churn。

  

### 9.3 对短命任务关闭不必要的 Accounting（减轻统计开销）

  

在这些 service 上显式关闭：

  

- `CPUAccounting=no`

- `IOAccounting=no`

- `MemoryAccounting=no`

- `TasksAccounting=no`

  

说明：

  

- 该措施的效果依赖于系统/发行版对 controller 启用策略，但通常能减少 systemd 在 cgroup 上做的额外统计动作。

  

### 9.4 评估系统级策略（谨慎）：避免启用/使用 memcg controller

  

由于本次根因明确落在 `mem_cgroup_css_alloc`，若业务不依赖 cgroup memory 限制、systemd-oomd 等能力，可评估系统层面降低对 memcg 的使用。

  

> 注意：这是系统策略调整，影响面可能较大（容器、资源限制、oomd 等），需要单独评审。

  

## 10. 验收方法（推荐标准流程）

  

### 10.1 验收目标

  

- `Percpu` 不再按“几秒几百 KB”持续增长

- `systemd` 的 `mem_cgroup_css_alloc -> cgroup_mkdir` 相关 alloc_percpu 触发显著下降

  

### 10.2 验收命令

  

- **观察 meminfo：**

  

```bash

watch -n 1 "grep -E 'Percpu|Slab|SUnreclaim|BPF|VmallocUsed' /proc/meminfo"

```

  

- **观察 systemd 的 alloc_percpu 栈是否仍高频出现：**

  

```bash

sudo bpftrace -e '

kprobe:__alloc_percpu_gfp /comm=="systemd"/ { @[kstack(20)] = sum(arg0); }

interval:s:10 { print(@); clear(@); }

'

```

  

### 10.3 重要注意事项

  

- 修复后 `Percpu` **可能不会立刻下降**（allocator 高水位）。

- 评估时以“增长停止/显著放缓”为准。

- 若需要将 `Percpu` 恢复到较低水平，通常需要重启（reboot）后在新策略下运行。

  

## 11. 附录：本次定位用到的命令清单（汇总）

  

### 11.1 cgroup/timer/service 信息

  

```bash

find /sys/fs/cgroup -type d | wc -l

grep -E 'Percpu|Slab|SReclaimable|SUnreclaim' /proc/meminfo

systemd-cgls --no-pager | head -n 200

systemctl list-units --state=failed

systemctl list-timers

systemctl status <xxx>.service

cat /usr/lib/systemd/system/<xxx>.service

uname -r

systemd --version

stat -fc %T /sys/fs/cgroup

```

  

### 11.2 CPU/percpu 启动信息

  

```bash

nproc

lscpu | egrep 'CPU\(s\)|On-line CPU\(s\) list|Thread\(s\) per core|Core\(s\) per socket|Socket\(s\)|NUMA'

cat /sys/devices/system/cpu/possible

cat /sys/devices/system/cpu/online

dmesg | grep -i 'percpu' | head -n 20

```

  

### 11.3 内核内存/分配痕迹

  

```bash

grep -E 'Percpu|VmallocUsed|VmallocTotal|VmallocChunk|Slab|SReclaimable|SUnreclaim|BPF' /proc/meminfo

sudo grep -iE 'pcpu|percpu' /proc/vmallocinfo | head -n 80

```

  

### 11.4 bpftrace 定位

  

```bash

# 1) (不推荐作为主手段) 仅在 percpu allocator 扩容/填充时更可能触发

sudo bpftrace -e 'kprobe:pcpu_mem_zalloc { @[kstack(20)] = count(); }'

  

# 2) 通用入口：统计谁在触发 percpu 分配

sudo bpftrace -e '

kprobe:__alloc_percpu_gfp

{

  @cnt[comm] = count();

  @bytes[comm] = sum(arg0);

}

interval:s:5

{

  printf("---- 5s window ----\n");

  print(@cnt);

  print(@bytes);

  clear(@cnt);

  clear(@bytes);

}

'

  

# 3) systemd 具体内核栈（定位到 memcg/cgroup_mkdir）

sudo bpftrace -e '

kprobe:__alloc_percpu_gfp /comm=="systemd"/

{

  @[kstack(20)] = sum(arg0);

}

interval:s:10

{

  printf("---- 10s window (systemd stacks) ----\n");

  print(@);

  clear(@);

}

'

```