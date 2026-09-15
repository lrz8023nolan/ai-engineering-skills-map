# 4.2 Making Product Decisions

> **Part 4** · Shaping the build
> **Status** · Complete
> **Last updated** · 2026-09-15

---

## 1. Overview

Making product decisions means **deciding the things the product spec does not cover** — including, when there is no spec, writing it.

It matters because the role of "implement what was specified" is disappearing. The old division of labour had product managers decide what to build and developers build it; now one person can carry a project end to end, so **the calls that used to be made by someone else land on whoever is doing the work**. Those calls include what the feature should look like, whether this path is the right one, whether it is worth funding — and the one most easily skipped: **whose problem are we actually solving.**

Without this skill the failure is usually not "cannot build it." It is **building something nobody wants.** And that failure reports back very slowly: the code runs, the tests pass, the interface is not ugly. The only thing wrong is that the problem it solves does not matter much to users.

## 2. In the map

Ng draws the boundary first, so this is not read as "developers should become PMs":

> "Developers don't have to become PMs, but you will make decisions the product spec doesn't cover. If you are asked to build without a spec, you know how to develop one."

He splits the skill into four, and names one of them as the root:

| Capability | What it covers |
|---|---|
| **Product sense** | Picking a direction that meets real user needs, without waiting for a PM to make every decision |
| **Design sense** | At least basic; building things that are not merely functional but pleasing to use |
| **Business sense** | Thinking through go-to-market, market size, unit economics and P&L, and making economically sensible trade-offs |
| **User empathy** | **The first three are rooted here**, and it is honed continuously by a range of methods |

The methods he lists span a wide cost range, from **quick informal interviews with 2–3 users** to **surveys of hundreds**, **large-scale A/B tests**, and **analysing the behaviour of thousands or millions of users**.

Source: [AI Engineering Skills Map Part 4 — Shaping the Build, 2026-09-11](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/)

## 3. Core concepts

### 3.1 With no spec, four things must be written down

Writing a spec does not require a full methodology. When one is missing, these four items prevent most disasters:

| What to write | The failure it prevents |
|---|---|
| **Who the user is** (concretely enough to list them) | "Everyone needs this" — which means no user |
| **What problem is being solved** (how they cope today) | Solving a problem that does not exist or does not matter |
| **How success is measured** (one observable metric) | Finishing without being able to say whether it helped |
| **What is explicitly out of scope** | Scope creep, and nursing a prototype as a production system |

The fourth is omitted most often and works best. With no upper bound on scope, any amount of output can be explained as "not finished yet."

### 3.2 Four ways to gather evidence, and where each is misused

Ng's methods run from light to heavy, and their costs differ by orders of magnitude. **Choosing wrong costs you either way: a heavy instrument used for a small question, or a light one used for a question that needs statistical power.**

| Method | Question it answers | Where it gets misused |
|---|---|---|
| **Informal interview with 2–3 users** | Is there a real problem, and how do they cope today? | Taking politeness for validation (see §4.1) |
| **Survey of hundreds** | How widespread is the problem, and in whom? | Asking "would you buy it" — the answer is not a behavioural prediction |
| **Large-scale A/B test** | Did this change **actually** move the metric? | Concluding from too small a sample; not suspecting a result that looks too good (see §4.2) |
| **Behavioural data analysis** | What users **actually** did | Using behavioural data to answer "why", which it cannot |

### 3.3 The minimum business vocabulary

You do not need to be a finance person, but you should be able to hold up your end:

| Concept | The question it asks |
|---|---|
| **Go-to-market** | How will the target users find out about this and start using it? |
| **Market size** | How many people are worth serving, and how many can you reach? |
| **Unit economics** | What does serving one user earn, and what does it cost? |
| **P&L** | Combining the above, does this make or lose money overall? |

Their shared function is to **put an upper bound on technical judgement**: a solution that is elegant technically and liked by users may not work out per unit.

### 3.4 Three minimum handles on design sense

"Pleasing to use" is not a matter of taste; it has operational criteria. The three most durable from classical human-computer interaction:

- **Discoverability**: without a manual, can a user tell what is possible here?
- **Feedback**: after each action, can the user tell what happened?
- **Constraints**: can the wrong action be made impossible, rather than reported after the fact?

### 3.5 How this manual relates to 4.1

[4.1](01-driving-the-build-loop.md) is the **rhythm** of the loop; 4.2 is the **basis for each decision inside it**. Without 4.2, 4.1 becomes a machine that produces the wrong direction efficiently.

## 4. Going deeper

### 4.1 Why asking users yields no real signal by default

This is the most counterintuitive and most practically useful section in the manual.

Rob Fitzpatrick diagnoses the problem precisely in *The Mom Test*: **it is not dishonesty, it is politeness.** People do not want to hurt your feelings, so they say the idea sounds interesting, that they would probably use it, that they would definitely recommend it to a friend — and none of that carries information. He calls these answers **fluff**, in three shapes:

| Shape | Phrasing |
|---|---|
| **Generic claims** | "I usually…", "I always…", "I never…" |
| **Future-tense promises** | "I would…", "I will…" |
| **Hypothetical maybes** | "I might…", "I could…" |

His three rules move the conversation off them:

1. **Talk about their life instead of your idea.**
2. **Ask about specifics in the past**, not generics or opinions about the future.
3. **Talk less and listen more.**

There are two mechanistic reasons, and the second is usually missed. The first is a **division of ownership**: you are not allowed to tell them what their problem is, and in exchange they are not allowed to tell you what to build — the problem belongs to them, the solution to you. The second is that **people cannot predict their own future behaviour**. The book's example is telling: ask people how they like their coffee and many say "strong"; watch what they actually order and it is weak and milky. That is not lying; it is the gap between the ideal self and the actual self. **So the only signal that comes close to trustworthy is commitment** — willingness to pay money, give up time, or make an introduction.

Source: [Fitzpatrick, *The Mom Test*, 2013](https://www.momtestbook.com/)

This is an important supplement to Ng's "quick informal interviews with 2–3 users": **the method is sufficient, provided the questions are right.** Otherwise what it produces is not data — it is comfort.

### 4.2 "Statistically significant" is not the same as actionable

The previous section was about **collection**; this one is about **interpretation**.

Start with the base rate: as in [4.1 §4.1](01-driving-the-build-loop.md), a single idea's success rate is low (about one third at Microsoft, ~85% failure at Bing, ~92% at Airbnb). That base rate has a statistical consequence that gets overlooked:

**When the true success rate is low, false positives make up a much larger share of "significant" results than your significance level suggests.** By Kohavi's numbers, in an environment like Airbnb's with an 8% success rate, a result significant at p < 0.05 **still has roughly a 26% chance of being a false positive**.

This is the thing to remember here: **a p-value is computed under the null hypothesis of zero true effect. It is not the probability that the result is trustworthy.** Those differ, and a low base rate widens the gap.

Two disciplines go with it:

- **Twyman's law: if a figure looks interesting or different, it is usually wrong.** When a surprise appears, check instrumentation and traffic split first.
- **Novelty effect**: an initial lift fades. Run the experiment long enough, and do not mistake freshness for effect.

Source: [Kohavi, Tang & Xu, *Trustworthy Online Controlled Experiments*, Cambridge University Press, 2020](https://doi.org/10.1017/9781108653985)

**What this means for product sense.** If success rates are in the single digits or around a third, then "strong product sense" does not mean "guesses right." It means two things: **producing a steady supply of cheap, verifiable bets**, and **not rushing to interpret a result as having been right**.

### 4.3 AI has swapped the interface paradigm, which changes what design sense is applied to

A claim often dismissed as marketing that nonetheless has structural content.

Jakob Nielsen divides computing history into three interface paradigms: **batch processing** (the user submits a complete workflow up front), **command-based interaction** (one command at a time, back and forth — dominant for over sixty years), and the arriving third, **intent-based outcome specification**.

His phrasing:

> "With the new AI systems, the user no longer tells the computer what to do. Rather, the user tells the computer what outcome they want."

The key is that **the locus of control is reversed**. In command-based interaction the user decomposes intent into steps and the system executes; in intent-based interaction the user supplies only the intent and the decomposition is taken on by the other side. So what design has to align with has changed: it used to be **controls and flows**, and it is now **helping a user narrow a vague intent into something executable**, and **making the credibility of the result visible**.

The three handles in §3.4 have not expired, but their carriers have changed: discoverability becomes "does the user know what can be asked for here"; feedback becomes "how was this result arrived at, and can it be checked"; constraints become "making the wrong thing unrequestable at the level of intent."

Nielsen adds two judgements of his own, worth knowing but not to be treated as findings. One is that **prompt engineering will not be a long-lasting career**. The other is his estimate that **a large share of people in rich countries cannot write the prose needed to get good results from current AI systems** — that is his judgement rather than a measurement, but it points at a design constraint: **requiring users to write a good paragraph is itself a usability barrier**.

Source: [Nielsen, *AI: First New UI Paradigm in 60 Years*, Nielsen Norman Group, 2023](https://www.nngroup.com/articles/ai-paradigm)

### 4.4 The other half of design sense: the fault is in the design, not the person

The most useful move in Don Norman's classic is **reversing the direction of blame**: when users repeatedly get something wrong, the problem is with the object, not the person. If a device needs a manual, a warning label or a sticker beside it in order to be operated correctly, **the design has already failed**.

He also draws a distinction that is especially useful for AI interfaces: **an affordance is what an object actually makes possible; a signifier is the perceivable cue telling you where and how to act.** Most usability incidents are failures of signifiers, not affordances — a door genuinely affords pushing, but if it carries a handle that signifies pulling, people will pull.

The engineering implication: **do not add a warning after the error happens; first ask why the wrong action was possible, or even attractive.** For AI products this matters more than it used to, because in an intent-based interface the user supplies a sentence rather than a sequence of explicit commands — **there is more room to be misread, and misreadings usually raise no error at all.**

Source: [Norman, *The Design of Everyday Things*, Revised and Expanded Edition, 2013](https://jnd.org/books/the-design-of-everyday-things-revised-and-expanded-edition/)

## 5. Capability checkpoints

1. I can say which capability product decisions are rooted in, and explain why this is not "becoming a PM early".
2. I can write a minimum spec without one being given, covering user, problem, success criterion and explicit non-goals.
3. I can choose a suitable evidence-gathering method for a specific question and say why I did not choose a lighter or heavier one.
4. I can spot fluff in interview notes (generic claims, future-tense promises, hypothetical maybes) and rewrite it into questions anchored on specific past behaviour.
5. I can explain why commitment is the only close-to-trustworthy signal, and give an example.
6. I can explain why, in a low-success-rate environment, a result at p < 0.05 still carries a substantial false-positive risk.
7. I can say how Twyman's law and the novelty effect each distort a product decision.
8. I can name the four business concepts I should be able to discuss without being a PM, and explain how they bound technical judgement.
9. I can review an interface against discoverability, feedback and constraints, and say which is weakest.
10. I can explain why intent-based interfaces make "the fault is in the design, not the user" more critical, not less.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 4 — Shaping the Build*, 2026-09-11 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-shaping-the-build/) |
| Tier 1 · Book | Fitzpatrick, *The Mom Test: How to talk to customers & learn if your business is a good idea when everyone is lying to you*, 2013 | [momtestbook.com](https://www.momtestbook.com/) |
| Tier 1 · Book | Kohavi, Tang & Xu, *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*, Cambridge University Press, 2020 | [doi.org](https://doi.org/10.1017/9781108653985) |
| Tier 1 · Book | Norman, *The Design of Everyday Things*, Revised and Expanded Edition, 2013 | [jnd.org](https://jnd.org/books/the-design-of-everyday-things-revised-and-expanded-edition/) |
| Tier 1 · Practitioner | Nielsen, *AI: First New UI Paradigm in 60 Years*, Nielsen Norman Group, 2023 | [nngroup.com](https://www.nngroup.com/articles/ai-paradigm) |
