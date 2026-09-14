# 2.2 Managing Data

> **Part 2** · Software engineering fundamentals
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Managing data means **deciding what to store, for how long, in what shape, and how to keep it clean and consistent enough to rely on**.

It deserves its own treatment for a plain reason: **data is the hardest layer of a system to change.** Code can be rewritten, interfaces redesigned, architecture evolved — but once a data structure and its contents have accumulated, changing them means migration, and migration is risky, needs downtime or dual writes, and often cannot be fully automated. Even a coding agent helps only so much: it can write the migration script, but **the context for why the field was designed this way in the first place lives only in a person's head.**

Without the skill, the characteristic outcome is that **one early data-model decision locks in everything that follows.** You want to add a query and find the existing structure cannot support it, so you write layers of workaround code. You want to scale and find the storage type you chose degrades into serial execution under concurrency. The defining property of these problems is that **they do not surface during development** — development data volumes and concurrency are small enough to hide any structural flaw.

## 2. In the map

This is the second of the five sub-skills of Part 2. Ng sets out a full checklist:

- Think through **access patterns**, and use them to decide what to store and for how long
- Identify the right **data models** and select appropriate **storage types** (relational tables, documents, key-value, graphs) and infrastructure — which in turn affects speed, scalability, availability, reliability and cost
- Understand **transactions and concurrency**, and ensure data is clean, consistent and fresh
- Ensure proper **privacy, governance and compliance** where needed
- Manage the **data lifecycle**
- **Evolve the data architecture** as the application evolves

And the AI-specific consequence he draws is the line worth memorising:

> "Your AI systems will get their own input context from your data source, so if data architecture is chosen poorly, the AI doesn't know what it doesn't know."

Source: [AI Engineering Skills Map Part 3, 2026-08-28](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals)

## 3. Core concepts

### 3.1 Choose access patterns first, storage second

This is the most important ordering in the manual. **Do not ask "which database should I use" first. Ask "how will this data be read and written".**

| Question to answer | What it determines |
|---|---|
| Who reads, who writes? What is the read/write ratio? | Whether to denormalise; whether to use read replicas |
| On which **dimensions** is data filtered when read? | Indexes and partition keys |
| Are individual records updated frequently? | How well a storage type fits |
| Are cross-entity queries needed? | Whether you need relational, or a graph |
| What is the growth curve? | Sharding strategy and cost model |
| How strong must consistency be? | Whether eventual consistency is acceptable |

**Access patterns determine the data model, not the other way round.** Doing it backwards — fixing the table structure first and then finding a way to serve the queries — is the root of most performance incidents.

### 3.2 Four storage types and the criteria between them

| Type | Shape | Strong at | Weak at |
|---|---|---|---|
| **Relational** | Tables, rows, columns, with constraints and joins | Cross-entity queries, transactions, constraints and integrity | Horizontal scaling is harder; schema changes need migration |
| **Document** | Self-contained documents (JSON-like) | Fast single-document reads and writes; flexible schema | Weaker cross-document queries and multi-document transactions |
| **Key-value** | Key → value | Extremely fast, trivially scalable | Accessible only by primary key; no range or relational queries |
| **Graph** | Nodes and edges | Multi-hop relationship queries | Smaller ecosystem; unsuitable for simple access patterns |

**These are not generations replacing one another; they are tools matched to different access patterns.** A single system can legitimately use several at once — core business data in a relational store, sessions in key-value, recommendation relationships in a graph.

### 3.3 Normalisation and denormalisation

| | Normalised | Denormalised |
|---|---|---|
| **Approach** | Each fact stored once, referenced elsewhere | Deliberate duplication to reduce joins |
| **Gains** | Updates stay consistent; contradictions are hard to create | Fast reads; simple queries |
| **Costs** | Reads need joins; complex queries slow down | Updates must touch several places; easy to drift out of sync |

**Denormalisation is a deliberate trade, not a design mistake.** The deciding factor is the read/write ratio: read-heavy workloads with latency sensitivity gain clearly; write-heavy or strict-consistency workloads find the synchronisation burden outweighs the gain.

One thing to keep in view: **denormalisation moves the responsibility for consistency out of the database and into application code.** The database can no longer prevent contradictions for you; you have to.

### 3.4 Transactions and concurrency

A **transaction** is a unit of work that either happens entirely or not at all. Its four guarantees are usually grouped as ACID: atomicity, consistency, isolation, durability.

**Isolation is the subtle one**, because it is not a switch but a set of levels, each permitting certain concurrency anomalies:

| Isolation level | Anomalies permitted |
|---|---|
| Read uncommitted | Dirty reads, non-repeatable reads, phantom reads |
| Read committed | Non-repeatable reads, phantom reads |
| Repeatable read | Phantom reads (under the standard definition) |
| Serializable | None — at the cost of the lowest concurrency |

Choosing a level is fundamentally **trading correctness against concurrent throughput**. Higher levels are safer, with more lock contention and lower throughput.

### 3.5 Consistency, availability and partitions

This is the classic triangle of distributed data systems, and it is **widely misread** (see §4.1). The correct reading:

- The three properties are **consistency (C), availability (A) and partition tolerance (P)**.
- **P is not an option you can decline** — network partitions inevitably occur in a distributed system.
- The real trade is therefore between **C and A, and only while a partition is in progress**.
- With no partition, you can have both C and A.

### 3.6 Data quality: three things

Ng uses three words: **clean, consistent, fresh**. Each maps to a set of concrete engineering actions:

| Dimension | Problems to handle | Means |
|---|---|---|
| **Clean** | Missing values, inconsistent formats, mixed units, obviously wrong values, duplicates | Validation on write, constraints, cleaning pipelines, deduplication |
| **Consistent** | The same fact differing in two places, broken references, contradictory timestamps | Transactions, foreign key constraints, a single write path |
| **Fresh** | Upstream updated but downstream not caught up; cache older than the source | Change capture, incremental sync, freshness monitoring |

**A principle that is easy to overlook: guarantee quality at the source, not downstream.** Downstream cleaning is always guessing at upstream intent, and the cost of guessing wrong propagates all the way to the final result.

### 3.7 The data lifecycle

Data does not have just two states, "stored" and "deleted". The full lifecycle runs: create → actively used → cold → archived → destroyed.

| Stage | Decisions to make |
|---|---|
| Create | Validation rules, required fields, defaults |
| Active | Indexing strategy, access control, backup frequency |
| Cold | When to move into cheaper storage |
| Archive | Whether queries must still work; how long to keep it |
| Destroy | How to be sure it is actually gone, including backups and replicas |

**"How long do we keep it" has to be answered explicitly**, because it is constrained both by business needs and by compliance requirements (§3.8).

### 3.8 Privacy, governance and compliance

| Concern | What it covers |
|---|---|
| **Classification** | Which data is personal, sensitive, or publishable |
| **Access control** | Who may read, write, or only see aggregates; whether field-level permissions are needed |
| **Erasability** | When a user asks for deletion, can it genuinely be removed — including derived data, backups and logs? |
| **Portability** | The user's right to export their own data |
| **Minimisation** | Collecting only the fields genuinely needed |
| **Audit** | Who accessed what, and when |
| **Compliance** | Following the relevant regime for each target market (GDPR in the EU, China's Personal Information Protection Law, and so on) |

**Erasability is the most underestimated item.** If data has been copied into caches, search indexes, warehouses and logs, then "delete" becomes a cross-system coordination exercise — and it has to be designed in, because it cannot be bolted on afterwards.

### 3.9 Evolving the data architecture with the application

Applications change, so the data architecture has to be able to change with them:

| Approach | Method | Risk |
|---|---|---|
| **Incremental migration** | Old and new structures coexist; read and write paths switch over gradually | Long duration with two sets of logic alive at once |
| **Dual write plus backfill** | Write to both places at once, then backfill history | Failure handling during dual write gets complicated |
| **One-off migration** | Stop the world and migrate | Simple, but requires an outage window |

**The common factor: migration is the hardest part of data management, and its difficulty grows faster than the data volume.** That is why the early choice matters so much — what you are choosing is not today's cost but the ceiling on how hard every future migration will be.

### 3.10 Data infrastructure for agents

This is the area Ng singles out as still evolving rapidly. Traditional data architecture assumes two consumers, **humans and application code**, neither of which much cares whether data is organised in a way that suits a model. An agent is a third kind of consumer:

| Dimension | For humans and code | For agents |
|---|---|---|
| Organisation | Table structure, interfaces | Semantic chunks, retrievable representations (see [1.2 Grounding Models with Data](../part-1-ai-applications/02-grounding-models-with-data.md)) |
| Access | Explicit queries | Vague natural-language intent |
| Failure mode | An error, or an empty result | **A plausible wrong answer** |

That last row is the key: **traditional consumers fail loudly when data is missing. Agents do not.** They produce an answer in exactly the same confident register, built on incomplete data. This is the concrete meaning of Ng's "the AI doesn't know what it doesn't know".

## 4. Going deeper

### 4.1 CAP was misread for a decade, and the author said so

The "pick two of three" formulation is extremely widespread, and **it is wrong**. Brewer himself wrote a paper in 2012 to correct it:

> Brewer, *CAP Twelve Years Later: How the "Rules" Have Changed*, IEEE Computer 45(2), 2012 — [DOI](https://doi.org/10.1109/mc.2012.37)

The original theorem (conjectured by Brewer in 2000, proved by Gilbert and Lynch in 2002) says: **in the presence of a network partition**, a distributed system cannot simultaneously guarantee strong consistency and availability.

Once misread as "pick two", three specific errors follow:

| Misreading | Reality |
|---|---|
| "You can choose to give up P" | P is not a choice — partitions necessarily occur in a distributed system |
| "You must permanently give up C or A" | The trade exists **only while a partition is in progress**; normally you can have both |
| "This trade defines a database's category" | Modern systems can restore consistency after a partition heals, so databases should not be labelled "CP" or "AP" |

**The practical implication**: do not justify a design with "we chose AP, so inconsistency is fine", because that sentence rests on the misreading. The real question is more specific — **during a partition, should this particular operation refuse service or return possibly-stale data** — and, once the partition heals, how the inconsistency gets repaired.

> Related: Kleppmann, *Please stop calling databases CP or AP*, 2015, discusses this in more detail.

### 4.2 Isolation levels: the standard's own definitions are ambiguous

One easily overlooked fact about names like "read committed" and "repeatable read": **the SQL standard's definitions of them are ambiguous, and real databases do not implement them identically.**

This was established by an influential paper:

> Berenson, Bernstein, Gray, Melton, O'Neil & O'Neil, *A Critique of ANSI SQL Isolation Levels*, SIGMOD 1995

The paper did two things: it showed that the standard's natural-language definitions of the isolation levels are ambiguous (the same wording can be read into different implementations), and it **introduced snapshot isolation** — a level absent from the standard at the time but already widely implemented in commercial databases.

**Two engineering consequences:**

1. **Do not assume "repeatable read" behaves the same across databases.** The same level name can carry materially different semantics in different implementations. Check the specific database's documentation when choosing or migrating.
2. **Do not judge safety by the level name.** What you need to know is which specific concurrency anomalies are possible *in this database at this level*, not a broad label.

The danger of this class of problem is that **it raises no error**: your code occasionally produces one wrong balance or stock count under high concurrency, and no single test can reproduce it.

### 4.3 The relational model's real contribution is data independence, not tables

The 1970 paper is often remembered as "proposing to store data in tables". Tables were not new. Its real contribution is in the opening sentence:

> "Future users of large data banks must be protected from having to know how the data is organized in the machine."

> Codd, *A Relational Model of Data for Large Shared Data Banks*, Communications of the ACM, 1970

This is **data independence**: physical independence (applications need not know how data is stored) and logical independence (applications need not know how the logical structure changes). Before this, hierarchical and network models required programmers to know physical locations and navigation paths — **changing one field's structure could mean rewriting large amounts of program code.**

**What it means today**: when evaluating a data approach, the question is not only "is it fast" but **"how much structural detail does it expose to callers"**. The more it exposes, the more code you will have to change when the structure evolves. This is also why ORMs, API abstraction layers and semantic layers have value — what they buy is not performance but **evolvability**.

### 4.4 Indexes are not free

A classic trade-off, but worth stating plainly: **every index makes writes slower.**

| With an index | Without one |
|---|---|
| Queries on that field are fast | That query degrades to a full scan |
| Every write must update the index structure | Writes are faster |
| Extra storage consumed | Space saved |

The deciding factors are the read/write ratio and how critical the query is. **Do not add an index because it "might be useful"** — on write-heavy systems, surplus indexes directly cut write throughput, and that cost is far less visible than the query speedup.

### 4.5 Schema-on-write versus schema-on-read

| | Schema-on-write | Schema-on-read |
|---|---|---|
| **Validation happens** | On write | On read |
| **Gains** | Quality guaranteed at the entrance; structure explicit | Flexible writes; suits evolving requirements |
| **Costs** | Structure changes require migration | Quality problems surface only at read time |

**Again a trade about which end to pay the validation cost at, not a question of better or worse.** A practical combination: core business data under schema-on-write for quality, exploratory and log-like data under schema-on-read for flexibility.

### 4.6 Migrations cannot be fully automated: expand-contract

Ng says data is "relatively hard to change (even if agents help with migrations)". The weight of that sentence is that migration's difficulty usually lies not in writing the script but in **switching over without an outage**.

A widely used pattern is **expand-contract**:

| Phase | Action |
|---|---|
| **Expand** | Add the new structure (column, table) while keeping the old; code tolerates both |
| **Migrate** | Backfill history; move reads and writes over gradually |
| **Contract** | Once nothing depends on the old structure, remove it |

The point is that **each phase is reversible**, and the system stays available throughout. The cost is a longer timeline and two sets of compatibility logic in the interim.

**The practical value of knowing this pattern** is that it gives you a default answer for every schema change that does not require downtime — instead of being forced to choose between stopping the service and living with the old structure.

### 4.7 Data architecture sets the ceiling on what the AI can know

Back to Ng's most important sentence. Unfolded into engineering terms, there are three layers:

1. **The AI's input context comes from your data sources.** If relevant information is scattered in corners that cannot be joined, or its meaning depends on oral tradition to interpret, then no amount of retrieval sophistication will find it.
2. **Missing information does not present as an error. It presents as a confident wrong answer.** Manual 1.1 covered how models do not know the edge of their own knowledge; the situation here is worse — **the model does not even know that the data is absent**.
3. **So the data architecture decision sets the capability ceiling for the AI.** This is the same principle as "retrieval quality is the ceiling" from Part 1, one layer further down.

**The most practical inference**: if you are building an AI system on your own data, **tidying the data returns more than tuning retrieval parameters**. This is the data-centric idea from machine learning, translated into data infrastructure.

## 5. Capability checkpoints

1. I can explain why access patterns must be settled before choosing a storage type, and name at least three decisions they affect.
2. I can compare relational, document, key-value and graph stores on data shape, strengths and weaknesses, and select for a given access pattern.
3. I can state the costs of normalisation and denormalisation, and identify where denormalisation moves the consistency responsibility.
4. I can list the four standard isolation levels with the anomalies each permits, and explain what choosing a level actually trades.
5. I can state CAP correctly and identify what is wrong with the popular "pick two of three" formulation.
6. I can give concrete engineering measures for each of the three quality dimensions — clean, consistent, fresh.
7. I can list the stages of the data lifecycle and explain why "how long do we keep it" must be answered explicitly.
8. I can explain why erasure has to be designed in and cannot be bolted on later.
9. I can describe the phases of the expand-contract migration pattern and state the core problem it solves.
10. I can explain how data architecture sets an AI system's capability ceiling and give the concrete engineering meaning of "the AI doesn't know what it doesn't know".

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 3*, 2026-08-28 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals) |
| Tier 1 — paper | Codd, *A Relational Model of Data for Large Shared Data Banks*, Communications of the ACM, 1970 | [ACM](https://dl.acm.org/doi/10.1145/362384.362685) |
| Tier 1 — paper | Brewer, *CAP Twelve Years Later: How the "Rules" Have Changed*, IEEE Computer 45(2), 2012 | [DOI](https://doi.org/10.1109/mc.2012.37) |
| Tier 1 — paper | Gilbert & Lynch, *Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services*, ACM SIGACT News 33(2), 2002 | [DOI](https://doi.org/10.1145/564585.564601) |
| Tier 1 — paper | Berenson, Bernstein, Gray, Melton, O'Neil & O'Neil, *A Critique of ANSI SQL Isolation Levels*, SIGMOD 1995 | [ACM](https://dl.acm.org/doi/10.1145/223784.223785) |
| Tier 1 — paper | DeCandia et al., *Dynamo: Amazon's Highly Available Key-value Store*, SOSP 2007 | [ACM](https://dl.acm.org/doi/10.1145/1294261.1294281) |
| Tier 1 — book | Kleppmann, *Designing Data-Intensive Applications*, O'Reilly, 2017 | [dataintensive.net](https://dataintensive.net/) |
| Tier 2 — practitioner | Kleppmann, *Please stop calling databases CP or AP*, 2015 | [martin.kleppmann.com](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html) |

> The expand-contract migration pattern and the schema-on-write / schema-on-read distinction are general engineering practice; no single source is cited. For the specific provisions of GDPR or China's Personal Information Protection Law, consult the official published texts.
