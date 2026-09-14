# 1.3 Building Agentic Systems

> **Part 1** · Building and deploying AI applications
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Building agentic systems means extending "a single model call" into a system that has **steps, tools, state, and the ability to decide its own next move**.

The problem it solves: many tasks cannot be done in one call. You need to look something up, judge the result, take an action, then verify the action took effect — and where each step goes depends on what the previous one produced. A single question-and-answer exchange cannot do this.

Without the skill, failure comes in two opposite shapes. One is **staying at single-turn question answering**, unable to do anything that requires fetching, judging or acting. The other is more common and more costly: **reaching for multi-agent architectures first**. The pull is strong and the demos look good, but measurement says multi-agent systems deliver far less on standard benchmarks than expected while failing 41%–86.7% of the time (see §4.3). You pay several times the tokens and latency for a worse, harder-to-debug result.

Worth noting how Ng orders his own description: architecture choice comes first — what to chain, what to parallelise, when to use code instead of a model — and agent loop design comes after. **The order is the point: work out the flow before deciding how much autonomy to hand over.**

## 2. In the map

This is the third of the six sub-skills of Part 1. Ng's definition places it on a spectrum:

> "Agentic systems range from workflows that execute a predefined sequence of LLM calls to ones based on an agent harness that lets an LLM repeatedly decide its own next step."

The judgements he asks for in this section:

- **Architecture choice**: what steps to chain, what to parallelise, when to use code and when to use an LLM
- **Engineering** the workflow or harness, **with fallbacks**
- **Designing the agent loop**: what tools the model can call (including MCP, CLI and sandbox execution environments), what memory architecture to use, **how to manage context over long sessions**, and when the task needs multi-agent orchestration rather than a single agent
- **Turning prototypes into production**: guardrails, adversarial inputs, identifying and working around key risks such as data exfiltration, and governance
- **Cutting-edge forms**: voice agents, computer-use agents, generative UI

Source: [AI Engineering Skills Map Part 2, 2026-08-21](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications)

## 3. Core concepts

### 3.1 The two ends of the spectrum: workflow and agent

The key distinction is not "with or without a model" but **who decides the execution path**.

| | Workflow | Agent |
|---|---|---|
| **Execution path** | Predefined in code | Decided by the model at runtime |
| **Predictability** | High; steps are fixed | Low; the path is not fixed |
| **Fits** | Clear tasks that can be decomposed in advance | Open-ended problems where the number of steps is unknown |
| **Cost** | Lower latency and cost | Higher latency and cost, traded for flexibility |

Anthropic's engineering guidance is to **start with the simplest approach and add complexity only where it demonstrably helps.** Many tasks do not need an agent at all — a single model call with retrieval is enough.

### 3.2 Five composable workflow patterns

This is the most practically useful menu, drawn from Anthropic's synthesis of dozens of teams' practice:

| Pattern | Approach | Fits |
|---|---|---|
| **Prompt chaining** | Split the task into fixed sequential steps, each feeding the next, with programmatic checkpoints between them | Tasks that decompose cleanly in sequence — outline, review, then draft |
| **Routing** | Classify the input first, then dispatch to a specialised path | Inputs that fall into clear categories needing different handling; also used to route simple requests to a small model |
| **Parallelization** | Sectioning (independent subtasks run in parallel, then combined) or voting (the same task run several times, taking consensus) | Multiple independent dimensions to assess; or where robustness matters |
| **Orchestrator-workers** | A central model decomposes the task at runtime, dispatches to workers, and synthesises results | Subtasks that cannot be known in advance, such as a coding task touching an unknown number of files |
| **Evaluator-optimizer** | One model generates, another critiques against a standard, looping until the bar is met | Where clear criteria exist and iteration genuinely helps, such as translation refinement |

**Orchestrator-workers differs from parallelization** in that the set of subtasks is decided at runtime rather than fixed in advance. That difference drives both cost and determinism.

### 3.3 Criteria for choosing an architecture

| Question | Points toward |
|---|---|
| Can the steps be listed in advance? | Yes → workflow. No → agent |
| Must the path change based on intermediate results? | Yes → move toward the agent end |
| Is the cost of an error high? | Yes → keep a human in the loop or programmatic checkpoints |
| Is the latency budget tight? | Tight → favour a workflow with fewer round trips |
| Does the task decompose cleanly into independent subtasks? | Yes → parallelise. No → do not (see §4.4) |
| Is there a clear success criterion? | Yes → an evaluator-optimizer loop becomes available |

### 3.4 Code or model

A practical division of labour:

- **Use code** for the deterministic parts — data transformation, schema validation, arithmetic, conditional branching, permission checks.
- **Use the model** for the parts needing judgement — interpreting vague intent, deciding the next step, generating natural language, handling exceptions.

The most common mistake is handing the model work that belongs in code: making it do arithmetic, decide permissions, or reformat data. Code is faster, cheaper and more reliable at all three.

### 3.5 Three execution environments for tools

| Type | Purpose | Watch for |
|---|---|---|
| **MCP (Model Context Protocol)** | Standardised access to external tools and data sources | The authorisation scope and data boundary of each tool |
| **CLI** | Letting the model invoke command-line tools | Command injection; restrict the executable set |
| **Sandbox** | Letting the model write and run code | Isolation and resource limits are mandatory — otherwise this is arbitrary code execution |

**Tool design is itself a skill.** Anthropic's experience is that time spent refining tools — parameter names, descriptions, examples, edge cases — pays back more than time spent tuning prompts. The standard to aim for is the care you would put into designing a human interface.

### 3.6 Memory and long-session context

Once an agent loop runs, the context grows continuously until it hits the window limit. Three problems need handling:

| Problem | Common approach |
|---|---|
| Context too long | Summarise, externalise large intermediate artefacts to files or storage, keep only key conclusions |
| Lost across sessions | Persist state and read it back when a new session begins |
| Inconsistent state | Be explicit about what is a known fact and what still needs verification |

**One boundary worth memorising: permission and state decisions should not live in the model.** A model can be *told* about a permission, but being told is not the same as being bound. Any switch that decides whether an action is currently allowed must live in code, read explicitly by the control loop.

### 3.7 Single agent or multi-agent

The default answer should be **single agent**. Multi-agent only pays off under specific conditions — see §4.2 for the criteria.

### 3.8 Four things that make it production-grade

| Area | What it covers |
|---|---|
| **Guardrails** | Input and output filtering, permission boundaries, human approval for high-risk actions |
| **Adversarial input** | Indirect prompt injection (§4.8), malicious tool return values, privilege-escalation attempts |
| **Key risks** | Data exfiltration, deletion of production data, cost blowouts from unbounded loops |
| **Governance** | Audit trails, traceable decision chains, clear accountability |

## 4. Going deeper

### 4.1 "Start simple" is not a slogan — it is measured

Anthropic's conclusion is that most tasks do not need an agent; an augmented LLM (model plus retrieval, tools and memory) suffices. Complexity should be added along the workflow → agent spectrum only once the simple approach demonstrably falls short.

That advice is backed by measurement. Cemri et al. ran a systematic study on today's most capable multi-agent frameworks:

> Cemri et al., *Why Do Multi-Agent LLM Systems Fail?*, [arXiv:2503.13657](https://arxiv.org/abs/2503.13657), NeurIPS 2025

Across seven popular open-source multi-agent frameworks they measured a **41%–86.7% failure rate**, and state plainly: "Despite enthusiasm for Multi-Agent LLM Systems (MAS), their performance gains on popular benchmarks are often minimal" — relative both to single-agent frameworks and to simple baselines such as best-of-N sampling.

In other words, **more agents does not mean more capability.** Complexity has to earn its place with evidence.

### 4.2 When multi-agent actually works

Multi-agent is not useless, and its effective settings have clear characteristics. Anthropic reports that its multi-agent research system beats single-agent Claude Opus 4 by **90.2%** on research tasks — at roughly **15× the token cost** of ordinary chat. The more interesting number: **token usage alone explained about 80% of the performance variance**. Much of the gain comes from simply doing more, not from dividing labour more cleverly.

The boundary matters:

| Works | Does not work |
|---|---|
| Subtasks that are **genuinely independent** and can run in parallel (parallel retrieval, reviewing different dimensions at once) | Subtasks that are **tightly coupled** and need to share substantial intermediate state (such as coding) |
| Context that can be isolated without losing information | Where one agent's output must line up precisely with another's assumptions |

Anthropic says it outright: the pattern is a **poor fit for tightly coupled work like coding**.

### 4.3 Why multi-agent fails: MAST's three categories, fourteen modes

This is the best-evidenced classification of multi-agent failure available. MAST (Multi-Agent System Failure Taxonomy) was built from 1,642 annotated execution traces across seven frameworks, with inter-annotator agreement at Cohen's κ = 0.88. It identifies **14 failure modes in three categories**:

| Category | Share | Principal modes |
|---|---|---|
| **1. Specification and system design** | ~44.2% | **Step repetition (15.7% alone — the single most frequent failure)**, disobeying the task specification (11.8%), disobeying role specification, loss of conversation history, unaware of termination conditions |
| **2. Inter-agent misalignment** | ~32.3% | Conversation resets, withholding information, task derailment, ignoring another agent's output, reasoning-action mismatch (13.2%) |
| **3. Verification and termination** | ~23.5% | Premature termination, no verification, incorrect verification |

The implication of that table is worth pausing on. **The largest category, at 44.2%, is not a runtime problem at all — it was baked in at design time.** The agent did not break; it faithfully executed a flawed setup. That is why swapping in a stronger model usually does not help — the model was never the problem.

The second category, by contrast, cannot occur in a single-agent system at all. It is a cost of the multi-agent structure itself.

### 4.4 Why "one more agent" often does not help

Cognition offered a mechanistic explanation in a widely discussed post:

> Walden Yan, *Don't Build Multi-Agents*, Cognition, 2025-06

The core argument: **actions carry implicit decisions, and conflicting decisions carry bad results.** When each agent holds only partial context, the implicit decisions it makes conflict with the others' in ways no individual agent can see.

The canonical example is asking a system to build a Flappy Bird clone: one subagent takes the background, another takes the bird. The first misreads the brief and produces a Super Mario Bros. background. The second produces a bird that neither moves like Flappy Bird nor matches the art it is flying over. **Neither subagent did anything wrong given what it was actually told.**

Mitigating this requires sharing **full agent execution traces**, not just the messages passed between agents. That is an architectural commitment, not a configuration change.

### 4.5 The highest-return single intervention: add a verification step

In the same MAST study, the authors ran direct intervention experiments with very concrete results:

| Intervention | Change in task success |
|---|---|
| Adding a high-level verification step (on ChatDev) | **+15.6 percentage points** |
| Tightening role specifications | +9.4 percentage points |

The verification step is the most contained, most measurable and largest-effect change available, and it does not belong to the first category of failures — it is an architectural component you can simply add. If you are prioritising work on a multi-agent system, this goes first.

### 4.6 Failure attribution is genuinely hard

An easily overlooked reality: even when you know the system failed, **automatically working out which agent failed and at which step is currently not something we can do well.**

Zhang et al. benchmarked automated failure attribution across 127 multi-agent systems (*Which Agent Causes Task Failures and When?*, [arXiv:2505.00212](https://arxiv.org/abs/2505.00212), ICML 2025). The best method identified the **responsible agent** with 53.5% accuracy and the **responsible step** with only 14.2% — while frontier reasoning models fell below the automated baseline on step attribution.

The reason is that failures are usually **cascades**: an early specification ambiguity surfaces ten steps later as a verification failure, and the trace does not mark the causal link. That is why "read the logs and find the cause" is especially inefficient on multi-agent systems.

### 4.7 Context management over long sessions

As turns accumulate, context grows monotonically until it hits the window limit — and even before that, lost-in-the-middle degrades the use of distant information. Three approaches:

1. **Summarise** — compress earlier turns into conclusions, discarding raw detail.
2. **Externalise** — write large intermediate artefacts to files or storage and fetch them when needed, rather than keeping them resident in context.
3. **Replay** — persist key state so a session can resume from an interruption instead of restarting.

The caution: **summarisation loses information, and it loses it in uncontrolled ways.** In the MAST taxonomy, "loss of conversation history" is itself a distinct failure mode. So summarisation should be conservative, and critical constraints — task specifications, role boundaries — should be resident content rather than part of what gets compressed.

> Related earlier work: Packer et al., *MemGPT: Towards LLMs as Operating Systems*, 2023 (cited at second hand; original not consulted)

### 4.8 Security: indirect prompt injection and data exfiltration

Once a system can read external content — web pages, documents, email, tool return values — that content becomes an **instruction channel**.

> Greshake et al., *Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection*, 2023 (cited at second hand; original not consulted)

The mechanism: an attacker hides instructions inside content the model will read, and the model cannot reliably separate "data I received" from "instructions I received". Combined with tool permissions, a successful injection can lead to data exfiltration — sending sensitive content to a location the attacker controls.

Three defensive principles: **least privilege** (strictly limit what each tool can do), **separate data from instructions** (mark external content explicitly as data, not instruction), and **put human gates in front of irreversible actions**. Note that this class of attack is not solved; genuinely high-risk settings have to be backstopped by permission design rather than by hoping the prompt holds.

### 4.9 Cutting-edge forms

The three directions Ng names — voice agents, computer-use agents, generative UI — share a property: the model's output is no longer just text but **enters an environment requiring real-time interaction and state synchronisation**. They amplify every problem above — context management, tool permissions, verification, failure attribution — because the environment's feedback is faster, the state is more complex, and the cost of error is more immediate.

## 5. Capability checkpoints

1. I can explain the fundamental difference between a workflow and an agent in terms of who decides the execution path, and place a given task on that spectrum.
2. I can describe the mechanism and conditions of each of the five workflow patterns, and identify what distinguishes orchestrator-workers from parallelization.
3. I can select an architecture for a given task using explicit criteria, and explain why "more autonomous" does not mean "better".
4. I can divide a task between code and model, and justify each division.
5. I can state the risk profile of each of the three tool execution environments — MCP, CLI and sandbox.
6. I can restate the conditions under which multi-agent works and fails, with the data that supports the judgement.
7. I can name the three MAST failure categories with their approximate shares, and explain why the largest category cannot be fixed by changing models.
8. I can explain why adding an agent often does not help, and what kind of architectural change mitigates it.
9. I can describe three approaches to long-session context management and name the risk of summarisation.
10. I can explain the mechanism of indirect prompt injection and give at least three defensive principles.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 2*, 2026-08-21 | [link](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications) |
| Tier 1 — paper | Cemri et al., *Why Do Multi-Agent LLM Systems Fail?* (MAST), NeurIPS 2025 | [arXiv:2503.13657](https://arxiv.org/abs/2503.13657) |
| Tier 1 — paper | Zhang et al., *Which Agent Causes Task Failures and When?*, ICML 2025 | [arXiv:2505.00212](https://arxiv.org/abs/2505.00212) |
| Tier 1 — paper | Yao et al., *ReAct: Synergizing Reasoning and Acting in Language Models*, ICLR 2023 | [arXiv:2210.03629](https://arxiv.org/abs/2210.03629) |
| Tier 1 — paper | Shinn et al., *Reflexion: Language Agents with Verbal Reinforcement Learning*, NeurIPS 2023 | [arXiv:2303.11366](https://arxiv.org/abs/2303.11366) |
| Tier 1 — paper | Schick et al., *Toolformer: Language Models Can Teach Themselves to Use Tools*, NeurIPS 2023 | [arXiv:2302.04761](https://arxiv.org/abs/2302.04761) |
| Tier 1 — official | Anthropic, *Building effective agents*, 2024-12 | [anthropic.com](https://www.anthropic.com/engineering/building-effective-agents) |
| Tier 1 — official | OpenAI, *A Practical Guide to Building Agents*, 2025-04 | [PDF](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) |
| Tier 1 — official | Anthropic, engineering write-up on its multi-agent research system, 2025-06 | [anthropic.com](https://www.anthropic.com/engineering/multi-agent-research-system) |
| Tier 2 — practitioner | Walden Yan (Cognition), *Don't Build Multi-Agents*, 2025-06 | [cognition.ai](https://cognition.ai/blog/dont-build-multi-agents) |
| Tier 2 | Packer et al., *MemGPT: Towards LLMs as Operating Systems*, 2023 (original not consulted) | — |
| Tier 2 | Greshake et al., *Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection*, 2023 (original not consulted) | — |
