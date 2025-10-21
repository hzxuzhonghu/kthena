# Kthena 演示文稿

---

## Slide 1: 标题页

# Kthena

### 为云原生时代而生，重新定义大语言模型（LLM）智能推理

**[您的姓名/团队]**
**[日期]**

---

## Slide 2: LLM 服务化的“最后一公里”困境

- **资源利用率低**：传统调度器无法感知 KV Cache，导致 GPU 资源浪费。
- **延迟与吞吐量难以兼顾**：Prefill (计算密集) vs Decode (访存密集) 混合调度效率低下。
- **多租户/多模型管理复杂**：公平性、优先级和动态路由难以实现。
- **缺乏原生集成**：现有方案与 Kubernetes 生态割裂或过于复杂。

---

## Slide 3: Kthena - 云原生 LLM 推理的智能大脑

**Kthena 是什么?**
一个专为 Kubernetes 设计的、云原生、高性能的 LLM 推理路由和调度系统。

**定位**
作为现有推理框架 (vLLM, TGI) 上层的**智能“交通枢纽”和“调度中心”**。

**核心组件**
- **Kthena Router**: 高性能数据平面，智能分发请求。
- **Kthena Controller Manager**: 控制平面，将声明式 API 转化为运行时资源。

---

## Slide 4: 核心特性 1: 生产级推理编排 (ModelServing)

- **三层架构 (ModelServing -> ServingGroup -> Roles)**：统一 Prefill/Decode 分离与非分离部署。
- **Prefill/Decode 分离部署**：资源最佳匹配，独立伸缩，应对复杂场景。
- **多并行范式**：灵活配置 TP/PP/DP/EP。
- **拓扑感知 + Gang 调度**：保证原子化落地，优化网络时延。

---

## Slide 5: 核心特性 2: 一站式模型上线 (ModelBooster)

- **极简部署模板**：针对主流大模型，一键生成所需全部资源。
  - `ModelRoute`
  - `ModelServer`
  - `ModelServing`
  - `Autoscaling`
- **覆盖通用场景**，同时允许通过 `ModelServing` 进行细粒度定制。

---

## Slide 6: 核心特性 3: 智能、模型感知的路由 (Kthena Router)

- **插件化调度算法**:
  - KV Cache 感知 / Prefix Cache 感知
  - LoRA 亲和 / GPU 利用率感知
  - 最少请求 / 最小时延 / 公平调度
- **LoRA 热插拔无中断**：感知并路由到已加载的 LoRA 适配器。
- **丰富流量治理**：权重路由、金丝雀发布、Token 级流控、故障转移。
- **独立二进制**：不依赖 Envoy，原生支持 PD 分离，简化架构。

---

## Slide 7: 核心特性 4: 成本驱动的自动扩缩容 (Autoscaler)

- **同构伸缩**：支持稳定、突发双模式，按业务指标（CPU/GPU/内存/自定义）精准扩缩。
- **异构部署优化**：在多推理引擎或异构加速器（GPU/NPU）组合中，按“成本-能力”贪心分配，最大化性价比，实现成本与 SLO 的动态平衡。

---

## Slide 8: 核心特性 5: 内置公平性与网络策略

- **公平调度**:
  - 基于**优先级**和**历史 Token 消耗**。
  - 兼顾高优先级用户服务质量，同时防止低优先级用户“饿死”。
- **精细化限流**:
  - 支持按**用户**、**模型**、**Token 长度**进行流量控制。

---

## Slide 9: 性能表现：KV Cache 感知调度的威力

**场景**: 长系统提示 (4096 tokens)
**对比**: “KV Cache 感知 + 最少请求” vs. 随机基线

- **吞吐量提升 ~2.73 倍**
- **首 Token 平均时延 (TTFT) 降低 ~73.5%**
- **端到端时延降低 > 60%**

| Plugin Configuration | Throughput (req/s) | Latency (s) | TTFT (s) |
|---|---|---|---|
| **Least Request + KVCacheAware** | **32.22** | **9.22** | **0.57** |
| Least Request + Prefix Cache | 23.87 | 12.47 | 0.83 |
| Random | 11.81 | 25.23 | 2.15 |

---

## Slide 10: 未来展望与社区

- **未来计划**:
  - 支持更高效的调度算法。
  - 提供更广泛的大模型最佳部署实践。
  - 持续深耕大规模部署和性能优化。

- **社区邀请**:
  > “Kthena 的目标是为社区提供一个强大、开放、标准的云原生 LLM 服务层...我们诚挚地邀请全球的开发者、研究人员和 AI 爱好者加入我们，共同塑造云原生 AI 的未来！”
  > — Kevin Wang, Kthena 项目负责人, CNCF TOC 副主席

---

## Slide 11: 加入我们

**立即开始探索 Kthena！**

- **GitHub**: [github.com/volcano-sh/kthena](https://github.com/volcano-sh/kthena)
- **文档**: [kthena.github.io](https://kthena.github.io)
- **社区**: Slack / Discord

## Q&A

