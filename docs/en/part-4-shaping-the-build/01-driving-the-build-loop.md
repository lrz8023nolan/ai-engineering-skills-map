# 4.1 Driving the Build Loop

> **Part 4** · Shaping the build
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

Driving the build loop means **continuously deciding what to do next** — build a little, get a little feedback, and let that decide where to go — rather than working through a list fixed in advance.

It matters because software development *is* such a loop, and the loop's quality depends on **when and in which direction each turn is made**. Given the same requirement, one person turns five times in two weeks, each turn grounded in something real they just learned; another works heads-down for two months without validating anything. The gap between them is not typing speed.

Without this skill, two failures recur: **going a long way in the wrong direction** because nothing was checked along the way, and **using an artefact for the wrong purpose** — nursing a throwaway prototype as if it were a system you will maintain, or building to enterprise standards when all you needed was a prototype.

## 2. In the map

Ng puts this skill first within shaping the build, and defines it as a combination of **temperament and discipline**:

> "You have a bias for action, and drive this loop at the high velocity that AI has made possible."

The decisions he lists: you might build **a quick prototype** to test a technical concept or a user feature, build **an MVP** to take to users and demonstrate value, **add features**, or **invest in an enterprise-grade system**. You frequently **ship in small batches** to keep up velocity. You know when to get feedback from users or other stakeholders, and when to run a technical experiment (such as training a model) to gather information for the next step. Those decisions take account of **product vision, stage of the project, technical feasibility, key risks, effort and budget**. For more mature projects, you know how to **define key metrics and project-manage to drive improvements to them**.

Source: [AI Engineering Skills Map Part 4 — Shaping the Build, 2026-09-11](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/)

## 3. Core concepts

### 3.1 What the loop is actually doing

Three steps: **build something → get feedback → decide the next step.**

The point worth stating: AI has made the first step cheap, and the third no cheaper at all. So the bottleneck in this loop is almost always **judgement** — deciding what this round of feedback means, and whether it justifies changing direction. That is why "driving" is a skill rather than a by-product of the process.

### 3.2 Four artefacts, and the distinction that matters

Ng names four. What really separates them is not scale but whether the artefact exists **to learn from** or **to keep**:

| Artefact | Purpose | When to choose it | What to do with it afterwards |
|---|---|---|---|
| **Quick prototype** | Test a technical concept or a user feature | When uncertainty is highest | **Throw it away or rewrite it** — its job is done |
| **MVP** | Show real users the value | When you need to know whether users want it | May evolve, but accept that it is not solid |
| **Added features** | Make something already in use more useful | Direction already validated | Maintain long-term |
| **Enterprise-grade system** | Depended on by a real business | Validated, and failure is expensive | Maintain long-term, to the highest standard |

**The most expensive mistake is letting a prototype quietly become the production system.** Every shortcut taken to make the prototype fast — hard-coded values, skipped error handling, no tests, no permissions — becomes technical debt at that moment, and nobody ever made a decision to accept that cost. It simply was never deleted.

### 3.3 Small batches

In Ng's framing, shipping in small batches is **how you keep velocity**, but it is simultaneously a **mechanism for better decisions**: the smaller the batch, the less has to be abandoned when you turn.

This is the same finding as in [2.5 Scaling and Operating in Production](../part-2-software-fundamentals/05-scaling-and-operating-in-production.md) — deployment frequency, lead time, change failure rate and time to restore are **positively correlated**, so shipping more often goes with fewer incidents, not more. There it was about stability; here it is about **how cheaply the loop can be reversed**.

### 3.4 Two kinds of feedback, answering different questions

There are two ways to gather evidence, and choosing the wrong one wastes a cycle:

| Evidence | Question it answers | When to use it |
|---|---|---|
| **User / stakeholder feedback** | Is this **worth** doing? | Uncertainty is about whether users want it |
| **A technical experiment** (benchmark, model training run, spike) | **Can** this be done, and at what cost? | Uncertainty is about technical feasibility |

These two uncertainties cannot be resolved by the same instrument. Using user interviews to answer "can this be brought under 200 ms" and a load test to answer "do users like this interaction" are both evidence gathered in the wrong direction.

### 3.5 Six things one decision has to hold at once

Ng names them: **product vision, stage of the project, technical feasibility, key risks, effort and budget**. Their function is to stop any single dimension from dominating — particularly feasibility dominating ("it can be built, so build it") and effort dominating ("it is cheap, so build it").

### 3.6 Mature projects shift to metrics

Early on, the loop runs on qualitative judgement. At maturity, Ng's instruction is to **define key metrics and project-manage to drive them**.

That transition has a hidden precondition: **the metrics themselves have to have been validated first.** A badly defined metric makes the whole loop run efficiently in the wrong direction — and the more smoothly it runs, the worse the outcome. This is the same problem as "an aggregate metric does not tell you where you are wrong" in [1.4 Evaluation-Driven Development](../part-1-ai-applications/04-evaluation-driven-development.md).

## 4. Going deeper

### 4.1 An uncomfortable base rate

Confidence in driving the loop should rest on a premise: **your intuition about whether the next idea will work has a poor historical record.**

In Microsoft's experiment data, **only about one third of the ideas tested improved the metrics they were designed to improve**. In more heavily optimised domains the rate is worse — by Ron Kohavi's figures, roughly two thirds of ideas failed across Microsoft, about 85% failed at Bing, and Airbnb's failure rate reached 92%. The discipline he draws from this is to **accept that most of your ideas will fail**.

Source: [Kohavi, Tang & Xu, *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*, Cambridge University Press, 2020](https://doi.org/10.1017/9781108653985)

That base rate is the **mathematical justification** for driving the loop fast, not an argument against it. If a single idea succeeds about a third of the time, the expected value of your output is set mainly by **how many you can try per unit time**, not by how well you guess. Turned around, the real cost of a slow loop is not the delay — it is that **your sample is too small**: having tried three of ten ideas, you are not entitled to conclude anything about which class of direction is better.

### 4.2 A big redesign is not just slower, it is worse

One class of decision looks like "do it all at once and save effort," and the results come out the other way.

In Kohavi's research, **full redesigns often fail not only to hit their goals but even to reach parity with the old version on key metrics**. His recommendation is to break a redesign into **many small factors** and validate them separately.

This upgrades §3.3 from a speed tactic to a **decision-quality** one: the problem with a big redesign is not merely that feedback arrives late, but that **it bundles many different changes into a single outcome**, so that whether it succeeds or fails, you cannot learn **which part did the work**. That is the product-decision face of "an aggregate metric conceals offsetting changes" from [1.4 §4.1](../part-1-ai-applications/04-evaluation-driven-development.md).

### 4.3 When a result looks too good, suspect the data

The easiest mistake in this loop is not ignoring bad results. It is **believing good ones too quickly**. Kohavi states it as **Twyman's law**: **any figure that looks interesting or different is usually wrong.**

He is not making a point about statistical purity. One quantitative scale from the book: **sample ratio mismatch alone makes about 8% of experiments invalid** — you believed traffic was split 50/50 and it was not, so the comparison never held in the first place.

The engineering implication is direct: **when the loop produces a surprise, the correct next step is verification, not celebration.** After an unexpectedly large improvement, the most likely explanations in order are a broken instrumentation, a broken split, and a broken interpretation.

### 4.4 When to stop: not significant does not mean "good enough"

The other half of the loop is **knowing what not to do**. Kohavi's principle reduces to: **if the result is not significant, do not ship** — rather than "the effect is not visible but it does no harm, so ship it."

The reason is the same as §4.2: a change with no positive effect still **adds maintenance cost and makes the codebase more complex**, and those two costs are almost never counted when the ship decision is made.

### 4.5 Why this loop is harder to drive in the AI era, not easier

Put the threads together:

- AI made **building and verifying** cheap (the measurements in [3.1](../part-3-coding-agents/01-directing-the-workflow.md) show the saved time landing mainly in coding and debugging).
- But §4.1 says the bottleneck was never capacity — it was the quality of the ideas.
- So when capacity stops being the constraint, **judgement becomes the only constraint left**.

That is the other face of Ng's line about not having to wait for a PM to work out what to do: the vacated seat did not disappear, it **moved to you**. The front half of the loop (building) got accelerated; the back half (deciding the next step) was left entirely intact — which is precisely why 4.2, Product Decisions, exists.

## 5. Capability checkpoints

1. I can name the three steps of the build loop and say which one AI accelerates and which one it does not.
2. I can decide, for a given situation, whether to build a prototype, an MVP, added features or an enterprise-grade system, and justify the choice.
3. I can recognise the signs that a prototype is quietly becoming a production system, and say what decision has to be made explicitly at that point.
4. I can explain why small batches are simultaneously a speed tactic and a decision-quality one.
5. I can separate "do users want it" from "can it be built" uncertainty and choose the matching instrument for each.
6. I can list the six factors a loop decision has to weigh and identify which of them is most likely to be over-weighted.
7. I can explain why a one-third success rate **supports** rather than undermines driving the loop fast.
8. I can explain why a big redesign is "not just slower but worse", and connect it to aggregate metrics concealing offsetting changes.
9. I can state Twyman's law and say what to do when an unexpected good result appears.
10. I can explain why "not significant" defaults to not shipping, and name the two costs that get ignored.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Shaping the Build*, 2026-09-11 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/) |
| Tier 1 · Book | Kohavi, Tang & Xu, *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*, Cambridge University Press, 2020 | [doi.org](https://doi.org/10.1017/9781108653985) |
| Tier 1 · Practitioner | Forsgren, Humble & Kim, *Accelerate*, 2018 (DORA's four metrics are positively correlated; see [2.5](../part-2-software-fundamentals/05-scaling-and-operating-in-production.md)) | [dl.acm.org](https://dl.acm.org/doi/10.5555/3235404) |
