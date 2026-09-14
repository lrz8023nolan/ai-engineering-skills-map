# 2.1 Building Full-Stack Applications

> **Part 2** · Software engineering fundamentals
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Building full-stack applications means understanding **what actually happens between a user clicking something and data landing in storage** — and which trade-offs exist at each step along the way.

The skill has become more valuable in the AI era, not less. Coding agents let people who worked only on the front end, or only on mobile, take on a much wider role. But that creates a specific trap: **you cannot constrain the agent if you do not know which trade-offs exist.** Someone who does not understand caching will never recognise that "should this response be cached?" is a decision at all, so the agent silently makes it for them — usually the more expensive way.

Without the skill, the typical failure is not "it doesn't run". It is **it runs, and the structure is wrong**: state that belongs on the server is kept on the client, a response that must not be cached gets cached, a hole opens between authentication and authorisation, or an ORM produces a query that takes milliseconds in development and seconds in production. None of these raise an error. They surface as "it feels slow" and "the data is occasionally wrong".

## 2. In the map

This is the first of the five sub-skills of Part 2. Ng names the central tension of the skill in two sentences:

> "A coding agent can help with parts of the development process that you might be less familiar with. However, understanding how the full stack actually works is important."

The components and concepts he lists span the whole stack:

| Layer | What to understand |
|---|---|
| Front end | UI components, page rendering |
| Data flow | Caching, asynchronous processing, data persistence |
| Interface | API choice and design |
| Identity | Authentication |
| State | State and session management |
| Quality | Testing, security, accessibility |

Source: [AI Engineering Skills Map Part 3, 2026-08-28](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals)

## 3. Core concepts

### 3.1 Where the front end ends and the back end begins

| Side | Runs on | Trustworthy? |
|---|---|---|
| **Front end** | The user's device | **No.** The user fully controls it, so every validation must be redone |
| **Back end** | Your server | Yes — the only place trust can live |

This is the foundation for every security discussion that follows: **any validation on the front end is a convenience, not a control.**

### 3.2 Four rendering strategies

| Strategy | When HTML is produced | Strengths | Costs |
|---|---|---|---|
| **CSR** (client-side) | In the browser, after JS loads | Smooth interaction, light server | Slow first paint; poor SEO |
| **SSR** (server-side) | On every request | Fast first paint; good SEO | Server bears the rendering cost |
| **SSG** (static generation) | At build time | Fastest, easiest to cache | A content change needs a rebuild |
| **ISR** (incremental static regeneration) | At build time, refreshed in the background | Speed with reasonable freshness | Added complexity |

These combine freely. The decision hinges on **how often the content changes** and **whether it must be personalised per user**: rarely changing content suits SSG, per-request content needs SSR, and interaction-heavy parts of the page can be CSR.

### 3.3 Caching is layered

Caching is not a switch. It is a structure that runs the length of the request path:

| Layer | Location | Controlled by |
|---|---|---|
| Browser cache | The user's device | HTTP response headers |
| CDN / edge cache | A node near the user | Response headers plus CDN configuration |
| Application cache | Inside your service process | Code |
| Distributed cache | A separate service (Redis, for example) | Code |
| Database cache | Inside the database | Query and index design |

**The first two layers have semantics standardised in HTTP** — RFC 9111 defines the exact meaning of `Cache-Control`, `Age`, `Vary` and the rest. That means you should not invent your own caching semantics: any proxy in between would not know how to handle your response.

### 3.4 API choice and design

| Style | Character | Fits |
|---|---|---|
| **REST** | Resource-centred, actions expressed through HTTP methods, leaning on HTTP semantics including caching and idempotency | Public, resource-oriented APIs |
| **RPC-style** | Action-centred (`POST /doThing`) | Explicit actions between internal services |
| **GraphQL** | The client declares which fields it wants, one endpoint | Where client needs diverge and over-fetching hurts |

Which you pick matters less than getting three things right:

1. **Idempotency** — GET, PUT and DELETE are idempotent by HTTP semantics; POST is not. This determines whether a client can safely retry.
2. **Error structure** — errors should be machine-readable, not just a sentence. A standard format already exists (RFC 9457 problem details); there is no need to invent one.
3. **Version evolution** — the interface will change. Decide in advance how.

### 3.5 Authentication and authorisation are two different things

This is the most commonly conflated pair, and the most expensive one:

| | Question answered | Example |
|---|---|---|
| **Authentication** | Who are you? | Signing in, verifying a password or token |
| **Authorisation** | What are you allowed to do? | May this user delete this record? |

**Doing authentication without authorisation is the classic shape of "broken access control"**: the user really is signed in, but they can reach someone else's data. OWASP's Top 10 has kept broken access control near the top of the list for exactly this reason.

The industry-standard framework for delegation is OAuth 2.0 (RFC 6749), which defines four grant types — authorisation code, implicit, resource owner password credentials, and client credentials. **The implicit grant is now widely regarded as unsuitable for new systems** (the token is exposed in the browser's address bar and history); authorisation code with PKCE has replaced it.

### 3.6 Two shapes of session management

| Shape | Approach | Strengths | Costs |
|---|---|---|---|
| **Stateful** (server-side session) | Session data on the server; the client holds only an opaque ID | Can be revoked at any time; data never leaves the server | Needs shared storage; horizontal scaling takes extra work |
| **Stateless** (token) | Session data encoded in the token; the server just verifies the signature | No shared storage, easy scaling | **Cannot be revoked before expiry** — unless you keep a denylist, which makes it stateful again |

**The core of this trade is revocation against scalability.** Any scenario needing an immediate cut-off — invalidating sessions after a password change, banning an account — has to take this seriously.

### 3.7 Asynchronous processing and persistence

- **Asynchronous processing**: move slow work off the request path (queues, background jobs) so the endpoint returns quickly. The cost is **eventual consistency** — when the user is told "submitted", the work may not be done, so progress has to be queryable.
- **Persistence**: not just "write it somewhere", but which layer the data lives in, who can reach it, and what the backup and recovery story is. See [2.2 Managing Data](02-managing-data.md).

### 3.8 Layers of testing

| Layer | What it covers | Character |
|---|---|---|
| **Unit** | One function or component | Very fast and stable, but far from real usage |
| **Integration** | Several components together | Covers real interaction; moderate speed |
| **End-to-end** | A full flow from the user's point of view | Closest to reality, but slow and brittle |

**The right proportions are genuinely disputed** (see §4.7), but "unit tests only" and "end-to-end only" are both known-bad.

### 3.9 A minimum security list

| Item | Requirement |
|---|---|
| Input validation | Redone on the server |
| Authentication | Use a mature scheme; do not build your own |
| Authorisation | Checked on every protected resource, down to the object level |
| Dependencies | Continuously scanned for known vulnerabilities |
| Secrets | In environment variables or a secret store, never in the repository |
| Transport | HTTPS everywhere |
| Output escaping | Prevention of injection (SQL, XSS and so on) |

### 3.10 Accessibility

Accessibility is not extra work for a minority. It requires semantic HTML, keyboard reachability, sufficient contrast, alternative text for images and non-text content, and correct form labelling.

The standard is the W3C's WCAG, with three conformance levels (A / AA / AAA). **AA is the usual target in practice.**

## 4. Going deeper

### 4.1 Rendering strategy is a trade-off matrix, not a preference

Discussions of "SSR or CSR" often collapse into team-picking, but it is an account that can actually be worked out. At least four quantities are in play:

| Strategy | Time to first byte | Time to interactive | Server cost | Content freshness |
|---|---|---|---|---|
| CSR | Fast (empty shell) | Slow | Low | Latest on every request |
| SSR | Slow (must render) | Medium | High — renders on every request | Latest |
| SSG | Very fast | Fast | Near zero | As of build time |
| ISR | Very fast | Fast | Low | Delayed |

**The key insight: server-side rendering moves cost from the user's device onto your servers.** With high traffic and slow-changing content, that is pure waste — the same HTML rendered a million times. Conversely, a heavily personalised page cannot be made static, and SSR or CSR has no substitute.

This is also why "don't optimise prematurely" does not apply here: **rendering strategy is a decision that changes the cost structure, not a parameter to be tuned later.** Choose wrongly and every subsequent performance fix is paying interest on that choice.

### 4.2 The hard part of caching is invalidation, and layered caches fail worst

The gains from caching are obvious; the risks are hidden. The real question is almost never "should we cache" but **"when does it become stale"**.

Once you have caches at the browser, the CDN, the application process, the distributed cache and the database, **the same data exists at five different levels of freshness**. A classic incident takes this shape:

1. The user updates their profile; the application cache is correctly invalidated.
2. But the CDN still holds the old response, with `Cache-Control: max-age=3600`.
3. For the next hour, some users see stale data and others see fresh — **entirely depending on which cache their request hit.**

Failures like this are very hard to reproduce because they depend on cache hit paths. The defence is to **state the freshness requirement per layer and express it with standard headers**, rather than relying on remembering to purge.

One useful related pattern is **stale-while-revalidate**: return possibly-stale content immediately for speed while refreshing in the background. It trades consistency for delayed consistency, which is a good deal for most content — but **a disaster for "I just changed it and must see it immediately"**, so the distinction has to be made per scenario.

### 4.3 Conflating authentication with authorisation is the most expensive bug

Judged by vulnerability counts, this matters more than any technical detail. **"The user is signed in" and "the user may do this" are independent checks, and developers routinely implement only the first.**

The archetype is **object-level access control failure**: the endpoint is `/api/orders/{id}`, the server verifies that the caller has a valid session, but never verifies that the order belongs to that caller. An attacker simply substitutes someone else's id and reads their orders — and **nothing raises an error**, because from the system's point of view the request is entirely legitimate.

The discipline is plain: **every endpoint touching a specific resource must explicitly check whether the current user may access that specific resource**, and that check belongs in the data access layer rather than relying on every endpoint author to remember it.

### 4.4 The revocation problem with stateless tokens

The appeal of JWT-style tokens is that the server stores nothing — verify the signature and you are done, with excellent scaling. But there is a **structural contradiction** here:

**A token that has been issued and has not expired remains valid until the server forgets it. And the server does not know about it in the first place — that is what stateless means.**

So all of these become problems: killing existing sessions after a password change, immediately blocking a compromised account, revoking a departing employee's access.

Common mitigations and their costs:

| Mitigation | Cost |
|---|---|
| Short expiry plus a refresh token | Narrows the exposure window but does not deliver immediate revocation |
| A denylist of revoked tokens | Stateful again, giving up the main advantage |
| A version claim in the token, checked against the user record | A storage lookup on every request — also state |

**This is not a problem that a correct choice of scheme solves; it is a trade that has to be made against business requirements.** If the business genuinely needs immediate revocation — finance, healthcare, enterprise permission systems — the honest conclusion is that "purely stateless" does not apply, and the cost of a storage lookup has to be accepted.

### 4.5 Idempotency is the precondition for retrying

Networks are unreliable: a timeout does not mean the request did not arrive, only that you did not receive the response. **If the endpoint is not idempotent, one "retry after timeout" can become two charges.**

This is why the HTTP protocol distinguishes methods by idempotency:

| Method | Idempotent? | Meaning |
|---|---|---|
| GET / HEAD | Yes | Inherently safe; retry freely |
| PUT / DELETE | Yes | Repeating produces the same result |
| **POST** | **No** | Repeating creates additional resources |

For cases that must use POST but need to be retryable — creating an order, initiating a payment — the standard answer is an **idempotency key**: the client generates a unique key and sends it with the request, and the server returns the first result rather than executing again on seeing the same key.

The value of this design is that it isolates **network-layer uncertainty** from business logic. **Without it, any automatic retry mechanism is dangerous.**

### 4.6 N+1 queries: the trap ORMs set most often

The most common performance trap when using an ORM: fetch a list (one query), then fetch related data for each item in the list (N queries). **With small development datasets this is milliseconds; in production N is in the hundreds or thousands, so it becomes seconds.**

It is dangerous precisely because **the code looks entirely normal** — no raw SQL inside a loop, just a natural property access. The ORM hides the performance problem behind syntax.

Prevention: learn your ORM's eager-loading mechanism, log the queries actually executed in development, and watch specifically for "accessing a related object inside a loop body" during review.

### 4.7 Testing pyramid versus testing trophy

There is a genuine disagreement worth knowing about the proportions between layers:

| | Position | Basis |
|---|---|---|
| **Testing pyramid** (Mike Cohn) | Many unit tests, fewer integration, fewest end-to-end | Lower levels are faster, more stable and cheaper |
| **Testing trophy** (Kent C. Dodds) | Integration tests dominate; few unit and few end-to-end | Unit tests tend to test implementation details rather than behaviour, producing large numbers of meaningless failures on refactor; integration tests give the best confidence-per-cost |

**The agreement matters more than the disagreement**: end-to-end should not be the primary tool (too slow, too brittle), and unit tests alone are not enough (they do not prove the pieces work together). The argument is about **how much of the middle layer there should be**, and the answer depends on how easy it is for the contracts between your components to break.

### 4.8 The curb-cut effect of accessibility

A counter-intuitive fact that keeps being confirmed: **design improvements made for disabled users end up benefiting everyone.**

Curb cuts were designed for wheelchairs and are used by people with prams, suitcases and delivery trolleys. The same pattern recurs in software: captions made for deaf users are heavily used by people in noisy places; semantic HTML made for screen readers also makes pages easier for search engines and AI agents to understand.

**This matters especially for AI applications**: an application with clear semantics and structure is not only friendlier to people, it is easier for an agent to operate correctly. Good accessibility is, incidentally, an agent-friendly interface.

### 4.9 Why these "outdated" fundamentals matter more in the agent era

A fair question: agents can write the code, so what is the point of learning this?

**There is one — precisely because agents can write the code.** A coding agent's default behaviour is to implement what it is asked. It will not volunteer that:

- adding a cache here introduces a consistency problem;
- the object-level permission check is missing;
- this query degrades to N+1 at production data volumes;
- choosing SSR for this page will multiply server cost by ten.

None of these are questions of "is the code right". They are questions of **what should be built at all**. An agent can only optimise within the objective it is given, and **recognising and defining those objectives is exactly what full-stack knowledge is for.** You do not need to write faster than the agent; you need to know what to tell it to write.

## 5. Capability checkpoints

1. I can explain the fundamental difference in trust model between front end and back end, and why front-end validation cannot replace server-side validation.
2. I can compare CSR, SSR, SSG and ISR on time to first byte, time to interactive, server cost and content freshness, and choose for a given scenario.
3. I can name the layers where caching occurs, and explain why standard HTTP cache headers should be used rather than invented semantics.
4. I can distinguish authentication from authorisation and give a concrete example of object-level access control failure.
5. I can explain the trade between stateful sessions and stateless tokens on revocation, and why "purely stateless" is unsuitable for some businesses.
6. I can explain idempotency, say which HTTP methods are idempotent, and explain what an idempotency key solves.
7. I can recognise the code shape of an N+1 query and name at least two ways to prevent it.
8. I can describe where the testing pyramid and the testing trophy disagree, and what they agree on.
9. I can list the minimum accessibility requirements, and explain why the curb-cut effect matters especially for AI applications.
10. I can explain why full-stack fundamentals matter more now that coding agents are widespread, with at least two risks an agent will not volunteer.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 3*, 2026-08-28 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map-in-detail-software-engineering-fundamentals) |
| Tier 1 — standard | IETF RFC 9111, *HTTP Caching*, 2022 (obsoletes RFC 7234) | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc9111.html) |
| Tier 1 — standard | IETF RFC 9110, *HTTP Semantics* (methods, status codes, idempotency) | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc9110.html) |
| Tier 1 — standard | IETF RFC 6749, *The OAuth 2.0 Authorization Framework*, 2012 | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc6749.html) |
| Tier 1 — standard | IETF RFC 7636, *PKCE* (strengthening the authorisation code flow) | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc7636.html) |
| Tier 1 — standard | IETF RFC 9457, *Problem Details for HTTP APIs* | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc9457.html) |
| Tier 1 — standard | W3C, *Web Content Accessibility Guidelines (WCAG) 2.2* | [w3.org](https://www.w3.org/TR/WCAG22/) |
| Tier 1 — official | OWASP Top 10 (web application security risks) | [owasp.org](https://owasp.org/www-project-top-ten/) |
| Tier 1 — paper | Fielding, *Architectural Styles and the Design of Network-based Software Architectures*, 2000 (the original definition of REST) | [ics.uci.edu](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) |
| Tier 2 — practitioner | Kent C. Dodds, *The Testing Trophy and Testing Classifications* | [kentcdodds.com](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) |

> The testing pyramid was introduced by Mike Cohn in *Succeeding with Agile* (2009); cited here at second hand, original not consulted. Rendering strategies and N+1 queries are general engineering practice; no single source is cited.
