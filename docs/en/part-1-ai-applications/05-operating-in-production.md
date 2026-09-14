# 1.5 Operating in Production

> **Part 1** · Building and deploying AI applications
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Operating in production means **keeping a system available and correct under real users, real traffic and real failures** — not just getting it to run once on your machine.

What separates it from ordinary software operations comes down to three properties: **output is unpredictable** (you cannot confirm "correct" with a single assertion), **cost scales linearly with usage** (every request spends money, where traditional software's marginal cost is near zero), and **latency becomes experience** (users will not wait). Together these make "shipping" not an endpoint but a state that has to be measured continuously.

Without this skill, three failure shapes recur. The system quietly degrades and nobody notices — the input distribution drifted while the model stayed the same. The bill runs away — some feature is being called far more than intended, or prompt caching stopped working entirely. And after an incident nobody can localise the cause, because the observability data was never captured; all you know is "users say it got worse".

## 2. In the map

This is the fifth of the six sub-skills of Part 1. Ng attributes its distinctiveness directly to three properties:

> "Operating AI software is different from traditional software because of its unpredictability, cost, and latency."

He sets out three groups of requirements:

| Requirement | What it covers |
|---|---|
| **Build observability** | Understand performance on real usage; track performance, **detect drift**, and respond quickly to model failures and security incidents such as adversarial prompt injection |
| **Regression testing and CI/CD** | Requires **more statistical evaluations** than traditional software; the testing effort should be **calibrated relative to the risk of a mistake** |
| **Cost and latency optimisation** | Selecting the right mix of techniques — model choice optimisation, distillation and fine-tuning, agentic workflow simplifications — especially at scale |

Source: [AI Engineering Skills Map Part 2, 2026-08-21](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications)

## 3. Core concepts

### 3.1 The three questions observability must answer

| Question | Signal needed |
|---|---|
| What happened? | **Traces** — the full path of one request: every LLM call, tool call, duration and token count |
| How is it going overall? | **Metrics** — aggregate latency, cost, success rate, quality scores |
| When did it start going wrong? | **Logs plus version stamps** — model version, prompt version and code version on one timeline |

The relationship: metrics tell you there **is** an anomaly, traces tell you **what it looks like**, and version stamps tell you **which change introduced it**. Remove any one and diagnosis degrades into guesswork. Cross-vendor standards already exist for unifying traces, metrics and logs (OpenTelemetry) — there is no need to invent your own scheme.

### 3.2 What to track

| Category | Key metrics | Why |
|---|---|---|
| **Quality** | Eval scores (see manual 1.4), user feedback, task completion rate | The only metric that genuinely matters for an AI system |
| **Cost** | Tokens per request, cache hit rate, cost per user | Cost blowout is a failure mode particular to AI systems |
| **Latency** | p50 / p95 / p99, with **time to first token** reported separately from total | Averages hide bad tail experience; in conversational settings the first token matters more to perceived responsiveness |
| **Business** | Conversion, retention, human-intervention rate | Good technical metrics with no business movement means you are measuring the wrong thing |

**A layered requirement that is easy to miss**: every metric should be sliceable — by user segment, input type, model version, time of day. Aggregate numbers hide the case where one category of user is being systematically harmed.

### 3.3 Three kinds of drift

| Kind | Meaning | Typical signal |
|---|---|---|
| **Data drift** (input distribution changed) | What users ask has changed, or the upstream data has | Shift in input length, topic or language distributions |
| **Concept drift** (input-to-output relationship changed) | The right answer for the same input has changed | Quality metrics fall while the input distribution is unchanged |
| **Upstream change** | A data source's schema, semantics or interface changed | Parse failure rate rises; fields suddenly arrive empty |

The third is the most overlooked and the most often misdiagnosed. In practice a large share of "the model got worse" incidents are **an upstream system quietly renaming a field or changing a format**.

### 3.4 Why CI/CD needs statistical evaluation

| | Traditional software | AI systems |
|---|---|---|
| Pass criterion | Assertions: equal / not equal | Statistics: did the score change significantly? |
| Threshold | Deterministic | Needs thresholds and tolerances |
| Coverage | Unit and integration tests | Plus eval sets (see manual 1.4) |
| Cost sensitivity | Low | High — every regression run costs real money |

This is what Ng means by calibrating testing effort to the cost of a mistake. Running a full LLM-judge regression suite for an internal tool is waste; running a handful of assertions for a high-stakes system is negligence. **The calibration criterion is: if this error reaches production, what does it cost?**

### 3.5 Release strategies

| Strategy | Approach | Fits |
|---|---|---|
| **Shadow** | A new model receives real traffic in parallel but its output is not returned to users, only logged for comparison | Validating correctness and real latency before launch |
| **Canary** | Send a small share of real traffic to the new version first | Progressive rollout with cheap failure |
| **Gradual rollout** | Shift traffic by user or proportion, keeping a fast rollback path | Routine releases |
| **Feature flags** | Model and prompt versions switchable at runtime | When you need second-level rollback |

**The critical precondition: model versions and prompt versions must be first-class rollback targets**, exactly like code versions. If rolling back a model means redeploying the whole application, you do not actually have rollback capability.

### 3.6 Four levers for cost and latency

In rough order of return on effort:

| Lever | Approach | Where the gain comes from |
|---|---|---|
| **Model choice and routing** | Simple requests to a small model, hard ones to a large model | Directly reduces high-cost model calls |
| **Agentic workflow simplification** | Remove unnecessary round trips; use code wherever a model is not needed | Fewer calls and fewer tokens overall |
| **Serving-layer efficiency** | Prompt caching, continuous batching, quantisation, speculative decoding | Serve more requests on the same hardware |
| **Distillation and fine-tuning** | Compress a large model's capability into a small one | Lower unit cost over the long run |

The first two are **architectural**, usually give the largest return, and require no training. The last two are **infrastructure and model** level: more investment, higher ceiling.

### 3.7 Incident response

AI systems add several incident classes that traditional software does not have:

| Class | Manifestation | Response |
|---|---|---|
| **Model failure** | Upstream API change, rate limiting, outage | Multi-vendor fallback; request queuing and degradation |
| **Adversarial input** | Prompt injection leading to privilege escalation or exfiltration (see manual 1.3) | Least privilege; human gates on irreversible actions |
| **Cost blowout** | Unbounded loops, runaway retries, caching gone cold | Hard budget caps and rate limits; alerts on cache hit rate |
| **Silent quality regression** | Metrics slide slowly, unnoticed | Online quality sampling; drift alerting |

The common requirement: **you must be able to stop the bleeding without shipping a new version.** Degradation paths — disable a feature, switch back to the old model, reject requests — have to be built in advance rather than improvised during an incident.

## 4. Going deeper

### 4.1 Machine learning technical debt is system-level

The hardest part of operating in production is the debt you cannot see.

> [Sculley et al., *Hidden Technical Debt in Machine Learning Systems*, NeurIPS 2015](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf)

The paper's first observation lands hard: in a real ML system, **the model code is a small fraction of the whole**; the rest is data pipelines, configuration, monitoring, serving and glue code.

The principle it introduces remains apt: **CACE — Changing Anything Changes Everything.** Change the distribution of one input feature and the weights and importance of the remaining features shift too; the same holds for hyperparameters, sampling methods and convergence thresholds. **No input is ever truly independent.**

The system-level anti-patterns it catalogues each map onto a class of operational trouble:

| Anti-pattern | Manifestation |
|---|---|
| Glue code | Prototype in one codebase, production in another; preprocessing diverges gradually |
| Pipeline jungles | A separate data pipeline per model version; duplicated logic, inconsistent results |
| Undeclared consumers | Another system quietly depends on your output structure and breaks silently when you change it |
| Hidden feedback loops | Model output shapes user behaviour, which generates future training data |
| Configuration debt | Settings scattered everywhere; nobody can confirm which parameters production is actually running |

**Why this debt is especially dangerous**: it compounds, and it accrues **silently**. Code debt has linters, review and tests. System-level debt has no equivalent mechanism — only deliberate design.

### 4.2 Train-serve skew

One of the most insidious and destructive problems in production: **the features computed at inference time are not the same features used in training.**

Common causes: training preprocessing written in Python and reimplemented in another language for serving; different missing-value handling on the two sides; different time-window definitions (a 7-day average in training, a rolling 24 hours in serving); normalisation statistics computed on training data while the production distribution has since shifted.

Its danger is that **it does not raise an error.** The system runs, the endpoint responds, and accuracy quietly falls.

Detection: log the model's actual inputs at serving time and periodically run statistical tests against the training input distribution (KS test, PSI and similar), alerting past a threshold. Prevention is more fundamental — **train and serve through a single feature-computation code path**, rather than maintaining two implementations.

### 4.3 Feedback loops reinforce themselves

A system-level risk Sculley et al. identify: when model output influences user behaviour, and user behaviour becomes future training data, you have a closed loop.

Recommendation systems are the canonical case: a category gets slightly more clicks, so it is shown more, generating more of that behaviour, so it is shown more still. **The system reinforces itself, and each individual round looks like "the data supports this direction".** There is no bug here — only a missing design consideration.

The response is not purely technical. It is to recognise at design time that **your training data has been contaminated by your own model** and cannot be treated as an unbiased observation of true preference.

### 4.4 Before optimising cost, break it down

A common mistake is seeing a large bill and reaching immediately for a cheaper model. But cost has several components, and which one dominates determines what to do:

| Component | Optimisation |
|---|---|
| Input tokens (including a repeated fixed prefix) | Prompt caching, leaner system prompts, compressing retrieved content |
| Output tokens | Cap output length, lower reasoning effort, avoid verbose formats |
| Number of calls | Fewer round trips, merged steps, code where a model is unnecessary |
| Concurrency and queuing | Serving-layer batching, capacity planning |

**Cache hit rate deserves its own monitoring.** Manual 1.1 covered the mechanism: a single differing token in the prefix invalidates everything after it. Putting a timestamp or session ID at the top of the system prompt takes the hit rate to zero — an extremely common mistake that shows up only on the bill, never as an error.

### 4.5 Routing: the highest-value single step

If you do only one thing to cut cost, it is usually **model routing** — sending simple requests to a small model.

> [Ong et al., *RouteLLM: Learning to Route LLMs with Preference Data*, LMSYS / UC Berkeley, arXiv:2406.18665](https://arxiv.org/abs/2406.18665)

The key design decision is training the router on **human preference data** rather than only judging "did the small model get it right". The difference matters: on the same problem both models may produce the correct answer, but the larger model's explanation is clearer and users actually prefer it — a correctness-only router would misclassify those requests as fine for the small model.

The reported results carry useful magnitude. On MT-Bench, routing every request to GPT-4 scores 9.2, while **sending only 50% of requests to GPT-4** scores 9.1 (BERT router with data augmentation) — essentially indistinguishable to users, at half the cost. The paper also introduces a practical metric, **CPT (call-performance threshold)**: their figures give CPT(50%) = 37%, meaning **only 37% of requests need the large model to capture 50% of the top-tier quality gain**.

The same experiments show the router learns the difficulty of the *problem*, so swapping the underlying model pair degrades performance only slightly (about 2–3 percentage points) without retraining. That matters for teams that will keep changing models.

### 4.6 Distillation: why soft targets carry more information

> [Hinton, Vinyals & Dean, *Distilling the Knowledge in a Neural Network*, 2015 — arXiv:1503.02531](https://arxiv.org/abs/1503.02531)

The core insight: **a large model's full probability distribution carries far more information than its argmax label.** When a strong classifier says "cat, 0.92", the remaining 0.08 spread across "lynx", "fox" and "dog" encodes the teacher's learned **similarity structure between classes**. Training a student to match those soft targets rather than just the hard label transfers that structure.

The mechanism is **temperature scaling**: divide both teacher and student logits by a temperature T (commonly 2–20) before the softmax, flattening the distribution so the small probabilities surface. The paper's MNIST result is worth remembering: teacher error 67, student improved from 146 to 74 — the student approaches the teacher. On speech recognition, distilling an ensemble of ten models into a single model lifted frame accuracy from 58.9% to 60.8%, close to the ensemble's 61.1%.

The engineering implication: distillation is the legitimate route from "expensive but good" to "cheap and nearly as good", but it presupposes a stable teacher — which is why it comes after routing and workflow simplification, not before.

### 4.7 Serving-layer efficiency gains are quantifiable

If you self-host or run your own inference service, the optimisation most worth knowing is KV cache memory management.

> [Kwon et al., *Efficient Memory Management for Large Language Model Serving with PagedAttention*, SOSP 2023（最佳论文）— arXiv:2309.06180](https://arxiv.org/abs/2309.06180)

The traditional approach pre-allocates a contiguous block of GPU memory per request, sized for the maximum possible output. But output length cannot be known in advance, producing three kinds of waste: reserved but unused, unfilled space inside a block, and fragmented free space that cannot be reassembled for a new request. What the paper measured: **of allocated KV cache memory, only 20.4%–38.2% actually held useful token states** — 60% to 80% was wasted.

PagedAttention borrows the operating-system idea of virtual memory: split the KV cache into fixed-size blocks, logically contiguous but physically scattered, with a block table mapping between them. The result is memory utilisation rising from 20%–40% to **over 96%**, and throughput improving **2–4×** on the same hardware.

The significance is not merely speed: **it moves the concurrency limit from "determined by model weights" to "determined by KV cache management"** — a systems problem, not a model problem. Related levers on the same layer include continuous batching, prefix caching and speculative decoding.

### 4.8 Setting thresholds for statistical evaluation

Evals in CI are statistics, so "it passed" needs a precise definition. Three points:

1. **Separate noise from signal.** With a small sample, a 1–2 point swing may be pure noise. Either enlarge the eval set or set a threshold above the noise floor.
2. **Report intervals, not points.** 0.83 versus 0.85 tells you nothing about whether the difference is real.
3. **Set per-slice thresholds.** An acceptable regression in the aggregate can mean one small slice is completely broken.

## 5. Capability checkpoints

1. I can explain why AI operations is distinctive because of unpredictability, cost and latency — not merely because "models make mistakes".
2. I can name the three signal types observability requires and the role each plays in diagnosis.
3. I can distinguish data drift, concept drift and upstream change, and give the typical signal for each.
4. I can explain why CI for AI systems needs statistical evaluation, and what calibrates the level of testing effort.
5. I can describe at least three release strategies with their use cases, and explain why model and prompt versions must be rollback-capable.
6. I can decompose cost into at least four components and name an optimisation direction for each.
7. I can restate the CACE principle and name at least three ML-specific technical-debt anti-patterns.
8. I can explain the causes of train-serve skew, why it raises no error, and how to detect and prevent it.
9. I can state the magnitude of gain available from model routing and its key design choice (preference data rather than correctness).
10. I can explain the problem PagedAttention solves and its quantified benefit.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 2*, 2026-08-21 | [link](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications) |
| Tier 1 — paper | Sculley et al., *Hidden Technical Debt in Machine Learning Systems*, NeurIPS 2015 | [paper](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf) |
| Tier 1 — paper | Ong et al., *RouteLLM: Learning to Route LLMs with Preference Data*, 2024 | [arXiv:2406.18665](https://arxiv.org/abs/2406.18665) |
| Tier 1 — paper | Hinton, Vinyals & Dean, *Distilling the Knowledge in a Neural Network*, 2015 | [arXiv:1503.02531](https://arxiv.org/abs/1503.02531) |
| Tier 1 — paper | Kwon et al., *Efficient Memory Management for Large Language Model Serving with PagedAttention*, SOSP 2023 | [arXiv:2309.06180](https://arxiv.org/abs/2309.06180) |
| Tier 1 — official | OpenTelemetry (cross-vendor standard for traces, metrics and logs) | [opentelemetry.io](https://opentelemetry.io/docs/) |
| Tier 1 — official | OpenAI, *Prompt caching* | [platform.openai.com](https://platform.openai.com/docs/guides/prompt-caching) |

> The detection techniques for train-serve skew (KS test, PSI) and the mechanics of shadow release are synthesised here from several secondary MLOps sources rather than a single original paper.
