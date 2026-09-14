# 1.4 评估驱动开发（Evaluation-driven development）

> **Part 1** · 构建与部署 AI 应用
> **状态** · 已完成
> **最后更新** · 2026-09-14

---

## 1. 概述

评估驱动开发，是指**根据系统实际出错的测量结果来决定下一步做什么**，而不是凭直觉、凭一个聚合指标，或凭哪个问题看起来更有意思就去修哪个。

它重要，是因为 AI 系统不按可预测的方式运行。传统软件可以事先规划，因为你能推断它接下来会做什么。AI 系统做不到：你不知道大语言模型会输出什么，也不知道模型对新样本会给出什么预测。所以构建过程必然是迭代的——做出一小块，看结果，再决定下一步试什么。而这个循环的质量，完全取决于你能多清楚地看见哪里出了问题。

没有这项技能，循环就会退化为猜测。你改了一处，头条数字动了一点，但你无从判断：是修好了一个真实的失败，还是引入了一个新的，还是仅仅把一类错误换成了另一类。进展因此变成随机的，无法累积。

## 2. 图谱中的定位

这是 Part 1 六个子技能之一。吴恩达用全篇最重的措辞单独点了它：

> "In my experience, the most important trait that distinguishes someone great at building AI systems is whether you can drive a disciplined evals/error analysis loop to drive development."
>
> 按我的经验，区分一个人是否真正擅长构建 AI 系统，最重要的特质是他能否驱动一个有纪律的 eval／错误分析循环来推进开发。

他对这项技能的描述包含四层。**作用**是聚焦——它「让你反复把精力投到更可能出成果的方向上」。**难在因项目而异**——正确的做法「在不同项目之间、甚至同一项目的不同阶段都差异很大」。**构建 eval 本身就是一项有深度的技术活**：你要查看系统的 trace 与输出、做探索性数据分析，再结合产品与业务洞察**决定该测什么**。而且你必须掌握**选项菜单**——什么时候用确定性的代码评估、什么时候用 LLM-as-a-judge、什么时候必须人在环中——其中还包括**如何评估你自己的 eval**，让它持续演进。这些评估结果再喂入迭代循环，让进展变得「系统化而不是随机的」。

出处：[AI Engineering Skills Map Part 2，2026-08-21](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications)

## 3. 核心概念

### 3.1 评估的三个对象

最常见的错误是只评估了第一个。

| 对象 | 回答的问题 | 证据来源 |
|---|---|---|
| **输出（output）** | 最终答案对不对？ | 模型的最终产物 |
| **轨迹（trajectory）** | 走过的步骤可接受吗？ | 完整的 trace：工具调用、中间推理、是否循环、是否越权 |
| **结果（outcome）** | 外部状态真的变对了吗？ | 数据库记录、文件、下游系统 |

输出和轨迹都可能在**实际是错的时候看起来是对的**。Agent 完全可以在回复里写「您的订单已确认，编号 CA1234」，而数据库里根本没有这条记录。正因如此，**三者之中结果最可信**——状态不会误报自己。

反向的错误同样有害：把 grader 写死成「必须按标准流程走」，会判错一个正确但出乎意料的解法。Anthropic 报告过一个案例：在 τ²-Bench 的航空任务中，Agent 发现了退改签政策的漏洞、帮用户省了钱，却被判为失败。评估应该问的是**目标有没有达成**，而不是 Agent 有没有走你预设的那条路。

### 3.2 三类 grader

术语沿用 Anthropic 的 Agent 评估指南，它区分基于代码、基于模型、人工三类。

| 类型 | 优势 | 弱点 | 什么时候用 |
|---|---|---|---|
| **基于代码（code-based）** | 快（毫秒级）、客观、可复现、免费 | 不理解语义上的细微差别 | 凡是能写成断言的：结构校验、数值容差、单元测试、工具调用检查、延迟与 token 统计 |
| **基于模型（LLM-as-a-judge）** | 理解语义、可扩展、能处理开放式任务 | 非确定性、按次付费、必须校准 | 主观质量、按评分细则打分、没有唯一正确答案的开放式产出 |
| **人工** | 专家判断；金标准 | 慢、贵、不可扩展 | 校准前两类 grader；高风险场景 |

实践上，好的评估套件是三者组合：少量高质量的人工标注充当标尺，LLM 裁判对照标尺完成校准，所有能量化的部分下推给代码 grader。

一个具体的坑：代码 grader 写得太严会产生**假失败**。用 `expected == actual` 判等时，模型返回 `96.12` 而标准答案是 `96.124991`，精度其实远远够用，却被判失败。修法是改成容差比较。

### 3.3 能力评估与回归评估

两类评估套件，目的不同，不能按同一种方式解读。

| 类型 | 回答的问题 | 用途 |
|---|---|---|
| **能力评估（capability eval）** | 能不能做到现在做不到的难任务？ | 爬山——衡量能力上限 |
| **回归评估（regression eval）** | 以前能做对的有没有被搞坏？ | 守住营地——防止无声退化 |

### 3.4 什么时候该投入

吴恩达指出正确做法取决于项目阶段，而误判阶段是常见的失败。大致形状：

| 阶段 | 工作状态 | 合适的 eval 投入 |
|---|---|---|
| **探索期** | 「要做什么」还在变 | 轻。读数据、建立直觉，用电子表格而不是基础设施 |
| **收敛期** | 方案基本定了，在比较不同实现 | 重。建立可复现的评估集，接入 CI |
| **生产期** | 有真实用户 | eval 成为基础设施，与线上监控打通 |

## 4. 深入原理

### 4.1 为什么单看聚合指标无法指导行动

一个准确率或 F1 数字告诉你出错的比例，但不告诉你**错误的结构**。模型在 95% 时，约有 5% 的样本是错的——而工程上的应对完全取决于这 5% 的分布：

- **集中在某一类** → 修这一类的特征或预处理。
- **分散且看起来随机** → 先怀疑标注，再怀疑模型。
- **集中在某个子系统或某个数据源** → 去看融合或整合策略。

在逐个看失败样本之前，这三条路径是无法区分的。更麻烦的是，聚合指标会**掩盖相互抵消的变化**：放宽某个阈值，可能让一类错误变好、另一类变差，而总数纹丝不动——这实际上是一次交易，却被呈现为「什么都没变」。

### 4.2 LLM-as-a-judge 已知的四种偏差

以下来自对这套方法的一次系统性研究：

> Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, NeurIPS 2023 — https://arxiv.org/abs/2306.05685

| 偏差 | 表现 | 缓解手段 |
|---|---|---|
| **位置偏差（position bias）** | 偏好先看到的那个回答 | 交换顺序各跑一次，只有结论一致时才接受判定 |
| **冗长偏差（verbosity bias）** | 偏好更长、更华丽的回答，哪怕冗余 | 在评分细则里约束长度，或做长度归一 |
| **自我偏好偏差（self-enhancement bias）** | 偏向同族模型生成的输出 | 换一个不同族的模型当裁判 |
| **推理能力不足** | 自己解不出的题就判不好（数学、逻辑） | 用思维链提示；提供参考答案 |

同一篇论文区分了三种评分模式——**成对比较（pairwise comparison）**（两个回答哪个更好）、**单回答评分**（对照评分细则给一个回答打分）、**有参考答案的评分（reference-guided grading）**（提供标准答案辅助判分）。它还报告称，像 GPT-4 这样的强裁判与人类专家的一致率约为 **80–85%**，与两个人类评审彼此之间的一致率相当。

### 4.3 一致率高，不等于 grader 有用

80–85% 这个数字常被拿来当作有效性证明。它不是。一个**永远返回 PASS** 的 grader 同样会显示出与人类标签极高的一致率，但它什么都没抓出来。一致率衡量的是 grader 在平均意义上是否与标签吻合，而**不衡量它是否具备区分能力**。要判断一个 grader 有没有用，得看它抓到了什么、漏掉了什么——假阴性和假阳性——而不只是那个一致率。

这就是吴恩达所说的「如何评估你的 eval」的具体含义。

### 4.4 起点是错误分析，不是评估集

目前可操作性最强的公开方法论来自 Hamel Husain 与 Shreya Shankar，提炼自大量生产环境实现。**顺序很关键，而且经常被颠倒：**

1. **收集真实 trace。** trace 是管线一次交互的完整记录：初始输入、每一次 LLM 的输入与输出、中间的推理与工具调用，以及最终面向用户的结果。
2. **人工读至少约 100 条多样化的 trace。** 10–15 条远远不够。目标是达到**理论饱和（theoretical saturation）**——再往下读也不会冒出没见过的失败模式。
3. **开放式编码（open coding）。** 写下简短的描述性笔记，记录哪里出了问题。这一步要刻意**不**去诊断根因，也不要把每个字放大看。每条 trace 只记一个主要的失败模式。在这里做根因分析是无底洞，而且会拖慢这一遍的节奏、并让你更倾向于注意某些东西而不是另一些。
4. **轴心编码（axial coding）。** 把开放式编码按主题聚类，得到一棵**失败分类学（failure taxonomy）**。
5. **按频次排序。** 修最高频的失败，而不是最好修的那个。
6. **只对已经稳定下来的失败模式写 evaluator。** 定义清晰、可量化的先交给代码 grader；只有无法量化的才上 LLM 裁判。
7. **校准并持续演进。** 拿每个 evaluator 与人工标签比对，并且预期评估标准本身会随项目推进而变化。

这套方法明确否定的做法包括：还没看数据就先去找「最好的 eval 工具或平台」；购买不懂你所在领域的通用现成指标；把标注外包出去——这恰好丢掉了整个过程本应建立的产品直觉；以及在还没读够数据、不知道要测什么之前就写下一大套 eval。

出处：[A Field Guide to Rapidly Improving AI Products](https://hamel.dev/blog/posts/field-guide/)、[Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/)、[AI Evals 笔记汇总](https://hamel.dev/notes/llm/evals/)。

### 4.5 从分类学到 evaluator

每一个高频失败模式对应一个 evaluator。类型的选择按 §3.2 展开：

| 失败的形态 | grader 类型 |
|---|---|
| 结构化输出格式错、类型错、超出取值范围 | 基于代码（结构校验） |
| 数值结果超出容差 | 基于代码（数值比较） |
| 该调的工具有没调，或参数传错 | 基于代码（轨迹检查） |
| 输出违反了某条明示的规则 | 基于模型（走评分细则，给出是/否判定） |
| 语气、有用程度、对来源的忠实度 | 基于模型 |
| 模糊、高风险或有争议的案例 | 人工 |

### 4.6 把 eval 接进 CI

评估集一旦稳定下来，就应当在每次改动时自动跑一遍。通常采用测试金字塔的形状：廉价的代码类 eval 每次提交都跑；昂贵的 LLM 裁判与人工评审只在关键节点跑。DeepLearning.AI 的 *Automated Testing for LLMOps* 课程讲的就是搭这条流水线。

## 5. 能力检查点

1. 我能区分输出、轨迹、结果三种评估对象，并说明为什么结果是最可信的证据，以及为什么只查轨迹的 grader 会判错一个正确的解法。
2. 我能说出三类 grader，并为给定任务挑出一套组合，逐项说明选择理由。
3. 我能说出 LLM-as-a-judge 有据可查的四种偏差，并给出各自的标准缓解手段。
4. 我能解释为什么一个与人类一致率达 90% 的裁判仍可能毫无用处。
5. 我能按正确顺序描述错误分析的流程，并解释为什么根因分析被刻意推迟到开放式编码之后。
6. 我能把一棵失败分类学逐条指派到某个 grader 类型，并说明理由。
7. 我能针对处于特定阶段的项目，判断此时建一套重型评估体系是不是合适的精力投向。
8. 我能解释为什么单凭一个聚合的准确率数字，不足以决定下一步的工程动作。

## 6. 来源

| 类型 | 来源 | 链接 |
|---|---|---|
| 原文 | 吴恩达，*AI Engineering Skills Map Part 2*，2026-08-21 | [链接](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications) |
| 原文 | 吴恩达，*The AI Engineering Skills Map*（总览），2026-08-14 | [链接](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map) |
| 一手 · 论文 | Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, NeurIPS 2023 | [arxiv.org/abs/2306.05685](https://arxiv.org/abs/2306.05685) |
| 实践者 | Hamel Husain, *A Field Guide to Rapidly Improving AI Products* | [hamel.dev](https://hamel.dev/blog/posts/field-guide/) |
| 实践者 | Hamel Husain, *Your AI Product Needs Evals* | [hamel.dev](https://hamel.dev/blog/posts/evals/) |
| 实践者 | Hamel Husain, AI Evals 笔记汇总 | [hamel.dev](https://hamel.dev/notes/llm/evals/) |
| 实践者 | Hamel Husain & Shreya Shankar, *AI Evals for Engineers and PMs*（课程） | [maven.com](https://maven.com/parlance-labs/evals) |
| 一手 · 官方 | Anthropic, *Building effective agents*, 2024-12 | [github.com/anthropics/anthropic-cookbook](https://github.com/anthropics/anthropic-cookbook) |
| 一手 · 官方 | OpenAI, *A Practical Guide to Building Agents*, 2025-04 | [PDF](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) |
| 课程 | DeepLearning.AI, *Automated Testing for LLMOps* | [链接](https://read.deeplearning.ai/courses/automated-testing-llmops) |
