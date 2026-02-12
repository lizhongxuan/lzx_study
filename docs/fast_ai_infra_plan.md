# 🚀 AI Infra & vLLM 快速学习指南 (10-12 周)

**核心方法论：** 垂直切片 (Vertical Slicing) + 问题驱动 (Problem-Driven)。
**目标：** 跳过纯学术推导，直接掌握工业界最急缺的“模型系统优化”能力，达到 AI Infra 工程师入职标准。

---
# 基础

## 🟢 Phase 1: 扫盲与工具准备 (第 1-4 周)
**目标：** 不求精通编程，但要能看懂“脚本”在干什么。AI Infra 的工作 80% 是在和 Linux 命令行及 Python 脚本打交道。

### 1. Linux 命令行 (基础设施的语言)
**必学理由：** AI 模型都跑在 Linux 服务器上，没有图形界面（Windows 很少用于生产环境）。
- [ ] **基本操作：** 学习 `ls`, `cd`, `mkdir`, `rm`, `cp`, `mv`。
- [ ] **系统监控：** 学习 `top`, `htop` (看 CPU/内存占用)，`nvidia-smi` (这是最重要的！看显卡显存有没有满)。
- [ ] **编辑文件：** 学会用 `nano` 或 `vim` 修改配置文件。

### 2. Python 基础 (只学胶水语言部分)
**必学理由：** AI 领域的工具全是 Python 写的。你不需要能写出复杂的算法，但要能读懂配置文件和简单的逻辑。
- [ ] **基础语法：** 变量、字符串、数字、布尔值。
- [ ] **容器：** **列表 (List)** 和 **字典 (Dictionary)** —— *这个最重要，因为模型配置全是 JSON 格式的字典。*
- [ ] **流程控制：** `if/else` 判断，`for` 循环。
- [ ] **函数：** 知道 `def` 是定义功能，`return` 是返回结果。
- [ ] **库的使用：** 学会 `pip install` 安装包，学会 `import` 导入包。

> **🛑 避坑：** 不要去学 Python 爬虫、网页开发、游戏开发。只学基础语法。

---
## 🔵 Phase 2: 建立直觉——AI 是个什么东西? (第 5-8 周)
**目标：** 跳过数学公式，建立物理直觉。把 AI 模型看作一个“巨大的、吃硬件的文件”。

### 1. 什么是大模型 (LLM)?
- [ ] **黑盒认知：** 不要管神经网络怎么传导。就把模型当成一个**函数**：输入一段字 -> 消耗显卡资源 -> 输出一段字。
- [ ] **文件认知：** 知道模型其实就是一堆存着数字的 `.bin` 或 `.safetensors` 文件。7B 的模型大概 14GB 大。
- [ ] **HuggingFace (AI 界的 GitHub):** 注册账号，学会搜索模型（如 `Llama-3`），学会看 Model Card（说明书）。

### 2. 数学概念 (Infra 特供版)
你不需要微积分，只需要**小学算术**和**空间想象力**。
- [ ] **矩阵 (Matrix):** 把它想象成 Excel 表格。
- [ ] **形状 (Shape):** 表格有几行几列？比如 `[1, 10, 4096]`。如果不匹配就会报错。
- [ ] **显存 (VRAM):** 显卡的空间。模型是货物，显存是仓库。货物太大，仓库装不下就会 OOM (Out Of Memory) 报错。

---

## 📅 Phase 3: 建立核心负载认知 
**目标：** 不依赖高级库，手写推理代码，痛彻理解“为什么 LLM 推理这么慢”。
### 理论基石
- [ ] **Transformer 架构 (Decoder-only):** 彻底搞懂 Input Embedding, MHA (Multi-Head Attention), FFN, LayerNorm 的矩阵维度变化。
- [ ] **KV Cache 原理:** 理解它是如何用“显存空间”换取“计算时间”的。
- [ ] **瓶颈认知:** 理解 Memory Bound (显存带宽限制) vs Compute Bound (算力限制)。

### 硬核实战 (代码量 < 300行)
- [ ] **任务 1:** 使用纯 PyTorch (`torch.nn.Linear` 等) 搭建一个微型 GPT 模型。
- [ ] **任务 2 (关键):** 编写两个版本的 `generate` 函数：
    - 版本 A: **不使用** KV Cache (每次重新计算所有 token)。
    - 版本 B: **使用** KV Cache (只计算新 token，拼接旧 KV)。
- [ ] **任务 3:** 记录并对比版本 A 和 B 在生成 100 个 token 时的耗时与显存占用。

> **💡 思考题:** 当 Batch Size = 1 时，GPU 显存利用率是多少？为什么这么低？

---
## 📅 Phase 4: 掌握 vLLM 核心架构 
**目标：** 从“用户”转变为“源码阅读者”，理解 vLLM 如何通过系统设计解决 Phase 1 的痛点。

### 核心概念
- [ ] **PagedAttention:** 结合操作系统“虚拟内存”概念，理解 Logical Block 到 Physical Block 的映射。
- [ ] **Continuous Batching:** 理解它是如何解决传统 Static Batching 的“气泡”问题的。
- [ ] **Copy-on-Write:** 理解 vLLM 如何通过共享物理块来实现高效的 Beam Search。

### 源码狙击 (基于 vLLM GitHub)
- [ ] **环境搭建:** 在 Linux/WSL 环境下源码安装 vLLM，跑通 Demo。
- [ ] **调度器逻辑:** 阅读 `vllm/core/scheduler.py`，理解 `running`, `waiting`, `swapped` 队列的状态流转。
- [ ] **块管理器:** 阅读 `vllm/core/block_manager.py`，搞懂 Block Table 是怎么维护的。
- [ ] **执行引擎:** 浏览 `vllm/model_executor/`，看懂它是如何调用 CUDA/Triton 算子的。

### 工程实操
- [ ] **压测实验:** 使用 `benchmark_throughput.py` 测试不同 Request Rate 下的 Throughput (RPS) 和 Latency (TTFT/TPOT)。

---

## 📅 Phase 5: 进阶优化与硬件感知 (Week 6-8)
**目标：** 掌握让模型“更小、更快”的手段，接触底层算子。

### 量化技术 (Quantization)
- [ ] **原理:** FP16 vs INT8 vs FP8。理解 Zero-point 和 Scale。
- [ ] **算法:** 学习 **AWQ** (Activation-aware Weight Quantization) 和 **GPTQ**。
- [ ] **实战:** 使用 vLLM 加载一个 AWQ 量化模型，对比其与 FP16 版本的显存占用减少了多少。

### 算子入门 (OpenAI Triton)
*Infra 岗位的加分项，不必精通 C++ CUDA*
- [ ] **学习:** 阅读 Triton 官方文档 Tutorials 01-03。
- [ ] **实战:** 用 Python (Triton) 写一个简单的 `Vector Add` 和 `Softmax` kernel。
- [ ] **集成:** 尝试看懂 vLLM 中简单的 Triton kernel (`vllm/model_executor/layers/quantization/`).

---

## 📅 Phase 6: 项目实战与面试准备 (Week 9-10)
**目标：** 产出简历上的亮点项目，模拟面试。

### 🏆 项目推荐 (三选一)

#### 项目 A: LLM 推理性能深度分析报告 (适合入门)
* **内容:** 搭建一套 Benchmark 流水线，对比 HuggingFace PyTorch vs vLLM vs TGI 在不同并发下的表现。使用 PyTorch Profiler 分析瓶颈。
* **产出:** 一篇详细的技术博客 + GitHub Repo。

#### 项目 B: 自定义 Sampling/Logits Processor (适合进阶)
* **内容:** 修改 vLLM 源码，增加一种特殊的采样逻辑（例如：强制输出特定格式的 JSON 约束）。
* **产出:** 提交 PR 或维护一个 Fork 版本，展示你对 Model Executor 的理解。

#### 项目 C: 基于 Triton 的算子优化 (适合高阶)
* **内容:** 为某个特定的小算子（如特殊的 Activation）编写 Triton 实现，并替换 PyTorch 原生实现，展示加速比。

### ⚔️ 面试必背题库 (Checklist)
- [ ] 为什么 LLM 推理分为 Prefill 和 Decode 两个阶段？计算特性有何不同？
- [ ] 详细解释 PagedAttention 如何解决显存碎片化问题。
- [ ] 什么是 FlashAttention？它如何减少 HBM 访问次数？
- [ ] 如何估算一个 7B 模型在推理时的显存占用？(权重 + KV Cache + Activation)。
- [ ] 什么是 Continuous Batching？它比 Static Batching 好在哪里？

---

## 📚 必备资源清单

1.  **论文:**
    * *Attention Is All You Need* (只看架构图)
    * *vLLM: Efficient Memory Management for Large Language Model Serving with PagedAttention*
    * *FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness*
2.  **代码库:**
    * [vLLM GitHub](https://github.com/vllm-project/vllm)
    * [OpenAI Triton Tutorials](https://triton-lang.org/main/getting-started/tutorials/index.html)
3.  **工具:**
    * `nvidia-smi` / `nvitop` (显卡监控)
    * `torch.profiler`

---

> **给未来的 Infra 工程师:**
> 在这个领域，**“跑通代码”比“推导公式”重要，“理解显存”比“理解模型”重要**。保持对 System 开销的敏感度，你就能赢。加油！