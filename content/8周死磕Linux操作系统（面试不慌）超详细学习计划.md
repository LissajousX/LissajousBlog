---
date: 2025-12-29T10:23:02+08:00
title: 8周死磕Linux操作系统（面试不慌）超详细学习计划
description: 8周死磕Linux操作系统（面试不慌）超详细学习计划
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - OS
categories: Tutorials
---

## 0. 计划使用说明（先读这个，避免两个月白忙）

  

这个计划目标是：你在 8 周内把 Linux 操作系统相关的面试与实战能力拉到「不慌」的水平。

  

你会得到的能力不是“背概念”，而是：

  

- **能解释**：把面试问题讲成机制 + 例子 + 取舍

- **能定位**：遇到线上 CPU/内存/IO/网络问题，有稳定的排障路径

- **能量化**：能用数据和实验验证假设，而不是猜

- **能落地**：给出止血与长期治理方案，并能验收

  

### 0.1 每天学习的固定节奏（建议 2.5 小时/天）

  

- **第1段（30min）概念学习**：看书/文章/讲义，做 10 条以内笔记

- **第2段（60min）动手实验**：必须跑命令或写小程序，得到“现象→数据”

- **第3段（30min）复盘输出**：写 10 行总结：现象、原因、证据、结论、下一步

- **第4段（30min）面试自测**：回答当天自测题（口头复述 + 写答案都行）

  

如果你每天只有 1 小时：保留第2段（实验）+ 第3段（复盘）。

  

### 0.2 你必须遵守的两条纪律（否则学不成）

  

- **一次只验证一个变量**：做实验必须有对照组（A/B），不然得不出可靠结论

- **每周必须有产出物**：脚本/实验记录/小程序/1页总结，不能只有“看了很多”

  

### 0.2.1 防止陷入细节与挫败感（强制执行规则）

  

- **学习模式分离**

  - **扫盲模式（80%时间）**：目标是“能用 3 句话讲清楚 + 能跑出一个现象 + 能给出排查路径”。禁止深挖源码/论文级细节。

  - **深挖模式（20%时间）**：只有在“现象/证据解释不通”时才开启，否则一律算偏题。

- **每个知识点的完成定义（DoD）**：满足以下任意 4 条就算当天完成

  - **一句话**：20 秒内讲清楚“它是什么/解决什么问题”。

  - **两条命令**：能用 2 条命令观察到相关指标/现象。

  - **一个对照实验**：A/B 只改一个变量，得到可复现结论。

  - **一道面试题**：用 3-5 句话回答一个高频追问。

- **时间盒与停止规则**

  - 单个细节最多 **20 分钟**。

  - 到点必须二选一：写入“停车场”并回到主线；或明确它能直接提升定位能力并继续。

- **停车场（Parking Lot）模板**：把焦虑“卸载”出去

  - **问题**：我卡在什么细节？

  - **触发**：卡在计划的哪一步？

  - **当前猜测**：A/B/C。

  - **需要的证据**：看哪个指标/日志/源码/文档。

  - **优先级**：高/中/低。

- **挫败感急救流程（卡住就照做）**

  - 写一句话：我现在卡住的具体问题是什么。

  - 判断：这是主线问题还是细节问题（细节→停车场）。

  - 做一个最小实验/最小命令拿到新证据。

  - 输出 5 行总结（不求完美，只求闭环）。

- **每日任务分层（必须/可选）**

  - **必须（Must）**：2 条命令 + 1 个小实验 + 1 道面试题。

  - **可选（Bonus）**：源码深挖、扩展阅读、论文与实现细节。

  

### 0.3 建议准备的环境（尽量简单）

  

- 一台 Linux（Ubuntu 22.04/24.04 都可），有 sudo 权限

- 8GB 内存起步（够做大部分实验），CPU 核数越多越好

- 尽量使用同一台机器完成 8 周，避免环境差异干扰实验

  

### 0.4 建议安装的工具（不强制，但强烈推荐）

  

- **基础**：`procps`（top/ps）、`util-linux`（dmesg）、`iproute2`（ip/ss）

- **系统统计**：`sysstat`（iostat/sar）、`smem`（可选）

- **性能工具**：`perf`、`strace`

- **eBPF（最后两周用）**：`bpftrace`（没有也能学，但少了“杀手锏”）

  

你可以在开始前确认一下工具是否存在：

  

```bash

which top ps free vmstat iostat sar ss ip strace perf || true

```

  

---

  

## 1. 总体知识地图（8周覆盖哪些主线）

  

- **进程与调度**：进程/线程/上下文切换/负载/优先级/抢占

- **内存管理**：虚拟内存/页表/page fault/RSS vs cache/NUMA/oom

- **文件系统与存储 IO**：page cache、writeback、fsync、iowait、ext4/xfs 基本语义

- **网络**：TCP 状态机、队列、重传、拥塞、TIME_WAIT、端口耗尽

- **systemd + cgroup + namespace**：现代 Linux 服务管理与资源治理

- **可观测性与性能工程**：USE/RED、strace/perf/bpftrace、证据链排障

  

---

  

## 2. 8周学习计划（细到每天）

  

> 约定：每周 6 天学习 + 1 天复盘与面试演练。

  

### Week 1：Linux 基础 + 观测入门（把“系统在干什么”看见）

  

**本周目标**：你看到 CPU/内存/IO/网络异常时，不再无从下手，知道第一批命令怎么跑。

  

**本周产出物**：

  

- `os-notes/week1.md`（你自己写的笔记，至少 200 行）

- 一个采样脚本：每秒采集 meminfo/loadavg/进程 topN（bash 或 python 均可）

  

#### Day 1：Linux 进程基础（ps/top 的正确打开方式）

  

- **知识点**

  - PID/PPID、进程状态 R/S/D/Z/T

  - VSZ/RSS/SHR 的含义（先知道“是什么”，下周再学“为什么”）

  - load average 是什么（与 CPU% 不同维度）

- **必做命令**

  

```bash

ps -eo pid,ppid,stat,comm,%cpu,%mem --sort=-%cpu | head

ps -eo pid,ppid,stat,comm,%cpu,%mem --sort=-%mem | head

top -H -p 1

cat /proc/1/status | head -n 40

cat /proc/loadavg

uptime

```

  

- **练习**

  - 找一个 CPU 占用高的进程，解释它的 `STAT` 与 `loadavg` 的关系（先讲直觉）

- **面试自测题**

  - “load average 代表什么？为什么 CPU 只有 10% 但 load 很高？”

  

#### Day 2：文件与权限（不要被基础题卡住）

  

- **知识点**

  - 文件权限 rwx、uid/gid、umask

  - 软链接/硬链接

  - 关键目录：`/proc`、`/sys`、`/etc`、`/var/log`

- **必做命令**

  

```bash

id

umask

ls -l /proc/1/exe

readlink -f /proc/1/exe

ls -l /var/log | head

```

  

- **练习**

  - 用一句话解释 `/proc` 和普通目录的不同

- **面试自测题**

  - “软链接和硬链接区别？删除原文件会怎样？”

  

#### Day 3：内存概览（先学会看 meminfo）

  

- **知识点**

  - `/proc/meminfo` 常用字段：MemFree/MemAvailable/Buffers/Cached/Slab

  - page cache 是什么（先能说直觉：缓存文件内容，提高读性能）

- **必做命令**

  

```bash

free -h

grep -E 'MemTotal|MemFree|MemAvailable|Buffers|Cached|Slab|SReclaimable|SUnreclaim' /proc/meminfo

cat /proc/zoneinfo | head -n 40

```

  

- **练习**

  - 解释“为什么 Cached 很大不代表内存泄漏”

- **面试自测题**

  - “MemAvailable 与 MemFree 的区别？”

  

#### Day 4：IO 与磁盘基础（iowait 到底是什么）

  

- **知识点**

  - iowait 的含义（CPU 等 IO 完成）

  - 进程的 IO 统计视角

- **必做命令**

  

```bash

vmstat 1 5

iostat -x 1 3 || true

pidstat -d 1 3 || true

lsblk

mount | head

```

  

- **练习**

  - 用 `dd` 制造一次写入，观察 `vmstat` 的 `wa` 变化（注意控制文件大小）

- **面试自测题**

  - “iowait 高意味着什么？一定是磁盘慢吗？”

  

#### Day 5：网络基础观测（先会看连接与端口）

  

- **知识点**

  - TCP 连接状态：ESTAB/LISTEN/TIME_WAIT/CLOSE_WAIT

  - 端口/FD 基础

- **必做命令**

  

```bash

ss -s

ss -lntp | head

ss -ant | head

ulimit -n

cat /proc/sys/net/ipv4/ip_local_port_range

```

  

- **练习**

  - 找出本机监听端口最多的服务，并解释它是什么

- **面试自测题**

  - “TIME_WAIT 为什么存在？多了会怎样？”

  

#### Day 6：日志与系统启动（能把系统状态串起来）

  

- **知识点**

  - `journalctl` 基本用法

  - systemd unit 基本概念：service/timer/target

- **必做命令**

  

```bash

systemctl status

systemctl list-units --type=service --state=running | head

systemctl list-timers | head

journalctl -b -p warning | head -n 50

```

  

- **练习**

  - 选一个 service，找到它的 ExecStart 和日志位置

- **面试自测题**

  - “systemd 和传统 init 有什么差别？你怎么排查一个 service 起不来？”

  

#### Day 7（复盘日）：做一份“系统体检报告”

  

- **产出**：写一页报告（Markdown）包含：CPU/内存/IO/网络/服务状态

- **必须包含的命令输出（截取关键行即可）**

  - `uptime`、`free -h`、`vmstat 1 3`、`ss -s`、`systemctl --failed`

  

---

  

### Week 2：进程/线程/调度（把 CPU 与并发问题讲明白）

  

**本周目标**：遇到 CPU 高、上下文切换多、线程打满，你能解释原因并定位到线程/锁/系统调用层面。

  

**本周产出物**：

  

- 一个 C/Go/Python 小程序：分别制造 CPU 密集/锁竞争/大量线程

- `week2-scheduler.md`：解释 loadavg、上下文切换、run queue 的直觉

  

#### Day 8：进程创建与执行（fork/exec 的面试必考）

  

- **知识点**

  - fork/exec/clone 的概念

  - copy-on-write（先理解：fork 不会立刻复制全部内存）

- **练习（建议写 C）**

  - 写一个程序 `fork()` 之后分别打印 pid/ppid

  - `execve()` 执行 `/bin/echo`

- **必做命令**

  

```bash

strace -f -e trace=process bash -lc 'echo hello'

cat /proc/$$/status | grep -E 'Pid|PPid|Threads'

```

  

- **面试自测题**

  - “fork 之后父子进程内存如何处理？为什么 fork 很快？”

  

#### Day 9：线程与上下文切换（从现象到指标）

  

- **知识点**

  - 线程 vs 进程（共享地址空间/文件描述符等）

  - voluntary/involuntary context switch

- **必做命令**

  

```bash

vmstat 1 5

pidstat -w 1 5 || true

cat /proc/1/sched | head -n 40

```

  

- **练习**

  - 写一个多线程程序，空转 vs 加锁竞争，对比上下文切换

- **面试自测题**

  - “上下文切换为什么贵？什么时候会变多？”

  

#### Day 10：CPU 利用率拆解（user/sys/softirq/irq）

  

- **知识点**

  - user/system/irq/softirq/iowait 的意义

  - 软中断的直觉（网络/块 IO 常见）

- **必做命令**

  

```bash

mpstat -P ALL 1 3 || true

cat /proc/softirqs | head

cat /proc/interrupts | head

```

  

- **面试自测题**

  - “system CPU 很高说明什么？softirq 很高可能是什么？”

  

#### Day 11：调度基本概念（CFS 讲人话版）

  

- **知识点**

  - 时间片、公平、优先级

  - nice 值、抢占

- **必做命令**

  

```bash

nice -n 10 bash -lc 'yes > /dev/null' &

renice -n 0 -p $!

ps -o pid,ni,pri,stat,comm -p $!

kill $!

```

  

- **面试自测题**

  - “nice 是什么？为什么改了 nice 但 CPU 还是很高？”

  

#### Day 12：性能热点定位入门（perf 最小闭环）

  

- **知识点**

  - 采样（sampling） vs 插桩（tracing）

  - 火焰图概念（先知道用途）

- **必做命令**

  

```bash

perf --version || true

perf stat -a -- sleep 1 || true

```

  

- **练习**

  - 对一个 CPU 空转进程做 `perf top`（如果权限允许）

- **面试自测题**

  - “perf 能回答什么问题？和 strace 有什么不同？”

  

#### Day 13：D 状态与卡死（最常见线上问题之一）

  

- **知识点**

  - D 状态（不可中断睡眠）通常与 IO/锁相关

  - 为什么 `kill -9` 也杀不掉 D 状态

- **必做命令**

  

```bash

ps -eo pid,stat,wchan:30,comm | awk '$2 ~ /D/ {print}' | head

cat /proc/$$/stack 2>/dev/null | head || true

```

  

- **面试自测题**

  - “什么是 D 状态？怎么排查？为什么 kill 不掉？”

  

#### Day 14（复盘日）：输出一份“CPU 高排障 SOP”

  

- **产出**：1 页 Markdown，包含：

  - 先看什么指标（load、us/sy、ctx switch）

  - 如何定位到进程/线程

  - 如何区分热点/锁/软中断

  

---

  

### Week 3：虚拟内存与 OOM（把内存问题从“玄学”变成“可解释”）

  

**本周目标**：你能解释 RSS/VSZ/PSS、page cache、page fault、swap、OOM killer 机制。

  

**本周产出物**：

  

- `week3-memory.md`：用你自己的话解释 10 个核心概念

- 一个“内存实验记录”：至少 3 个对照实验

  

#### Day 15：虚拟内存基本直觉（地址空间/页）

  

- **知识点**

  - 虚拟地址空间、页（page）、页表（page table）

  - user space vs kernel space

- **必做命令**

  

```bash

getconf PAGE_SIZE

cat /proc/$$/maps | head -n 30

cat /proc/$$/smaps | head -n 40

```

  

- **面试自测题**

  - “为什么有虚拟内存？它解决了什么问题？”

  

#### Day 16：page fault（major/minor）与缺页开销

  

- **知识点**

  - minor fault（页在内存但未映射） vs major fault（需要读盘）

- **必做命令**

  

```bash

vmstat 1 5

perf stat -e page-faults,major-faults,minor-faults -- sleep 1 || true

```

  

- **练习**

  - 读一个大文件两次，对比第二次 major fault 下降（体现 page cache）

  

#### Day 17：RSS/VSZ/PSS/Shared（面试高频）

  

- **知识点**

  - RSS：进程实际驻留

  - VSZ：虚拟地址空间

  - PSS：共享内存按比例分摊（排查更靠谱）

- **必做命令**

  

```bash

ps -o pid,vsz,rss,comm -p $$

cat /proc/$$/status | grep -E 'VmSize|VmRSS'

```

  

- **面试自测题**

  - “VSZ 很大一定有问题吗？什么时候 VSZ 大是正常的？”

  

#### Day 18：page cache 与回收（为什么 free 看起来“满了”）

  

- **知识点**

  - page cache、reclaim、dirty pages、writeback

- **必做命令**

  

```bash

grep -E 'Dirty|Writeback|Cached|Buffers|MemAvailable' /proc/meminfo

cat /proc/vmstat | grep -E 'pgscan|pgsteal|pgfault' | head

```

  

- **练习**

  - 写入一个文件后马上读，观察 cache 命中（可用 `time` 感受）

  

#### Day 19：swap 与抖动（什么时候该开 swap）

  

- **知识点**

  - swap in/out

  - 何为 thrashing（抖动）

- **必做命令**

  

```bash

swapon --show || true

vmstat 1 5

cat /proc/sys/vm/swappiness

```

  

- **面试自测题**

  - “swap 开还是不开？swappiness 是什么？”

  

#### Day 20：OOM killer（从日志到根因）

  

- **知识点**

  - OOM 触发条件（内存回收失败）

  - oom_score/oom_score_adj

- **必做命令**

  

```bash

dmesg | grep -i -E 'oom|killed process' | tail -n 20

cat /proc/$$/oom_score

cat /proc/$$/oom_score_adj

```

  

- **练习**

  - 读一篇 OOM 日志样例（网上/历史事故），写出“谁被杀、为什么是它”

  

#### Day 21（复盘日）：输出“内存问题排障 SOP + 术语表”

  

- **产出必须包含**：

  - RSS/Cache/Slab/Vmalloc/Percpu 的区别

  - OOM 排查步骤

  - 你认为最容易被问住的 10 个点的答案

  

---

  

### Week 4：文件系统与存储 IO（延迟、抖动、fsync）

  

**本周目标**：你能解释 page cache、writeback、fsync、iowait、IO 队列，能定位“慢在用户态还是内核/磁盘”。

  

**本周产出物**：

  

- 一份“IO 延迟排障报告”（对一次实验的完整记录）

  

#### Day 22：文件系统基础语义（读写与一致性）

  

- **知识点**

  - inode/dentry（先会讲直觉）

  - 文件读写路径：用户态→syscall→VFS→FS→block layer

- **必做命令**

  

```bash

stat .

mount | head -n 20

cat /proc/filesystems | head

```

  

- **面试自测题**

  - “为什么 `write()` 返回成功不代表数据落盘？”

  

#### Day 23：page cache 与 direct IO（概念即可）

  

- **知识点**

  - buffered IO vs direct IO

  - 为什么数据库关心 fsync

- **练习**

  - 对比读取同一文件两次的耗时差（体现缓存）

  

#### Day 24：writeback 与 dirty（抖动来源之一）

  

- **知识点**

  - dirty ratio、writeback 周期

- **必做命令**

  

```bash

sysctl vm.dirty_background_ratio vm.dirty_ratio vm.dirty_expire_centisecs vm.dirty_writeback_centisecs

grep -E 'Dirty|Writeback' /proc/meminfo

```

  

- **面试自测题**

  - “为什么某些场景会出现周期性 IO 抖动？”

  

#### Day 25：iostat 指标解读（把磁盘指标讲人话）

  

- **知识点**

  - `await`、`svctm`（老版本）、`util`、队列深度

- **必做命令**

  

```bash

iostat -x 1 3 || true

lsblk -o NAME,SIZE,TYPE,MOUNTPOINT

```

  

- **练习**

  - 制造一次写入负载，解释 `await` 与 `util` 变化

  

#### Day 26：strace 看 IO 系统调用（从应用到内核入口）

  

- **知识点**

  - `open/read/write/fsync` 的基本含义

- **必做命令**

  

```bash

strace -f -tt -T -e trace=openat,read,write,fsync bash -lc 'echo 123 > /tmp/t1; cat /tmp/t1'

```

  

- **面试自测题**

  - “你怎么判断应用慢是慢在系统调用还是慢在锁/计算？”

  

#### Day 27：文件描述符与泄漏（经典线上问题）

  

- **知识点**

  - fd 是什么，为什么会耗尽

- **必做命令**

  

```bash

ulimit -n

ls -l /proc/$$/fd | head

lsof -p $$ | head || true

```

  

- **面试自测题**

  - “fd 泄漏怎么排查？为什么会导致 accept 失败？”

  

#### Day 28（复盘日）：写“IO 问题排障 SOP”

  

- **产出必须包含**：

  - 如何区分：磁盘慢 vs 文件系统/回写抖动 vs 应用同步写

  - 你会用哪些指标证明（vmstat/iostat/strace）

  

---

  

### Week 5：网络（把超时、重传、连接堆积讲透）

  

**本周目标**：面试里讲 TCP 不慌；线上遇到超时/连接数暴涨/端口耗尽能定位。

  

**本周产出物**：

  

- 一份“网络问题排障清单”（含 ss/tcpdump 的组合拳）

  

#### Day 29：TCP 基本状态机与 ss 用法

  

- **知识点**

  - 三次握手、四次挥手

  - 常见状态：SYN-SENT/SYN-RECV/ESTAB/TIME_WAIT/CLOSE_WAIT

- **必做命令**

  

```bash

ss -ant

ss -s

ss -ant state time-wait | head

```

  

- **面试自测题**

  - “CLOSE_WAIT 多说明什么？一般是谁的锅？”

  

#### Day 30：端口与连接耗尽（NAT/本机端口）

  

- **知识点**

  - ephemeral port range

  - TIME_WAIT 与端口复用

- **必做命令**

  

```bash

cat /proc/sys/net/ipv4/ip_local_port_range

sysctl net.ipv4.tcp_tw_reuse net.ipv4.tcp_fin_timeout

```

  

- **面试自测题**

  - “为什么压测时会出现 Cannot assign requested address？”

  

#### Day 31：TCP 重传与丢包直觉

  

- **知识点**

  - 重传发生的常见原因：丢包/拥塞/队列溢出

- **必做命令**

  

```bash

netstat -s 2>/dev/null | grep -i retrans | head || true

ss -ti | head

```

  

- **面试自测题**

  - “怎么证明是丢包导致慢？你看哪些指标？”

  

#### Day 32：tcpdump 入门（抓一条证据链）

  

- **知识点**

  - 抓包不是为了“看热闹”，是为了证明握手/重传/RTT

- **必做命令（示例）**

  

```bash

sudo tcpdump -i any -nn 'tcp' -c 50

```

  

- **练习**

  - 抓一次你访问某个服务的包，找到 SYN、SYN-ACK、ACK

  

#### Day 33：DNS 与超时（不要忽略外部依赖）

  

- **知识点**

  - DNS 超时会伪装成“业务慢”

- **必做命令**

  

```bash

cat /etc/resolv.conf

getent hosts localhost

```

  

- **面试自测题**

  - “线上请求偶发 5s 超时，你怎么排除 DNS？”

  

#### Day 34：网络队列与软中断（概念 + 观测）

  

- **知识点**

  - 软中断与网络处理关系（不要求你讲代码）

- **必做命令**

  

```bash

cat /proc/softirqs | head

mpstat -P ALL 1 2 || true

```

  

#### Day 35（复盘日）：输出“网络问题排障 SOP”

  

- **产出必须包含**：

  - 超时/拒绝连接/连接重置的区别

  - ss 如何定位连接堆积

  - tcpdump 如何证明握手与重传

  

---

  

### Week 6：systemd + cgroup + namespace（现代 Linux 的“真实形态”）

  

**本周目标**：你能把“服务治理”和“资源隔离”讲清楚，理解容器与 systemd 的本质。

  

**本周产出物**：

  

- 一份“cgroup v2 速记”：目录结构、常见控制器、典型故障

  

#### Day 36：systemd unit 深入（service/timer/target）

  

- **知识点**

  - unit 文件结构：`[Unit]/[Service]/[Install]`

  - ExecStart、Restart、TimeoutStartSec

- **必做命令**

  

```bash

systemctl cat ssh.service | head -n 80 || true

systemctl show ssh.service | head -n 80 || true

```

  

- **面试自测题**

  - “服务启动慢，你怎么定位？systemd 超时在哪里调？”

  

#### Day 37：cgroup v2 目录与控制器

  

- **知识点**

  - `/sys/fs/cgroup` 结构

  - controller（cpu/memory/io/pids）概念

- **必做命令**

  

```bash

stat -fc %T /sys/fs/cgroup

cat /sys/fs/cgroup/cgroup.controllers

cat /sys/fs/cgroup/cgroup.subtree_control

```

  

- **练习**

  - 找到一个 service 的 cgroup 路径（`systemctl status` 里通常能看到）

  

#### Day 38：memory controller 基础（限额与统计）

  

- **知识点**

  - `memory.current`、`memory.max`、`memory.stat`

- **必做命令**

  

```bash

cat /sys/fs/cgroup/memory.current 2>/dev/null || true

cat /sys/fs/cgroup/memory.stat 2>/dev/null | head || true

```

  

- **面试自测题**

  - “容器内存限制怎么生效？为什么有时看起来不准？”

  

#### Day 39：pids controller 与 fork bomb 防护

  

- **知识点**

  - `pids.max` 作用

- **必做命令**

  

```bash

cat /sys/fs/cgroup/pids.max 2>/dev/null || true

cat /sys/fs/cgroup/pids.current 2>/dev/null || true

```

  

#### Day 40：namespace 概念（容器隔离的另一半）

  

- **知识点**

  - pid/net/mnt/uts/ipc/user namespace 是什么

- **必做命令**

  

```bash

lsns | head || true

readlink /proc/1/ns/pid

readlink /proc/1/ns/net

```

  

- **面试自测题**

  - “容器到底隔离了什么？cgroup 和 namespace 分别解决什么？”

  

#### Day 41：systemd 与 cgroup 的关系（把两者串起来）

  

- **知识点**

  - systemd 用 cgroup 管理 unit 的资源与进程树

- **必做命令**

  

```bash

systemd-cgls | head -n 60

systemctl status | head -n 80

```

  

#### Day 42（复盘日）：写“systemd/cgroup 面试 30 题答案”

  

- **要求**：每题 3-5 句话，能讲机制+例子。

  

---

  

### Week 7：可观测性 + 性能方法论（你要形成“证据链思维”）

  

**本周目标**：任何性能/稳定性问题，你都能用 USE/RED/四大黄金信号组织排查，不乱。

  

**本周产出物**：

  

- 一份通用排障模板（Markdown）：现象→影响→假设→实验→证据→结论→止血→治理→验收

  

#### Day 43：USE 方法（资源视角）

  

- **知识点**

  - Utilization/ Saturation/ Errors

  - CPU/内存/IO/网络各自怎么看 U/S/E

- **练习**

  - 用你机器做一次 USE 体检，写成表格

  

#### Day 44：RED 方法（服务视角）

  

- **知识点**

  - Rate/Errors/Duration

- **练习**

  - 选一个 HTTP 服务（哪怕是本机），定义 RED 指标与阈值

  

#### Day 45：strace 深入（定位慢系统调用）

  

- **知识点**

  - `-tt -T -f` 的意义

- **必做命令**

  

```bash

strace -f -tt -T -e trace=network,read,write,openat,fsync -p 1 2>/dev/null | head || true

```

  

#### Day 46：perf 进阶（热点与调用栈）

  

- **知识点**

  - `perf record/report`

- **练习**

  - 对一个 CPU 密集程序采样，写出 top 3 热点函数（哪怕是 libc）

  

#### Day 47：日志与指标如何对齐（时间线思维）

  

- **知识点**

  - 把“指标变化点”与“日志事件点”对齐

- **练习**

  - 从 journalctl 里找一次服务重启，结合 uptime/load/meminfo 写时间线

  

#### Day 48：故障演练（你自己造一次事故）

  

- **要求**：从下面选一个造故障并写报告：

  - 内存持续上涨（用户态泄漏或 cache 抖动）

  - IO 抖动（周期性写回）

  - 连接堆积（TIME_WAIT/CLOSE_WAIT）

  

#### Day 49（复盘日）：输出“面试讲故事模板（STAR + 证据链）”

  

- **要求**：用你 Week48 的故障演练写一份面试叙事稿。

  

---

  

### Week 8：高级定位工具（bpftrace/内核视角）+ 面试冲刺

  

**本周目标**：你能在“系统级难题”里拿到关键证据（谁触发、栈是什么、数量级如何解释）。

  

**本周产出物**：

  

- 3 份 bpftrace 脚本（或命令）+ 对应解释

- 一份“OS 面试终极复习清单”（你自己的薄弱点）

  

#### Day 50：bpftrace 入门（kprobe 与聚合）

  

- **知识点**

  - kprobe/tracepoint 的区别（知道概念即可）

  - 聚合：count/sum/hist

- **练习（示例）**

  

```bash

sudo bpftrace -e 'kprobe:__x64_sys_openat { @[comm] = count(); } interval:s:5 { print(@); clear(@); }'

```

  

#### Day 51：把“谁触发”定位到“内核路径”（kstack）

  

- **练习（示例）**

  

```bash

sudo bpftrace -e 'kprobe:__x64_sys_openat { @[kstack(10)] = count(); } interval:s:5 { print(@); clear(@); }'

```

  

- **面试自测题**

  - “你为什么选这个探针？怎么避免误判？”

  

#### Day 52：percpu/slab/vmalloc 的关联（把内核内存讲透一遍）

  

- **知识点**

  - `/proc/meminfo` 里这些字段的关系：Slab/VmallocUsed/Percpu

- **练习**

  

```bash

grep -E 'Percpu|VmallocUsed|Slab|SReclaimable|SUnreclaim' /proc/meminfo

sudo grep -iE 'pcpu|percpu' /proc/vmallocinfo | head -n 50

```

  

#### Day 53：cgroup 相关追踪（把 Week6 的知识用工具固化）

  

- **练习（示例）**

  

```bash

sudo bpftrace -e 'kprobe:cgroup_mkdir { @[comm] = count(); } interval:s:5 { print(@); clear(@); }'

```

  

#### Day 54：做一题综合题（模拟面试现场排障）

  

- **题目**：给你一个现象：CPU 不高但 load 高、偶发超时、内存看似充足但 OOM。

- **要求**：写出你的排查路径（先看什么→为什么→预期看到什么→下一步）。

  

#### Day 55：OS 高频面试题背诵与纠错（查漏补缺）

  

- **必须覆盖的题目清单**

  - 进程 vs 线程

  - 上下文切换

  - load average

  - 虚拟内存与 page fault

  - page cache

  - OOM killer

  - iowait

  - fsync

  - TIME_WAIT/CLOSE_WAIT

  - cgroup vs namespace

  - strace/perf/bpftrace 分别解决什么

  

#### Day 56（最终复盘日）：做一次“全流程面试演练”

  

- **要求**：

  - 你讲 5 分钟：用 STAR 讲一个你做过的排障/优化（哪怕是练习题）

  - 你回答 10 分钟：从上面的题单随机抽 10 题

  - 你写 10 分钟：写一份 1 页故障报告模板

  

---

  

## 3. 每周验收标准（你是否“学到了”）

  

- **Week1**：你能独立做“系统体检报告”（CPU/内存/IO/网络/服务）

- **Week2**：你能解释 loadavg、线程数、上下文切换，并用命令证明

- **Week3**：你能区分 RSS/page cache/slab/vmalloc/percpu，并解释 OOM 日志

- **Week4**：你能用 iostat/vmstat/strace 对一次 IO 慢做证据链

- **Week5**：你能解释 TIME_WAIT/CLOSE_WAIT，并用 ss/tcpdump 给出证据

- **Week6**：你能讲清楚 cgroup/namespace/systemd 的关系，并能定位某 service 的 cgroup

- **Week7**：你能用 USE/RED 写一份通用排障模板并完成一次演练报告

- **Week8**：你能用 bpftrace 做“谁触发 + 栈路径 + 数量级解释”的最小闭环

  

---

  

## 4. 面试速查（你最容易慌的点，建议每天复述 5 分钟）

  

- **CPU**：loadavg ≠ CPU%；us/sy/si/hi/wa 代表什么；上下文切换为何贵

- **内存**：MemAvailable；RSS vs cache；page fault；OOM 日志怎么看

- **IO**：iowait；fsync；writeback 抖动；iostat 指标如何解释

- **网络**：TIME_WAIT/CLOSE_WAIT；端口耗尽；重传；超时排查路径

- **隔离**：cgroup vs namespace；systemd 如何管理 cgroup

- **工具**：strace（系统调用）、perf（热点）、bpftrace（内核路径证据）

  

---

  

## 5. 你执行计划时的常见坑（提前避坑）

  

- **只看书不做实验**：两周后你会觉得“好像懂了”，面试一问就露馅

- **一次改很多变量**：你会得出错误结论，越学越乱

- **追求把所有细节学完**：两个月做不到；目标是“面试不慌 + 能定位常见问题”

- **没有产出物**：没输出就没沉淀；建议每周至少一份 SOP 文档