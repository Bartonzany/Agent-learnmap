# 🚀 定制化 HPC 与并行计算学习路线图（下班时间版 · 12个月大满贯顶配）

> **核心理念**：以 **Stanford CS149** 为宏观骨架，以 **Roofline Model** 为分析圭臬；向下通过 **《C++ Concurrency》** 与 **MIT 6.824** 夯实高级并发与一致性思维；向上通过 **OpenMP** 与 **OpenBLAS** 掌握工业级并行 API 与顶级数学库的工程实现。100% 聚焦单节点多核与内存优化，直接反哺 Ascend 算子开发。

---

## 🗺️ 四阶段学习路线图（10-12 个月）

### 🟢 阶段一：并行计算核心范式与 Roofline 建模（第 1-3 个月）
**核心目标**：建立“计算受限”与“访存受限”的量化直觉，掌握现代共享内存并行编程基础。

* **👑 主攻资源 1**：**Stanford CS149: Parallel Computing**
  * **怎么学**：重点攻克 Lecture 1-7（并行思想、Amdahl 定律、SIMD、多核架构、Roofline 模型）。
* **👑 主攻资源 2**：**OpenMP 官方规范与实战**
  * **怎么用（下班时间专属）**：**不要当书读！** 结合 CS149 的 Lab，在 C++ 代码中直接使用 `#pragma omp`。
  * **核心突破**：掌握 `parallel for`、`collapse`（多维循环并行）、`schedule`（循环调度策略，如 dynamic/guided 解决负载不均）、以及 `simd`（提示编译器向量化）。
* **🛠️ 必做验证任务 (结合 Agent)**：
  1. **GEMM (矩阵乘法) 并行化初探**：编写朴素的 C++ 矩阵乘法，使用 OpenMP 进行多核加速。
  2. **Roofline 初体验**：使用 `perf` 收集该程序的 FLOPs 和内存带宽数据，绘制 Roofline 图，观察程序是卡在“算力墙”还是“带宽墙”上。

### 🟡 阶段二：内存层次优化与 OpenBLAS 工程标杆（第 4-6 个月）
**核心目标**：掌握解决“内存墙”的核心技术，并学习工业界顶级数学库是如何将硬件性能榨干的。

* **主攻资源 1**：**MIT 6.172: Performance Engineering (核心章节)**
  * **怎么学**：死磕 **Caching & Loop Unrolling** 和 **Software Pipelining (Double Buffering)**。
* **主攻资源 2**：**OpenBLAS / GotoTomas 算法思想**
  * **怎么用（下班时间专属）**：**不需要去读 OpenBLAS 的底层汇编源码！** 重点阅读关于 **GotoTomas 算法** 的经典论文或技术博客。
  * **核心突破**：理解 OpenBLAS 是如何做 **L2/L3 Cache Blocking** 的，以及它是如何设计 **Micro-kernel (微内核)** 来最大化寄存器复用和指令流水线利用率的。
* **🛠️ 必做验证任务**：
  **GEMM 极致优化实验**：在 Ubuntu 下，逐步实现并对比以下版本的性能（使用 `perf` 验证）：
  1. Naive 三层循环。
  2. + OpenMP 多线程。
  3. + Cache Blocking (Tiling，分块优化)。
  4. + Loop Unrolling (循环展开) + SIMD (ARM NEON / x86 AVX)。
  5. **对比 Baseline**：调用 `cblas_sgemm` (OpenBLAS)，看看你的极致优化距离工业级天花板还有多远，分析差距在哪里。

### 🟠 阶段三：C++ 高级并发与分布式一致性思维降维（第 7-9 个月）
**核心目标**：深入理解 C++ 内存模型，引入高级并发一致性思维，解决多核算子中的复杂同步问题。

* **主攻资源 1**：**《C++ Concurrency in Action》（C++ 并发编程实战）**
  * **怎么读**：跳过基础线程创建。**死磕 Part 2 (高级线程管理)**，特别是 **第 5 章 (C++ 内存模型和原子操作)**。
  * **核心突破**：彻底搞懂 `memory_order_relaxed`, `acquire`, `release`, `seq_cst` 的硬件语义。理解为什么滥用 `seq_cst` 会导致多核算子性能雪崩。
* **主攻资源 2**：**MIT 6.5840 (原 6.824): Distributed Systems**
  * **怎么用（防沉迷）**：**绝对不要深陷分布式网络细节！** 重点学习其 **Lab (用 Go 实现 Raft)** 中关于 **并发控制 (Locking)、状态机一致性、以及多协程/多线程下的竞态条件处理**。用这种经过学术界严苛检验的并发设计思维，来审视你 Ascend 多核算子中的 Barrier 和锁设计。
* **🛠️ 必做验证任务**：
  编写一个多核并行规约 (Reduction) 或生产者-消费者模型。刻意制造 False Sharing（伪共享），然后用 `perf` 观察 Cache Miss 飙升；接着通过 Padding（内存对齐填充）或调整内存序（使用 `acquire/release` 替代 `seq_cst`）来消除瓶颈。

### 🔴 阶段四：Ascend 业务深度反哺与知识沉淀（第 10-12 个月）
**核心目标**：将前 9 个月的通用 HPC 与并发理论，100% 收敛到 Ascend 3403 算子开发中。

* **主攻资源**：**Ascend C 编程指南** + **你的实际业务代码**
  * **概念映射**：
    * *Cache Blocking (OpenBLAS思想)* ➔ **Ascend Tiling 策略设计** (计算 L1/L0A/L0B 容量，确定最优分块)。
    * *Software Pipelining* ➔ **Ascend C 的 Pipe 机制与 Multi-Core 流水线调度** (数据搬运与计算重叠)。
    * *C++ 内存序 / 6.824 并发思维* ➔ **Ascend 多核任务调度与轻量级同步原语设计**。
* **🎯 核心产出（结合你的发展目标）**：
  准备一次**组内技术分享**。主题例如：《从 Roofline 模型与 OpenBLAS 思想，看 Ascend 算子 Tiling 与多核并发优化》。
  * **执行通路**（复用导师 PL 方法论）：
    1. **问题提出**：大分辨率输入下，算子耗时未达预期，多核加速比存在瓶颈。
    2. **需求背景**：Roofline 分析显示处于 Memory-bound 区域；且多核同步存在潜在的伪共享或过度锁竞争。
    3. **设计方案**：借鉴 OpenBLAS 的 Micro-kernel 思想重新设计 Tiling；引入 Double Buffering 隐藏搬运延迟；运用 C++ 内存序知识优化多核间的同步逻辑。
    4. **实现效果**：展示优化前后 Profiling 数据中 Cache Miss 率的下降、流水线 Bubble 的减少，以及最终耗时的显著缩短。

---

## 📚 核心资源字典与查阅指南

| 资源名称 | 定位 | 特点与下班时间专属用法 |
|---|---|--- |
| **Stanford CS149** | **👑 宏观主线** | 并行计算神课。聚焦 Multicore, SIMD, Roofline Model，建立正确的性能分析世界观。 |
| **OpenMP 官方文档/实战** | **并行 API 工具** | 结合 CS149 Lab 使用。掌握 `collapse`, `schedule`, `simd`，快速实现数据并行。 |
| **OpenBLAS / GotoTomas 算法** | **工程标杆与思想** | **不读底层汇编**。研读其 Cache Blocking 和 Micro-kernel 设计思想，调用其 API 作为性能对比 Baseline。 |
| **MIT 6.172 (核心章节)** | **底层性能工程** | 聚焦 Caching, Loop Unrolling, Software Pipelining。作为内存优化部分的深度补充。 |
| **《C++ Concurrency in Action》** | **并发与内存模型圣经** | **死磕第 5 章内存序**。理解 `acquire/release`，解决多核算子中的同步开销与数据竞争问题。 |
| **MIT 6.5840 (原 6.824)** | **并发思维试炼场** | **防沉迷**：只学其 Lab 中的并发控制、锁设计与一致性思维，用于降维打击多核算子同步问题。 |
| **Linux `perf` / Ascend Profiling** | **数据收集工具** | 没有测量就没有优化。为 Roofline 建模和 Cache Miss 分析提供真实输入。 |

---

## 💡 针对“下班时间”的防放弃执行守则

1. **坚决不碰分布式集群部署**：学习 6.824 时，只在本地单机运行 Go 的 Raft Lab，绝对不浪费时间去配置多台虚拟机。
2. **OpenBLAS 思想 > 源码**：下班时间极其宝贵，不要试图去读懂 OpenBLAS 几百万行的底层汇编。学习它的**分块策略和微内核设计思想**，然后自己用 C++/Intrinsic 实现一个简化版，性价比最高。
3. **微习惯驱动**：工作日目标设定为：“今晚用 C++ 写一个带 Tiling 的矩阵乘法小块，并用 `perf` 跑一下看 L1 Cache Miss 是否下降”或“阅读《C++ Concurrency》内存序的 5 页内容”。
4. **AI 辅助提效（核心）**：严格使用下方的 **Agent Prompt 模板**。让 AI 帮你生成不同 Tiling Size 的代码，或帮你分析 `perf` 数据。

---

## 🛠️ 附：你的专属 Agent 协作 Prompt 模板 (建议收藏)

每次需要验证 HPC 优化技巧、分析性能或编写并发代码时，套用此模板：

```text
【角色设定】
你是一位资深的 HPC (高性能计算) 专家和 C++ 性能优化工程师，精通 Stanford CS149 的并行计算理论、Roofline Model、OpenMP/OpenBLAS 工程实践，以及《C++ Concurrency in Action》中的高级内存模型。

【上下文】
我正在优化一个 [填写具体场景，如：矩阵乘法 GEMM / Ascend hifloat8 算子 / 大规模数组规约] 的性能。
当前遇到的瓶颈是：[填写现象，如：Cache Miss 率高 / 处于 Memory-bound 区域 / 多核加速比不理想，存在锁竞争]。
以下是我当前的 C++ 核心代码片段：
"[粘贴你的 C++ 代码]"

【任务目标】
请帮我分析当前代码的性能瓶颈，并提供基于 HPC 最佳实践的优化版本。

【严格约束条件】
1. 优化方向：请优先考虑 Cache Blocking (Tiling)、循环展开 (Unrolling)、数据预取 (Prefetching)、Software Pipelining 或 OpenMP 数据并行 (如 `collapse`, `schedule`)。
2. 语言与规范：使用现代 C++ (C++14/17)，代码需简洁、可读。**严格检查数据类型匹配**，避免 `int` 与 `int64_t` 混用导致的隐式转换或编译错误。
3. 并发与内存序：如果涉及多线程同步，请确保同步原语轻量且正确。避免过度使用 `std::memory_order_seq_cst`，优先使用 `acquire/release`，并警惕 False Sharing（必要时提供 Padding 方案）。
4. 理论支撑：在提供优化代码后，请用 **Roofline Model** 的视角简要解释：该优化是如何提升“计算强度”或减少“内存访问延迟”的。如果涉及 OpenBLAS 思想，请指出其对应的 Cache 层级。
5. 验证方法：请提供如何在 Ubuntu 下使用 `perf` 或 `g++ -O3 -march=native -fopenmp` 编译和测量该优化效果的命令。