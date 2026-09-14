# Glossary 术语表

Bilingual terminology for this repository. **Authoritative** — if a manual uses a term that is not here, the glossary must be updated in the same change.

本仓库的中英术语对照。**具有权威性**——若某份手册用到表中没有的术语，必须在同一次改动中补入本表。

---

## How to use this table 使用规则

1. **First mention** in a manual uses the Chinese term with the English original in parentheses, formatted `中文（English）`. Thereafter the Chinese term alone.
2. **Untranslatable terms** are marked `保留英文` (keep English). Do not invent a Chinese translation for these. When such a term first appears in Chinese prose, gloss it once, e.g. `grader（评分器）`, then use the English form.
3. Where a Chinese translation is common but imprecise, the note column says why the preferred form was chosen.

---

## General 通用

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| AI Engineering | AI 工程 | 中文 |
| AI Engineer | AI 工程师 | 中文 |
| skill map | 技能图谱 | 中文 |
| sub-skill | 子技能 | 中文 |
| continuous learning | 持续学习 | 中文 |
| best practice | 最佳实践 | 中文 |
| trade-off | 取舍 | 中文 |
| pipeline | 管线 | 中文。不要译作「管道」，本仓库统一用「管线」 |
| workflow | 工作流 | 中文 |
| prototype | 原型 | 中文 |
| in production | 生产环境 / 线上 | 中文。指真实用户场景时用「生产环境」 |

---

## Part 1 — Building and deploying AI applications 构建与部署 AI 应用

### Evaluation 评估

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| eval / evals | 评估 | **保留英文**。中文圈「评估」可指任何评价行为，无法承载这里特指的工程化评测体系。首次出现写 `eval（评估）`，其后用 eval |
| evaluation-driven development | 评估驱动开发 | 中文 |
| error analysis | 错误分析 | 中文 |
| eval set | 评估集 | 中文 |
| grader | 评分器 | **保留英文**。首次出现写 `grader（评分器）`。译作「评分器」在句中使用别扭，且 grader 已在实际工作中通用 |
| code-based grader | — | 见 grader |
| model-based grader | — | 见 grader |
| human grader | — | 见 grader |
| LLM-as-a-judge | — | **保留英文**。「大模型评审」等译法尚未形成共识，保留英文 |
| rubric | 评分细则 | 中文。指分项打分标准时用「评分细则」，不译作「量规」 |
| pairwise comparison | 成对比较 | 中文 |
| reference-guided grading | 有参考答案的评分 | 中文 |
| ground truth | 标准答案 | 中文。在标注语境下用「标准答案」，不译作「真值」（那是测量学语境） |
| trace | — | **保留英文**。指管线一次交互的完整记录。译作「轨迹」会与 trajectory 冲突 |
| trajectory | 轨迹 | 中文。指 Agent 走过的步骤序列 |
| output | 输出 | 中文 |
| outcome | 结果 | 中文。特指外部状态的改变 |
| transcript | 交互记录 | 中文。指人可读的对话记录，与机器可读的 trace 区分 |
| failure taxonomy | 失败分类学 | 中文 |
| open coding | 开放式编码 | 中文。首次出现可括注英文 |
| axial coding | 轴心编码 | 中文。首次出现可括注英文 |
| theoretical saturation | 理论饱和 | 中文 |
| capability eval | 能力评估 | 中文 |
| regression eval | 回归评估 | 中文 |
| position bias | 位置偏差 | 中文 |
| verbosity bias | 冗长偏差 | 中文。不译作「长度偏差」，偏好的实质是冗长而非长度本身 |
| self-enhancement bias | 自我偏好偏差 | 中文 |
| red teaming | 红队测试 | 中文。首次出现可括注英文 |
| observability | 可观测性 | 中文 |
| regression testing | 回归测试 | 中文 |
| drift | 漂移 | 中文。指数据分布或模型表现随时间偏移 |
| adversarial input | 对抗性输入 | 中文 |
| prompt injection | 提示注入 | 中文 |
| guardrail | 护栏 | 中文 |
| human in the loop | 人在环中 | 中文 |

### LLM and agentic systems LLM 与 Agentic 系统

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| LLM (large language model) | 大语言模型 | 中文。首次出现给全称，其后用 LLM |
| context window | 上下文窗口 | 中文 |
| context engineering | 上下文工程 | 中文 |
| RAG (retrieval-augmented generation) | 检索增强生成 | 中文。首次出现给全称，其后用 RAG |
| grounding | 锚定 | 中文。指用数据为模型提供可靠上下文 |
| vector index | 向量索引 | 中文 |
| knowledge graph | 知识图谱 | 中文 |
| semantic layer | 语义层 | 中文 |
| agentic system | Agentic 系统 | 混合 |
| agent harness | — | **保留英文**。「框架」会与 framework 混淆；harness 特指包裹 LLM 的那层执行外壳 |
| agent loop | Agent 循环 | 混合 |
| tool calling | 工具调用 | 中文 |
| memory architecture | 记忆架构 | 中文 |
| multi-agent orchestration | 多智能体编排 | 中文 |
| MCP (Model Context Protocol) | 模型上下文协议 | 中文。首次出现给全称，其后用 MCP |
| fine-tuning | 微调 | 中文 |
| self-hosting | 自托管 | 中文 |
| distillation | 蒸馏 | 中文 |
| knowledge cutoff | 知识截止时间 | 中文 |
| reasoning effort | 推理强度 | 中文 |
| sampling parameter | 采样参数 | 中文 |
| cache hit | 缓存命中 | 中文 |
| prompt caching | 提示缓存 | 中文 |
| KV cache | — | **保留英文**。指注意力机制中缓存的键值状态 |
| prefix | 前缀 | 中文 |
| tokenization | 分词 | 中文 |
| tokenizer | 分词器 | 中文 |
| BPE (byte-pair encoding) | — | **保留英文**。首次出现写 BPE（字节对编码），其后用 BPE |
| autoregressive | 自回归 | 中文 |
| logits | — | **保留英文**。指 softmax 之前的原始分数 |
| attention | 注意力 | 中文 |
| transformer | Transformer | 保留原文。专有架构名，不译 |
| temperature | 温度 | 中文 |
| top-p | — | **保留英文**。即核采样；参数名不译 |
| nucleus sampling | 核采样 | 中文 |
| top-k | — | **保留英文**。参数名不译 |
| greedy decoding | 贪心解码 | 中文 |
| hallucination | 幻觉 | 中文 |
| calibration | 校准 | 中文 |
| scaling law | 缩放定律 | 中文 |
| retrieval | 检索 | 中文 |
| latency | 延迟 | 中文 |
| throughput | 吞吐量 | 中文 |
| multimodal | 多模态 | 中文 |
| data exfiltration | 数据外泄 | 中文 |
| generative UI | 生成式 UI | 混合 |

### Retrieval and grounding 检索与锚定

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| grounding | 锚定 | 中文。指用数据为模型提供可靠的输入上下文 |
| sparse retrieval | 稀疏检索 | 中文 |
| dense retrieval | 密集检索 | 中文 |
| hybrid retrieval | 混合检索 | 中文 |
| BM25 | — | **保留英文**。算法名，不译 |
| embedding | 嵌入 | 中文 |
| embedding model | 嵌入模型 | 中文 |
| chunk / chunking | 切块 | 中文。统一用「切块」，不用「分块」；动词与名词同形 |
| rerank / reranking | 重排 | 中文 |
| query rewrite | 查询改写 | 中文 |
| context compression | 上下文压缩 | 中文 |
| OCR | — | **保留英文**。缩略已通用 |
| hit rate | 命中率 | 中文 |
| MRR (mean reciprocal rank) | 平均倒数排名 | 中文。首次出现给全称，其后用 MRR |
| recall@K | 召回率@K | 混合 |
| faithfulness | 忠实度 | 中文。指生成内容是否基于检索材料 |
| context precision / recall | 上下文精确度 / 上下文召回率 | 中文 |
| freshness | 新鲜度 | 中文 |

### Agent architecture Agent 架构

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| prompt chaining | 提示链 | 中文 |
| routing | 路由 | 中文 |
| parallelization | 并行化 | 中文 |
| sectioning | 分段式 | 中文 |
| voting | 投票式 | 中文 |
| orchestrator-workers | 编排者-工作者 | 中文 |
| evaluator-optimizer | 评估者-优化者 | 中文 |
| sandbox | 沙箱 | 中文 |
| CLI | — | **保留英文**。缩略已通用 |
| fallback | 回退 | 中文 |
| termination condition | 终止条件 | 中文 |
| context isolation | 上下文隔离 | 中文 |
| step repetition | 步骤重复 | 中文 |
| reasoning-action mismatch | 推理与行动不一致 | 中文 |
| task derailment | 任务脱轨 | 中文 |
| least privilege | 最小权限 | 中文 |
| indirect prompt injection | 间接提示注入 | 中文 |
| privilege escalation | 越权 | 中文 |
| audit trail | 审计留痕 | 中文 |
| best-of-N sampling | — | **保留英文**。参数化命名，不译 |
| ACI (agent-computer interface) | — | **保留英文**。与 HCI 类比而生，中文无对应 |
| tool design | 工具设计 | 中文 |

### Machine learning 机器学习

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| supervised learning | 监督学习 | 中文 |
| reinforcement learning | 强化学习 | 中文 |
| deep learning | 深度学习 | 中文 |
| bias / variance | 偏差 / 方差 | 中文 |
| data engineering | 数据工程 | 中文 |

---

## Part 2 — Software engineering fundamentals 软件工程基础

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| full-stack | 全栈 | 中文 |
| front-end / back-end | 前端 / 后端 | 中文 |
| caching | 缓存 | 中文 |
| page rendering | 页面渲染 | 中文 |
| state management | 状态管理 | 中文 |
| session management | 会话管理 | 中文 |
| asynchronous processing | 异步处理 | 中文 |
| data persistence | 数据持久化 | 中文 |
| accessibility | 无障碍 | 中文 |
| data model | 数据模型 | 中文 |
| relational / document / key-value / graph store | 关系型 / 文档型 / 键值 / 图存储 | 中文 |
| transaction | 事务 | 中文 |
| concurrency | 并发 | 中文 |
| data lifecycle | 数据生命周期 | 中文 |
| monolith / microservices | 单体 / 微服务 | 中文 |
| system decomposition | 系统拆分 | 中文 |
| rate limit | 速率限制 | 中文 |
| graceful degradation | 优雅降级 | 中文 |
| blast radius | — | **保留英文**。指故障影响范围的边界，中文无简洁对应说法 |
| shift left | 安全左移 | 中文。首次出现可括注英文，说明其含义是在生命周期中提前 |
| supply chain injection | 供应链注入 | 中文 |
| SDLC (software development lifecycle) | 软件开发生命周期 | 中文。首次出现给全称，其后用 SDLC |
| CI/CD | 持续集成 / 持续交付 | 中文。首次出现给全称，其后用 CI/CD |
| IaaS (infrastructure as a service) | 基础设施即服务 | 中文。首次出现给全称，其后用 IaaS |
| load balancing | 负载均衡 | 中文 |
| sharding | 分片 | 中文 |
| indexing | 索引 | 中文 |
| replication | 复制 | 中文 |
| version control | 版本控制 | 中文 |
| code review | 代码审查 | 中文 |
| dependency maintenance | 依赖维护 | 中文 |
| technical debt | 技术债 | 中文 |
| incident management | 事故管理 | 中文 |

---

## Part 3 — Using coding agents 使用编程 Agent

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| coding agent | 编程 Agent | 混合 |
| agentic coding | Agentic 编程 | 混合 |
| spec | — | **保留英文**。「规格说明」在工程语境中已特指别的文档类型，且 spec 在本主题中高频出现，保留英文更精确 |
| greenfield | 全新项目 | 中文。指从零构建 |
| brownfield | 存量项目 | 中文。指已有代码库上改造 |
| agent autonomy | Agent 自主性 | 混合 |
| subagent | 子 Agent | 混合 |
| orchestrator | 编排者 | 中文 |
| hook | 钩子 | 中文。指在固定时点自动触发的脚本 |
| plugin | 插件 | 中文 |
| skill | 技能 | 中文 |
| standing context | 常驻上下文 | 中文。指 AGENTS.md、CLAUDE.md 一类长期常驻的说明文件 |
| AGENTS.md / CLAUDE.md | — | **保留原文**。文件名不翻译 |
| verification loop | 验证回路 | 中文 |
| overengineering | 过度设计 | 中文 |
| retrospective | 复盘 | 中文 |
| context window management | 上下文窗口管理 | 中文 |
| token | token | **保留英文** |

---

## Part 4 — Shaping the build 塑造构建

| English | 中文 | Usage 本仓库用法 |
|---|---|---|
| shaping the build | 塑造构建 | 中文 |
| build loop | 构建循环 | 中文 |
| MVP (minimum viable product) | 最小可行产品 | 中文。首次出现给全称，其后用 MVP |
| product sense | 产品嗅觉 | 中文 |
| design sense | 设计审美 | 中文 |
| business sense | 商业意识 | 中文 |
| user empathy | 用户同理心 | 中文 |
| go-to-market | — | **保留英文**。「进入市场」在句中使用生硬，且缩写 GTM 已通用 |
| market size | 市场规模 | 中文 |
| unit economics | 单位经济性 | 中文 |
| P&L (profit and loss) | 损益 | 中文。首次出现给全称，其后用 P&L |
| A/B test | A/B 测试 | 中文 |
| stakeholder | 利益相关方 | 中文 |
| agency | 主动性 | 中文。首次出现可括注英文；本仓库不译作「主体性」 |
| high-agency ownership | 高主动性担当 | 中文 |
| PM (product manager) | 产品经理 | 中文 |

---

## Terms deliberately kept in English 不译清单

汇总 §2 中标注为「保留英文」的术语。这些词在中文技术写作中被直接使用，强行翻译反而降低准确性：

| Term | Why 保留原因 |
|---|---|
| `eval` / `evals` | 中文「评估」无法承载其作为工程化评测体系的特指含义 |
| `grader` | 与 `evaluator`、`scorer` 需区分；中文译名未成共识 |
| `LLM-as-a-judge` | 直译冗长，业内直接用英文 |
| `trace` | 避免与 trajectory（轨迹）的中文译名冲突 |
| `agent harness` | 避免与 framework（框架）混淆 |
| `KV cache` | 中文无通用对应；作为技术术语直接用英文 |
| `logits` | 中文无简洁对应 |
| `top-p` / `top-k` | 参数名，不译 |
| `BPE` | 缩略已通用；仅在首次出现时括注「字节对编码」 |
| `Transformer` | 专有架构名 |
| `BM25` | 算法名，不译 |
| `OCR` | 缩略已通用 |
| `CLI` | 缩略已通用 |
| `best-of-N sampling` | 参数化命名，不译 |
| `ACI` | 由与 HCI 类比而生，中文无对应 |
| `blast radius` | 中文无简洁对应 |
| `spec` | 高频词，「规格说明」在工程语境另有特指 |
| `go-to-market` | 缩略 GTM 已通用 |
| `token` | 中文技术语境直接用英文 |
| `AGENTS.md` / `CLAUDE.md` | 文件名 |
