# 3.4 Customizing the Agent and Its Environment

> **Part 3** · Using coding agents
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

Customizing the agent and its environment means **deliberately reshaping two things — the agent itself (which skills it has, which tools it can reach) and the environment it works in (repo structure, standing instructions, state files)** — so that it gets the right context on the first try in your project.

It matters because an agent does not know anything you have not written down. Why your team chose PostgreSQL over MySQL, why `/legacy` must not be touched, which command runs the tests, why data access has to go through a particular layer — in a human's head these are common knowledge; to an agent they are blank. Its only way to fill that blank is to **explore**, which costs time and tokens and frequently guesses wrong.

Without this skill, the most common symptom is that **you repeat the same sentences every week**. "Use pnpm, not npm." "Tests run with this command." "Don't touch that file." Every new session needs the same briefing. Worse is the case where you do not say it: the agent falls back on the model's generic defaults, and those are usually not your project's.

## 2. In the map

Ng frames it as a two-sided capability:

> "Your ability to update both the agent and the environment it works in allows your agents to efficiently get the context they need, access tools, and build correctly and efficiently."

He lists seven actions, which group into four:

| Group | Actions |
|---|---|
| **Give the agent capabilities** | Integrate agent skills, plugins, MCP servers; **occasionally prune them** (for example when a new model obsoletes an old skill) |
| **Automate the fixed steps** | Use **hooks** to automate repeatable parts of the development process, such as triggering automated code reviews or CI/CD |
| **Maintain the environment** | Update the **standing context** (AGENTS.md, CLAUDE.md and the like) with information on the codebase, **key architectural assumptions**, code style and data access patterns; establish consistent conventions and structure so the codebase is navigable to the agent; **occasionally clear out agent-generated debt** |
| **Across sessions and teams** | Preserve state across multiple sessions and parallel agents; accumulate agent learnings over time, perhaps by running **post-run retrospectives** to capture what did and did not work; in a team, coordinate context across different developers' agents |

Source: [AI Engineering Skills Map Part 4, 2026-09-04](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents)

## 3. Core concepts

### 3.1 Four extension layers, four different jobs

These get conflated, but they load at different times and do different things. Choosing the wrong layer typically wastes a scarce resource.

| Layer | Solves | When it enters context | Example |
|---|---|---|---|
| **Standing context** | What this project is and what its conventions are | At session start, in full | AGENTS.md, CLAUDE.md |
| **Skill** | How a class of task is done (procedural knowledge) | Body loads only when relevant | "How to fill a PDF form" |
| **MCP server** | Connectivity to external services and data | Tool definitions enter context once connected | Filesystem, database, browser |
| **Hook** | Actions that must happen **every** time | Never enters context; deterministic code runs it | Lint before commit, trigger CI |

A rough rule: **things that need judgement go into skills or standing context; things that must be guaranteed go into hooks.** Ng's phrasing is that hooks automate the *repeatable* parts of the development process — note the keyword is **repeatable**, not **important**. Something important but not fixed does not belong in a hook.

### 3.2 Progressive disclosure: why skills do not cost context

The core design of a skill is not "packaging knowledge" but **loading it in three stages on demand**:

| Level | Content | When loaded | Cost |
|---|---|---|---|
| 1: metadata | `name`, `description` (YAML frontmatter) | Pre-loaded into the system prompt at startup, for every skill | Tiny, scales with skill count |
| 2: body | The `SKILL.md` content | When the skill is judged relevant and triggered | Moderate |
| 3: bundled files and scripts | Referenced docs, templates, executable scripts | Read when the body points to them; scripts run and **only their output** enters context | On demand; script source never enters context |

The third level is the counterintuitive part: **a script can be large, and its code never enters the context.** The model sees only the output. This is why the amount of context a skill can bundle is described by Anthropic as "effectively unbounded."

Source: [Anthropic Engineering, *Equipping agents for the real world with Agent Skills*, 2025-10-16](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) (published as an open standard on 2025-12-18)

### 3.3 What belongs in standing context

Ng's four items are **codebase information, key architectural assumptions, code style, and data access patterns**. Better than a checklist is one test:

> **Write what the agent cannot guess. Do not write what it already knows.**

For standard tools and conventions — `pytest`, `npm test`, PEP 8 — the model already knows; copying them into the standing file just spends context. What earns its place is **the deviation from the norm**: non-standard build tooling, bespoke directory conventions, technology choices with historical reasons, modules that must be avoided, and operational workflows that hold only in your organisation.

One anti-pattern deserves its own warning: **do not write a persona** ("You are a senior engineer…"). It contains no information about this project and is pure standing cost.

### 3.4 Preserving state across sessions

Ng's requirement is to preserve state across multiple sessions and parallel agents. The only reliable way is: **state lives in files, not in the model's memory.**

A workable minimal set:

| Artefact | Purpose |
|---|---|
| **Feature list** (structured, e.g. JSON) | Tracks the status of every requirement, starting from "failing" |
| **Progress log** | What each session did, what worked, what was left unfinished |
| **Bootstrap script** | One command to bring the dev environment up, instead of re-discovering it |
| **Git history** | The authoritative record of what actually shipped |

### 3.5 Pruning and debt cleanup

Ng names two things easily deferred: **pruning** (removing a skill when a new model obsoletes it) and **clearing out agent-generated debt**.

What they share is that **their cost is invisible.** An unused skill merely occupies a little context; an ever-growing standing file merely costs a few more tokens per session; duplicated code merely makes changes slower. Nothing raises an error. Neither will happen on its own — only on a schedule.

### 3.6 Coordinating across a team

When several people use different tools, the usual failure is that **one set of conventions gets copied into N files which then drift apart.** The workable arrangement is to maintain **one authoritative file** and let each tool read the name it recognises (for example, symlinking `CLAUDE.md` to `AGENTS.md`).

The test is not the format but: **how many sources of truth does this project have?**

## 4. Going deeper

### 4.1 A controlled experiment: context files generally do not improve success rates, and add over 20% to cost

This is the finding to remember from this manual, because it overturns what the entire industry recommends.

In February 2026, researchers at ETH Zurich and LogicStar.ai ran the first controlled study of standing context files. They used two benchmarks: **SWE-bench Lite** (300 tasks across 11 popular Python repositories, which had no context files, so one was generated by an LLM) and a purpose-built **AGENTbench** (138 instances across 12 **less popular** repositories that already carried developer-written context files — deliberately niche, to avoid contaminating results with model memory). Four agent/model combinations took part: Claude Code + Sonnet 4.5, Codex + GPT-5.2, Codex + GPT-5.1 mini, and Qwen Code + Qwen3-30b-coder. Each repository was run in three settings: **no context file, an LLM-generated one, and a developer-written one**.

The abstract's conclusion:

> "Across multiple coding agents and LLMs, we find that context files tend to reduce task success rates compared to providing no repository context, while also increasing inference cost by over 20%."

The mechanism is more useful than the headline, and the paper explicitly rejects the obvious explanation:

> "Behaviorally, both LLM-generated and developer-provided context files encourage broader exploration (e.g., more thorough testing and file traversal), and coding agents tend to respect their instructions."

In other words, **the failure is not disobedience — it is obedience.** Every line in a context file is a requirement the model now tries to satisfy, and satisfying requirements costs steps and tokens. The measurable costs include more execution steps, 14–22% more reasoning tokens, and more testing and file traversal.

One specific result demolishes the most-recommended section. A **codebase overview** — the directory tree plus module notes — appeared in essentially every LLM-generated file and in 8 of 12 human ones, and it **did not help agents find the relevant files any faster**: on the metric the authors built for exactly this (steps before first touching a file the real fix changed), it made no difference. Modern agents are already good at exploring a repo unaided; handing them a tour they did not ask for mostly gives them something extra to read.

The paper's conclusion: **unnecessary requirements from context files make tasks harder, and human-written context files should describe only minimal requirements.**

Source: [Gloaguen, Mündler, Müller, Raychev & Vechev, *Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?*, 2026](https://arxiv.org/abs/2602.11988)

### 4.2 Tool descriptions are an instruction channel

MCP lets an agent reach external tools by reading each tool's name, description and parameter schema into context. The problem: **those descriptions are text, and a model does not read text the way you do — it cannot distinguish "an instruction from the developer" from "a description from a tool server."**

In April 2025, Invariant Labs disclosed the resulting attack class, which they named the **tool poisoning attack** and classified as a specialised form of **indirect prompt injection**. The technique is to embed instructions in a tool's description field. For a tool called `add` that sums two numbers, the description can carry instructions to first read `~/.cursor/mcp.json` and `~/.ssh/id_rsa`, pass their contents as a `sidenote` parameter, and not mention to the user that reading those files was necessary. The user sees an addition tool. The model reads the whole text and complies.

Three structural reasons make this work, and each is worth remembering:

| Reason | What it means |
|---|---|
| **Information asymmetry** | The user sees a tool name and a one-line summary; the model reads the full description. The attacker writes for the **larger audience** and bets the human reads the smaller one |
| **The instruction layer is flattened** | User messages, system prompt, tool descriptions and tool output all become one token sequence. The model can only guess which text is "meant to be obeyed" |
| **Models are trained to follow instructions** | A well-crafted instruction in a tool description is simply another instruction. Dressing it as a "required precondition" or "implementation detail" makes it more effective |

Two properties amplify the danger:

- **Cross-server escalation.** Every connected server shares one context, so **one malicious server can steer how the model uses other, trusted servers** — for instance by having it read files with a filesystem tool and send them out with an HTTP tool. Auditing servers one at a time is therefore insufficient.
- **Rug pulls.** Most clients request approval only the first time a tool is seen, and cache that approval by **tool name** rather than by content. A server can therefore pass review, then silently rewrite the same tool's description, with no second consent step.

Source: [Invariant Labs, *MCP Security Notification: Tool Poisoning Attacks*, 2025-04](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks)

**This is the same problem as the lethal trifecta in 3.2, arriving through a different door.** 3.2 describes three legs — access to private data, exposure to untrusted content, ability to send data out. Tool poisoning makes the point that **the moment you connect an MCP server, you have already placed untrusted content into the context** — usually without noticing that you made that decision. The remedies match: withhold unnecessary egress, keep credentials out of any session that reads outside content, and actually read a tool's description before installing it.

### 4.3 Why debt cleanup is not "someday" work

Ng specifically calls out "occasionally clear out agent-generated debt." The reason is mechanical, not merely hygienic.

The main input to an agent generating code is **the code it reads**. A pattern that is everywhere in a codebase reads to the model as this project's normal practice. Conversely, **deleting an unwanted pattern from the codebase is more effective than banning it in a standing file** — the former changes the model's local prior; the latter is just an instruction it may or may not follow.

This is the other face of §4.1: **rather than writing more requirements into a file, shape the environment into the thing you want imitated.**

## 5. Capability checkpoints

1. I can distinguish what standing context, skills, MCP servers and hooks each solve, and say which layer a given need belongs to.
2. I can explain the three loading levels of progressive disclosure and why the amount of context a skill can bundle is effectively unbounded.
3. I can write a standing context file containing only what the agent cannot guess, and name what I removed because it already knows it.
4. I can list the artefacts needed to preserve state across sessions and explain why state should not be trusted to model memory.
5. I can say which class of work hooks should carry, and give an example of something that should not be a hook.
6. I can explain why "context files do not improve success rates and add over 20% cost" has nothing to do with agents ignoring instructions, and describe the actual mechanism.
7. I can explain why a codebase overview section delivers less than its popularity suggests.
8. I can describe the mechanism of a tool poisoning attack and explain why flattening the instruction layer makes it hard to solve with filtering.
9. I can say what cross-server escalation and rug pulls each exploit, and give two matching mitigations.
10. I can explain how clearing agent-generated debt directly changes the agent's future default behaviour.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Coding Agents*, 2026-09-04 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-using-coding-agents) |
| Tier 1 · Paper | Gloaguen, Mündler, Müller, Raychev & Vechev, *Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?*, 2026 | [arXiv:2602.11988](https://arxiv.org/abs/2602.11988) |
| Tier 1 · Official | Anthropic Engineering, *Equipping agents for the real world with Agent Skills*, 2025-10-16 (open standard from 2025-12-18) | [anthropic.com](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) |
| Tier 1 · Vendor security research | Invariant Labs, *MCP Security Notification: Tool Poisoning Attacks*, 2025-04 | [invariantlabs.ai](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks) |
| Tier 1 · Standard | Model Context Protocol (specification and documentation) | [modelcontextprotocol.io](https://modelcontextprotocol.io/) |
| Tier 1 · Standard | AGENTS.md (the cross-tool standing-context format) | [agents.md](https://agents.md/) |
