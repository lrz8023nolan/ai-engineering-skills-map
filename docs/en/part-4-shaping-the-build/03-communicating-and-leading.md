# 4.3 Communicating and Leading

> **Part 4** · Shaping the build
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

Communicating and leading means translating technical judgement into language other functions can use, and keeping a project moving through cross-functional work.

AI engineering skills widen the range of work one person can hold. You can carry something from requirements through implementation to release while also dealing with marketing, finance, legal and users, and all of those functions affect whether the project proceeds. They do not need to read your code. They need one judgement from you: can this be done, roughly how long will it take, and if it cannot, what is blocking it. Without that judgement the project stops at the point where someone has to nod.

The failure shows up in two ways from the same place. You explain, but you explain the implementation, so your listener never gets the judgement they came for and the meeting schedules another meeting. Or you do not explain, and someone forms a view of AI from headlines and makes the call for you.

## 2. In the map

Ng splits the skill into cross-functional collaboration and explaining the technology to the organisation.

> "Your technical skill in AI Engineering puts you ahead of the game and allows you to play a unique role in shaping these perspectives."

On the first half, he writes that you may participate in other functions affecting your project, including marketing, finance and legal, which makes "your ability to communicate with these other functions more important than before" and lets you move the project forward by aligning and coordinating among stakeholders. He also notes that communication is the foundation for the conversations with users that build user empathy, which [4.2](02-making-product-decisions.md) covers.

On the second half, his reason is that the technology is changing fast and many people outside engineering are trying to work out what it does, what it means for their jobs, and what new practices and products it makes possible. His example: you can explain why a given initiative is or is not technically feasible, and that lets you help lead the wider organisation.

Source: [AI Engineering Skills Map Part 4 — Shaping the Build, 2026-09-11](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/)

## 3. Core concepts

### 3.1 Different functions need different judgements

Explaining one thing to five audiences is not a matter of tone. What changes is where the judgement lands.

| Audience | The judgement they need | Useful to say | A waste of their time |
|---|---|---|---|
| Marketing | Can we build it, and when can we talk about it publicly | How complete it is, when it can be demoed, what is unresolved | Architecture choices |
| Finance | What it costs, when the money is spent, what it saves | Cost structure, where per-unit and fixed costs separate | Model parameters |
| Legal | Where the data comes from, whether it can leave, who is liable | Data flows, retention, third-party dependencies | Training methods |
| Users | Whether it solves their problem | Their problem, what they use instead today | Every internal term |
| Leadership | Whether to keep funding it, and where the risk sits | What is validated, what is not, what the next validation costs | Technical detail as achievement |

### 3.2 Answering "can this be done"

A useful answer has four parts: what can be done, what cannot, what you base that on, and what happens if you are wrong.

The second part is the one that gets dropped. Say only what can be done and your listener hears "anything can be done", then runs into an unstated limit somewhere else. Putting the boundary first keeps later expectations in place. The experiments below show that this boundary is invisible not only to outsiders but to the people using the tool.

### 3.3 Leading means coordinating, not deciding for people

Ng's phrase is aligning and coordinating among stakeholders. In practice that is three things: getting everyone onto one definition of success, surfacing cross-functional dependencies and blockers, and turning conflicts between technical feasibility and business demands into options rather than refusals.

The third is where leadership most often collapses into a veto. "That can't be done" ends the discussion. "This way takes three months, that way takes two weeks and costs you X" hands the decision back and keeps the project moving.

## 4. Going deeper

### 4.1 The same people, 19 percentage points apart

In 2023 Harvard Business School, MIT, Wharton and BCG ran a preregistered randomised experiment with 758 BCG consultants across 18 simulated consulting tasks. Three groups: no AI, GPT-4, and GPT-4 plus a prompt engineering overview.

On tasks inside GPT-4's capability, the AI groups completed 12.2% more tasks, worked 25.1% faster, and produced work rated over 30% higher in quality.

On a task deliberately built to sit outside that capability, the AI groups were 19 percentage points less likely to answer correctly: the control group was right about 84.5% of the time, the AI groups landed between 60% and 70%.

The researchers recorded one more detail. When the AI produced a wrong recommendation, that wrong recommendation still scored higher on clarity and coherence than the correct human answers. The error did not arrive looking like an error. It arrived looking like the strongest material in the room.

Source: [Dell'Acqua et al., *Navigating the Jagged Technological Frontier*, Organization Science 37(2), 2026](https://doi.org/10.1287/orsc.2025.21838)

### 4.2 The boundary is invisible, so explaining it is scarce work

The paper calls this boundary the jagged technological frontier. Two tasks that look equally hard to a person can sit on opposite sides of it, and the boundary is jagged rather than smooth: one step forward keeps you inside, one step sideways drops you off.

The consultants in the experiment could not tell which side they were on. That is where this skill sits. You can judge which side a given task falls on and the people you work with cannot. Being the person who states that clearly is one of the few things here that nobody else on the team is positioned to supply.

The same experiment produced a number that changes how you pitch training. Split by baseline performance, the bottom half improved about 43% with AI and the top half about 17%. The gains are not evenly spread, and most of the increase lands on people who were weaker to begin with. That tells you where a training budget does the most work.

### 4.3 Your audience's confidence does their judging for them

Microsoft Research and Carnegie Mellon surveyed 319 knowledge workers about 936 real uses of generative AI at work (CHI 2025).

The result runs in two directions. Higher confidence in the AI predicted less critical thinking. Higher confidence in their own expertise predicted more. The workers who rated themselves least able to judge the AI's output were the most likely to accept it.

The study also found that the effort moves rather than disappears. Less time gathering information and producing a first draft, more time verifying the output, integrating it, and deciding whether it is good enough to ship.

For anyone explaining this technology, the practical consequence is that confidence substitutes for judgement, and confidence tracks the tool's reputation rather than its reliability on the specific task. A feasibility note therefore has to say more than where the boundary sits. It has to say which parts can be taken at face value and which parts have to be checked.

Source: [Lee, Sarkar, Tankelevitch et al., *The Impact of Generative AI on Critical Thinking*, CHI 2025](https://doi.org/10.1145/3706598.3713778)

### 4.4 One unsettled piece of evidence, and its rebuttal

MIT Media Lab posted a preprint in June 2025. Sixty students from five Boston universities wrote three timed SAT-style essays over four months in one of three conditions: GPT-4o, a search engine, or no internet. EEG was recorded throughout.

The no-tools group showed the strongest and most widespread connectivity between brain regions, the search group sat in the middle, and the LLM group showed the least. Asked afterwards, 83% of the LLM group could not accurately quote a sentence from the essay they had submitted minutes earlier, against 11% in the other two groups. The authors call the accumulating pattern cognitive debt.

The rebuttal is worth reading alongside it. Vitomir Kovanovic and Rebecca Marrone, at the University of South Australia, argue that the fourth-round result, where switching from AI back to writing unaided produced worse performance, is explained by a familiarity effect: the no-tools group had three rounds of practice and the switching group had one, so the two were not matched on task experience. On that reading, the difference comes from the design rather than from the model.

Kosmyna states her own limits: the study did not show the brain becoming "dumb" or "on vacation", and the paper, in her words, has no answers.

This sits here because explaining AI's impact to an organisation means being asked what happens to people's abilities. The honest answer today is that nobody knows. The evidence supports a direction, that judgement handed over repeatedly weakens, and nothing about magnitude. Settling magnitude needs matched practice across rounds, larger samples, and repetition across task types.

Source: [Kosmyna et al., *Your Brain on ChatGPT*, MIT Media Lab, 2025 (preprint)](https://arxiv.org/abs/2506.08872)

## 5. Capability checkpoints

1. I can name the judgement each of five audiences actually needs: marketing, finance, legal, users, leadership.
2. I can answer "can this be done" in four parts: what can, what cannot, the basis, and what happens if I am wrong.
3. I can restate a technical limit as a trade-off with options instead of a refusal.
4. I can explain what the jagged frontier is and why the people using the tool cannot tell which side they are on.
5. I can report both directions of the BCG experiment and explain why a wrong recommendation was harder to spot.
6. I can explain why the gains are unevenly spread and what that implies for where training money goes.
7. I can describe how an audience's confidence in AI and in itself each change how much they check.
8. I can state the evidence level of the EEG study and which conclusions it does and does not support.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Shaping the Build*, 2026-09-11 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/) |
| Tier 1 · Paper | Dell'Acqua et al., *Navigating the Jagged Technological Frontier*, Organization Science 37(2), 2026 | [doi.org](https://doi.org/10.1287/orsc.2025.21838) |
| Tier 1 · Paper | Lee, Sarkar, Tankelevitch et al., *The Impact of Generative AI on Critical Thinking*, CHI 2025 | [doi.org](https://doi.org/10.1145/3706598.3713778) |
| Tier 1 · Preprint | Kosmyna et al., *Your Brain on ChatGPT*, MIT Media Lab, 2025 | [arXiv:2506.08872](https://arxiv.org/abs/2506.08872) |
