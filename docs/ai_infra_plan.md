# AI Infra 高级开发工程师转型路线图 


## 一、 技术栈

### 1. 绝对核心：vLLM & 推理引擎 

* **Memory Management:** 彻底精通 **PagedAttention**。理解 Block Table、Logical/Physical Block 映射，以及它如何解决显存碎片化问题。
* **Scheduling:** 精通 **Continuous Batching** (Iteration-level scheduling)。理解 Scheduler 如何在每个 Step 决定哪些 Request 进 Running Queue，哪些被 Preempt（抢占），哪些被 Swap Out（换出到 CPU 内存）。
* **Distributed Inference:** 理解 **Tensor Parallelism (TP)** 和 **Pipeline Parallelism (PP)** 在 vLLM 中的实现。熟悉 Ray 是如何被 vLLM 用来做分布式 Worker 管理的。
* **Kernels:** 能看懂 vLLM 的 Pytorch Custom Ops (C++/CUDA)。重点关注 Attention Kernel 和 Activation Kernel。

### 2. 算力与硬件亲和性 

* **GPU Topology:** 必须懂 **NVLink** vs **PCIe**。知道为什么 TP (Tensor Parallelism) 必须跑在 NVLink 互联的卡上。
* **Quantization (量化):** 掌握 **AWQ**、**GPTQ**、**FP8**。理解它们如何通过降低精度来换取更高的 **Memory Bandwidth**（显存带宽），从而提升 Token 生成速度。
* **CUDA Streams:** 理解 vLLM 是如何利用 CUDA Streams 实现计算和数据拷贝重叠的。

### 3. 云原生与编排 
* **K8s + Ray:** 现在的标配是 **KubeRay**。K8s 管资源 (Pod)，Ray 管任务 (Actor)。vLLM 通常作为一个 Ray Actor 运行。
* **Autoscaling:** 基于 vLLM 内部指标（`num_requests_waiting` 或 `gpu_cache_usage`）的 **KEDA** 弹性伸缩，而不是 CPU 使用率。
* **Observability:** 搭建基于 vLLM Metrics 的 Prometheus/Grafana 大盘。

### 4. 开发语言
* **Python (100%):** vLLM 的控制流全在 Python。必须精通 `asyncio` 和 `multiprocessing`。
* **C++ (10%):** 阅读底层 Kernel，无需手写但需看懂。
* **Golang (Expert):** 仅用于外围系统（网关、Sidecar、Operator）。**严禁用于推理热路径。**

---

## 二、 深度学习规划 

### 阶段一：vLLM 解剖学 (单卡原理)
**目标：** 能够回答“一个 Token 是怎么在显存里生出来的”所有细节。

#### 1. 源码阅读 (Core Components)
* **重点文件:** `vllm/core/scheduler.py`, `vllm/core/block_manager.py`, `vllm/engine/llm_engine.py`。
* **任务:** 画出 vLLM 的核心状态机。当一个 Request 到达时，Block Manager 是如何分配 Block 的？当显存不足时，Scheduler 选择哪个 Request 进行 Preemption？
* **验收标准:** 修改源码，增加日志，追踪一个 Request ID 在整个生命周期内的 Block 分配和释放过程。

#### 2. 显存管理机制 (PagedAttention)
* **概念:** 对比 OS 的 Virtual Memory 和 vLLM 的 PagedAttention。
* **任务:** 手动计算 Llama-3-8B 在 Context Window 为 8192 时，KV Cache 需要多少显存？Block Size 设置为 16 和 32 对碎片率有什么影响？
* **验收标准:**  能够解释为什么 vLLM 不需要 Padding，以及这对吞吐量带来的具体数学提升。

---

### 阶段二：分布式推理与性能压测
**目标：** 解决“单卡装不下”和“QPS 上不去”的问题。

#### 1. Tensor Parallelism (TP) 实战
* **概念:** 理解 Matrix Multiplication (MatMul) 是如何被切分到多张卡上并行计算，最后通过 AllReduce 同步结果的。
* **任务:** 配置 vLLM 启动多卡推理 (`tensor_parallel_size=2` 或 `4`)。
* **验收标准:** 使用 `nsys` (Nsight Systems) 抓取 Profile，识别出 NCCL 通信（AllReduce）在 Timeline 上的位置，判断通信是否成为瓶颈。

#### 2. 极限压测与参数调优
* **工具:** `vllm/benchmarks/benchmark_serving.py`。
* **指标:** 区分 **TTFT** (Time To First Token) 和 **TPOT** (Time Per Output Token)。
* **任务:** 保持 QPS 不变，调整 `--max-num-seqs` 和 `--max-num-batched-tokens`，观察 Latency 的变化曲线。
* **验收标准:** 找到特定硬件下 Llama-3-70B 的“甜点配置”（Sweet Spot），即吞吐量最大且延迟可接受的参数组合。

---

### 阶段三：生产级部署与稳定性
**目标:** 既然你是 Infra，你要保证这玩意儿在 K8s 上不炸。

#### 1. Ray on Kubernetes (KubeRay)
* **任务:** 使用 Helm 部署 KubeRay Operator。编写 RayService CRD 来管理 vLLM 集群。
* **验收标准:** 模拟 Worker 节点宕机，观察 Ray 如何自动拉起新的 vLLM Worker 并重新加入集群。

#### 2. 基于 Metrics 的弹性伸缩
* **痛点:** 显存满了请求会排队，导致延迟飙升。
* **任务:** 部署 Prometheus + KEDA。
* **逻辑:** 监听 vLLM 暴露的 `/metrics` 接口。当 `vllm:num_requests_waiting > 5` 持续 30秒时，触发 Pod 扩容。
* **验收标准:** 压测直到触发扩容，记录扩容期间的请求延迟抖动。

---

### 阶段四：前沿优化 (Expert Level)
**目标:** 成为团队里的技术权威。

#### 1. 量化部署 (Quantization)
* **任务:** 对比 **FP16** 版本和 **AWQ (INT4)** 版本的 vLLM 性能。
* **观察:** 记录显存占用减少了多少？TPOT 提升了多少？
* **验收标准:** 解释为什么在 Batch Size 较小时，量化带来的加速不明显（Compute Bound vs Memory Bound）。

#### 2. Speculative Decoding (投机采样)
* **概念:** 用小模型（Drafter）猜 Token，大模型验证。
* **任务:** 在 vLLM 中开启 Speculative Decoding。
* **验收标准:** 测算在不同 Prompt 场景下的“接受率”（Acceptance Rate），评估是否值得开启此功能。

---

## 三、 面试必问的 vLLM 杀手锏问题
*(自己对着镜子练，答不上来就回头重学)*

1.  **"请详细描述 vLLM 的 Continuous Batching 和传统的 Static Batching 有什么区别？为什么它能显著提升吞吐？"**
  * *关键词:* Padding 浪费, Iteration-level 调度, 提前结束的序列释放资源。

2.  **"如果 vLLM 出现显存碎片导致 OOM，调整哪个参数最有效？block_size 设置为 16 和 128 有什么 trade-off？"**
  * *关键词:* 内部碎片, Kernel 读取效率, 寻址开销。

3.  **"在多卡推理中，Ray 和 NCCL 分别扮演什么角色？"**
  * *关键词:* Ray 负责控制面（进程启动/元数据），NCCL 负责数据面（Tensor 通信）。

4.  **"我发现 GPU 利用率只有 60%，但请求延迟已经很高了，可能的原因是什么？"**
  * *关键词:* CPU Overhead (Python 代码慢), Kernel Launch Bound, 通信瓶颈, 或者是 Batch Size 太小导致没吃满带宽。