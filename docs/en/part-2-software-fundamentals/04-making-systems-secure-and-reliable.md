# 2.4 Making Systems Secure and Reliable

> **Part 2** · Software engineering fundamentals
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Security and reliability mean **keeping a system doing the right thing in a world that will go wrong** — neither leaking through a security hole nor collapsing wholesale when something fails.

The two are often discussed separately, but they share a premise: **assume something will go wrong.** Security assumes someone will try to break in; reliability assumes components will break. That is why both work the same way — not by hoping nothing fails, but by keeping the consequences of failure bounded.

Without the skill, the typical failure is **treating security as a gate just before launch**: the feature is finished, a scanner is run over it, the reported issues are fixed, and it ships. The problem is that many security problems cannot be found by scanning code, because they live in the design — where permission boundaries fall, where state is kept, who can reach what. Discovering them after the code is written makes them expensive to fix. Reliability behaves the same way: bolting fault tolerance onto a running system usually means restructuring the critical path.

## 2. In the map

This is the fourth of the five sub-skills of Part 2. Ng splits it into two halves.

**Reliability**, in two layers:

| Layer | What it covers |
|---|---|
| **Testing strategy** | What mix of unit and integration tests, which frameworks, what level of coverage |
| **Designing around failure** | Handling failures (an API hitting a rate limit), graceful degradation, and **minimising the blast radius** |

**Security**, which he frames as a question of timing — the "shift left" movement:

> "You can now use AI tools to scan your code for vulnerabilities, check dependencies for supply chain injections, and examine your cloud configuration for attack surfaces. But doing this well still requires some knowledge of security."

He also notes a change at the role level: just as all developers are moving toward becoming full stack developers, **many developers are now also partly security engineers**.

Source: [AI Engineering Skills Map Part 3, 2026-08-28](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals)

## 3. Core concepts

### 3.1 The three legs of reliability

| Leg | Purpose | Without it |
|---|---|---|
| **Testing** | Find logic errors early | Defects go straight to production |
| **Designing for failure** | Assume components break; keep failures survivable | A single point of failure takes everything down |
| **Observability** | Localise problems quickly | Outage time is multiplied by diagnosis time |

All three are needed. Testing without failure design means any untested path takes the system down. Failure design without observability means incidents recur without anyone finding out why.

### 3.2 Testing strategy

| Layer | Covers | Speed |
|---|---|---|
| Unit | A single function or component | Milliseconds |
| Integration | Several components together | Seconds |
| End-to-end | A complete user flow | Minutes, and brittle |

One judgement about **coverage** worth having firmly: **it is a useful diagnostic metric and a terrible target metric.**

Coverage measures whether a line was *executed*, not whether it was *verified*. Code can be 100% covered with no assertions at all, or with assertions of the wrong thing. Making coverage a KPI most directly incentivises writing assertion-free tests to move the number — which is the problem §4.2 expands on.

### 3.3 Designing around failure

Do not ask "what if nothing goes wrong". Ask "what happens when it does":

| Failure scenario | Response |
|---|---|
| A downstream API hits a rate limit | Backoff with jitter, circuit breaking, queuing |
| A dependency is unavailable | **Graceful degradation** — return a reduced result rather than an error |
| An instance dies | Health checks, automatic removal, redundant instances |
| One region's failure affects everything | **Shrink the blast radius** — cut the system into independent units |
| A traffic spike | Rate limiting, queues, backpressure |

**Blast radius deserves its own note**: it means **how much a single failure can reach**. An instance dying affects every user if all users share that instance, and one unit's worth of users if the system is cut into independent units. **That is an architectural reliability decision, not something code can compensate for.**

### 3.4 Shift left

"Left" means earlier on a conventional project timeline. Moving security work left means:

| Phase | What happens there |
|---|---|
| Design | Threat modelling: how would this be attacked? Where are the permission boundaries? |
| Coding | Secure coding standards, no secrets in the repository, input validation |
| Commit | Secret scanning, static analysis (SAST), dependency vulnerability scanning |
| Build | Software bill of materials (SBOM), artefact signing |
| Test | Dynamic analysis (DAST), fuzzing |
| Deploy | Configuration validation, least privilege, gradual rollout |
| Operate | Runtime monitoring, anomaly detection, incident response |

**Why it works**: the earlier a problem is found, the cheaper it is. A permission flaw in the design takes ten minutes to fix before coding starts, and may require changing the data model after launch.

### 3.5 Supply chain security

In a modern application, **the code you wrote yourself is usually a small fraction** of what runs; the rest is dependencies. So attackers have moved upstream.

| Concept | Meaning |
|---|---|
| **SBOM** (software bill of materials) | An ingredients list — every component and version. When a new vulnerability is published, you can immediately tell whether you are affected |
| **SLSA** | A tiered framework for build-process integrity and tamper resistance |
| **Artefact signing** | Proof that this binary is the one you built, unmodified |
| **Provenance** | A record of who built this artefact, when, and by what process |

**A concrete motivation**: real supply chain attacks (the 2024 xz-utils incident, for example) show that an attacker need never touch your code — a backdoor planted in one of your dependencies travels in through the normal update path.

### 3.6 What AI tools can and cannot do

Ng mentions using AI to scan code for vulnerabilities, check dependencies for supply chain injections, and inspect cloud configuration for attack surfaces. The boundaries of that capability are worth stating:

| Can | Cannot |
|---|---|
| Quickly surface known-pattern vulnerabilities (injection, hard-coded secrets) | Judge whether a permission boundary is designed correctly |
| Compare dependency versions against known CVEs | Decide whether an operation should be permitted at all |
| Detect drift in cloud configuration | Recognise authorisation flaws at the business-logic level |
| Triage large codebases | Replace an understanding of the security model |

**In one line**: AI tools make *finding known classes of problem* cheap, but **knowing which problems matter is still human work.** That is what Ng means by "doing this well still requires some knowledge of security".

## 4. Going deeper

### 4.1 Most severe failures come from error-handling code that was never tested

This is an empirically established conclusion that should change what you decide to test:

> Yuan et al., *Simple Testing Can Prevent Most Critical Failures: An Analysis of Production Failures in Distributed Data-Intensive Systems*, OSDI 2014

The authors analysed severe production failures across several widely used distributed storage systems (Cassandra, HBase, HDFS, MapReduce, Redis, ZooKeeper). Two findings stand out:

| Finding | The number |
|---|---|
| Failures are easy to reproduce | **77% of production failures could be reproduced with three or fewer nodes** |
| Failures cluster in one place | Most were triggered by **error-handling code** |

The second is the important one. **Error-handling code is code that never runs ordinarily** — those `catch` blocks and `if error` branches are not executed on the happy path. And it is precisely that never-verified code that gets invoked at the moment production goes wrong, failing in ways nobody anticipated.

**This explains why "decent test coverage" and "reliable in production" are two different things**: coverage counts executed lines, and error branches are exactly the lines least likely to be executed by tests.

**It also points directly at the remedy**: fault injection — **manufacturing failures on purpose** so those dormant paths actually run. That is the theoretical basis of chaos engineering: not random vandalism, but deliberately striking where nothing has struck before.

### 4.2 Coverage is a misaligned metric

Following from the above, coverage-as-a-target produces three specific distortions:

| Problem | Mechanism |
|---|---|
| Rewards assertion-free tests | Executing a line counts as covered; checking the result is optional and costs nothing |
| Rewards testing implementation details | To execute a particular branch, tests hug the implementation, producing masses of false failures on refactor |
| Ignores importance | A line in the payment logic and a line formatting a log carry equal weight |

A better signal is **mutation testing**: the tool deliberately alters your source (flips `>` to `>=`, changes a constant) and checks whether the tests notice. **A mutation nobody catches means that code was never genuinely verified.**

### 4.3 Why "designing around failure" has to be explicit

An easily overlooked point: **failure-handling code does not get written by the normal development process.**

During development you walk the success path — the endpoint responded, the data saved, the page rendered. Failure paths are only walked when something is already on fire, at which point nobody has capacity to redesign.

So designing around failure has to be **a deliberate act performed during development**, in the form of a set of hypothetical questions:

- This downstream call times out for 30 seconds — what does the user experience?
- This queue backs up to a hundred thousand messages — what happens to the system?
- This cache is wiped entirely — can the database take the load?
- This dependency goes down completely — which features still work?

**The last question matters most**, because it determines what is left after degradation. A well-designed system degrades to "fewer features but the core works", not "unavailable".

### 4.4 Shift left is not a slogan — there are executable standards

"Finding problems earlier is cheaper" is common sense; the hard part is **knowing what to do specifically, and how far to go**. That ground is well covered; there is no need to invent your own programme.

> NIST, *Secure Software Development Framework (SSDF)*, SP 800-218

It organises security work into four practice groups:

| Group | Content |
|---|---|
| **Prepare the Organization (PO)** | Policy, process, people: who owns security, with what tooling |
| **Protect the Software (PS)** | Protect components from tampering: branch protection, signed commits, repository permissions |
| **Produce Well-Secured Software (PW)** | Produce secure software across design, coding, testing and build |
| **Respond to Vulnerabilities (RV)** | Continuously identify and remediate vulnerabilities after release |

**A signal of how real this is**: the framework is mandated for US federal software procurement (via executive order EO 14028) — it is being used as a contract condition, not merely recommended.

Alongside it sits **SLSA** (Supply-chain Levels for Software Artifacts, driven by Google and OpenSSF), which constrains the build process in tiers:

| Level | Requirement |
|---|---|
| L1 | The build process is documented and traceable |
| L2 | The build runs on a hosted service that signs the provenance it generates |
| L3 | The build environment is hardened and isolated, preventing interference between builds |

**The value of tiering is that it admits not everyone must reach the top**: L1 is cheap (document your build), L3 is expensive (isolated build infrastructure). You choose where to stop based on risk and cost.

### 4.5 Supply chain: from "trust your dependencies" to "verify your dependencies"

The traditional stance toward dependencies is trust: install it and use it. Supply chain security is the shift **from trust to verification**.

Three concrete actions:

| Action | Problem it solves |
|---|---|
| Generate an SBOM | When a new vulnerability is published, decide whether you are affected in minutes rather than weeks |
| Pin and review dependency changes | Dependency updates also carry risk; they should be reviewed, not auto-merged |
| Verify artefact provenance | Confirm that what runs in production really came from your pipeline, and was not substituted |

**One practical detail**: SBOMs have standard formats (SPDX is now ISO/IEC 5962:2021; CycloneDX is maintained by OWASP), which means this is no longer everyone doing their own thing but an interoperable specification.

### 4.6 Blast radius is architectural reliability

Mentioned above; here is how it is done. The core idea is to **cut the system into independent cells**: each cell serves a subset of users and owns its full dependency stack, and cells do not share a failure domain. A failure then affects at most one cell's users.

| Technique | Effect |
|---|---|
| **Cell-based architecture** | Shard users into independent cells; a failure stays inside one |
| **Shuffle sharding** | Arrange each user's resource combination to overlap as little as possible, driving down the probability of correlated failure |
| **Request isolation** | One tenant's abnormal traffic does not affect others |
| **Kill switches** | Deliberately disable non-critical features during an incident to protect the critical path |

**Why this beats making individual components more reliable**: a component's reliability has an upper bound, and a system's reliability is roughly the product of its components' — so more components means a less reliable whole. Blast-radius thinking does not raise any component's reliability; it **stops one component's failure from propagating**.

The early theoretical statement of this idea:

> Fox & Brewer, *Harvest, Yield, and Scalable Tolerant Systems*, HotOS 1999

That paper proposes describing a system's degradation along two axes — **harvest** (the fraction of the data that should be returned which actually is) and **yield** (the fraction of requests answered) — **rather than as a binary up/down**. This is the quantitative basis of graceful degradation: a system can lose harvest while holding yield, meaning "the answer is incomplete, but we did not refuse service".

### 4.7 What scanning code with AI cannot replace

Back to Ng's caution. The boundary is cleanly stated by one criterion:

**AI tools are good at pattern matching. Humans are responsible for judging intent.**

| Kind | Example | Who |
|---|---|---|
| Pattern matching | This string is concatenated into SQL, this key is hard-coded, this dependency version has a known CVE | **Tools — and AI makes it cheaper** |
| Intent judgement | Who should be allowed to call this endpoint, should this data be stored at all, does this action need human approval | **A person** |

**Authorisation flaws are the clearest example**: the code is entirely "correct" — it checks that the user is signed in, that parameters are valid, that types match. No scanner finds anything. What is missing is a business judgement: whether this user may access this record (see [2.1 §4.3](01-building-full-stack-applications.md)). **The only defence for that class is a permission check implemented consistently in the data access layer — not a hope that a scanner will catch it.**

## 5. Capability checkpoints

1. I can name the three legs of reliability and explain what fails when each is missing.
2. I can explain why coverage is a useful diagnostic but a poor target, with at least two specific incentive distortions.
3. I can explain blast radius and why it is an architectural rather than a code-level concern.
4. I can state what shift left requires specifically at the design and coding phases.
5. I can explain what SBOM and SLSA each solve, and give the rough requirements of SLSA's three levels.
6. I can state the limits of AI tooling in security, with an example a scanner cannot find and a person must decide.
7. I can restate Yuan et al.'s empirical findings on production failures and explain why error-handling code needs dedicated testing.
8. I can explain the theoretical basis of fault injection and chaos engineering.
9. I can describe a system's degradation using the harvest and yield axes, and explain why up/down is too coarse.
10. I can list at least four ways to deliberately shrink a system's blast radius.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 3*, 2026-08-28 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals) |
| Tier 1 — paper | Yuan et al., *Simple Testing Can Prevent Most Critical Failures: An Analysis of Production Failures in Distributed Data-Intensive Systems*, OSDI 2014 | [usenix.org](https://www.usenix.org/conference/osdi14/technical-sessions/presentation/yuan) |
| Tier 1 — paper | Fox & Brewer, *Harvest, Yield, and Scalable Tolerant Systems*, HotOS 1999 | [DOI](https://doi.org/10.1109/HOTOS.1999.798396) |
| Tier 1 — standard | NIST, *Secure Software Development Framework (SSDF)*, SP 800-218 | [nist.gov](https://csrc.nist.gov/pubs/sp/800/218/final) |
| Tier 1 — standard | SLSA, *Supply-chain Levels for Software Artifacts* (OpenSSF) | [slsa.dev](https://slsa.dev/spec/v1.0/levels) |
| Tier 1 — standard | OWASP Top 10 (web application security risks) | [owasp.org](https://owasp.org/www-project-top-ten/) |
| Tier 1 — official | OWASP, *Software Assurance Maturity Model (SAMM)* — supplies the maturity dimension SSDF lacks | [owaspsamm.org](https://owaspsamm.org/) |
| Tier 1 — official | Google, *Site Reliability Engineering* (free online) — blameless postmortems, change management | [sre.google](https://sre.google/sre-book/table-of-contents/) |
| Tier 2 — practitioner | Principles of Chaos Engineering | [principlesofchaos.org](https://principlesofchaos.org/) |

> Of the two mainstream SBOM formats, SPDX is now ISO/IEC 5962:2021 and CycloneDX is maintained by OWASP; both standard numbers are cited here at second hand.
