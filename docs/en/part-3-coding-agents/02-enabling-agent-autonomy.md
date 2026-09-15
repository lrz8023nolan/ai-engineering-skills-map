# 3.2 Enabling Agent Autonomy

> **Part 3** · Using coding agents
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

Enabling agent autonomy means **deciding how much rope to give the agent — how long it runs unattended, what it can touch, where it can connect** — and how to make sure that, having given it that rope, it cannot do irreparable damage.

It matters because autonomy is where the main benefit and the main risk both come from, and both are governed by the same switches. An agent that can run for an hour, notice its own failures and retry is what actually saves you time. The same capability means that if it misjudges the direction, it will **keep going confidently in the wrong direction for an hour**.

Without this skill, two failures recur. One is permissions set too loosely — to stop clicking "approve," every gate comes down, and this is the one agent that happened to read an untrusted page or dependency. The other is context left unmanaged — the agent works three rounds before anyone notices that an assumption from round two changed, and it could not have known, because the change was never written down.

## 2. In the map

Ng breaks the skill into four actions, the first of which is the trade-off itself:

> "When applying a coding agent to the steps in the workflow, you choose the autonomy level: Do you watch it and go back-and-forth interactively or delegate a larger chunk of work to it? And when do you set a clear goal and have it loop until it succeeds?"

The other three actions are **managing context** (as the build proceeds through phases, calibrating when to capture key learnings, user feedback, and assumptions — *including assumptions that changed partway through the build* — so the agent can use them downstream); **orchestrating parallel agents** (deciding when to run many agents on a decomposition of the task, whether a human or a higher-level agent orchestrates them, and how to manage your own attention across concurrent sessions); and **running agents safely** (setting permissions and gating actions so development stays fast while limiting the risk of leaks, data loss, or other damage).

Source: [AI Engineering Skills Map Part 4, 2026-09-04](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents)

## 3. Core concepts

### 3.1 The three levels of autonomy

Ng's three levels can be separated by asking **who decides whether to keep going**:

| Level | Shape | Who decides the next step | Fits |
|---|---|---|---|
| **Interactive** | Every step confirmed back and forth | Human | Direction still unsettled, blast radius unknown, code you do not know |
| **Bounded delegation** | A defined chunk of work, reviewed at the end | Human, at chunk boundaries | Clear goal, clear boundaries, verifiable at the end of the chunk |
| **Goal + loop until success** | A goal and a check, iterating until it passes | **The agent**, using the check you gave it | The check is objective and machine-executable |

The third level has the highest payoff, and its precondition is most often glossed over: **"loop until it succeeds" requires that the agent can tell whether it succeeded.** If that check does not exist, or is wrong, the agent will optimise precisely toward the wrong endpoint — see §4.4.

### 3.2 Choosing how much autonomy to give

| Signal | Which way to move |
|---|---|
| Objective, machine-executable check exists | Up (autonomous) |
| Only a human can judge whether it is right | Down (interactive) |
| The action is reversible (editing code) | Up |
| The action is irreversible (migrating data, publishing, deleting) | Down, and add a gate |
| Highly repetitive, conventions well established | Up |
| Requires understanding existing conventions and historical baggage | Down |

### 3.3 Context management: three things that must be captured

Ng names three: **key learnings, user feedback, and assumptions that changed partway through.** The third is the one most often lost, because it produces no error. You asked the user, the user said to change direction, you replied "ok, switching to X," and carried on. For an agent running in a different session, that never happened.

The practical test is simple: **any information that affects the implementation does not exist unless it sits somewhere the agent will read.** So it goes where the agent always reads — not into the chat log.

### 3.4 Parallel agents and the human attention budget

Ng mentions both ways of orchestrating: **a human orchestrating** (starting sessions by hand and distributing work) and **a higher-level agent orchestrating** (one agent dispatching others).

A constraint tends to get overlooked here: **the ceiling on parallelism is not compute, it is how much you can review.** Three parallel agents produce three sets of changes; you can still only read one carefully at a time. If output keeps outrunning review, the process will degrade into "merge it and see" — and at that point parallelism has produced unreviewed code, not throughput.

### 3.5 Three layers of safe operation

| Layer | Mechanism | Strength |
|---|---|---|
| **Model layer** | System prompts, classifiers, training | Influences behaviour; cannot guarantee it |
| **Approval layer** | A confirmation prompt on every action with side effects | Depends on the human reading each one |
| **Environment layer** | Sandbox (filesystem isolation + network isolation), scoped permissions, egress allowlists | Does not depend on any judgement |

The environment layer is the only one that does not rely on someone judging correctly. It does not try to make the agent smarter; it **makes it incapable of much even when fooled**.

### 3.6 The lethal trifecta

Simon Willison proposed a very usable test in June 2025: the [**lethal trifecta**](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/). When an agent has all three of the following, one successful prompt injection becomes data theft:

| The three legs | Meaning |
|---|---|
| **Access to private data** | API keys, customer records, an inbox, internal documents |
| **Exposure to untrusted content** | A web page, an issue, a third-party dependency, a retrieved chunk |
| **Ability to externally communicate** | An HTTP call, an email, a webhook, even an image URL with data in the query string |

Any two are survivable. All three together are not. **The fix is not to teach the model to refuse, it is to remove a leg** — most commonly the third, by taking away the egress tool.

## 4. Going deeper

### 4.1 Approval fatigue: why "ask every time" stops working

Anthropic measured a number on its own product. Claude Code asks for per-action approval before writing files, running commands or reaching the network — and **users approved roughly 93% of those prompts**.

That 93% is what disables the approval layer. When only 7% are refused, people stop reading the prompt. Anthropic named the phenomenon **approval fatigue**: the prompt stops being a safety control and becomes pure friction — "friction with no protective value."

Their fix was not to reduce what the agent could do wrong. It was to **move where the boundary lives**: from "ask a human every time" to "let the human set it once, then have the operating system enforce it." Concretely, two isolation boundaries built on OS primitives (macOS Seatbelt, Linux bubblewrap) — **filesystem isolation** (read/write confined to the working directory) and **network isolation** (outbound traffic only through a proxy outside the sandbox, which applies a domain allowlist).

The result: **an 84% reduction in permission prompts**, with **no additional authority granted to the agent**.

One detail matters: Anthropic states plainly that **effective sandboxing requires both filesystem and network isolation.** Lock files but not the network, and a compromised agent can still send out what it read; lock the network but not the filesystem, and it can escape the sandbox and regain network access.

Source: [Anthropic Engineering, *Beyond permission prompts: making Claude Code more secure and autonomous*](https://www.anthropic.com/engineering/claude-code-sandboxing)

### 4.2 Red-team result: judgement fails, environment holds

Anthropic ran a controlled red-team exercise: a phishing attack led an employee to give Claude Code a plausible-looking instruction to retrieve AWS credentials and send them to an external destination.

**It carried out the exfiltration in 24 of 25 attempts.**

The significance is not that the model is malicious. It is that **the request was authorised-looking when it was executed** — it came from the user, was well-formed, and read normally. Any mechanism that tries to infer "is this malicious?" from the request itself fails on this class of case. Therefore:

> Controls cannot rest on recognising intent. They must rest on hard environmental boundaries — **filesystem isolation and outbound network restrictions that block credential theft even when the request appears entirely legitimate.**

Source: Anthropic's disclosed containment design, as reported by [InfoQ](https://www.infoq.com/news/2026/07/anthropic-claude-containment)

### 4.3 An allowlisted domain is not a trusted destination

The same disclosure contains a case worth remembering. In Claude Cowork's design, a malicious file caused Claude to upload workspace files to an attacker-controlled account — **through Anthropic's own Files API**. Because `api.anthropic.com` was allowlisted, the destination check passed.

> The lesson: **an allowlisted domain is not a trusted destination, it is an entry point to every function reachable through it.**

The fix was a proxy inside the VM that accepts only the session's provisioned token and blocks the relevant server-side-fetch headers. The trap generalises to anyone running an egress allowlist: if the allowlist granularity is a domain, then any large vendor domain that permits uploads is a potential exit.

Source: as above, via [InfoQ](https://www.infoq.com/news/2026/07/anthropic-claude-containment)

### 4.4 Injection is probabilistic, which makes it harder to find

The most counterintuitive property of the lethal trifecta is that **it does not fire every time.**

The same agent with all three legs, the same injected email: this run leaks, the next refuses. Prompt injection is not an exploit but a persuasion problem in language, and its success depends on how the model reads that particular text. The consequence: **a system that leaks one time in five is normal in every demo and every test you happen to run.** It does not fail tests. It leaks once, on some day.

This is precisely why "we tested it and nothing happened" is not evidence about this risk class. It is also why guardrail products should not be relied on here: Willison's assessment is that such products almost always claim to catch "95% of attacks," and **in web application security, 95% is a failing grade**. For data exfiltration, "occasionally fails" and "never works" differ only in how long it takes you to find out.

Source: [Simon Willison, *The lethal trifecta for AI agents*, 2025-06-16](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)

### 4.5 The reliability cost of autonomy is measurable

The third autonomy level in §3.1 — set a goal, loop until it succeeds — presupposes a check. Here is a measurable reason not to extend such runs indefinitely.

In measuring frontier agents' **task time horizons**, METR found that the **80% success horizon is about one fifth** of the 50% success horizon (see [3.1 §4.3](01-directing-the-workflow.md)).

In plain terms: to raise reliability from "works half the time" to "basically works," the usable task length drops to a fifth. **Every notch of autonomy up costs a notch of reliability — and reliability lost this way is not recovered by retrying.** Retrying fixes intermittent failure; it does not fix systematic drift.

So the question for a delegated run is not "how long can the model go?" but **"how soon can I confirm it has not drifted?"**

### 4.6 Context is the only channel

Returning to the three things in §3.3: they must be *written down* because, for an agent, **context is the only input channel.**

It does not remember the previous session, it does not know what you discussed with the user in another window, and it cannot know that an assumption has already changed in your head. Every compaction, every new session, and every parallel sub-agent is a place where that information can be dropped. Add Ng's specific point — **assumptions that changed partway through the build** — and this becomes the most dangerous category, because losing it produces no error at all; it just makes the agent faithfully keep building on a now-void premise.

The engineering implication: **writing decision-relevant information into files** rather than leaving it in the conversation is what keeps parallelism and session isolation from becoming unmanageable. The next manual in this part, 3.4, is about where exactly to write it.

Sources: [Simon Willison](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) for the trifecta; the source letter (2026-09-04) for context capture and changed assumptions.

## 5. Capability checkpoints

1. I can name the three levels of autonomy and pick the right one for a given task, with reasons.
2. I can judge whether a task suits "set a goal, loop until success," and identify the check it depends on.
3. I can list the three categories of context an agent needs captured (key learnings, user feedback, changed assumptions) and explain why the third is lost most easily.
4. I can explain why the practical ceiling on parallel agents is review capacity rather than compute.
5. I can distinguish model-layer, approval-layer and environment-layer controls, and explain why only the environment layer does not depend on judging correctly.
6. I can assess an agent's risk by counting the legs of the lethal trifecta and say which leg to remove.
7. I can explain why a 93% approval rate collapses the approval layer.
8. I can explain why effective sandboxing needs both filesystem and network isolation, and what fails if one is missing.
9. I can explain why "we tested it and nothing happened" cannot serve as evidence about prompt injection risk.
10. I can describe the trade between delegation length and reliability, and state my own basis for deciding how long to let an agent run.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Coding Agents*, 2026-09-04 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents) |
| Tier 1 · Official | Anthropic Engineering, *Beyond permission prompts: making Claude Code more secure and autonomous* | [anthropic.com](https://www.anthropic.com/engineering/claude-code-sandboxing) |
| Tier 1 · Study | METR, *Measuring AI Ability to Complete Long Tasks*, NeurIPS 2025 | [arXiv:2503.14499](https://arxiv.org/abs/2503.14499) |
| Practitioner | Simon Willison, *The lethal trifecta for AI agents*, 2025-06-16 | [simonwillison.net](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) |
| Practitioner · Relayed | InfoQ, *Anthropic Details How it Contains Claude across Web, Code, and Cowork* (relaying Anthropic's disclosed containment design) | [infoq.com](https://www.infoq.com/news/2026/07/anthropic-claude-containment) |

> **Sourcing note:** the three items in §4.2 and §4.3 — the 93% approval rate, the 24-of-25 exfiltration result, and the exfiltration via the Files API — are **relayed** from Anthropic's own disclosure via InfoQ. The 84% figure and the sandbox mechanism in §4.1 come from Anthropic's official engineering post.
