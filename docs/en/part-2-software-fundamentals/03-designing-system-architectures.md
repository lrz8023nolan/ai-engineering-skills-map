# 2.3 Designing System Architectures

> **Part 2** · Software engineering fundamentals
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Designing system architectures means deciding **what parts a system is made of, where the boundaries fall, which layer holds the state, and how large each piece should be**.

It is not drawing diagrams. It is a chain of trade-offs — and the premises behind those trade-offs change. When the user count changes, or the latency requirement, or the cost constraint, an architecture that was right can become wrong. So the core of the skill is not memorising a correct answer but **knowing what each choice trades away**, and **when to re-evaluate**.

Without it, two failure shapes recur. One is **introducing complexity before it is needed**: an internal tool split into eight microservices, with operational cost far exceeding business value. The other is subtler and more common — **treating architecture as something to sort out later**. Architecture is the layer hardest to adjust after the fact: data boundaries, service boundaries and state ownership become exponentially more expensive to move once large amounts of code depend on them. Ng notes that coding agents can help, but the tooling addresses "how fast can this be changed", not "what should it be changed to".

## 2. In the map

This is the third of the five sub-skills of Part 2. Ng first states its precondition — you have to understand the full stack and the data before you are in a position to judge how the pieces fit:

> "When you understand the major components of the full stack of software and data, you are then better positioned to decide how to put the pieces together."

His list of architectural decisions:

| Decision | Question to answer |
|---|---|
| Application platform | What does it run on? Self-managed, cloud instances, managed services? |
| Front-end / back-end boundary | Which logic lives on each side? |
| System decomposition | How many parts, and where do the boundaries fall? |
| State placement | Which layer holds state? |
| Architectural granularity | Monolith or microservices? |
| Stack | Languages, runtimes, frameworks, data technologies |

He stresses that all of this is premised on understanding **what the software is intended to do** — "how many users? how important is latency? how important is cost?" — before any choice is meaningful.

And the line that matters most in this section:

> "Further, the right architecture is a moving target, depending on the phase of the project."

Source: [AI Engineering Skills Map Part 3, 2026-08-28](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals)

## 3. Core concepts

### 3.1 Requirements first, architecture second

The first question in architecture is not technical — it is **what the software has to achieve**:

| Question | What it determines |
|---|---|
| How many users? What growth rate? | Whether to design for scale up front |
| How important is latency? | Caching layers, edge deployment, sync versus async |
| How cost-sensitive? | Self-managed versus managed, single machine versus distributed |
| How large is the team? | The upper bound on service granularity (see §4.2) |
| How strong must consistency be? | Whether eventual consistency is acceptable |
| What is the expected lifespan? | How much front-loaded design is justified |

**Without answers to these, any architecture debate is just a clash of preferences.**

### 3.2 Monolith and microservices

| | Monolith | Microservices |
|---|---|---|
| **Deployment** | One unit | Independently deployed |
| **Team** | One team collaborating | Each service can have an owning team |
| **Cross-module calls** | In-process function calls | Network calls (they fail, they have latency) |
| **Transactions** | A database transaction suffices | Distributed transactions or compensation |
| **Debugging** | One process, clear stack | Across services; needs distributed tracing |
| **Early speed** | Fast | Slow — infrastructure must come first |
| **Scaling** | Scale the whole thing | Scale the hot services |

**The critical thing to grasp: the cost of microservices is not "writing more services", it is replacing in-process calls with network calls.** The latter fail, have latency, and can partially succeed — which drags in retries, timeouts, idempotency, tracing and service discovery. That infrastructure is the hidden price.

### 3.3 What to decompose along

This is the hardest and most important step. Some common lines to draw along:

| Basis | Example | Problem |
|---|---|---|
| Technical layer | Front-end service / business service / data service | One requirement change crosses every layer, so coupling gets worse |
| Process step | Take order → process → notify | Data dependencies between steps force synchronous calls |
| **Decisions likely to change** | Encapsulate "how prices are computed" and "how users are stored" separately | Requires each module to absorb the impact of change (see §4.1) |
| Team boundary | Each team owns one end-to-end slice | Aligns with Conway's Law (§4.2) but may sacrifice technical merits |

### 3.4 Where state lives

| Location | Strengths | Weaknesses |
|---|---|---|
| **Database** | Durable, queryable, backed up | Every access costs latency |
| **Inside the service process** | Extremely fast | Cannot scale horizontally; lost on restart |
| **Distributed cache** | Fast and shareable | Invalidation problems (see [2.1 §4.2](01-building-full-stack-applications.md)) |
| **Client** | Consumes no server resources | Untrusted; tamperable |

**A practical rule: make services stateless wherever you can.** A stateless service can have instances added and removed at will, which is the precondition for horizontal scaling. Concentrating state in the database or cache costs one network hop.

### 3.5 How to choose a stack

Ng mentions a practice that is easy to overlook: **"sometimes by running experiments to evaluate options before settling on one."**

This deserves its own note. Stack choice is often treated as a matter of faith — which language is better, which framework is more advanced — when it can be **a measurable experiment**: take one genuinely critical scenario, implement it twice with the two candidates, and compare actual development and runtime behaviour. That beats any benchmark, because it reflects your real situation.

### 3.6 Architecture evolves with the phase

| Phase | Appropriate architecture | Why |
|---|---|---|
| **Prototype** | The simplest thing that runs, usually a single-machine monolith | The goal is proving feasibility, not carrying load |
| **First production system** | Add deployment, monitoring, backups, basic isolation | Real users now exist; availability matters |
| **Scale** | Split the hot spots, add caching and replicas, possibly split services | Bottlenecks have appeared and need targeted relief |

**An anti-pattern to watch for here: designing phase one for phase three.** Splitting before you know where the bottleneck is has certain cost and hypothetical benefit. But the reverse risk is equally real: **assuming "we can refactor later"** — if data boundaries are depended on by large amounts of code, refactoring may become prohibitively expensive (§4.5).

### 3.7 The eight fallacies of distributed computing

Assumptions that beginners take for granted and that are all false in a distributed system:

| Fallacy | Reality |
|---|---|
| The network is reliable | Packets drop, requests time out, calls partially succeed |
| Latency is zero | Cross-region round trips can be hundreds of milliseconds |
| Bandwidth is infinite | Large payloads become a bottleneck |
| The network is secure | Internal networks get breached too |
| Topology doesn't change | Nodes are added and removed constantly |
| There is one administrator | Many people change configuration concurrently |
| Transport cost is zero | Cross-region traffic is a real bill |
| The network is homogeneous | Different datacentres, clouds and protocols |

(This set originates with Waldo et al., *A Note on Distributed Computing*, 1994, and subsequent elaborations.)

## 4. Going deeper

### 4.1 Parnas 1972: decompose by information hiding, not by flowchart

This is the most important paper ever written on software architecture, and it fits in five pages:

> Parnas, *On the Criteria To Be Used in Decomposing Systems into Modules*, Communications of the ACM 15(12), 1972

He ran a deceptively ordinary demonstration: take a KWIC index generator (input some lines, output every circular shift, sorted) and **decompose it two different ways**.

| Decomposition | Basis | Modules |
|---|---|---|
| **One (flowchart)** | Processing steps | Input, Circular Shifter, Alphabetizer, Output, Master Control |
| **Two (information hiding)** | **Design decisions likely to change** | Line Storage (hides how lines are stored), Circular Shifter (hides how shifts are represented), and so on |

The two decompositions have **the same number of modules, similar names, and look equally clean on the surface**. The difference appears only under change:

- Change line storage from an array to a linked list: decomposition one touches almost every module (they all manipulate the shared array); decomposition two **changes only the Line Storage module**.
- Replace the sorting algorithm: decomposition one changes the sorter and everything that depends on how the sorted result is indexed; decomposition two changes one module.

Parnas's own words:

> "It is almost always incorrect to begin the decomposition of a system into modules on the basis of a flowchart."

**Why this remains counter-intuitive**: a flowchart describes the **order of execution**, and the order of execution is precisely not where change happens. People still habitually draw architecture along "request in → process → store → respond", which is exactly the decomposition Parnas showed to be fragile.

**How to use it**: before drawing a boundary, ask **"which part is most likely to change?"** Encapsulate that, so its change does not leak. A fifty-year-old KWIC generator and a modern microservice are structurally identical on this point: a service that owns its own database and refuses external direct access is running Parnas's experiment at network scale.

> One piece of follow-up worth knowing: Fred Brooks was sceptical of information hiding in the first edition of *The Mythical Man-Month*, and later publicly corrected himself — "Parnas was right, and I was wrong about information hiding."

### 4.2 Conway's Law: architecture mirrors the organisation

> "organisations which design systems are constrained to produce designs which are copies of the communication structures of these organisations."

> Conway, *How Do Committees Invent?*, Datamation 14(4), 1968

The implication: **architecture is not only a technical decision, it is also an organisational one.** If two modules must collaborate closely but the teams owning them barely talk, then no matter what the architecture diagram says, the interface between them will degrade into "throw it over the wall".

A practical corollary: **either align team boundaries with the architecture you want, or accept that the architecture will converge on your team boundaries.** This is also why "decompose services along team boundaries" is legitimate — it goes with Conway's Law rather than against it.

Conversely, if there is a clear technical reason to split something (say one part has entirely different compute characteristics) but the team structure is left unchanged, the split will most likely fail.

### 4.3 Microservices are not the default: a public reversal

Microservices are often assumed to be the more advanced choice, but published engineering cases show the direction can reverse.

In 2023 an AWS engineering blog post became widely discussed: after moving Prime Video's audio/video monitoring service **from microservices back to a monolith**, costs fell by roughly 90%.

What matters is not "monoliths are better" but **the reason they moved back**. That service's workload involved heavily coupled communication between many components. Once split, every orchestration step and every cross-service call amplified the volume of data moved. **For that access pattern, the abstraction bought nothing and cost a great deal.**

**The practical value of this case**: the benefit of microservices comes from "the parts have different scaling needs and change at different rates", and the cost is "in-process calls become network calls" (§3.2). If several components **always scale together and always have to change together**, splitting pays the cost without collecting the benefit.

### 4.4 The "distributed monolith": the worst of both

An anti-pattern worth memorising: **split into many services, but so tightly coupled that they must be deployed and modified together.**

The result is both sets of drawbacks at once:

| Source | Drawback inherited |
|---|---|
| Microservices | Unreliable network calls, distributed debugging, operational complexity |
| Monolith | Tight coupling, one change touching many places, no independent deployment |

**The test is simple**: if delivering one requirement means changing three services and deploying them together, you do not have microservices — you have "a monolith that is harder to deploy". This state is rarely designed deliberately; it emerges naturally after choosing the wrong decomposition basis (§3.3).

### 4.5 "We can refactor later" is a dangerous assumption

Ng says the right architecture is a **moving target**, and that hides a common misreading: if architecture changes anyway, build something quick now and adjust later.

The problem is that **the cost of architectural change is not linear — it rises systematically with the number of dependants**:

| Degree of dependence | Cost of change |
|---|---|
| Only code depends on it | Change code, test, release |
| Data boundaries are depended on | Requires data migration (see [2.2 §4.6](02-managing-data.md)) |
| The interface is depended on externally | Requires versioning, dual writes, gradual rollout |
| Many third parties depend on it | Effectively immutable |

**So the legitimate scope of "later" is limited to the parts that have no external dependants yet.** Once a boundary is published — an external API, written data, a message other services consume — it moves from "refactorable" to "irreversible".

The practical corollary: **where you are still uncertain, be conservative in whichever layer is hardest to change.** Concretely, data boundaries and public interfaces deserve extra thought; internal implementation, framework choice and deployment method can all be adjusted later, because the first group is irreversible and the second is not.

### 4.6 The eight fallacies of distributed computing

The table in §3.7 is worth using as a checklist, because **each fallacy corresponds to a class of bug that only appears in distributed settings**:

| Fallacy | The real failure it produces |
|---|---|
| Network is reliable | No timeout or retry logic; requests hang |
| Latency is zero | Synchronous calls chained, latency accumulating |
| Bandwidth is infinite | A large response body overwhelms downstream |
| Network is secure | Plaintext traffic on an internal network, intercepted |
| Topology doesn't change | Hard-coded IPs break after scaling |
| One administrator | Configuration conflicts |
| Transport cost is zero | Cross-region traffic bill runs away |
| Network is homogeneous | Assumed a cloud's behaviour; breaks on another |

**The value of these eight is that they explain why "works locally, breaks in production" happens.** On a single machine all eight hold, so single-machine testing cannot expose them — which is the premise behind the empirical result in §4.1 (most severe failures come from code paths never exercised).

### 4.7 Making architectural decisions traceable

The distinctive thing about architectural decisions is that **the reasoning evaporates while the conclusion remains.** Three months later nobody remembers why that database was chosen, so newcomers either follow it blindly or tear it down.

A non-ceremonial remedy is to record each decision with four fields: what was decided, what alternatives were considered, why this one, and **under what conditions it should be revisited**.

The value is not in the format but in that last field — it turns Ng's "moving target" into something operational: rather than re-evaluating when something feels wrong, the trigger conditions are written down in advance.

## 5. Capability checkpoints

1. I can explain why architecture must start from what the software has to achieve, and list at least four requirement dimensions that shape it.
2. I can compare monoliths and microservices on deployment, team structure, cross-module calls, transactions and debugging, and name the hidden cost of microservices.
3. I can describe the risks of decomposing by technical layer, by process step, by change point and by team boundary.
4. I can explain why services should be stateless wherever possible, and state the cost of concentrating state in a database or cache.
5. I can restate Parnas's conclusion on modular decomposition and explain why decomposing by flowchart is wrong.
6. I can use Conway's Law to explain why architecture is not purely technical, and state its practical corollary.
7. I can cite a published case where microservices proved worse than a monolith, and state the criterion for whether splitting pays off.
8. I can recognise a distributed monolith and explain why it inherits the drawbacks of both.
9. I can distinguish reversible from irreversible parts of an architecture and explain what that means for the order of decisions.
10. I can list the eight fallacies of distributed computing and explain how they account for "works locally, breaks in production".

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 3*, 2026-08-28 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals) |
| Tier 1 — paper | Parnas, *On the Criteria To Be Used in Decomposing Systems into Modules*, CACM 15(12), 1972 | [ACM](https://dl.acm.org/doi/10.1145/361598.361623) |
| Tier 1 — paper | Conway, *How Do Committees Invent?*, Datamation 14(4), 1968 | [melconway.com](http://www.melconway.com/Home/Committees_Paper.html) |
| Tier 1 — paper | Waldo, Wyant, Wollrath & Kendall, *A Note on Distributed Computing*, Sun Microsystems, 1994 | [PDF](https://scholar.harvard.edu/files/waldo/files/waldo-94.pdf) |
| Tier 1 — paper | Fox & Brewer, *Harvest, Yield, and Scalable Tolerant Systems*, HotOS 1999 | [DOI](https://doi.org/10.1109/HOTOS.1999.798396) |
| Tier 1 — official | Google, *Site Reliability Engineering* (free online) — release engineering, monitoring hierarchy, blameless postmortems | [sre.google](https://sre.google/sre-book/table-of-contents/) |
| Tier 1 — engineering case | AWS, *Scaling up the Prime Video audio/video monitoring service and reducing costs by 90%*, 2023 | [aws.amazon.com](https://aws.amazon.com/blogs/media/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90/) |
| Tier 2 — practitioner | Lewis & Fowler, *Microservices*, 2014 | [martinfowler.com](https://martinfowler.com/articles/microservices.html) |
| Tier 2 — practitioner | Fowler, *MonolithFirst* | [martinfowler.com](https://martinfowler.com/bliki/MonolithFirst.html) |

> Architecture Decision Records originate with Michael Nygard's 2011 article *Documenting Architecture Decisions*; cited here at second hand.
