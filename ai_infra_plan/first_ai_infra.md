# AI-Infra 高级研发工程师深度精通计划 (每日详尽版)

## 使用说明

- 时间单位：每一天 (Day) 代表一个约 4小时 的深度学习单元。
- 执行标准：必须完成“核心要解决问题”的自测，并运行过“AI 互动”指令才算通关。
- 资料获取：由于无法直接提供外链，请使用“学习资料”中的关键词在 Google/GitHub/Arxiv 中搜索。

## 第一阶段：物理算力与互联底座 (The "Iron")

**阶段目标**：建立扎实的硬件与网络物理层认知，这是做 Infra 的底气。

### Day 1: GPU 核心架构解密 (SM & Tensor Core)

- **学习方向**：深入理解 GPU 内部的流多处理器 (SM)、Tensor Core 的计算原理、以及 HBM (高带宽内存) 与 L2 Cache 的层级关系。
- **核心要解决问题**：
  - 为什么大模型训练更看重 HBM 带宽而不是 GPU 主频？
  - Tensor Core 在矩阵乘法中是如何实现 4x4 或 8x8 混合精度计算加速的？
- **AI 互动**：
  - "我是 Infra 初学者，请用通俗的语言（类比 CPU 的核）解释 GPU 的 Streaming Multiprocessor (SM) 结构。Tensor Core 是如何像'专用加速器'一样处理矩阵运算的？"
- **学习建议**：不要陷入 CUDA 编程的细节，重点关注数据流动：数据从 HBM 搬到 SM 的速度往往决定了性能（Memory Bound）。
- **学习资料**：
  - NVIDIA A100 Tensor Core Architecture Whitepaper
  - Understanding GPU Memory Hierarchy

### Day 2: 旗舰架构演进 (H100 vs B200)

- **学习方向**：对比 NVIDIA H100 与 B200 (Blackwell) 的架构差异，重点关注双芯片互联带来的挑战。
- **核心要解决问题**：
  - Transformer Engine 是硬件层面的什么东西？
  - B200 的双芯片设计（Chiplet）对互联带宽提出了什么新要求？
- **AI 互动**：
  - "请对比 NVIDIA H100 和 B200 架构，重点解释 B200 的双芯片设计对互联带宽的挑战，以及 NVLink Switch 在其中的作用。"
- **学习建议**：关注数字的变化（显存带宽从 3TB/s 到 8TB/s），建立数量级敏感度。
- **学习资料**：
  - NVIDIA Blackwell Architecture Technical Brief
  - NVIDIA GTC Keynote Summary H100 vs B200

### Day 3: 混合精度训练原理 (FP8/BF16)

- **学习方向**：理解不同浮点精度（FP32, FP16, BF16, FP8）的位宽结构及其对训练吞吐的影响。
- **核心要解决问题**：
  - 为什么 BF16 (Brain Float 16) 不需要 Loss Scaling？（提示：看指数位范围）。
  - FP8 训练需要什么特殊的硬件支持？它是如何平衡精度损失的？
- **AI 互动**：
  - "请列出一个表格，对比 FP32, FP16, BF16, FP8 的位宽（符号位、指数位、尾数位）、动态范围和适用场景。"
- **学习建议**：画出不同精度的 bit 结构图，直观感受“动态范围”和“精度”的取舍。
- **学习资料**：
  - NVIDIA Transformer Engine Documentation
  - Google Brain Float 16 paper

### Day 4: 卡间通信协议 (NVLink vs PCIe)

- **学习方向**：理解机内通信的物理链路，区分带宽 (Bandwidth) 和延迟 (Latency)。
- **核心要解决问题**：
  - 为什么 PCIe 4.0/5.0 的带宽不足以支撑模型并行 (Model Parallelism)？
  - NVLink 的 SerDes 链路是如何工作的？
- **AI 互动**：
  - "请画图解释 NVIDIA HGX H100 的物理拓扑。如果有 8 张卡，它们是如何通过 NVSwitch 实现 All-to-All 全互联的？"
- **学习建议**：把 NVLink 想象成 GPU 之间的高速公路，把 PCIe 想象成连接 CPU 的乡间小道。
- **学习资料**：
  - NVLink and NVSwitch Architecture
  - PCIe vs NVLink bandwidth comparison

### Day 5: 物理拓扑优化 (NVSwitch & Rail-only)

- **学习方向**：深度解析 NVSwitch 的全互联架构与 "Rail-only" 优化拓扑。
- **核心要解决问题**：
  - 解释 "Rail-only" 优化拓扑是什么？为什么它能减少跨交换机通信？
  - 在千卡集群中，如何设计物理布线以匹配 Rail-only 拓扑？
- **AI 互动**：
  - "请解释在多机训练中，为什么我们要把同一 Rail（比如每台机器的 GPU 0）连接到同一台 ToR 交换机上？"
- **学习建议**：这部分比较抽象，建议找一些集群布线图来看。
- **学习资料**：
  - NVIDIA SuperPOD Reference Architecture
  - Rail-optimized network topology AI training

### Day 6: 拓扑工具实战 (nvidia-smi topo)

- **学习方向**：实战解读 nvidia-smi topo -m，识别性能瓶颈。
- **核心要解决问题**：
  - 输出矩阵中的 PIX, NV, SYS, PHB 分别代表什么物理路径？哪个最快？
  - 如何通过拓扑图判断两张卡是否处于同一个 PCIe Root Complex？
- **AI 互动**：
  - "我将提供一个 nvidia-smi topo -m 的输出结果，请帮我分析这台机器内部的 GPU 物理连接图，并指出可能存在的通信瓶颈。"
- **学习建议**：找一台真实的 GPU 服务器（或者网上的截图）去逐行分析。
- **学习资料**：
  - nvidia-smi documentation topo matrix interpretation

### Day 7: RDMA 核心原理 (Zero Copy)

- **学习方向**：理解传统 TCP/IP 的内核拷贝瓶颈与 RDMA 的 Zero Copy (零拷贝) 原理。
- **核心要解决问题**：
  - 为什么说 CPU "不参与 RDMA 数据搬运"？
  - 详细拆解一次 RDMA Write 操作中，CPU、网卡、内存的交互流程。
- **AI 互动**：
  - "请用大白话解释 RDMA 的 'Kernel Bypass' 技术。当应用程序调用 ibv_post_send 时，数据是如何直接到达网卡的？"
- **学习建议**：重点理解 User Space (用户态) 直接访问硬件的概念。
- **学习资料**：
  - Remote Direct Memory Access (RDMA) Introduction
  - RoCE vs InfiniBand

### Day 8: RDMA 传输交互流程

- **学习方向**：深入 RDMA 的 Verbs API 概念（Send/Recv, Write/Read）。
- **核心要解决问题**：
  - RDMA Write 和 Send 操作的区别是什么？为什么训练中多用 Write？
  - 什么是 QP (Queue Pair)？
- **AI 互动**：
  - "请图解 RDMA 通信中的 Work Queue Element (WQE) 和 Completion Queue (CQ) 的流转过程。"
- **学习建议**：不要去写 C 代码，看懂流程图即可。
- **学习资料**：
  - RDMA Mojatatu Blog (非常经典的教程)

### Day 9: RoCE v2 协议详解

- **学习方向**：理解 RoCE v2 如何在以太网上实现 RDMA，以及包结构。
- **核心要解决问题**：
  - RoCE v1 和 v2 的区别？（v2 基于 UDP，可路由）。
  - RoCE v2 的包头包含哪些关键信息？
- **AI 互动**：
  - "我是网络运维，请解释 RoCE v2 协议包结构。为什么它是基于 UDP 的？这是否意味着它不可靠？"
- **学习建议**：使用 Wireshark 的截图来学习包结构。
- **学习资料**：
  - RoCE v2 Packet Format

### Day 10: 拥塞控制与流控 (PFC/ECN)

- **学习方向**：掌握 PFC (优先流控) 死锁、ECN 拥塞控制与 PFC Storm（风暴）排查。
- **核心要解决问题**：
  - 什么是 PFC Storm？它如何导致整个集群网络瘫痪？
  - 交换机上的 Watchdog 是怎么工作的？
- **AI 互动**：
  - "模拟故障：交换机监控显示某端口 PFC 帧激增，导致全网吞吐下降。请给出排查思路。"
- **学习建议**：这是面试中“网络故障排查”的高频考点，务必理解“无损网络”的代价。
- **学习资料**：
  - Understanding PFC and ECN in RoCE networks
  - PFC Deadlock explanation

### Day 11: K8s 网络集成 (Multus & SR-IOV)

- **学习方向**：学习如何在 K8s 中配置多网卡，实现管理网与业务网分离。
- **核心要解决问题**：
  - 如何在 K8s Pod 中配置多个网络接口？
  - Pod 如何知道哪张网卡对应哪个 GPU？(NUMA 亲和性)。
- **AI 互动**：
  - "请给我一个 K8s Pod 的 YAML 示例，展示如何通过 k8s.v1.cni.cncf.io/networks 注解挂载一张支持 RoCE 的 Mellanox 网卡。"
- **学习建议**：阅读 Multus CNI 的官方文档架构图。
- **学习资料**：
  - Multus CNI concept
  - NVIDIA k8s-network-operator

### Day 12: NCCL 算法原理 (Ring vs Tree)

- **学习方向**：掌握 NCCL 库的核心通信原语与底层算法。
- **核心要解决问题**：
  - 为什么小包通信适合 Tree 算法，大包通信适合 Ring 算法？
  - Ring All-Reduce 分为哪两个阶段？（Reduce-Scatter + All-Gather）。
- **AI 互动**：
  - "请一步步图解 Ring All-Reduce 算法。如果有 4 张卡，数据块是如何被切分并在卡之间流转的？"
- **学习建议**：在纸上画圆圈（Ring），模拟数据块的移动。
- **学习资料**：
  - NVIDIA NCCL Developer Guide Algorithms

### Day 13: NCCL 性能调优实战

- **学习方向**：学习 nccl-tests 基准测试工具及环境变量调优。
- **核心要解决问题**：
  - busbw 和 algbw 的换算公式是什么？
  - 测试结果如果只有理论带宽的 50%，排查思路是什么？
- **AI 互动**：
  - "解释环境变量 NCCL_IB_GID_INDEX 的作用，配置错误会导致什么后果？我运行 nccl-tests 发现带宽很低，该如何排查？"
- **学习建议**：下载 nccl-tests 源码看一眼 README。
- **学习资料**：
  - GitHub nccl-tests
  - NCCL Environment Variables

### Day 14: GPU Direct RDMA (GDR)

- **学习方向**：理解显存直通网卡技术原理与内核模块依赖。
- **核心要解决问题**：
  - GDR 如何消除 CPU 内存拷贝的瓶颈？
  - nvidia-peermem 模块的作用是什么？
- **AI 互动**：
  - "请解释 GPUDirect RDMA 的工作流程。当 GPU 需要发送数据到远程机器时，数据路径是怎样的？"
- **学习建议**：对比有无 GDR 时的数据路径图。
- **学习资料**：
  - NVIDIA GPUDirect RDMA technology

### Day 15: 阶段复盘 (算力集群交付)

- **学习方向**：总结第一阶段，制定集群验收标准。
- **核心要解决问题**：
  - 如何计算集群的 Rmax 和 Rpeak？
  - 验收一台 GPU 服务器需要检查哪些关键指标？
- **AI 互动**：
  - "请帮我草拟一份《GPU 训练集群交付验收清单》，包含硬件健康度、PCIe 带宽、RDMA 连通性、NCCL 性能四个维度的具体测试工具和合格标准。"
- **学习建议**：尝试用思维导图画出从硬件到网络的完整架构。

## 第二阶段：AI 负载与训练框架深度 (The Workload)

**阶段目标**：读懂 DeepSpeed 和 vLLM 的源码，理解其内存管理与调度逻辑。

### Day 16: PyTorch DDP 源码解析

- **学习方向**：深入 PyTorch DDP 的实现机制，特别是 Bucketing。
- **核心要解决问题**：
  - DDP 的 Bucketing 机制是如何优化小梯度通信的？
  - 梯度同步发生在 Forward 还是 Backward 阶段？
- **AI 互动**：
  - "请解释 PyTorch DDP 中的 bucket_cap_mb 参数的作用。为什么将小的梯度 Tensor 打包成桶能提升通信效率？"
- **学习建议**：关注 "Overlap"（通信计算重叠）的概念。
- **学习资料**：
  - PyTorch DDP internal design

### Day 17: DeepSpeed ZeRO 原理 (切分一切)

- **学习方向**：彻底搞懂 ZeRO-Stage 1/2/3 分别切分了什么。
- **核心要解决问题**：
  - ZeRO-1 (Optimizer State), ZeRO-2 (Gradient), ZeRO-3 (Parameter) 的区别？
  - 为什么说 ZeRO-3 是显存优化的极致但牺牲了通信？
- **AI 互动**：
  - "请画图解释 DeepSpeed ZeRO-3 的参数获取流程：在前向传播时，参数是如何从其他节点广播过来的？"
- **学习建议**：用“拼图”的概念来理解 ZeRO-3 的参数分片。
- **学习资料**：
  - DeepSpeed ZeRO paper (ZeRO: Memory Optimizations Toward Training Trillion Parameter Models)

### Day 18: ZeRO-Offload 与卸载技术

- **学习方向**：理解如何利用 CPU 内存和 NVMe 扩展显存。
- **核心要解决问题**：
  - ZeRO-Offload 将哪部分数据卸载到了 CPU？
  - PCIe 带宽如何成为 Offload 的瓶颈？
- **AI 互动**：
  - "分析 ZeRO-Offload 的性能损耗。在什么情况下（比如模型多大），启用 Offload 是划算的？"
- **学习建议**：关注数据在 GPU 显存和 CPU 内存之间的换入换出策略。
- **学习资料**：
  - DeepSpeed ZeRO-Offload tutorial

### Day 19: ZeRO 通信开销分析

- **学习方向**：深入分析 Gather/Scatter 过程中的网络压力。
- **核心要解决问题**：
  - ZeRO-3 相比 DDP，通信量增加了多少（1.5x?）？
  - 如何通过参数调优减少通信次数？
- **AI 互动**：
  - "对比 DDP 和 ZeRO-3 的通信模式。DDP 做的是 All-Reduce，那 ZeRO-3 做的是什么？（All-Gather + Reduce-Scatter）。"
- **学习建议**：回顾 Day 12 的 NCCL 原语。
- **学习资料**：
  - DeepSpeed communication analysis

### Day 20: Megatron-LM 之 Tensor Parallel (TP)

- **学习方向**：理解张量并行（切层内）的原理与通信模式。
- **核心要解决问题**：
  - TP 是如何将一个矩阵乘法切分到两张卡上的？（列切分 vs 行切分）。
  - 为什么 TP 必须依赖高带宽的 NVLink？
- **AI 互动**：
  - "请用伪代码演示 Megatron-LM 中 MLP 层的切分逻辑：第一个 Linear 层做列切分，第二个 Linear 层做行切分，中间需要 All-Reduce 吗？"
- **学习建议**：在纸上画矩阵乘法的切分图。
- **学习资料**：
  - Megatron-LM paper (Efficient Large-Scale Language Model Training on GPU Clusters)

### Day 21: Pipeline Parallel (PP) 与通信重叠

- **学习方向**：理解流水线并行及其气泡（Bubble）问题。
- **核心要解决问题**：
  - 什么是 PP 中的 "Bubble"？
  - 1F1B (One-Forward-One-Backward) 策略是如何减少气泡的？
- **AI 互动**：
  - "请解释 Pipeline Parallel 中的 GPipe 和 1F1B 调度策略的区别。为什么 1F1B 能节省显存？"
- **学习建议**：想象工厂流水线，中间有空闲就是 Bubble。
- **学习资料**：
  - GPipe vs 1F1B schedule

### Day 22: FlashAttention 瓶颈分析

- **学习方向**：理解标准 Attention 算法的 HBM 访问瓶颈。
- **核心要解决问题**：
  - 为什么标准 Attention 的显存占用是 $O(N^2)$？
  - 什么是 IO-Aware（IO 感知）优化？
- **AI 互动**：
  - "分析标准 Softmax 操作在 GPU 上的读写开销。为什么它需要多次读写 HBM？"
- **学习建议**：复习 Day 1 的 GPU 存储层级（HBM vs SRAM）。
- **学习资料**：
  - FlashAttention paper

### Day 23: FlashAttention 技术详解 (Tiling)

- **学习方向**：掌握 Tiling（分块）与 Recomputation（重计算）技术。
- **核心要解决问题**：
  - FlashAttention 如何通过分块计算利用 L1 Cache (SRAM)？
  - 为什么重计算反而比读写 HBM 更快？
- **AI 互动**：
  - "用大白话解释 FlashAttention 的 Tiling（分块）技术是如何利用 GPU SRAM（L1 Cache）的。它如何避免了存储巨大的 $N \times N$ 注意力矩阵？"
- **学习建议**：这是一个“以算力换带宽”的经典案例。
- **学习资料**：
  - FlashAttention v2 blog

### Day 24: 显存估算实战 (手算模型)

- **学习方向**：综合运用前面的知识，手算显存需求。
- **核心要解决问题**：
  - 训练一个 175B 模型，显存主要被谁占用了？
  - Adam 优化器状态占多少？（参数量的 2 倍还是 3 倍？）。
- **AI 互动**：
  - "计算题：训练 175B 参数的模型，使用 AdamW 优化器，混合精度 (FP16)，Batch Size=1。请列出权重、梯度、优化器状态分别占用的显存大小。"
- **学习建议**：建立 Excel 表格，输入参数量自动计算。
- **学习资料**：
  - Transformer memory estimation calculator

### Day 25: Checkpoint 存储优化

- **学习方向**：学习异步保存与分布式 Checkpointing。
- **核心要解决问题**：
  - 保存 TB 级模型时，如何避免阻塞训练进程？
  - Checkpoint 保存瞬间 GPU 利用率为什么会掉底？
- **AI 互动**：
  - "设计一个存储架构方案：支持千卡集群同时 Checkpoint 写入，要求 GPU IO Wait 低于 5%。提示：考虑 CPU 内存缓冲和异步上传。"
- **学习建议**：结合 Day 33 的 JuiceFS 学习。
- **学习资料**：
  - PyTorch Distributed Checkpointing

### Day 26: vLLM 之内存管理困境

- **学习方向**：深入理解传统 KV Cache 的显存碎片与预分配浪费问题。
- **核心要解决问题**：
  - 为什么 Transformer 推理时 KV Cache 会随着 Sequence 长度动态增长？
  - 预分配最大长度会导致多少显存浪费？
- **AI 互动**：
  - "请解释 KV Cache 是什么？为什么在推理阶段，每生成一个 Token 就需要读取所有的 KV Cache？"
- **学习建议**：回顾操作系统的“内存碎片”概念。
- **学习资料**：
  - vLLM blog post PagedAttention

### Day 27: PagedAttention 核心原理

- **学习方向**：学习 vLLM 如何模仿操作系统虚拟内存，设计 Block Table。
- **核心要解决问题**：
  - PagedAttention 如何通过逻辑块到物理块的映射，实现显存的零浪费？
  - Block Size 的大小（如 16 或 32）对性能有什么影响？
- **AI 互动**：
  - "请用 Python 字典类比 PagedAttention 的 Block Table，解释它是如何将逻辑上的连续 Token 映射到物理上不连续的显存块的。"
- **学习建议**：画出 逻辑块 -> Block Table -> 物理块 的映射图。
- **学习资料**：
  - vLLM paper

### Day 28: vLLM 源码阅读

- **学习方向**：浏览 vLLM 代码库，定位核心逻辑。
- **核心要解决问题**：
  - 定位 BlockManager 类，看它是如何分配和释放 Block 的。
  - 定位 CacheEngine 类，看它是如何操作 GPU 显存的。
- **AI 互动**：
  - "请帮我解读 vLLM 源码中 scheduler.py 的 _schedule 方法。它是如何判断当前显存是否足够容纳新的 Token 的？"
- **学习建议**：Clone 代码库到本地，使用 IDE 跳转阅读。
- **学习资料**：
  - GitHub vllm-project/vllm

### Day 29: 静态批处理 vs 连续批处理

- **学习方向**：对比 Static Batching 与 Continuous Batching。
- **核心要解决问题**：
  - 静态批处理的“短板效应”是什么？（等待最长请求）。
  - Continuous Batching 如何在迭代级（Iteration-level）插入新请求？
- **AI 互动**：
  - "模拟一个推理场景：同时来了两个请求，A 请求输入长输出短，B 请求输入短输出长。请描述 Continuous Batching 是如何调度 GPU 计算周期的。"
- **学习建议**：画甘特图对比两种调度方式的空闲时间。
- **学习资料**：
  - Orca paper (OSDI 2022) - Continuous Batching 的理论基础。

### Day 30: 推理延迟指标 (TTFT vs TPOT)

- **学习方向**：理解推理性能的关键指标。
- **核心要解决问题**：
  - TTFT (Time to First Token) 受什么影响最大？（Prefill 阶段）。
  - TPOT (Time Per Output Token) 受什么影响最大？（Decode 阶段）。
- **AI 互动**：
  - "为什么 Batch Size 增大可以提升吞吐量（Throughput），但可能会恶化 TPOT？请从 GPU 计算能力和带宽角度分析。"
- **学习建议**：在压测中观察这两个指标的互斥关系。
- **学习资料**：
  - LLM inference performance metrics

### Day 31: 模型量化原理 (AWQ/GPTQ)

- **学习方向**：W4A16 量化原理与反量化计算。
- **核心要解决问题**：
  - W4A16 量化在推理时是如何反量化计算的？带宽需求减少了多少？
  - Weight-only 量化和 Activation 量化的区别？
- **AI 互动**：
  - "在 vLLM 中部署一个 AWQ 量化的模型，相比 FP16 原版，显存占用和推理延迟（Latency）会有什么具体的量级变化？"
- **学习建议**：关注量化对 "Memory Bound" 场景的缓解作用。
- **学习资料**：
  - AWQ: Activation-aware Weight Quantization

### Day 32: Triton Inference Server

- **学习方向**：NVIDIA 官方推理服务框架的功能。
- **核心要解决问题**：
  - Triton 的 Dynamic Batching 和 vLLM 的 Continuous Batching 有什么区别？
  - Model Ensemble 如何串联预处理和推理步骤？
- **AI 互动**：
  - "请解释 Triton Inference Server 的 Model Ensemble 功能，如何用它串联 Pre-processing (Python) 和 TensorRT Inference 两个步骤？"
- **学习建议**：Triton 是通用的，vLLM 是专用的，理解它们的生态位。
- **学习资料**：
  - NVIDIA Triton Architecture

### Day 33: 存储 I/O 优化 (JuiceFS)

- **学习方向**：针对 AI 场景的存储优化（元数据与缓存）。
- **核心要解决问题**：
  - 训练 ImageNet 这种百万小文件数据集，JuiceFS 元数据缓存如何优化？
  - 什么是 juicefs warm？原理是什么？
- **AI 互动**：
  - "解释 JuiceFS 的 'Client Write Cache'。开启它对训练 Checkpoint 保存速度有多大提升？"
- **学习建议**：结合你在 JuiceFS 的经验，思考其在 AI 场景的特殊配置。
- **学习资料**：
  - JuiceFS best practices for AI training

### Day 34: 故障分析 Lab (Hang/Loss Spike)

- **学习方向**：常见 AI 任务故障的全链路排查。
- **核心要解决问题**：
  - 训练过程中 Loss 突然变成 NaN，可能的原因有哪些？
  - 训练任务卡住（Hang），GPU 利用率为 0，显存不释放，如何排查？
- **AI 互动**：
  - "训练任务卡住（Hang），GPU 利用率为 0，但显存没释放。请给出一个排查 Checklist，包括 NCCL 环境变量和堆栈分析（py-spy）。"
- **学习建议**：积累一份自己的 "Debug Checklist"。
- **学习资料**：
  - PyTorch troubleshooting guide

### Day 35: OOM 故障排查 Lab

- **学习方向**：Out Of Memory (OOM) 的深度分析。
- **核心要解决问题**：
  - 如何区分是显存碎片导致的 OOM 还是真的显存不足？
  - PYTORCH_CUDA_ALLOC_CONF 环境变量如何缓解碎片？
- **AI 互动**：
  - "请解释 max_split_size_mb 环境变量的作用。它是如何通过禁止大块内存切分来减少碎片的？"
- **学习建议**：学习使用 torch.cuda.memory_summary()。
- **学习资料**：
  - PyTorch Memory Management

## 第三阶段：K8s 平台开发与高级调度 (The Platform Code)

**阶段目标**：真的去写 Go 代码。这是区别于普通运维的关键。

### Day 36: K8s Informer 机制

- **学习方向**：深入 Client-go 的 List-Watch 与 Informer 源码。
- **核心要解决问题**：
  - Informer 的 Resync 机制是做什么的？它会触发 Update 事件吗？
  - SharedInformerFactory 解决了什么问题？
- **AI 互动**：
  - "请画图解释 K8s Client-go 中 Reflector, DeltaFIFO, Indexer 和 Informer 的数据流向。Controller 从哪里读取数据？"
- **学习建议**：看 Client-go 源码架构图。
- **学习资料**：
  - K8s client-go under the hood

### Day 37: Kubebuilder 入门

- **学习方向**：Operator 模式概念，Kubebuilder 项目初始化。
- **核心要解决问题**：
  - CRD (Custom Resource Definition) 和 CR (Custom Resource) 的区别是什么？
  - kubebuilder init 生成了哪些核心文件？
- **AI 互动**：
  - "使用 Kubebuilder init 一个项目，请解释生成的 api/v1/group_types.go 文件中 Status 字段的作用。"
- **学习建议**：实际动手跑一遍 Kubebuilder 的 Tutorial。
- **学习资料**：
  - Kubebuilder Book

### Day 38: Go 结构体与 API 设计

- **学习方向**：理解 main.go 和 types.go 结构。
- **核心要解决问题**：
  - 如何利用 Go 的 Tag (json:"spec,omitempty") 定义 CRD 字段？
  - Scheme 在 K8s API 中的作用是什么？
- **AI 互动**：
  - "请解释 K8s Runtime Scheme 的概念。它是如何实现 Go Struct 和 YAML 之间的序列化/反序列化的？"
- **学习建议**：复习 Go 语言的 Struct 和 Interface。
- **学习资料**：
  - Programming Kubernetes (书籍)

### Day 39: 编写 CRD Controller (API 定义)

- **学习方向**：实战定义一个 CRD 结构体。
- **核心要解决问题**：
  - 如何定义一个包含 Replicas 和 Image 字段的 AppService CRD？
- **AI 互动**：
  - "请帮我写一个 Go Struct 定义，对应一个名为 AppService 的 CRD，包含 image (string) 和 replicas (int) 字段。"
- **学习建议**：模仿 K8s 原生 Deployment 的结构。
- **学习资料**：
  - Kubernetes API conventions

### Day 40: 编写 Reconcile 逻辑

- **学习方向**：编写 Controller 的核心调谐循环。
- **核心要解决问题**：
  - Reconcile 函数如果返回 error，Controller 会怎么做？
  - 如何通过 client.Get 获取 CR 对象？
- **AI 互动**：
  - "请给我一段 Kubebuilder 的 Reconcile 代码示例：如何获取 CR 对象，并判断关联的 Pod 是否存在？如果不存在则创建。"
- **学习建议**：理解“声明式 API”和“最终一致性”。
- **学习资料**：
  - Controller Runtime documentation

### Day 41: Controller 本地测试与部署

- **学习方向**：使用 make install 和 make run 进行测试。
- **核心要解决问题**：
  - 如何在本地运行 Controller 而不是打包到集群中？(连接 Kubeconfig)。
- **AI 互动**：
  - "解释 make install 和 make deploy 的区别。make install 是把 CRD 注册到集群吗？"
- **学习建议**：学会看 Controller 的启动日志。
- **学习资料**：
  - Kubebuilder Quick Start

### Day 42: Device Plugin 源码 (gRPC 接口)

- **学习方向**：阅读 NVIDIA Device Plugin 源码，理解 gRPC 接口。
- **核心要解决问题**：
  - 插件是如何通过 gRPC 告诉 Kubelet 设备 ID 的？
  - ListAndWatch 接口是如何持续上报设备状态的？
- **AI 互动**：
  - "请解释 K8s Device Plugin API 中的 Allocate 方法。它的输入是什么？输出的环境变量和挂载路径是如何传递给容器运行时的？"
- **学习建议**：下载 k8s.io/kubelet/pkg/apis/deviceplugin/v1beta1 看 protobuf 定义。
- **学习资料**：
  - K8s Device Plugin Framework documentation

### Day 43: Device Plugin 源码 (Server 实现)

- **学习方向**：阅读 NVIDIA 插件的 server.go。
- **核心要解决问题**：
  - NVIDIA 插件是如何调用 NVML (NVIDIA Management Library) 获取显卡信息的？
- **AI 互动**：
  - "分析 NVIDIA Device Plugin 的源码逻辑：当一张卡出现 XID 错误时，插件是如何将其标记为 Unhealthy 的？"
- **学习建议**：关注 NVML 的 Go 绑定库。
- **学习资料**：
  - GitHub NVIDIA/k8s-device-plugin

### Day 44: 编写自定义 Device Plugin (Stub)

- **学习方向**：实战用 Go 写一个 Mock Plugin 骨架。
- **核心要解决问题**：
  - 编写 Device Plugin 时，如何处理 Kubelet 重启的情况？(重新注册)。
- **AI 互动**：
  - "请给我一个 Go 语言编写的简单的 Device Plugin 骨架代码，实现向 Kubelet 注册名为 'https://www.google.com/search?q=example.com/dummy' 的资源。"
- **学习建议**：先跑通官方的 sample device plugin。
- **学习资料**：
  - K8s sample device plugin

### Day 45: 编写自定义 Device Plugin (Allocate)

- **学习方向**：实现 Allocate 逻辑。
- **核心要解决问题**：
  - 如何让 Allocate 返回一个特定的环境变量（如 DUMMY_DEVICE=1）给容器？
- **AI 互动**：
  - "如何在 Device Plugin 的 Allocate 响应中，指定将宿主机的 /tmp/dummy 目录挂载到容器内？"
- **学习建议**：这是理解“硬件如何透传进容器”的关键。
- **学习资料**：
  - Container Device Interface (CDI) (进阶知识)

### Day 46: K8s 调度器框架 (Scheduling Framework)

- **学习方向**：深入理解 Filter, Score, Bind 扩展点。
- **核心要解决问题**：
  - 调度器插件中的 PreFilter 和 Filter 有什么区别？
  - 如果我想根据显存大小调度，应该用哪个扩展点？
- **AI 互动**：
  - "请解释 K8s Scheduling Framework 的扩展点。如果我要实现一个'显存感知'调度器，应该在哪个扩展点注入逻辑？"
- **学习建议**：看 K8s 调度器的官方架构图。
- **学习资料**：
  - K8s Scheduling Framework

### Day 47: 编写自定义调度插件 (Score 插件)

- **学习方向**：实战编写 BinPacking 插件。
- **核心要解决问题**：
  - Bin Packing 算法在 K8s Score 阶段是如何实现的？(即资源利用率越高得分越高)。
- **AI 互动**：
  - "请写一段 Go 代码，实现一个 Scoring Plugin 的逻辑：优先将 Pod 调度到已分配资源较多的节点（Bin Packing 策略）。"
- **学习建议**：参考 NodeResourcesFit 插件的源码。
- **学习资料**：
  - GitHub kubernetes-sigs/scheduler-plugins

### Day 48: 编写自定义调度插件 (部署)

- **学习方向**：如何将自定义调度器部署到集群。
- **核心要解决问题**：
  - 如何通过 scheduler-config.yaml 启用我的自定义插件？
  - 如何运行第二个调度器 (Secondary Scheduler) 而不替换默认调度器？
- **AI 互动**：
  - "请给我一个 K8s Scheduler Configuration 的 YAML 示例，展示如何启用名为 'my-binpacking' 的自定义 Score 插件。"
- **学习建议**：多调度器共存是生产环境的常见做法。
- **学习资料**：
  - Configure Multiple Schedulers K8s

### Day 49: Volcano 调度器架构 (PodGroup)

- **学习方向**：理解 Volcano 的核心 CRD：PodGroup 和 Queue。
- **核心要解决问题**：
  - Volcano 是如何解决 K8s 默认调度器不支持 All-or-Nothing 的问题的？
  - PodGroup 状态机是如何流转的？(Pending -> Scheduling -> Running)。
- **AI 互动**：
  - "请演示一个 Volcano Job 的 YAML，重点展示 minMember 字段是如何保证 10 个 Pod 同时启动否则全部等待的。"
- **学习建议**：部署一个 Volcano 到测试集群玩一下。
- **学习资料**：
  - Volcano sh documentation

### Day 50: Gang Scheduling 源码分析

- **学习方向**：阅读 Volcano 的 Gang Plugin 源码。
- **核心要解决问题**：
  - Volcano 在调度循环中是如何检查整个 Group 的资源是否满足的？
- **AI 互动**：
  - "解释 Volcano 调度器中的 'Action' 和 'Plugin' 的概念。Gang Scheduling 属于哪个阶段？"
- **学习建议**：理解“整体视图”与“单个 Pod 视图”的区别。
- **学习资料**：
  - Volcano Gang Scheduling design

### Day 51: 拓扑感知调度 (Topology Manager)

- **学习方向**：源码层面理解 NUMA 亲和性计算逻辑。
- **核心要解决问题**：
  - K8s Topology Manager 的 SingleNUMANode 策略是如何工作的？
  - 它是如何协调 CPU Manager 和 Device Manager 的？
- **AI 互动**：
  - "解释 K8s Topology Manager 是如何通过 Bitmask (位掩码) 算法计算 CPU、内存和 GPU 的最佳亲和性节点的。"
- **学习建议**：Bitmask 算法是核心。
- **学习资料**：
  - K8s Topology Manager Feature Gate

### Day 52: Kubeflow Training Operator 原理

- **学习方向**：Operator 如何管理 Master/Worker 服务发现。
- **核心要解决问题**：
  - PyTorchJob 如何处理 Worker 0 (Master) 的服务发现？
  - Headless Service 起什么作用？
- **AI 互动**：
  - "详细解释 Kubeflow Training Operator 是如何利用 K8s Service 实现分布式节点 DNS 自动发现的。"
- **学习建议**：查看 PyTorchJob 创建的 Service 和 Pod 的对应关系。
- **学习资料**：
  - Kubeflow Training Operator architecture

### Day 53: 弹性训练 (TorchElastic)

- **学习方向**：理解 c10d 后端与动态节点增删。
- **核心要解决问题**：
  - rdzv_backend (etcd/c10d) 的作用是什么？
  - Worker 挂了如何不重启训练？
- **AI 互动**：
  - "请解释 C10d 的 Rendezvous (集合点) 机制。当一个 Worker 失败时，TorchElastic Agent 是如何触发集群重启的？"
- **学习建议**：区分 Static 训练和 Elastic 训练的启动命令差异。
- **学习资料**：
  - PyTorch Elastic Training tutorial

### Day 54: 资源配额与多租户 (ElasticQuota)

- **学习方向**：学习多租户借贷模型实现。
- **核心要解决问题**：
  - 如何实现“组 A 没用完的资源，组 B 可以借用，但组 A 要用时必须归还”？
- **AI 互动**：
  - "请对比 K8s 原生 ResourceQuota 和 Volcano 的 ElasticQuota。ElasticQuota 的 'Min' 和 'Max' 字段分别控制什么？"
- **学习建议**：这是提升集群资源利用率的关键。
- **学习资料**：
  - Volcano ElasticQuota

### Day 55: KEDA 自定义指标扩容

- **学习方向**：KEDA Prometheus Scaler 源码分析。
- **核心要解决问题**：
  - 为什么 HPA 用 CPU 指标无法准确扩容推理服务？
  - KEDA 是如何将 Prometheus 查询结果转化为 HPA 的 Metric Value 的？
- **AI 互动**：
  - "请给我一个 KEDA Scaler 的配置 YAML，通过查询 Prometheus 中的 vllm_num_requests_waiting 指标来触发扩容。"
- **学习建议**：理解 External Metrics API。
- **学习资料**：
  - KEDA Prometheus Scaler

### Day 56: Serverless 冷启动挑战

- **学习方向**：Knative 的 Scale-to-Zero。
- **核心要解决问题**：
  - 当 Pod 数为 0 时，流量进来会发生什么？(Activator 组件)。
  - 大模型动辄 50GB，如何实现秒级冷启动？
- **AI 互动**：
  - "解释 Knative 的 Activator 组件。当服务缩容到 0 后，新请求是如何触发 Pod 创建的？"
- **学习建议**：思考存储预热对 Serverless 的重要性。
- **学习资料**：
  - Knative Scaling to Zero

### Day 57: P2P 模型分发 (Dragonfly)

- **学习方向**：P2P 预热技术原理。
- **核心要解决问题**：
  - 大规模集群如何避免 Image Pull BackOff？
  - P2P 分发为什么比 Registry 快？
- **AI 互动**：
  - "设计一个 P2P 模型分发方案：利用 Dragonfly 或 Kraken，在 100 个节点并发拉取 50GB 模型镜像时，如何避免打爆 Registry？"
- **学习建议**：理解 Seed Peer 和 Peer 的概念。
- **学习资料**：
  - Dragonfly P2P image distribution

## 第四阶段：稳定性、运维与综合冲刺 (The Stability)

**阶段目标**：学会“炸”集群并修好它，建设生产级稳定性。

### Day 58: 深度监控 (DCGM & Prometheus)

- **学习方向**：编写复杂的 PromQL 监控 GPU。
- **核心要解决问题**：
  - DCGM_FI_DEV_ECC_DBE_VOL_TOTAL (双比特错误) 意味着什么？
  - 如何计算集群整体的 GPU 利用率（排除闲置节点）？
- **AI 互动**：
  - "写一个 PromQL：计算过去 5 分钟内，集群中 GPU 利用率低于 10% 且显存占用高于 80% 的节点列表（疑似僵尸任务）。"
- **学习建议**：熟练掌握 PromQL 的 group by 和 avg_over_time。
- **学习资料**：
  - DCGM Exporter metrics list

### Day 59: 性能剖析 (Nsys)

- **学习方向**：使用 Nsys 读懂 Timeline，识别 CPU Bound。
- **核心要解决问题**：
  - 在 Timeline 视图中，如果看到大量的 CPU Gap (GPU 空闲)，通常是什么原因？
  - 如何判断是 Kernel 计算慢还是数据加载慢？
- **AI 互动**：
  - "如何使用 Nsight Systems (nsys) 抓取一个 PyTorch 分布式训练任务的 Profile？请给出命令行参数示例。"
- **学习建议**：下载 NVIDIA Nsight Systems 并在本地查看报告。
- **学习资料**：
  - NVIDIA Nsight Systems User Guide

### Day 60: 混沌工程 (Chaos Mesh)

- **学习方向**：使用 Chaos Mesh 主动注入网络故障。
- **核心要解决问题**：
  - 如果随机丢弃 10% 的 RDMA 包，训练速度会下降多少？NCCL 会报错吗？
- **AI 互动**：
  - "请给我一个 Chaos Mesh 的 YAML 示例，模拟在 K8s Pod 的 RDMA 网卡上增加 50ms 的网络延迟。"
- **学习建议**：在测试环境中进行，千万别在生产环境跑！
- **学习资料**：
  - Chaos Mesh NetworkChaos

### Day 61: 混沌工程实战 (Pod 故障)

- **学习方向**：模拟 Pod 随机被杀。
- **核心要解决问题**：
  - 当 Master 节点的 Pod 被杀时，Operator 能否自动恢复任务？
- **AI 互动**：
  - "模拟 Pod Failure：每隔 10 分钟随机杀掉一个训练 Worker Pod，观察 TorchElastic 的恢复行为。"
- **学习建议**：观察任务恢复的时间成本。
- **学习资料**：
  - Chaos Mesh PodChaos

### Day 62: 故障自愈 Operator 设计

- **学习方向**：设计自动化运维逻辑。
- **核心要解决问题**：
  - 遇到 XID 61 (掉卡) 错误，标准处理流程是什么？
- **AI 互动**：
  - "设计一个故障自愈状态机：监控到 XID 错误 -> Taint Node -> Drain Pod -> 重置 GPU 驱动 -> 移除 Taint。"
- **学习建议**：画出状态转移图。
- **学习资料**：
  - NVIDIA GPU Operator self-healing

### Day 63: 网络故障排查 (NCCL Timeout)

- **学习方向**：NCCL Timeout 全链路排查。
- **核心要解决问题**：
  - 怎么用 tcpdump 或 ibdump 抓包分析？
  - NCCL_DEBUG=INFO 能看到什么信息？
- **AI 互动**：
  - "当训练日志中出现 'NCCL Timeout'，请给出一个从物理网卡、RoCE PFC 配置到 K8s CNI 的全链路排查 Checklist。"
- **学习建议**：学会阅读 NCCL 的调试日志。
- **学习资料**：
  - NCCL Troubleshooting Guide

### Day 64: 存储故障排查 (JuiceFS)

- **学习方向**：JuiceFS 性能排查。
- **核心要解决问题**：
  - JuiceFS 的 block_cache_hits 低说明什么？
  - 对象存储的带宽限制会如何影响训练？
- **AI 互动**：
  - "帮我分析：训练任务 GPU 利用率忽高忽低（锯齿状），怀疑是 I/O 瓶颈。如何使用 juicefs stats 确认是否是存储读写慢拖累了 GPU？"
- **学习建议**：理解缓存穿透的影响。
- **学习资料**：
  - JuiceFS performance tuning

### Day 65: FinOps 成本治理

- **学习方向**：闲置资源治理与 Spot 实例。
- **核心要解决问题**：
  - 如何发现申请了 GPU 但没跑任务的“僵尸 Pod”？
  - Spot 实例被回收时，如何优雅终止训练？
- **AI 互动**：
  - "请写一个 Python 脚本逻辑，通过查询 Kubelet API 和 DCGM 指标，找出过去 24 小时 GPU 平均利用率低于 5% 的 Pod 及其 Owner。"
- **学习建议**：成本是老板最关心的指标之一。
- **学习资料**：
  - K8s FinOps best practices

### Day 66: 异构算力适配 (AMD/Huawei)

- **学习方向**：非 NV 生态的适配难点。
- **核心要解决问题**：
  - ROCm (AMD) 和 CUDA 在 K8s 调度上有什么区别？
  - Ascend NPU 的 Device Plugin 是怎么写的？
- **AI 互动**：
  - "作为平台方，如何设计一套统一的 Operator，屏蔽底层是 NVIDIA GPU 还是华为 Ascend NPU 的差异？"
- **学习建议**：了解一下 "Hardware Agnostic" 的概念。
- **学习资料**：
  - Huawei Ascend Docker Runtime

### Day 67: 模拟面试 - 系统设计

- **学习方向**：万卡集群架构设计。
- **核心要解决问题**：
  - "请设计一个支持万卡训练的 AI 基础设施，从机房选址到 K8s 调度。"
- **AI 互动**：
  - "我是一名应聘 AI Infra 高级工程师的候选人。请扮演面试官，对我进行压力面试。请出一道关于 '万卡集群训练稳定性保障' 的系统设计题。"
- **学习建议**：关注可扩展性（Scalability）和故障域（Failure Domain）。

### Day 68: 模拟面试 - 现场编程

- **学习方向**：Go 语言 K8s 编程面试。
- **核心要解决问题**：
  - "用 Go 写一个简单的 K8s Informer，监听 Pod 变化并打印日志。"
- **AI 互动**：
  - "请出题：要求我用 client-go 写一个简单的程序，列出集群中所有 GPU 节点的名称。并给我的代码评分。"
- **学习建议**：背诵 Informer 的 Boilerplate 代码。

### Day 69: 模拟面试 - 故障排查

- **学习方向**：实际场景故障分析。
- **核心要解决问题**：
  - "训练速度突然慢了 50%，排查思路。"
- **AI 互动**：
  - "扮演面试官，描述一个'训练 Loss 突然不下降，且 GPU 利用率波动'的故障场景，让我进行排查，并根据我的回答给出反馈。"
- **学习建议**：展现逻辑性，先排查大概率事件（网络/IO），再看小概率事件（硬件 bug）。

### Day 70+: 前沿技术加餐 (SGLang)

- **学习方向**：RadixAttention 原理。
- **核心要解决问题**：
  - SGLang 相比 vLLM，在处理多轮对话或 JSON 格式强制约束时有哪些性能优势？
- **AI 互动**：
  - "请对比 vLLM 的 PagedAttention 和 SGLang 的 RadixAttention。在 'Few-shot Learning'（少样本学习）场景下，RadixAttention 是如何避免重复处理 Examples 的？"
- **学习建议**：阅读 SGLang 的论文，关注 Agent 场景的优化。
- **学习资料**：
  - SGLang paper
