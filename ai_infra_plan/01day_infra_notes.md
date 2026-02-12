# AI Infra 学习笔记（Day 01）

## 0. 学习目标与一页速览

### 0.1 Day 01 目标

- 建立 GPU 计算与存储的整体认知框架。
- 理解 `SM / Warp / HBM / Tensor Core` 在性能中的角色。
- 形成第一版性能排查直觉（为什么“显存高但利用率低”）。

### 0.2 一页速览

- `SM` 是 GPU 的基本计算单元，可理解为“标准化车间”。
- `CUDA Core` 负责通用算术，`Tensor Core` 负责高吞吐矩阵乘加（MMA）。
- 大模型瓶颈常在数据搬运，而不是纯算力。
- CUDA 代码中的 `Grid / Block / Thread` 映射到硬件，真实执行单位是 `Warp(32 threads)`。
- `Kernel Fusion` 的核心价值：减少中间结果回写 HBM。
- LLM 常见于 `Memory Bound` 区域，因此 HBM 带宽经常比主频更关键。

---

## 1. 计算单元：SM、CUDA Core、Tensor Core

### 1.1 SM 是什么

在 CPU 里，一个 Core 更偏“低延迟 + 强控制”；在 GPU 里，一个 `SM` 更偏“高吞吐 + 高并行”。

`SM (Streaming Multiprocessor)` 可理解为一个独立车间，主要包含：

- `Warp Scheduler`：调度可运行的 Warp，隐藏访存延迟。
- `CUDA Cores`：执行通用指令。
- `Tensor Cores`：执行矩阵乘加高吞吐路径。
- `Registers / Shared Memory / L1`：就近数据存放与线程协作资源。

### 1.2 CUDA Core vs Tensor Core

- `CUDA Core`：通用 ALU，灵活，适合标量/向量级计算。
- `Tensor Core`：矩阵专用单元，适合 Transformer 中高频的 MatMul。

Tensor Core 的核心操作可抽象为：

`D = A * B + C`（MMA, Matrix Multiply-Accumulate）

### 1.3 “4x4 连线网络”一步到位的实现原理

可以把 Tensor Core 里“4x4 连线网络”理解成一块硬连线的 MAC（乘加）阵列，它直接实现：

`d_ij = c_ij + Σ(a_ik * b_kj)`（`i,j,k` 在小 tile 范围内）

从软件视角看是一条 `mma.sync` 指令，从硬件视角通常是固定流水线：

1. 从寄存器读取 A/B/C 的 fragment，并做 lane 重排（swizzle）。
2. 把 A 的行元素和 B 的列元素分发到固定连线的乘法器阵列。
3. 并行产生乘积并通过加法树/累加链归约。
4. 与 C 做累加（常见是低精度乘法 + FP32 累加）。
5. 把结果 fragment 写回寄存器。

“一步到位”不表示物理上只经历 1 个晶体管动作，而是表示：

- 对程序员来说是 1 条张量指令。
- 对硬件来说走 1 条专用数据通路，而不是通用 ALU 的逐指令循环。

补充：`4x4` 是教学化模型。真实架构里常见的是更大的 warp 级 tile（如 `m16n8k16` 一类），原理一致。

### 1.4 为什么 Tensor Core 更快

- 专用数据通路，减少通用指令调度成本。
- 更高并行度的乘加执行。
- 混合精度常见策略：
1. 低精度乘法（FP16/BF16）换吞吐。
2. 高精度累加（FP32）保稳定。

### 1.5 对比表

| 维度 | CPU Core | GPU SM (CUDA Cores) | Tensor Core |
| --- | --- | --- | --- |
| 设计重心 | 低延迟、复杂控制 | 高吞吐、大并行 | 矩阵吞吐最大化 |
| 擅长任务 | 分支逻辑、串行流程 | 大量相似算术 | MatMul/Conv/Attention 核心路径 |
| 处理粒度 | 标量 | 向量/线程束 | 矩阵块 |

---

## 2. 内存层级与数据流

### 2.1 层级（从慢到快）

1. `HBM`：容量最大、带宽高、延迟相对高（主显存）。
2. `L2 Cache`：片上共享缓存，连接 HBM 与各 SM。
3. `L1 / Shared Memory`：SM 内高速层，适合数据复用。
4. `Registers`：线程私有，访问最快。

### 2.2 为什么会有 Memory Wall

现代 GPU 计算能力增长很快，但数据供给路径（尤其到 HBM）跟不上时，SM 会出现等待。

直观理解：

- 算得再快，数据喂不进来，Tensor Core 也会停工。
- 因此“算力很强但利用率不高”在 LLM 场景很常见。

### 2.3 工程含义

- `OOM`：通常是 HBM 容量耗尽。
- 利用率低：可能是搬运瓶颈、访存模式差、Occupancy 低。
- 利用率波动大：常见于分支发散或 kernel 间同步等待。

---

## 3. CUDA 执行模型与硬件映射

### 3.1 三层映射

| CUDA 概念 | 硬件对应 | 直观类比 |
| --- | --- | --- |
| `Grid` | 整块 GPU | 整个订单 |
| `Block` | 分配到某个 SM | 一支施工小队 |
| `Thread` | 在核心上执行 | 小队中的单个工人 |

### 3.2 两条规则（必须记住）

1. `Block` 是调度基本单位，不能拆到多个 SM。
2. 一个 SM 可同时驻留多个 Block（受寄存器/Shared 等资源约束）。

这直接影响 `Occupancy`：

- Occupancy 高：更容易隐藏延迟。
- Occupancy 低：SM 空转概率更高。

### 3.3 Warp 是真实执行单位

硬件按 `Warp(32 threads)` 发指令（SIMT）。

分支发散示例：

```cpp
if (thread_id < 16) {
  // path A
} else {
  // path B
}
```

同一 Warp 内线程走不同分支时，常见结果是串行化执行，吞吐下降。

### 3.4 编程视角下的内存映射

- `Global Memory` 对应 HBM：所有线程可见，但访问最慢。
- `Shared Memory`：同一 Block 可见，适合块内复用。
- `Registers`：线程私有，速度最快但配额有限。

---

## 4. Kernel Fusion：从“多次搬运”到“就地处理”

### 4.1 为什么要做 Fusion

GPU 常出现“算得快，搬得慢”。
Fusion 的目标是减少中间结果在 HBM 的往返。

### 4.2 示例：`Linear + ReLU`

目标：`Y = ReLU(XW + b)`

无 Fusion（两次 kernel）：

1. 计算 `Z = XW + b`，写回 HBM。
2. 读回 `Z`，执行 ReLU，再写回 HBM。

Fusion（一次 kernel）：

1. 读入 `X/W/b`。
2. 在寄存器或 Shared 中完成 `XW + b` 后立刻做 ReLU。
3. 只把最终 `Y` 写回 HBM。

收益：

- 减少中间张量读写。
- 减少 kernel launch 开销。
- 提高计算强度：

`Arithmetic Intensity = FLOPs / Bytes Moved`

### 4.3 代表案例：FlashAttention

核心思想：通过 `tiling + 重计算`，尽量让中间结果留在片上 SRAM（Shared/L1），减少 HBM 往返。

常见收益：

- 更低显存占用。
- 更高吞吐。
- 更长上下文支持能力。

---

## 5. 为什么 LLM 更看重 HBM 带宽，而不是主频

### 5.1 直观比喻

把 Tensor Core 想象成大胃王，把 HBM 想象成厨房上菜系统：

- `GPU 主频` 类似咀嚼速度（计算速度）。
- `HBM 带宽` 类似上菜速度（供数速度）。

如果上菜慢，咀嚼再快也要等。这个状态就是 `Memory Bound`。

### 5.2 LLM 为什么更容易卡带宽

1. 参数量大：每轮前向/反向都要搬大量权重与激活。
2. 许多场景数据复用不高：常见模式接近“读一次、算一次”。
3. Roofline 视角：大量 LLM 工作点落在带宽受限斜坡区，而非算力受限平顶区。

### 5.3 A100 vs H100 的现实信号

| 指标 | A100 | H100 | 提升倍数 |
| --- | --- | --- | --- |
| FP16 Tensor Core 理论算力 | 312 TFLOPS | 989 TFLOPS | ~3.2x |
| HBM 带宽 | 2.0 TB/s | 3.35 TB/s | ~1.7x |

经验现象：许多大模型任务中，端到端提速更接近带宽提升比例，而不是纯算力提升比例。

### 5.4 工程结论

- 主频决定“理论上限”。
- 带宽决定“喂料能力”。
- 在 LLM 常见路径里，带宽往往更接近真实瓶颈。

一句话：

`算力决定天花板，带宽决定你能否真正接近天花板。`

---

## 6. 性能排查清单（实用版）

### 6.1 现象：显存高、利用率低

优先检查：

1. Batch Size 是否过大导致 HBM 压力高。
2. Block 参数是否让 Occupancy 偏低。
3. 是否有可融合算子却被拆成多个 kernel。

### 6.2 现象：利用率锯齿波动

优先检查：

1. Warp 分支发散（大量 `if/else`）。
2. 访存是否连续（coalesced）与缓存命中情况。
3. kernel launch / 同步等待是否过多。

### 6.3 现象：算力强但加速不明显

优先检查：

1. 工作负载是否本质上 `Memory Bound`。
2. 中间张量是否频繁回写 HBM。
3. 是否启用了合适的精度与融合路径（如 `torch.compile`/Triton/FlashAttention）。

---

## 7. 高频术语速查

- `SM`：Streaming Multiprocessor，GPU 基本执行单元。
- `Warp`：32 线程执行组，硬件执行/调度关键粒度。
- `Occupancy`：SM 驻留线程/Block 的饱和程度。
- `HBM`：High Bandwidth Memory，GPU 主显存。
- `Shared Memory`：Block 内线程共享的片上高速存储。
- `Kernel Fusion`：融合多个算子，减少中间读写与 launch。
- `Memory Bound`：瓶颈在内存带宽/延迟。
- `Compute Bound`：瓶颈在计算单元吞吐。
- `Roofline Model`：用计算强度与峰值带宽/算力分析性能上限。

---

## 8. Day 01 小结与 Day 02 建议

### 8.1 Day 01 小结

- 建立了“计算单元 + 存储层级 + 执行模型”的统一视角。
- 理解了为什么 LLM 常常是带宽瓶颈。
- 明确了 Fusion 在 Infra 优化中的核心地位。

### 8.2 Day 02 建议

1. 用 Nsight Systems 看 kernel 时间线和同步点。
2. 用 Nsight Compute 看访存效率、Occupancy、Tensor Core 利用率。
3. 选一个简单算子做“无 Fusion vs Fusion”实测对比。

---

## 9. 面试 5 分钟答题模板（可直接复述）

### 9.1 开场一句话

`GPU 的设计哲学是吞吐优先。SM 负责并行执行，Tensor Core 负责矩阵加速，HBM 负责持续供数。`

### 9.2 分点回答框架（总-分-总）

1. `SM`：以 Warp 为单位调度，通过 latency hiding 保持流水线满载。
2. `Tensor Core`：用 MMA + 混合精度提升矩阵计算吞吐。
3. `HBM/L2`：很多 LLM 场景是带宽受限，供数速度决定实际性能。
4. `Kernel Fusion`：减少中间结果回写 HBM，把瓶颈从搬运推向计算。

### 9.3 收束结论

`看 GPU 性能不能只看理论 TFLOPS，要结合带宽、占用率、访存效率和融合程度。`

---

## 10. 常见误区与纠正

1. 误区：`GPU 主频越高，训练就一定越快。`
纠正：LLM 常见瓶颈是 `HBM` 带宽，主频只影响计算上限，不保证端到端加速。

2. 误区：`nvidia-smi 利用率高就说明优化很好。`
纠正：还要看 `SM Efficiency`、`Memory BW Utilization`、stall 原因与 kernel 时间线。

3. 误区：`显存没满就不是内存问题。`
纠正：带宽瓶颈与容量瓶颈不同，显存容量没满也可能严重 `Memory Bound`。

4. 误区：`Tensor Core 一开就一定加速。`
纠正：数据布局、精度模式、算子形状、融合路径不匹配时，提速会明显打折。

---

## 11. 自测题（含答案要点）

1. 问：为什么 Warp 分支发散会降速？
答：同一 Warp 线程走不同分支时会串行化执行，降低并行效率。

2. 问：为什么 `Kernel Fusion` 能提速？
答：减少中间张量读写 HBM 和 kernel launch 开销，提高计算强度。

3. 问：如何解释“算力强但加速不明显”？
答：先判断是否 `Memory Bound`，再看访存效率、Occupancy 和融合覆盖率。

4. 问：`HBM` 与 `L2` 的角色差异是什么？
答：HBM 是主显存大仓库，L2 是片上共享中转层，用于减少远距离访存。

5. 问：Tensor Core 的“一步到位”是什么意思？
答：软件层面 1 条 MMA 指令，硬件层面走固定专用乘加流水线，不是通用 ALU 的逐条循环。

---

## 12. 案例分析模板（现象 -> 假设 -> 验证 -> 结论 -> 动作）

### 12.1 模板

1. 现象：记录吞吐、延迟、利用率、显存、带宽等指标异常。
2. 假设：给出 1-3 个最可能瓶颈（带宽、发散、占用率、同步）。
3. 验证：用 Nsight 或 profiler 指标逐条排除。
4. 结论：明确主瓶颈与次瓶颈。
5. 动作：列出可执行优化项和预期收益区间。

### 12.2 Day 01 示例（简化）

1. 现象：显存占用 80%，SM 利用率仅 45%。
2. 假设：中间张量频繁回写 HBM，且 block 配置导致 Occupancy 偏低。
3. 验证：时间线显示多个小 kernel 串行；访存统计显示带宽接近打满。
4. 结论：主瓶颈是 HBM 搬运，次瓶颈是 kernel 粒度过碎。
5. 动作：做 `Linear + Activation` 融合，调整 block 参数并复测。


---

## 13. 面试延展版（5 分钟，可直接复述）

### 13.1 开场一句话

`GPU 的设计哲学是吞吐优先：SM 负责并行执行，Tensor Core 负责矩阵加速，HBM/L2 负责持续供数。`

### 13.2 总-分-总回答框架

1. `SM`：GPU 的基本执行单元，以 Warp（32 线程）为粒度调度，通过 latency hiding 掩盖访存延迟。
2. `Tensor Core`：专用 MMA 路径，常见是 FP16/BF16 乘法 + FP32 累加，在矩阵运算上显著高于通用 CUDA Core。
3. `HBM + L2`：LLM 任务常见 `Memory Bound`，供数速度直接限制端到端吞吐。
4. `Kernel Fusion`：尽量把中间结果留在寄存器/Shared，减少 HBM 往返和 launch 开销。
5. 收束结论：`评估 GPU 性能不能只看 TFLOPS，要联合看带宽、访存效率、Occupancy 与融合覆盖率。`

### 13.3 面试追问加分点

1. 问：为什么 Warp Divergence 会降速？
答：同一 Warp 出现分支分流会串行化执行，等效并行度下降。

2. 问：为什么 `nvidia-smi` 利用率高不等于优化到位？
答：还要看 `SM Efficiency`、`Memory BW Utilization`、stall 原因和时间线分布。

3. 问：为什么 H100 理论算力提升很大，但实测提速常不到同倍数？
答：很多工作负载受 HBM 带宽限制，端到端收益更接近带宽提升比例。

---

## 14. Tensor Core “4x4 硬连线一步到位”补充（教学化模型）

### 14.1 数学目标

`D = A * B + C`  
`d_ij = c_ij + Σ_k(a_ik * b_kj)`

### 14.2 电路视角的基本单元

1. 教学上可把 Tensor Core 看成 `4x4` MAC 阵列（真实硬件通常是更大的 warp 级 tile）。
2. 每个交叉点是一个 `MAC` 单元：输入低精度操作数，执行乘加，并把部分和累加到更高精度寄存器。
3. 本质是“专用数据通路 + 空间并行”，不是复用通用 ALU 串行完成。

### 14.3 “连线网络”为何能快

1. A 的行元素按连线广播到同一行多个 MAC，B 的列元素广播到同一列多个 MAC。
2. 一次读入的元素可被多个 MAC 复用，显著降低单位计算的取数压力。
3. MAC 阵列并行产出部分和，通过固定流水线归约后写回寄存器。

### 14.4 “一步到位”的准确含义

1. 软件层面：体现为一条 `mma.sync/hmma` 指令。
2. 硬件层面：走专用矩阵乘加流水线，而非“取数-乘法-加法-写回”的通用指令循环。
3. 这不等于“物理上只需一个时钟完成全部计算”，而是强调控制开销小、并行度高、数据复用强。

### 14.5 常见误解纠正

1. `4x4` 主要是教学模型，真实 ISA 常见 `m16n8k16` 等 tile。
2. 开启 Tensor Core 不保证总是线性提速；还受数据布局、对齐、精度模式、访存与融合路径影响。
