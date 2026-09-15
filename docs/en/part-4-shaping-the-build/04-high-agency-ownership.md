# 4.4 High-Agency Ownership

> **Part 4** · Shaping the build
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

High-agency ownership means finding the work worth doing, ordering it, making it happen, and answering for the result.

It comes last in this map, and its premise is simple: in many organisations nobody at the top yet understands what AI can do, which means nobody can say which directions are worth funding. The people steering are not withholding direction. They do not have any to give. That gap needs filling, and the qualification for filling it is technical skill plus a willingness to push work forward without a precise instruction.

Without this skill, the usual result is putting yourself in a waiting position: waiting for requirements, for prioritisation, for schedule sign-off. By the time all three arrive, your work has been defined as execution. That is not a wrong place to stand, but it hands the decisive part of the work to someone else.

## 2. In the map

Ng first explains why the gap exists, then what it demands:

> "This creates an opening for someone with technical skill to bridge this gap: You can spot problems, propose solutions, and execute on them — being respectful of the organization's priorities and constraints, but without waiting for precise top-down direction."

Both halves matter. You act without waiting for precise top-down direction, and you respect the organisation's priorities and constraints.

He then lists four moves: identify opportunities, prioritise what matters, act on them, and own an initiative end to end. The requirements that go with them are taking accountability for problems that arise, acting in the face of ambiguity, persisting through setbacks, and measuring your work by the value you create rather than by task completion.

His closing paragraph on continuous learning belongs here rather than anywhere else: you track the frontier, pick up new tools, tune your workflows, and get better over time.

Source: [AI Engineering Skills Map Part 4 — Shaping the Build, 2026-09-11](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/)

## 3. Core concepts

### 3.1 How "respect the constraints" and "don't wait for instructions" both hold

The two sound contradictory. What separates them is the kind of thing you push forward.

| What you push forward | What it is | What follows |
|---|---|---|
| A working artefact, evidence it runs, a set of options | Filling the gap | Other people get something they can judge, and the decision stays theirs |
| Your own ordering of priorities, a release someone else owned, a commitment of their resources | Overstepping | It looks fast for a week and costs you their trust after that |

The test reduces to one question: did you produce the information and the evidence a decision needs, or did you make the decision? The first is agency. The second reads as overreach.

### 3.2 Acting under ambiguity

Ambiguity does not resolve by waiting for it to clear. The workable move is to cut a large uncertainty into one step that can be checked this week.

Product direction unclear, build a prototype three people can look at. Cost unclear, run one probe on the metered service. Whether users want it unclear, talk to three of them. What these share is that you get a result inside a week and, whether it lands well or badly, you know where to go next.

### 3.3 Measuring by the value you create

Ng's instruction to measure your work by the value you create has a concrete counter-example. Kohavi, writing about organisational metrics, describes how easily an organisation drifts toward measuring percent of plan delivered. That number is the easiest to collect: write a plan, execute the plan, declare success. Whether the feature moved any key metric never enters the reporting.

Anyone working to that metric gets one consistent signal, which is that shipping more items is better. The optimisation target becomes clearing the list rather than making something genuinely useful.

Three questions work better: did anyone use it after release, did it move a metric, and would anyone have noticed if it had never been built.

### 3.4 Why continuous learning sits inside this skill

Ng puts tracking the technology frontier here because it runs on the same action as everything else in this manual: going to look. Wait for someone to digest the new tools and practices for you and you are working from a second-hand conclusion with a delay attached, and in this field the delay is measured in months.

## 4. Going deeper

### 4.1 Agency pays, but not every form of it

A two-year longitudinal study tracked 180 full-time employees and their supervisors to test how a proactive personality leads to career outcomes.

The path is clear. Proactive personality measured at the start predicted innovation, political knowledge and career initiative two years later, and those in turn predicted salary growth, the number of promotions, and career satisfaction.

The same study produced a result running the other way. Voice, meaning raising suggestions and concerns, was negatively related to career progression. In one sample of people, making things and saying things did not pay the same, and the second went the wrong direction.

Source: [Seibert, Kraimer & Crant, *What Do Proactive People Do?*, Personnel Psychology 54(4), 2001](https://doi.org/10.1111/j.1744-6570.2001.tb00234.x)

This is not a reason to stay quiet. A more useful reading is that agency gets counted when it lands as something visible and assessable. Advice with no artefact behind it costs the receiver more to process than it returns.

### 4.2 Persistence is oversold

Ng writes about persisting through setbacks, which in popular writing maps onto the trait of grit. One meta-analysis is worth knowing about.

It pooled 88 independent samples, 584 effect sizes and 66,807 people, and found three things: grit's higher-order structure did not hold up, its correlation with performance and retention was only moderate, and its correlation with conscientiousness was very strong. In the authors' assessment, the construct validity of grit is itself in question.

The more useful part is the breakdown. Grit has two facets, perseverance of effort and consistency of interest. Perseverance carries significantly stronger criterion validity, and it explains variance in academic performance even after controlling for conscientiousness. Interventions aimed at raising grit generally look weak.

Source: [Credé, Tynan & Harms, *Much Ado About Grit*, Journal of Personality and Social Psychology 113(3), 2017](https://doi.org/10.1037/pspp0000102)

Against that, training persistence as a virtue does not hold up well. What does hold up is narrower: attend to the observable behaviour of sustained effort rather than to a self-assessment of how gritty you are.

### 4.3 Your bets on your own initiative have to match how reversible they are

[4.1](01-driving-the-build-loop.md) carries a set of numbers: about one third of ideas improved the metric they targeted at Microsoft, roughly 85% failed at Bing, and 92% failed at Airbnb.

The consequence for agency is direct. If most judgements are wrong, what you produce over time depends on how many attempts you get and what each one costs. So point your initiative at reversible moves: prototypes that can be deleted, features behind a switch that can be turned off, proposals that can be withdrawn. One-way decisions, meaning data migrations, public commitments, and spending someone else's budget, should get agreement first even when you are confident.

Source: [Kohavi, Tang & Xu, *Trustworthy Online Controlled Experiments*, Cambridge University Press, 2020](https://doi.org/10.1017/9781108653985)

## 5. Capability checkpoints

1. I can state the test that separates acting without instructions from overstepping, and classify a specific action.
2. I can cut a vague direction into a step that produces a result this week and tells me where to go next either way.
3. I can test a piece of work with three questions: did anyone use it, did it move a metric, would anyone notice its absence.
4. I can explain the distortion that measuring percent of plan delivered produces.
5. I can describe the path from proactive personality to career outcomes in that two-year study.
6. I can explain why voice came out negatively related to career progression, and give a reading that does not collapse into "stop speaking up".
7. I can report the three findings of the grit meta-analysis and identify which facet still holds up.
8. I can explain why initiative should go to reversible moves first, and which decisions count as one-way.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Shaping the Build*, 2026-09-11 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/) |
| Tier 1 · Paper | Seibert, Kraimer & Crant, *What Do Proactive People Do? A Longitudinal Model Linking Proactive Personality and Career Success*, Personnel Psychology 54(4), 2001 | [doi.org](https://doi.org/10.1111/j.1744-6570.2001.tb00234.x) |
| Tier 1 · Paper | Credé, Tynan & Harms, *Much Ado About Grit: A Meta-Analytic Synthesis of the Grit Literature*, Journal of Personality and Social Psychology 113(3), 2017 | [doi.org](https://doi.org/10.1037/pspp0000102) |
| Tier 1 · Book | Kohavi, Tang & Xu, *Trustworthy Online Controlled Experiments*, Cambridge University Press, 2020 | [doi.org](https://doi.org/10.1017/9781108653985) |
