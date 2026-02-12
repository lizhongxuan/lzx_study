# AI Infra 技术栈

### ☁️ 云原生与容器编排 (Cloud Native & Orchestration)
* **Kubernetes Core**: 精通 Kubernetes 核心架构，深入理解 Informer、Controller 及 Reflector 机制；具备 Operator/CRD 开发经验，能够基于 Kubebuilder/Controller-runtime 编写自定义控制器。
* **Container Runtime**: 熟悉 CRI/CNI/CSI 标准接口，深入理解 Docker/Containerd 容器运行时原理，能够排查 Pod 启动、网络通信及资源隔离等底层问题。
* **Delivery**: 熟练使用 Helm 进行应用封装，了解 GitOps (ArgoCD) 流程。
* **Networking Base**: 熟悉高性能网络插件（CNI）配置，了解 Multus CNI 及 SR-IOV/RDMA 在 AI 集群中的应用。

### 🧠 AI 基础设施与异构调度 (AI Infra & Heterogeneous Scheduling)
* **Device Management**: 深入掌握 Kubernetes Device Plugin 机制，能够配置和调试 NVIDIA GPU 在 K8s 中的发现、注册与分配流程。
* **Advanced Scheduling**: 熟悉 AI 批处理调度器 Volcano / Kueue，理解 Gang Scheduling (帮派调度)、Bin-packing (装箱) 及 Topology Aware (拓扑感知) 调度算法，能解决分布式训练中的资源死锁与碎片化问题。
* **Resource Efficiency**: 了解 GPU 资源隔离与共享技术，熟悉 MIG (Multi-Instance GPU) 配置及 MPS 原理，致力于提升集群 GPU 资源利用率。
* **AI Workloads**: 熟悉 PyTorch (DDP/FSDP) / DeepSpeed 等分布式训练框架在 K8s 上的运行模式；熟悉 vLLM / Triton 推理引擎的部署与调优；了解基于 KubeRay 的分布式计算集群管理。

### 💾 高性能存储与数据加速 (AI Storage & Acceleration)
* **Architecture**: 精通 JuiceFS + MinIO 的分层存储架构设计。能够利用 JuiceFS CSI Driver 为 AI 训练/推理任务提供高性能的 POSIX 文件系统接口。
* **Optimization**: 擅长解决 AI 场景下的 IO 瓶颈问题，能够利用 JuiceFS 的 Metadata Caching 和 Data Warm-up (预热) 机制，显著加速大模型权重加载（Model Loading）及海量小文件数据集读取。
* **Storage-Compute Separation**: 熟悉对象存储协议（S3），具备构建存算分离架构的实战经验，有效管理 Checkpoint 及海量日志与模型版本。

### 🚀 高性能网络与通信 (High-Performance Networking & NCCL)
* **Network Fabric**: 了解高性能网络基础，熟悉 RDMA / RoCE v2 在 K8s 环境下的配置（如 Multus CNI, SR-IOV）。
* **NCCL**: 熟悉 NCCL 通信原理，能够利用 `nccl-tests` 压测集群通信带宽，排查分布式训练中的通信瓶颈与网络抖动问题。

### 🔬 可观测性与开发语言 (Observability & Languages)
* **Golang**: 核心开发语言，精通 GMP 调度模型、Channel 并发模式及内存管理，具备高并发系统开发经验。
* **Python**: 熟悉 Python 脚本编写，能够使用 PyTorch 编写基础的模型加载与测试脚本，协作算法工程师进行环境排错。
* **Monitoring**: 精通 Prometheus + Grafana + DCGM-Exporter，建立涵盖 GPU 利用率、显存、ECC 错误及 NVLink 带宽的完整监控大盘；配置基于自定义指标的 HPA 自动扩缩容。

### 🛠 分布式训练框架的“运维视角”
* 熟悉主流分布式训练框架（PyTorch DDP/FSDP, DeepSpeed, Megatron-LM）的启动流程与资源需求。
* 具备在 K8s 上部署大规模分布式训练任务（Multi-Node Multi-GPU）的实战经验，能够排查任务挂起（Hang）及训练中断问题。
