# 1.4 Evaluation-Driven Development

> **Part 1** · Building and deploying AI applications
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Evaluation-driven development is the practice of deciding what to build next based on measurements of how your system actually fails, rather than on intuition, on aggregate metrics, or on whatever seems most interesting to fix.

It matters because AI systems do not behave predictably. Traditional software can be planned in advance, because you can reason about what it will do. An AI system cannot: you do not know what an LLM will output, or what prediction a model will make on a new example. So building one is necessarily iterative — you build a piece, look at the results, and decide what to try next. The quality of that loop depends entirely on how well you can see what is going wrong.

Without this skill, the loop degrades into guesswork. You change something, the headline number moves a little, and you have no way to tell whether you fixed a real failure, introduced a new one, or merely traded one category of error for another. Progress becomes random and cannot accumulate.

## 2. In the map

This is one of six sub-skills of Part 1. Ng singles it out with the strongest claim in the letter:

> "In my experience, the most important trait that distinguishes someone great at building AI systems is whether you can drive a disciplined evals/error analysis loop to drive development."

His description of the skill has four parts. Its **purpose** is focus — it "allows you to repeatedly focus your effort on directions that are more likely to be fruitful". It is **hard to master** because the right approach "varies significantly by project and even according to the stage of the project". **Building evals is itself a deep technical skill**: you examine a system's traces and outputs, carry out exploratory data analysis, and combine that with product and business insight to decide what to measure. And you must know the **menu of options** — when to use deterministic code-based evaluation, when an LLM-as-a-judge, when a human in the loop — including how to evaluate your own evals so they keep improving. The results then feed an iteration loop that makes progress "systematic rather than random".

Source: [AI Engineering Skills Map Part 2, 2026-08-21](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications)

## 3. Core concepts

### 3.1 Three targets of evaluation

The most common mistake is to evaluate only the first of these.

| Target | Question it answers | Evidence |
|---|---|---|
| **Output** | Is the final answer correct? | The model's end product |
| **Trajectory** | Were the steps acceptable? | The full trace: tool calls, intermediate reasoning, loops, permissions |
| **Outcome** | Did the external state actually change correctly? | Database rows, files, downstream systems |

Both output and trajectory can look right while being wrong. An agent can report "your order is confirmed, number CA1234" while no such row exists in the database. For that reason, **outcome is the most trustworthy of the three** — state does not misreport itself.

The reverse mistake is equally damaging: a grader that hard-codes "must follow the standard procedure" will fail a correct-but-unexpected solution. Anthropic reported a case in the τ²-Bench airline task where an agent found a loophole in the refund policy, saved the user money, and was scored as a failure. Evaluation should ask whether the goal was achieved, not whether the agent walked the path you assumed.

### 3.2 Three grader types

The terminology follows Anthropic's agent-evaluation guidance, which distinguishes code-based, model-based and human graders.

| Type | Strengths | Weaknesses | Use when |
|---|---|---|---|
| **Code-based** | Fast (milliseconds), objective, reproducible, free | No sense of semantic nuance | Anything expressible as an assertion: schema validation, numeric tolerance, unit tests, tool-call checks, latency and token counts |
| **Model-based** (LLM-as-a-judge) | Understands semantics, scales, handles open-ended tasks | Non-deterministic, costs per call, requires calibration | Subjective quality, rubric scoring, open-ended output with no single right answer |
| **Human** | Expert judgement; the gold standard | Slow, expensive, does not scale | Calibrating the other two; high-risk scenarios |

In practice a good suite combines all three: a small set of high-quality human labels acts as the yardstick, an LLM judge is calibrated against it, and everything quantifiable is pushed down to code graders.

One specific trap: an over-strict code grader produces false failures. Comparing `expected == actual` fails a model that returns `96.12` when the key says `96.124991`, even though the precision is more than sufficient. The fix is a tolerance comparison.

### 3.3 Capability evals and regression evals

Two kinds of eval suite, with different purposes — do not read them the same way.

| Type | Question | Purpose |
|---|---|---|
| **Capability eval** | Can the system do hard tasks it currently fails? | Climb the hill — measure the ceiling |
| **Regression eval** | Did behaviour that used to work break? | Hold the camp — prevent silent degradation |

### 3.4 When to invest

Ng notes that the right approach depends on the project's stage, and that misjudging this is a common failure. The rough shape:

| Stage | State of the work | Appropriate eval effort |
|---|---|---|
| **Exploration** | What to build is still moving | Light. Read data, build intuition, use spreadsheets rather than infrastructure |
| **Convergence** | Approach is settled; comparing implementations | Heavy. Build a reproducible eval set, wire it into CI |
| **Production** | Real users | Eval becomes infrastructure, connected to online monitoring |

## 4. Going deeper

### 4.1 Why an aggregate metric is not enough to act on

A single accuracy or F1 figure tells you the error rate but not the error structure. If a model is at 95%, roughly 5% of cases are wrong — and the engineering response depends entirely on how those cases are distributed:

- **Concentrated in one category** → fix the features or preprocessing for that category.
- **Dispersed and apparently random** → suspect the labels before the model.
- **Concentrated in one subsystem or data source** → look at the fusion or integration strategy.

These three paths are indistinguishable until you look at individual failures. Worse, aggregate metrics hide offsetting trades: loosening a threshold can improve one error category and degrade another while the total stays flat — a change that was actually a trade, presented as no change at all.

### 4.2 Documented biases of LLM-as-a-judge

From a systematic study of the approach:

> [Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, NeurIPS 2023](https://arxiv.org/abs/2306.05685)

| Bias | Manifestation | Mitigation |
|---|---|---|
| **Position** | Favours whichever answer appears first | Run the judge twice with the order swapped; accept a verdict only if consistent |
| **Verbosity** | Prefers longer, more elaborate answers even when redundant | Constrain length in the rubric, or normalise for it |
| **Self-enhancement** | Favours output from its own model family | Use a judge from a different family |
| **Limited reasoning** | Grades poorly on problems it cannot itself solve (math, logic) | Chain-of-thought prompting; supply a reference answer |

The same paper distinguishes three grading modes — **pairwise comparison** (which of two answers is better), **single-answer grading** (score one answer against a rubric), and **reference-guided grading** (provide a gold-standard answer). It also reports that a strong judge such as GPT-4 reaches roughly **80–85% agreement** with human experts, comparable to the agreement between two humans.

### 4.3 High agreement does not mean a useful grader

The 80–85% figure is often quoted as validation. It is not. A judge that always returns PASS will also show very high agreement with human labels while catching nothing at all. Agreement measures whether the grader matches the labels on average; it does not measure whether the grader discriminates. To know whether a grader is useful you need to look at what it catches and what it misses — the false negatives and false positives — not just the headline agreement rate.

This is the concrete meaning of Ng's point about "how to evaluate your evals".

### 4.4 Error analysis is the starting point, not the eval set

The most actionable published method comes from Hamel Husain and Shreya Shankar, distilled from production implementations. The order matters and is frequently inverted:

1. **Collect real traces.** A trace is the full record of one pipeline interaction: the initial input, every LLM input and output, intermediate reasoning and tool calls, and the final user-facing result.
2. **Read at least ~100 diverse traces by hand.** Ten to fifteen is nowhere near enough. The goal is *theoretical saturation* — reading further introduces no new failure modes.
3. **Open coding.** Write short descriptive notes on what went wrong. At this stage, deliberately **do not** diagnose root causes, and do not zoom into every word. Record one dominant failure mode per trace. Root-cause analysis is unbounded and counterproductive here: it slows the pass and biases what you notice.
4. **Axial coding.** Cluster the open codes into themes, producing a **failure taxonomy**.
5. **Rank by frequency.** Fix the most common failure, not the most tractable one.
6. **Write evaluators only for failures that have stabilised.** Concrete, well-defined failures go to code graders first; only what cannot be quantified goes to an LLM judge.
7. **Calibrate and evolve.** Check each evaluator against human labels, and expect the criteria themselves to change as the project moves.

Practices this method explicitly rejects: shopping for the best eval tool or platform before looking at data; buying generic off-the-shelf metrics that do not understand your domain; outsourcing annotation, which discards precisely the product intuition the exercise is meant to build; and writing a large eval suite before you have read enough data to know what to test.

Sources: [A Field Guide to Rapidly Improving AI Products](https://hamel.dev/blog/posts/field-guide/), [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/), [AI Evals notes index](https://hamel.dev/notes/llm/evals/).

### 4.5 From taxonomy to evaluators

Each high-frequency failure mode maps to one evaluator. The choice of type follows §3.2:

| Failure shape | Grader type |
|---|---|
| Structured output malformed, wrong type, out of range | Code-based (schema validation) |
| Numeric result outside tolerance | Code-based (numeric comparison) |
| A required tool was not called, or called with bad arguments | Code-based (trajectory inspection) |
| Output violates a stated policy | Model-based (rubric with a yes/no verdict) |
| Tone, helpfulness, faithfulness to a source | Model-based |
| Ambiguous, high-stakes, or contested cases | Human |

### 4.6 Wiring evals into CI

Once an eval set is stable, run it on every change. The usual shape follows a testing pyramid: cheap code-based evals on every commit; expensive LLM judges and human review at key gates only. DeepLearning.AI's *Automated Testing for LLMOps* course covers building this pipeline.

## 5. Capability checkpoints

1. I can distinguish output, trajectory and outcome evaluation, and explain why outcome is the most trustworthy evidence — and why a trajectory-only grader can reject a correct solution.
2. I can name the three grader types and select a combination for a given task, justifying each choice.
3. I can name the four documented LLM-as-a-judge biases and give the standard mitigation for each.
4. I can explain why a judge showing 90% agreement with humans may still be useless.
5. I can describe the error-analysis workflow in the correct order, and explain why root-cause analysis is deliberately postponed until after open coding.
6. I can take a failure taxonomy and assign each entry to a grader type, with a reason.
7. I can judge, for a project at a given stage, whether building a heavy eval suite is the right use of effort.
8. I can explain why an aggregate accuracy figure alone is insufficient to choose the next engineering step.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 2*, 2026-08-21 | [link](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications) |
| Source letter | Andrew Ng, *The AI Engineering Skills Map* (overview), 2026-08-14 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map) |
| Tier 1 — paper | Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, NeurIPS 2023 | [arxiv.org/abs/2306.05685](https://arxiv.org/abs/2306.05685) |
| Tier 2 — practitioner | Hamel Husain, *A Field Guide to Rapidly Improving AI Products* | [hamel.dev](https://hamel.dev/blog/posts/field-guide/) |
| Tier 2 — practitioner | Hamel Husain, *Your AI Product Needs Evals* | [hamel.dev](https://hamel.dev/blog/posts/evals/) |
| Tier 2 — practitioner | Hamel Husain, AI Evals notes index | [hamel.dev](https://hamel.dev/notes/llm/evals/) |
| Tier 2 — practitioner | Hamel Husain & Shreya Shankar, *AI Evals for Engineers and PMs* (course) | [maven.com](https://maven.com/parlance-labs/evals) |
| Tier 1 — official | Anthropic, *Building effective agents*, 2024-12 | [github.com/anthropics/anthropic-cookbook](https://github.com/anthropics/anthropic-cookbook) |
| Tier 1 — official | OpenAI, *A Practical Guide to Building Agents*, 2025-04 | [PDF](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) |
| Tier 3 — course | DeepLearning.AI, *Automated Testing for LLMOps* | [link](https://read.deeplearning.ai/courses/automated-testing-llmops) |
