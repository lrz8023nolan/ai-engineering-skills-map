# 2.5 Scaling and Operating in Production

> **Part 2** · Software engineering fundamentals
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Scaling and operating in production means **actually delivering software to users and keeping it available as load grows** — covering deployment, release, observability, capacity, and keeping the codebase maintainable years later.

**First, how this differs from [1.5 Operating in Production](../part-1-ai-applications/05-operating-in-production.md)**, since the two are easy to confuse:

| | 1.5 | 2.5 (this manual) |
|---|---|---|
| Perspective | Operations **specific to AI systems** | Operations for **general software engineering** |
| Core problems | Unpredictable output, cost scaling with usage, quality drift | Deployment, release, carrying load, long-term maintenance |
| Typical content | Evals in CI, drift detection, model routing, inference cost | SDLC, release strategy, error budgets, capacity, code review, dependency maintenance |

They are concentric: this manual is the outer ring (applies to any software), 1.5 is the inner ring (the AI-specific increment). Anything already covered there is cross-referenced rather than repeated.

Without the skill, the typical outcome is **a system that ships but cannot carry load** — not because any particular piece of code is bad, but because delivery and operations were never treated as part of the design. The symptoms: a release requires downtime; failures are discovered by users rather than by instrumentation; traffic goes up and the only response is adding machines; or three years later nobody dares change the code because nobody knows what will break.

## 2. In the map

This is the last of the five sub-skills of Part 2. Ng splits it into three parts.

**Delivery.** Knowing how to execute the software development lifecycle (SDLC) — which, beyond building and testing, includes **configuring the deployment environment, deciding a release strategy, applying deployment automation (CI/CD), and understanding infrastructure as a service (IaaS)**.

**Operations.** Putting observability tools in place, setting alerts, and managing incidents.

**Scaling**, which is where he is most specific about the technique:

> "to scale your application, you should understand the real load and know how to scale servers, load-balance, and adapt your data infrastructure (via sharding, indexing, replication) or make architecture changes to allow your system to adapt to scale."

Finally he names a set of long-term practices that are easily dismissed as process trivia: **version control, code review, dependency maintenance, and managing technical debt** — which "help you keep evolving your system over time".

Source: [AI Engineering Skills Map Part 3, 2026-08-28](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals)

## 3. Core concepts

### 3.1 The full scope of the SDLC

| Phase | What it covers |
|---|---|
| Plan | Requirements, technical approach, risk assessment |
| Build | Coding, code review |
| Test | Unit, integration, end-to-end (see [2.4 §3.2](04-making-systems-secure-and-reliable.md)) |
| Configure environments | Managing dev/staging/production differences; separating configuration from code |
| Release | Release strategy, rollback plan, change approval |
| Automate | CI/CD pipelines |
| Operate | Monitoring, alerting, on-call, incident response |

**The most overlooked phase is configuring the deployment environment.** When something runs in development but not in production, the cause is almost always environmental difference — dependency versions, environment variables, network policy, permissions. **Configuration should be separated from code and version-controlled**, so that "which configuration is production actually running" is a question with an answer.

### 3.2 Release strategy

Covered in [2.1 §3.5](01-building-full-stack-applications.md); the connection to this manual is that **release strategy determines the blast radius of a failure.** The value of a canary release is not just caution — it bounds the impact of a bad release to a fraction of traffic. That is the same idea as blast radius from [2.4 §4.6](04-making-systems-secure-and-reliable.md), operating along the time axis rather than the user axis.

### 3.3 Observability, alerting and incident management

The technical detail lives in [1.5 §3.1–3.2](../part-1-ai-applications/05-operating-in-production.md). This manual adds only the two principles that matter most from a general software perspective: error budgets (§4.2) and alerting on symptoms (§4.4).

### 3.4 Measure the real load before scaling

Ng's "understand the real load" deserves emphasis. The first class of scaling mistake is **measuring the wrong thing**:

| What to measure | Why |
|---|---|
| Arrival rate and peak shape | Averages hide peaks, and capacity is sized for peaks |
| Distribution of cost per request | Not all requests are equal; a few heavy ones can dominate |
| Latency tails (p95/p99) | Averages hide poor tail experience |
| Where the bottleneck actually is | Capacity added in the wrong place is pure waste |
| What drives the growth | User count, data volume, or one particular feature? |

**That last row is the crucial one.** Each driver implies a completely different scaling approach: user growth means more instances, data growth means sharding, one feature getting heavier means optimising that feature. **Without knowing which, scaling is guesswork.**

### 3.5 Load balancing and horizontal scaling

Horizontal scaling requires that **instances be interchangeable**, which in turn requires stateless services or externalised state (see [2.3 §3.4](03-designing-system-architectures.md)).

| Mechanism | Purpose |
|---|---|
| Load balancing algorithm | Round robin, least connections, consistent hashing (pin a user or key to an instance) |
| Health checks | Automatically remove broken instances from traffic |
| Graceful shutdown | Finish in-flight requests before an instance exits |
| Autoscaling | Add and remove instances from metrics |
| Rate limiting and queues | Protect yourself from traffic spikes |

### 3.6 Scaling the data layer

The data layer is far harder to scale than stateless services, because data cannot be freely copied or discarded. Three basic levers (detail in [2.2](02-managing-data.md)):

| Lever | Solves | Costs |
|---|---|---|
| **Indexing** | Slow queries | Slower writes, extra storage |
| **Replication** | Read scaling, disaster recovery | Replication lag means stale reads |
| **Sharding** | A single machine cannot hold it; write scaling | Cross-shard queries get complex; resharding is heavy work |

**The usual order is: index first, then add read replicas, and shard last.** Sharding is the most expensive and hardest to reverse of the three, and should only be reached once the first two are exhausted.

### 3.7 Keeping a codebase maintainable

The set of practices Ng names is often dismissed as process trivia, but these determine whether the system can still be changed three years from now:

| Practice | What it solves |
|---|---|
| Version control | Rollback, traceability, parallel work |
| Code review | Catches problems before merge; also the main channel for knowledge transfer |
| Dependency maintenance | Dependencies rot: vulnerabilities, breaking changes, abandonment |
| Managing technical debt | Debt compounds (see [1.5 §4.1](../part-1-ai-applications/05-operating-in-production.md)) |

**Dependency maintenance deserves its own mention**: dependencies are **the one category of thing that delivers you new risk without you changing anything** — your code is untouched, and a library you depend on is found to have a vulnerability. "Updating dependencies" is therefore not an optional chore but an ongoing maintenance cost.

## 4. Going deeper

### 4.1 Speed and stability are not a trade-off — they reinforce each other

This is the most counter-intuitive finding in the manual, and it rests on large-scale empirical evidence.

> [Forsgren, Humble & Kim, *Accelerate: The Science of Lean Software and DevOps*, 2018](https://dl.acm.org/doi/10.5555/3235404) (based on years of DORA research)

DORA measures delivery performance with four metrics:

| Metric | Meaning |
|---|---|
| **Deployment frequency** | How often you deploy |
| **Lead time for changes** | Commit to running in production |
| **Change failure rate** | What proportion of deployments cause a problem needing a fix |
| **Time to restore (MTTR)** | How long recovery takes after a failure |

The gap between elite and low performers is concrete:

| | Elite | Low |
|---|---|---|
| Deployment frequency | On demand, multiple times a day | Monthly or less |
| Lead time for changes | Under 1 hour | 1–6 months |
| Change failure rate | 0–15% | 46–60% |
| Time to restore | Under 1 hour | A week to a month |

**The key finding is that these four metrics correlate positively.** Teams that deploy more frequently **also** have lower failure rates and faster recovery. Intuition says the opposite — "deploy more and of course more things break" — but the data runs the other way.

The mechanism is not mysterious:

| Step | Why |
|---|---|
| Changes are smaller | A deploy containing 3 commits versus 300 — when it breaks, you know where to look |
| Feedback is faster | Frequent deploys mean problems surface in minutes, not weeks |
| Localisation is easier | The problem is in the small change just deployed, so the scope is narrow |
| Rollback is simpler | Reverting a small change is far safer than reverting a large release |

**So "we deploy less often to be safer" is a self-fulfilling failure strategy**: fewer deploys means larger change batches, and larger batches mean harder-to-localise failures and riskier rollbacks.

### 4.2 Error budgets: the right reliability target is not 100%

A common default assumption is that higher reliability is always better. But 100% availability means **no change may carry any risk**, which means no change at all — unacceptable commercially.

> [Google, *Site Reliability Engineering*](https://sre.google/)

SRE's answer is the **error budget**, which turns the conflict into a quantifiable decision:

**Error budget = 1 − SLO**

An SLO of 99.9% gives an error budget of 0.1% — roughly 43 minutes of unavailability per month.

| Budget state | What to do |
|---|---|
| Ample | Ship features quickly |
| Nearly exhausted | Pause feature releases and shift to reliability work |

**The value of this mechanism is that it converts "development wants speed, operations want stability" from an interdepartmental argument into a shared objective.** Instead of two teams arguing, the remaining budget decides what happens next. It also openly admits that 100% is the wrong target — chasing it costs far more than users can perceive.

### 4.3 Toil and the 50% ceiling

SRE has a term for it: **toil** — manual, repetitive operational work that could be but has not been automated. The tests are: manual, repetitive, automatable, tactical, devoid of enduring value, and **scaling linearly with system growth**.

Google's discipline: **no more than 50% of an SRE team's time should go to toil.** Above that, the team is understaffed or under-automated.

**The practical value of this threshold** is that it turns automation from "when there is time" into a hard requirement with a trigger. Toil has a specific pathology: it accumulates fastest when you are busiest, and the more it accumulates the less time you have to automate it — a loop that only an external threshold breaks.

### 4.4 Alert on symptoms, not causes

A principle that materially affects how well you sleep:

| What you alert on | The result |
|---|---|
| **Causes** (high CPU, disk at 80%, a process restarted) | You get woken for things users never noticed, and you miss the things they did |
| **Symptoms** (error rate up, latency over target, requests failing) | You get woken only when users are actually affected |

The reason is that **causes and impact are not one-to-one**. CPU at 90% may affect nothing at all; CPU at 40% may coincide with a spiking error rate because a dependency is down. Alerting on causes looks more professional but produces large volumes of noise — and noise teaches people to ignore alerts, at which point the real alert fails too.

**The correct arrangement: alert on symptoms, diagnose with causes.** Dashboards and logs are for the investigation, not for paging.

### 4.5 Two hidden traps in load balancing

Neither shows up in a test environment, because test environments have neither real traffic nor instances entering and leaving.

**Trap one: health checks without graceful shutdown drops requests.**

If the code cannot "finish in-flight requests before exiting", then the gap between an instance being removed from traffic and actually stopping cuts live requests off mid-flight. Users see random failures — and only during deploys or scaling events, which makes them very hard to reproduce.

**Trap two: retries amplify failures.**

This is the classic mechanism behind cascading failure: a dependency slows down → callers time out → callers retry → the dependency receives double the requests → slower still → more callers time out and retry…

```
Normal:      100 requests → dependency
Slower:      100 requests, 40 time out and retry → 140 reach it
Slower still: 140 are all slow, 70 retry → 210
...
```

**Two defences are needed, and they only work together:**

- **Retries must be bounded and jittered.** Without jitter, every client hits the dependency again at the same instant.
- **A circuit breaker:** once a downstream dependency fails past a threshold, stop calling it entirely for a while and fail fast instead of piling up. This protects the dependency and saves callers from waiting out timeouts they know will fail.

### 4.6 Autoscaling cannot fix a slow query

A tempting but wrong diagnostic path: latency rises → CPU looks high → add instances.

The problem is that **scaling addresses "concurrency exceeds capacity", not "a single request got slower"**:

| Symptom | Where to look |
|---|---|
| Concurrency up, latency up, CPU up | Scaling will help |
| Concurrency flat, latency up, CPU up | **Individual requests got heavier** — look for slow queries, N+1s, lock contention |
| Concurrency flat, latency up, CPU flat | The bottleneck is IO or an external dependency — scaling will not help |

**In the second case, adding machines only spreads the same problem more thinly**, raising cost without changing what users experience. Which loops back to the first point in §3.4: **measure the real load, then decide what to scale.**

### 4.7 Dependency maintenance is an ongoing cost, not a one-off task

Dependencies occupy a peculiar position in software: **they are the only thing that hands you new risk while you change nothing.**

| Fact | Implication |
|---|---|
| Your code is unchanged, but a library you depend on is found vulnerable | Risk is injected from outside |
| Dependencies ship breaking changes | Not upgrading also means being left behind by the ecosystem |
| Dependencies get abandoned | A library that works today may have nobody maintaining it in three years |
| Your dependencies' dependencies count as your risk | Supply chain depth is typically dozens of layers |

**Three things to actually do:**

1. **Pin versions** (with a lockfile) so builds are reproducible — otherwise "it built yesterday" becomes an unexplainable statement.
2. **Scan continuously** for known vulnerabilities, with an explicit remediation deadline.
3. **Upgrade proactively on a schedule** rather than letting it accumulate to the point where you must jump several major versions at once — the riskiest possible upgrade.

## 5. Capability checkpoints

1. I can explain what general production operations covers versus AI-specific operations, and why they should not be conflated.
2. I can list the phases of the SDLC, explain why configuring the deployment environment is the most overlooked, and relate release strategy to blast radius.
3. I can name at least four things to measure before scaling, and explain why identifying the growth driver is the most critical.
4. I can state the precondition for horizontal scaling, and the roles of load balancing, health checks and graceful shutdown.
5. I can state the costs of the three data-layer scaling levers and their sensible order of application.
6. I can restate the four DORA metrics with elite benchmarks, and explain why speed and stability reinforce rather than trade off.
7. I can explain how an error budget converts a development-versus-operations conflict into a shared objective, and compute the monthly unavailability for a 99.9% SLO.
8. I can give an example of why alerts should be based on symptoms rather than causes.
9. I can explain how retries produce cascading failure, and which two defences must work together to prevent it.
10. I can distinguish "concurrency overload" from "individual requests got slower", and explain why scaling does not help the latter.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 3*, 2026-08-28 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals) |
| Tier 1 — official | Google, *Site Reliability Engineering* (free online) — SLI/SLO/error budgets, toil, monitoring hierarchy, release engineering, blameless postmortems | [sre.google](https://sre.google/sre-book/table-of-contents/) |
| Tier 1 — official | Google, *The Site Reliability Workbook* — how to implement SLOs | [sre.google](https://sre.google/workbook/table-of-contents/) |
| Tier 1 — research | Forsgren, Humble & Kim, *Accelerate: The Science of Lean Software and DevOps*, 2018 | [itrevolution.com](https://itrevolution.com/product/accelerate/) |
| Tier 1 — research | DORA, *State of DevOps Report* (annual; contains the benchmark figures) | [dora.dev](https://dora.dev/research/) |
| Tier 1 — paper | Hamilton, *On Designing and Deploying Internet-Scale Services*, LISA 2007 | [usenix.org](https://www.usenix.org/legacy/events/lisa07/tech/full_papers/hamilton/hamilton.pdf) |
| Tier 1 — standard | NIST, *The NIST Definition of Cloud Computing*, SP 800-145 (definitions of IaaS / PaaS / SaaS) | [nist.gov](https://csrc.nist.gov/pubs/sp/800/145/final) |
| Tier 2 — practitioner | *The Twelve-Factor App* — config separated from code, stateless processes, logs as event streams | [12factor.net](https://12factor.net/) |
| Tier 2 — practitioner | AWS Builders' Library — engineering detail on backoff and jitter, timeouts, retries, circuit breaking | [aws.amazon.com](https://aws.amazon.com/builders-library/) |

> The figures and cases in this manual come from [DORA's public research]((https://dora.dev/research/2025/dora-report/)) and *Accelerate* (2018), benchmarks are updated in each annual report. The toil ceiling and error-budget arithmetic come from Google's SRE book.
