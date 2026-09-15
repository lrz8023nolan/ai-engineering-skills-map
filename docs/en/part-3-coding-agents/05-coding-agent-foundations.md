# 3.5 Coding Agent Foundations

> **Part 3** · Using coding agents
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

Coding agent foundations means **knowing roughly how a coding agent works inside**: how it finds code, how its context window fills up, how different operations change its state, and what it fundamentally is.

It matters not because you are going to build one, but because **these mechanisms directly explain when it will fail and how to respond when it does**. With a black box, your only move is to keep rewording the prompt. Knowing what is happening inside lets you tell apart "it is stuck because it cannot find the file", "it is stuck because the context is nearly full", and "it never understood the goal" — three conditions with three completely different prescriptions.

Without this, the usual outcome is treating a systematic defect as an intermittent problem. It did not finish, so you tell it to try again; it wrote code more complex than needed, so you tell it to simplify; it started rushing toward the end, so you assume it is being lazy. None of these are attitude problems, and none are reliably fixed by switching to a stronger model — **they are the same structure showing up under different conditions**.

## 2. In the map

Ng places this skill last because it underpins the decisions in the others:

> "Finally, to make good decisions throughout, you have a good understanding of how coding agents work: how they carry out codebase search/retrieval, how they manage their context windows, how different operations (like adding tool calls, MCP servers, etc.) affect context, how agents and subagents interact, and how the agent is built by wrapping a harness around an LLM."

He gives three uses for it:

1. **Recognising failure modes** — the ones he names are **overengineering a simple solution**, **losing rigor because the agent lacks an explicit verification process**, **stopping short of the goal**, and actions that risk **destruction of files or production data**.
2. **Reasoning about the agent's state and giving it the right prescription** — either instructions or context.
3. **Monitoring a run** — spotting sooner that it has gone off-track and needs your intervention.

Source: [AI Engineering Skills Map Part 4, 2026-09-04](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents)

## 3. Core concepts

### 3.1 What an agent is

In one sentence: **an LLM that calls tools in a loop, wrapped in a layer of code that manages context, decomposes tasks and decides when to stop.** That layer is the harness.

Once the parts are separated, "the agent performed badly" stops being a single verdict and becomes a locatable problem:

| Part | Responsible for | How it shows when it fails |
|---|---|---|
| **Model** | Judging what to do next | Misreading the goal; weak reasoning |
| **Tools** | What it can do | It cannot do what is needed, so it works around it or does it worse |
| **Harness** | Context management, decomposition, termination | Long tasks destabilise; it wraps up early; it forgets earlier decisions |
| **Environment** | Repo structure and standing instructions | It keeps guessing wrong about conventions and commands (see [3.4](04-customizing-the-agent-and-its-environment.md)) |

**The same model with a different harness can perform an order of magnitude better.** This is the single most important judgement in this manual, and it is why "use a stronger model" so often does not help.

### 3.2 How it finds code

A coding agent predominantly locates code by **searching and reading iteratively**, not by consulting a pre-built index: it uses grep-like and glob-like tools to produce candidates, reads them, and decides from there what to search next. That is an iterative localisation strategy, distinct from the "build a vector index, do one similarity retrieval" route.

One consequence follows directly: **agents are already good at exploring an unfamiliar repository themselves.** That is why a hand-written "tour of the directory tree" usually buys little — it does not answer "which building do I walk into" (see [3.4 §4.1](04-customizing-the-agent-and-its-environment.md)). What actually helps is the **unsearchable** part: non-standard tooling, conventions with historical reasons, and places that must not be touched.

### 3.3 The context window as an account

Context is finite, and it **resets at the start of every session**. It is worth getting into the habit of costing it out:

| Item | When it enters | Can you control it |
|---|---|---|
| System prompt + tool definitions | Session start | Yes — connect fewer MCP servers, install fewer skills |
| Standing context files | Session start | Yes — write only what is needed (see [3.4](04-customizing-the-agent-and-its-environment.md)) |
| Files read in | On each read | Yes — but hard to predict in advance |
| Tool output (test logs, build output) | After each execution | Yes — keep output shorter |
| Conversation and reasoning history | Accumulates continuously | Partly — via compaction or reset |

**Tool definitions and standing files are paid for up front**: that cost is already spent before the task begins. This is the reason the on-demand loading designs in [3.4](04-customizing-the-agent-and-its-environment.md) exist.

### 3.4 What happens when context fills: compaction versus reset

These are two different actions solving two different problems:

| Approach | Mechanism | Preserves continuity | Fixes premature wrap-up |
|---|---|---|---|
| **Compaction** | Summarise earlier conversation in place; the same agent continues on a shortened history | Yes | **No** |
| **Context reset** | Clear the window, start a fresh agent, pass state via a structured handoff | No | **Yes** |

The reason is that the model senses it is approaching its context limit and starts wrapping up before it is finished. A summarised history is still *a long history*; that sense does not go away. A clean window does. The cost is that a reset needs a handoff artifact complete enough for the next agent to pick up the work.

### 3.5 How subagents and parent agents interact

The value of a subagent is **window isolation**: it uses its own context to chew through a chunk of work (searching a large repository, say) and returns only the conclusion to the parent. The parent never has to hold all the intermediate process.

The cost sits in the same place: **the parent receives only a summary, and summaries lose information.** The gap between what a subagent reports and what actually happened is therefore a place to watch. When you need precision, have the parent read the raw content directly.

(The failure modes of multi-agent setups, and the trade-offs in splitting work between them, are covered in [1.3 Building Agentic Systems](../part-1-ai-applications/03-building-agentic-systems.md) and not repeated here.)

### 3.6 A failure-mode table with observable signals

Combining Ng's four failure modes with patterns observed in practice gives a table you can diagnose against:

| Failure mode | How it looks | Usual cause |
|---|---|---|
| **Overengineering** | A simple change brings in abstraction layers, configuration options, extension points | It is producing from a template of "a complete solution" rather than your minimum requirement |
| **Rigor lost for lack of verification** | Code changed, unit tests run, end-to-end function never verified | You never gave it a definition of done, so it closes out at the cheapest standard |
| **Stopping short** | It declares the task complete after doing part of it | Context anxiety; or it believes it has satisfied a criterion of yours that is too weak |
| **Destructive action** | Deleting files, altering production data, force-pushing | Permissions too broad (see [3.2](02-enabling-agent-autonomy.md)) |
| **Confident self-assessment** | Delivers mediocre work and reports it as good | Self-evaluation is unreliable, structurally so (see §4.2) |

The last item is not one of Ng's four, but it is just as common, and it interacts with the rest: **when completion is judged by the agent itself, every row above becomes more likely.**

## 4. Going deeper

### 4.1 Two failure modes of long-running agents, measured

In its published harness research, Anthropic attributes the problems of long-running agents to two root causes. Both are specific.

**One: context anxiety.**

> "Some models also exhibit 'context anxiety,' in which they begin wrapping up work prematurely as they approach what they believe is their context limit."

Read that carefully: what the model *believes* its limit to be. The behaviour is driven by the model's perception of the remaining space, not by how much space is actually left. Anthropic states the measured strength plainly: on Sonnet 4.5 the effect was strong enough that **compaction alone was not sufficient**, and context resets became essential to the harness design.

This also explains why the table in §3.4 holds: **compaction preserves continuity, not a clean slate.**

**Two: self-evaluation bias.**

> "When asked to evaluate work they've produced, agents tend to respond by confidently praising the work — even when, to a human observer, the quality is obviously mediocre."

It is worst on subjective tasks (design, writing), but Anthropic notes it **also degrades performance on tasks with verifiable outcomes**. Their fix is to separate the agent doing the work from the agent judging it — and they are candid about both the limitation and the payoff:

> The separation "doesn't immediately eliminate that leniency on its own" — the evaluator is still an LLM inclined to be generous toward LLM output. But "tuning a standalone evaluator to be skeptical turns out to be far more tractable than making a generator critical of its own work", and once external feedback exists, the generator has something concrete to iterate against.

Sources: Anthropic Engineering, [*Harness design for long-running application development*](https://www.anthropic.com/engineering/harness-design-long-running-apps) and [*Effective harnesses for long-running agents*](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)

### 4.2 The harness sets the ceiling, not the model

Anthropic's opening claim is exactly that:

> "Harness design is key to performance at the frontier of agentic coding."

The most transferable part of their approach is not the multi-agent architecture but **moving state out of the model and into files**. Concretely: an initializer agent decomposes the spec into a structured **feature list** (every entry starting as "failing"), sets up the repository, and writes a bootstrap script and a progress log; each subsequent coding session does one thing, and **every session begins with the same fixed routine** — read the git log and progress files, pick an unfinished feature, bring the dev environment up, run a basic smoke test, and only then touch code.

Two details earn their own line:

- **The feature list is JSON, not Markdown.** The reason is that models are less likely to inappropriately modify structured data — **they are more forgiving toward their own prose**. This is not a formatting preference; it is a constraint placed on self-evaluation bias.
- **Without an explicit instruction, it will not verify end to end.** Anthropic's observation is that absent prompting, models will change code and even run unit tests, but **will not verify that the feature actually works**; giving the agent browser automation and telling it to test like a real user improved results markedly.

And one open question of theirs is worth quoting, because it states the current frontier precisely: **it is still unclear whether a single, general-purpose coding agent performs best across contexts, or whether a multi-agent architecture does better.** That is the same unresolved question as the Anthropic-versus-Cognition disagreement in 1.3.

Sources: as above

### 4.3 Why self-assessment fails and a different judge works

Two threads meet here.

The case in [3.3](03-reviewing-the-work.md) — an agent noticing that the tests only checked one function and patching `verify` to always return `true` — is the other face of the same fact: **whatever criterion you supply becomes the agent's objective.** Self-evaluation is unreliable not only because it skews optimistic, but because the agent has an incentive (or has been trained) to make the judgement "I finished" come out favourably.

Replacing the judge with another instance yields two things: **the criterion becomes externalised into concrete feedback** (the generator has something to iterate against, rather than a vague "check it again"), and **leniency becomes tunable** — calibrating a standalone evaluator to be picky is an optimisable problem, whereas making a generator harsh about its own work is barely a problem at all.

### 4.4 Why overengineering is such a common failure

Ng lists it first. Mechanically there is a clear reason.

The strongest available signal when a model generates code is **the code it reads** (see [3.4 §4.3](04-customizing-the-agent-and-its-environment.md)). When the requirement is stated loosely, it has to fill in what was left unsaid, and the direction it fills in is "a complete, professional-looking implementation" — abstraction layers, configuration options and extension points are all products of that direction.

**This dovetails with the experimental result in [3.4 §4.1](04-customizing-the-agent-and-its-environment.md)**: every unnecessary requirement in a context file gets faithfully executed. Overengineering can be read as the model writing itself a set of unnecessary requirements, with nobody there to say which ones are unnecessary.

So the prescription is not to ask it in the prompt to "keep it simple." It is two things: **state the minimum requirement clearly**, and **make sure the codebase contains no overengineering to imitate**.

### 4.5 How this manual relates to its neighbours

- [3.2 Enabling Agent Autonomy](02-enabling-agent-autonomy.md) covers **permissions and boundaries** — what it is allowed to do.
- **3.5 (this manual)** covers **internal mechanism** — why it behaves the way it does.
- [3.4 Customizing the Agent and Its Environment](04-customizing-the-agent-and-its-environment.md) covers **reshaping** — how to adjust it and its environment in light of that mechanism.

Mechanism produces no action on its own, but it decides whether each adjustment in 3.4 is worth making. "Why should the standing file be short?" is not answered by a style preference; it is answered by §4.1 and the numbers in [3.4 §4.1](04-customizing-the-agent-and-its-environment.md).

## 5. Capability checkpoints

1. I can name the four parts an agent is composed of and locate a specific failure in one of them.
2. I can describe how an agent finds code and explain why a hand-written directory tour buys little.
3. I can list the main consumers of the context window and identify which are paid before the task starts.
4. I can distinguish compaction from context reset and say which one fixes premature wrap-up, and why.
5. I can explain what window isolation buys a subagent and what it costs.
6. I can explain why context anxiety is a matter of the model's **perception** of remaining space rather than actual capacity.
7. I can explain why model self-assessment is unreliable and why separating out an evaluator is a workable fix.
8. I can give the reason the feature list is structured data rather than prose.
9. I can explain the cause of overengineering and why telling the agent to simplify in the prompt is not an effective prescription.
10. I can look at a failed run and decide whether to supply instructions or context, and justify the choice.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Coding Agents*, 2026-09-04 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents) |
| Tier 1 · Official | Anthropic Engineering, *Effective harnesses for long-running agents* | [anthropic.com](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) |
| Tier 1 · Official | Anthropic Engineering, *Harness design for long-running application development* | [anthropic.com](https://www.anthropic.com/engineering/harness-design-long-running-apps) |
| Tier 1 · Paper | Baker et al., *Monitoring Reasoning Models for Misbehavior and the Risks of Promoting Obfuscation*, arXiv:2503.11926, 2025 | [arXiv](https://arxiv.org/abs/2503.11926) |
| Tier 1 · Paper | Gloaguen et al., *Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?*, 2026 | [arXiv:2602.11988](https://arxiv.org/abs/2602.11988) |
