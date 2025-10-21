### **Kthena: 为云原生时代而生，重新定义大语言模型（LLM）智能推理**

**摘要**：今天，我们激动地向全球开发者和 MLOps 工程师宣布，正式开源 Kthena 项目！Kthena 是一个专为 Kubernetes 设计的、云原生、高性能的 LLM 推理路由和编排、调度系统。它旨在解决在生产环境中大规模部署和服务 LLM 所面临的核心挑战，通过其独特的 KV Cache 感知调度、Prefill/Decode 分离路由等高级功能，显著提升 GPU 资源利用率，降低推理延迟，并赋予企业前所未有的灵活性和控制力。

**项目地址**: [kthena](https://github.com/volcano-sh/kthena)

---

#### **LLM 服务化的“最后一公里”困境**

大语言模型（LLM）正在以前所未有的速度重塑各行各业，但将其高效、经济地部署在生产环境中，特别是基于 Kubernetes 的云原生平台上，仍然困难重重。开发者们普遍面临以下挑战：

1.  **资源利用率低**：LLM 推理，尤其是其独特的 KV Cache 机制，对 GPU 显存的占用是动态且巨大的。传统的负载均衡一般采用Round-Robin算法，无法感知这种负载特性，导致 GPU 资源闲置与请求排队并存，成本高昂。
2.  **延迟与吞吐量难以兼顾**：LLM 推理分为“Prefill”（处理输入提示）和“Decode”（生成 Token）两个阶段，前者是计算密集型，后者是访存密集型。将两者混合调度，常常导致无法针对性优化，影响整体服务的响应速度和吞吐能力。因此PD分离的部署已经成为主流，但如何高效路由和调度，仍是一个难题。
3.  **多租户与多模型管理复杂**：在企业环境中，通常需要同时服务多个业务、多个不同版本或经过 LoRA 微调的模型。如何实现请求的公平调度、优先级管理以及动态路由，是一个复杂的工程难题。
4.  **缺乏原生集成**：许多现有的解决方案要么是外部系统，与 Kubernetes 生态割裂；要么过于复杂，无法满足生产级所需的简单易用性和运维的灵活性。

#### **Kthena：云原生 LLM 推理的智能大脑**

为了攻克上述难题，Kthena 应运而生。它并非要取代现有的 LLM 服务框架（如 vLLM, TensorRT-LLM），而是作为它们上层的智能“交通枢纽”和“调度中心”，深度集成于 Kubernetes 之中。

Kthena 的核心由两大组件构成：

*   **Kthena Router**：一个高性能的数据平面，负责接收所有推理请求，并根据 `ModelRoute` 规则，智能地将请求分发到后端的 `ModelServer`。
*   **Kthena Controller Manager**：Kubernetes 控制平面的控制器，它主要包含多种控制器，主要负责 LLM 工作负载的编排与生命周期管理。它持续调谐并联动多类 CRD（如 `ModelBooster`、`ModelServing`、`AutoScalingPolicy`/`AutoScalingPolicyBinding`、以及 `ModelRoute`/`ModelServer`），将声明式API转化为运行时资源：ModelServing 控制器编排 `ServingGroup` 与 `Prefill/Decode` 角色分组；支持网络拓扑亲和调度和Gang调度、滚动升级与故障恢复；基于 `AutoScalingPolicy` 实现弹性扩缩容。

这种架构使得 Kthena 成为连接用户请求与 LLM 模型的、高度可编程的桥梁。

#### **核心特性与优势**

Kthena 的强大之处在于其专为 LLM 推理场景设计的核心功能：


**1) 生产级推理编排（ModelServing）**

- 三层架构：ModelServing -> ServingGroup -> Roles，一种API统一Prefill/Decode 分离或者非分离的多机分布式并行部署
- Prefill-Decode分离部署：将计算密集型的 Prefill 实例调度到配备高性能计算卡的节点组，而将访存密集型的 Decode 实例调度到配备高带宽显存的节点组，实现资源的最佳匹配和极致的端到端延迟优化。另可以独立伸缩，动态调整角色的比例，更灵活的应对各种复杂的业务场景（如长短句混合、实时推理等。
- 多并行范式：TP/PP/DP/EP 等并行模式灵活配置
- 支持业界主流的推理引擎vLLM、SGLang、Triton/TGI 等
- 拓扑感知 + Gang 调度：Gang调度确保ServingGroup/Role“成组原子化”落地，避免资源浪费；拓扑感知调度通过将Role内的一组Pod调度到网络拓扑更优的节点，提升并行计算的数据传输时延。

**2) 一站式模型上线（ModelBooster）**

- 针对主流的大模型，提供极简的部署模板，自动生成ModelRoute/ModelServer/ModelServing/Autoscaling等路由策略和生命周期管理资源
- 覆盖通用的场景，复杂的编排可通过ModelServing细粒度的定制

**3) 智能、模型感知的路由（Kthena Router）**

- 多模型服务：兼容OpenAI API，根据请求头或内容，将流量分发到不同的基础模型。
- 提供插件化调度：最少请求、最小时延、KV Cache 感知、Prefix Cache 感知、LoRA 亲和、GPU 利用率感知、公平调度等，用户在不同场景下可灵活选择和组合
- LoRA 热插拔无中断：感知推理引擎加载的LoRA 适配器，提供无中断的路由能力
- 丰富流量治理策略：基于权重的模型路由，金丝雀发布、Token级流控、故障转移·
- 独立二进制，不依赖特定的 Envoy Gateway，原生支持PD分离的流量调度，将多层路由合并成一层，易于维护

**4) 成本驱动的自动扩缩容（Autoscaler）**

- 同构伸缩：支持稳定、突发双模式，按业务指标（CPU/GPU/内存/自定义）精准扩缩
- 异构部署优化：在多推理引擎/异构加速器组合中按“成本-能力”贪心分配，最大化性价比

**5) 多引擎与异构硬件**

- 支持多种主流引擎vLLM、SGLang、Triton/TGI 等，统一API抽象、标准化指标
- 支持GPU/NPU 等异构混部，配合异构 Autoscaling 做成本与 SLO 的动态平衡

**6) 网络与公平性内置**

- 公平调度：支持基于优先级和历史Token消耗的的公平调度，既兼顾用户的优先级，对高优先级用户提供更好的服务，又防止低优先级用户“饿死”
- 限流：支持按照用户、模型、token长度进行精细化流量控制

#### **性能**

基于 Kthena Router 的调度插件架构，在长系统提示场景（如 4096 tokens）下，采用“KV Cache 感知 + 最少请求”策略相较随机基线：

- 吞吐可提升约 2.73 倍
- TTFT 降低约 73.5%
- 端到端时延降低超过 60%


| Plugin Configuration         | Runs | Success Rate (%) | Throughput (req/s) | Latency (s) | TTFT (s) |
|:-----------------------------|:----:|:----------------:|:------------------:|:-----------:|:--------:|
| Least Request + KVCacheAware |   3  |       100.0      |      **32.22**     |   **9.22** | **0.57** |
| Least Request + Prefix Cache |   3  |       100.0      |        23.87       |    12.47    |   0.83   |
| Random                       |   3  |       100.0      |        11.81       |    25.23    |   2.15   |


短提示词场景差距会收敛，但在多轮对话、模板化生成、前缀高度相似的业务中，KV Cache 感知策略优势显著。实际收益与模型规模、Prompt长短、硬件紧密相关，但“按需组合、按场景选型”已被验证有效。

#### **未来展望与社区邀请**

Kthena 的发布只是一个开始。我们计划在未来支持更高效的调度算法、更广泛的大模型最佳部署实践，并持续深耕 LLM 推理的大规模部署和性能优化。

“我们相信，AI 的未来构建在开放和协作之上，” Kthena 项目负责人、CNCF TOC 副主席Kevin Wang表示，“Kthena 的目标是为社区提供一个强大、开放、标准的云原生 LLM 服务层，让每一位开发者都能像部署普通微服务一样，轻松、高效地部署和管理复杂的 LLM 应用。我们诚挚地邀请全球的开发者、研究人员和 AI 爱好者加入我们，共同塑造云原生 AI 的未来！”

**立即开始探索 Kthena：**

*   **GitHub 仓库**: [https://github.com/volcano-sh/kthena](https://github.com/volcano-sh/kthena)
*   **文档**: [https://kthena.github.io](https://kthena.github.io)
*   **社区**: [加入我们的 Slack/Discord 频道]

让我们一起，为 LLM 插上云原生的翅膀，释放 AI 的全部潜能！
