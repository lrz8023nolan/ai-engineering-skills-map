# 3.1 Directing the Workflow

> **Part 3** · Using coding agents
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

Directing the workflow means **deciding, at every step, who does the work, how deep they go, and when to go back a step** — rather than letting the agent decide on its own how far the whole thing should get.

It matters because the bottleneck in working with a coding agent is no longer typing. The agent writes faster than you do and gets most of it right. What it does not know is what you actually want, or when it has guessed wrong. Development therefore shifts from writing the code to *stating what should be built* and *judging whether what came back is correct*. **Neither of those can be handed over** — hand them over and there is nobody left who can judge the result.

Without this skill, two extremes recur. One is throwing a task over the wall and getting back a pile of code that runs but solves the wrong problem, then spending longer steering it back. The other is watching every step closely, which makes the agent fast but turns the human into the new bottleneck — often slower overall than writing it yourself.

## 2. In the map

In this letter Ng first lays out a high-level workflow, then names the five sub-skills that support it. The workflow has three stages:

| Stage | What it contains |
|---|---|
| **Planning** | Brainstorming (which may include research, experimentation, and understanding the existing codebase); writing a **spec** capturing requirements, technical design and architecture, then generating an execution plan; reviewing that plan to interrogate key assumptions and check for security, overengineering and other gaps |
| **Execution** | Building, testing and verifying, balancing agent autonomy against human oversight |
| **Deployment and monitoring** | Deploying, perhaps gated by CI/CD or additional human gates; then using agents to watch logs, surface issues, and propose and execute improvements |

His definition of the skill itself:

> "This involves deciding how much human and how much agent effort to spend on each and when to go back to an earlier step to iterate. It requires deeply understanding the tradeoffs of speed, cost, technical risk, and human effort..."

He is explicit about two things. **Duration varies and steps can be omitted** — the spec for a greenfield prototype might be loosely described in a quickly written prompt, whereas a brownfield project with many users may require much more effort to write *and verify*. And the workflow is **highly iterative**: "skilled developers know when feedback from a later step should lead them back to an earlier one."

Source: [AI Engineering Skills Map Part 4, 2026-09-04](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents)

## 3. Core concepts

### 3.1 The three tiers of planning artefacts

How much to write down is the most common misjudgement here. The criterion is **what it costs when this turns out wrong** — high cost, write more; low cost, write less.

| Tier | Artefact | Fits | Cost of being wrong |
|---|---|---|---|
| **A prompt** | State what you want in the conversation | Greenfield prototypes, throwaway scripts, disposable exploration | Redo it; a few minutes lost |
| **A spec** | Requirements + technical design + architecture, as a document | Features with real users; changes that must fit existing code | Has to be unpicked from code |
| **Spec + execution plan** | The above, decomposed into ordered, verifiable steps | Multi-person work, review required, wide blast radius | Affects several modules and other people |

On brownfield projects Ng's point is that the spec must not only be written but **verified** — the real constraints of an existing system usually live in the code, not in documentation.

### 3.2 Three classes of gap to look for when reviewing a plan

He names them directly: **security, overengineering, and other gaps.** Overengineering is the one most often missed, because its symptom is that the code looks *more* professional. Concrete things to look for:

- abstractions, configuration options or frameworks that nothing currently needs
- extension points built "in case we need them later"
- a simple conditional written as a strategy pattern
- What is the key assumption? If it is wrong, does the whole design fall over?

### 3.3 The criterion for decomposition: verifiability

When splitting work into steps for an agent, there is a practical test for whether a step is a good unit: **can you write down its check?** If you can, the step can be handed over to run autonomously. If you cannot, either it needs splitting further or someone needs to sit beside it.

This also explains why "give the agent the whole requirement and let it decompose" usually underperforms — it decomposes along the logic *it* sees, not along the boundaries *you* can verify.

### 3.4 What stays with humans

Ng's phrasing is **"when to retain human ownership over critical work."** Two dimensions decide it:

| Dimension | Signal to keep it human |
|---|---|
| **Irreversibility** | Data migration, deletion, public release, real money |
| **Unverifiability** | There is no objective check on whether the result is right — only judgement |

When both are low, delegating is the better trade. If either is high, keep the decision — not by writing the code yourself, but by owning the **conclusion**.

### 3.5 When to go back a step

This is where directing separates from following. Three typical triggers:

| Symptom | Go back to |
|---|---|
| Verification failed, but the cause is that the requirement was never clear | Planning — rewrite the spec |
| Implementation revealed that an architectural assumption does not hold | Planning — change the design |
| Deployment or monitoring surfaced a problem | Execution (or planning, depending on its nature) |

**Going back is not failure.** What is genuinely expensive is pressing on and amplifying the error.

## 4. Going deeper

### 4.1 A measured result that contradicts intuition

In 2025 METR ran a randomised controlled trial with 16 experienced developers who each maintained large open-source projects (averaging over 22,000 GitHub stars, with over a million lines of code each), working on 246 real issues drawn from their own repositories.

The result: **with AI tools, these developers took 19% longer to complete the same tasks.**

The perception is the more striking half. Before starting, they predicted the tools would speed them up by 24%. After finishing — facing a measured slowdown — they still believed the tools had made them 20% faster.

Where the time went was also measured. Analysing over 140 hours of screen recordings, the researchers found the tools did save time in three places — **active coding, testing/debugging, and reading/searching for information** — but those savings were outweighed by two costs:

- **reviewing AI output, prompting the AI, and waiting for generation**
- **idle and overhead time** (periods with no on-screen activity)

Two more numbers from the same study: these developers **accepted AI-generated code without modification less than 44% of the time**, and in the AI-assisted half of the study **about 9% of total task time went to reviewing the AI's output**.

Source: [METR, *Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity*, 2025](https://metr.org/Early_2025_AI_Experienced_OS_Devs_Study.pdf) ([arXiv:2507.09089](https://arxiv.org/abs/2507.09089))

### 4.2 Why "faster" and "feels faster" come apart

This should not be read as "the tools are useless." It says something else: **the bottleneck moved.**

METR's explanation is that coding benchmarks trade realism for scale — tasks are self-contained, need no prior context to understand, and are scored algorithmically. Both properties **overestimate** capability. Real tasks require reading a codebase you do not know, and correctness usually has no ready-made grader.

That resolves the paradox. If the bottleneck was *writing*, the agent wins. If the bottleneck is *understanding existing constraints* and *judging whether the result is right*, the time saved on typing gets absorbed — or more than absorbed — by the added comprehension and review cost. **The more familiar you are with your own codebase, the higher those costs are**, because you can see where the agent's solution conflicts with that codebase's conventions, and seeing that takes time.

The engineering implication is direct: **point the agent at places where the bottleneck really is writing** — greenfield code, well-specified modules, bulk mechanical rewrites — not at places where the bottleneck is understanding, such as altering a subsystem you do not know well either.

### 4.3 How long a task you can delegate depends on how long you can verify

METR's other study proposed a way to measure capability: the **time horizon** — translating what an agent can reliably do into how long a human expert takes for the same task. They measure the **50% time horizon**: the length of task the agent completes successfully half the time.

Two findings are worth keeping:

- **The growth is exponential.** Across 2019–2025, the 50% time horizon for frontier model agents has been **doubling roughly every 7 months**. At publication (March 2025), Claude 3.7 Sonnet's 50% time horizon was about **50 minutes**.
- **Reliability and task length are a steep trade.** The same models' **80% time horizon is about one fifth** of their 50% time horizon.

The second is the most engineering-relevant and least quoted. It means: if you want "this will basically work" rather than "this works half the time," the length of task you can hand over is one fifth as long. **Move autonomy up one notch and the usable task length drops one notch.**

METR later published an updated dataset (Time Horizon 1.1), growing the task suite from 170 to 228 tasks and fitting a shorter doubling time: about 131 days after 2023, tightening to roughly 89 days for 2024 onward. In the same version, Claude Opus 4.5's 50% time horizon is around 4 hours 49 minutes. METR also warns that, **with the current task suite, measurements above 16 hours are unreliable** — the suite is approaching saturation at the high end.

Source: [METR, *Measuring AI Ability to Complete Long Tasks*, 2025](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/) ([arXiv:2503.14499](https://arxiv.org/abs/2503.14499), NeurIPS 2025), [Time Horizons tracking page](https://metr.org/time-horizons)

### 4.4 Ng's own caution about long autonomous runs

This deserves its own section, because it comes from the source document itself and runs against the prevailing tone:

> "For example, it is sometimes useful to get agents to run autonomously for hours and burn millions or tens of millions of tokens. But currently the practical utility of very long-horizon tasks — especially relative to their cost — has been amplified beyond reality."

His conclusion: "most effective coding agent use is a complex, highly iterative process, and being able to intervene with high-skill judgement gives much better results." That is the same fact as §4.3 stated differently — how far you can delegate is set by how quickly you notice it going wrong.

### 4.5 Why planning is worth more, not less, in the agent era

There is a mechanical reason worth stating plainly.

Previously, a wrong spec was contained by the **cost of implementation**: if the idea was bad, it hurt partway through, and people naturally stopped to reconsider. That brake has largely gone. A wrong spec will still be implemented — faithfully, completely, and quickly — and because the resulting code looks tidy, it is harder, not easier, to recognise as wrong.

That is why planning, and the interrogation of key assumptions in the plan, carry *more* weight in the overall workflow, not less. Planning does not save writing time. It saves rework on a faithfully implemented mistake.

## 5. Capability checkpoints

1. I can draw the planning → execution → deployment-and-monitoring loop and say what the human's specific action is in each stage.
2. I can decide, for a given task, which tier of planning artefact it needs (a prompt / a spec / a spec plus execution plan) and justify the choice.
3. I can take an agent-produced execution plan and point out its key assumptions, overengineering, and security gaps.
4. I can decompose a requirement into steps, state the check for each step, and say which ones can run autonomously.
5. I can name which work should retain human ownership and give the criteria behind that judgement.
6. I can give at least three signals that it is time to go back a step, and say which step each returns to.
7. I can explain why experienced developers measured *slower* with these tools, and what that implies about which tasks to delegate.
8. I can describe the trade between task length and reliability, and use it to pick a sensible delegation length for a specific task.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Coding Agents*, 2026-09-04 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents) |
| Tier 1 · Study | METR, *Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity*, 2025 | [PDF](https://metr.org/Early_2025_AI_Experienced_OS_Devs_Study.pdf) · [arXiv:2507.09089](https://arxiv.org/abs/2507.09089) |
| Tier 1 · Study | Kwa et al., *Measuring AI Ability to Complete Long Tasks*, NeurIPS 2025 | [arXiv:2503.14499](https://arxiv.org/abs/2503.14499) |
| Tier 1 · Research org | METR, *Measuring AI Ability to Complete Long Software Tasks* (project page) | [metr.org](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/) |
| Tier 1 · Research org | METR, *Time Horizons* (continuously updated tracker, incl. Time Horizon 1.1) | [metr.org/time-horizons](https://metr.org/time-horizons) |

> **A note on sourcing:** the methods and conclusions of the two METR studies are taken from the papers themselves (abstract and body), and the figures 19% slower, the perception gap, under 44%, and about 9% review time come from the papers' public write-ups. The Time Horizon 1.1 figures come from METR's official tracking page rather than the original paper; treat that page as authoritative for them.
