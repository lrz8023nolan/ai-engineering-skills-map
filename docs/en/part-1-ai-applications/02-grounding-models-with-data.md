# 1.2 Grounding Models with Data

> **Part 1** · Building and deploying AI applications
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Grounding models with data means **getting relevant, clean external data into the model's input at the moment it needs it**, so that its answers rest on real material rather than on a blurred memory of its parameters.

The problem it solves is simple. A model's parameters hold knowledge compressed in during training, and that knowledge is both potentially out of date and simply missing your context — your internal documents, last week's meeting notes, the records in your customer database. And the model does not know it does not know; it fabricates in the same confident register.

Without this skill, the characteristic outcomes are self-inflicted. Stuffing an entire knowledge base into the prompt until cost and latency run away. Retrieved material that has nothing to do with the question. Key figures silently lost during document parsing. And the subtler failure: **assuming that adding retrieval solves the problem** — when in fact retrieval quality sets the ceiling for the whole system, and nothing downstream can rescue a retrieval stage that went wrong.

## 2. In the map

This is the second of the six sub-skills of Part 1. Ng frames it explicitly as a question of the **menu** — not a single technique, but knowing what the options are and what criteria separate them:

- What to **put in the prompt** versus what to **let the model retrieve on demand** using tools
- Which **representation** fits the data and the queries: a vector index, a knowledge graph, or a semantic layer over structured data
- Turning documents (text, PDFs, HTML, images) into **LLM-ready inputs**
- Engineering pipelines to keep the data **clean and fresh**

His closing sentence carries the point of the section:

> "When you understand the menu of techniques available to get data, you are better able to give your LLM relevant context."

Source: [AI Engineering Skills Map Part 2, 2026-08-21](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications)

## 3. Core concepts

### 3.1 Two paths: put it in the prompt, or let the model fetch it

| Path | Approach | Fits | Cost |
|---|---|---|---|
| **Into the prompt** | Retrieve in advance and splice the content into the context | Content is fixed, needed every time, manageable in size | Pay for the tokens every call; any change invalidates the cache |
| **Fetched on demand (tools)** | Give the model a retrieval tool and let it decide when and what to look up | Content depends on the question, is large, or needs several rounds | Depends on the model choosing well; adds round trips and latency |

The deciding question is **whether the content is the same every time**. A fixed style guide or an API schema belongs on the first path; a knowledge base where the answer depends on what the user asked belongs on the second. The two also combine: put the stable material in the prompt behind a cache breakpoint, and leave the variable part to a tool.

### 3.2 Three generations of retrieval

| Generation | Method | Character | Representative work |
|---|---|---|---|
| **Sparse** | Term statistics (BM25 and similar) | Fast, interpretable, strong on exact matching; blind to paraphrase | Robertson & Zaragoza, BM25 (2009) |
| **Dense** | Encode query and documents as vectors, compare by cosine similarity | Captures meaning; handles synonyms and vague phrasing | DPR (Karpukhin et al., [arXiv:2004.04906](https://arxiv.org/abs/2004.04906)) |
| **Hybrid** | Retrieve with both, then fuse | Exact matching and semantics together; the current default | HybridRAG and similar |

Worth keeping the rough magnitude from the DPR paper: using dense representations alone, top-20 passage retrieval accuracy improved by **9–19 percentage points** over a strong Lucene-BM25 baseline.

### 3.3 Choosing a representation

This is the judgement Ng singles out. Four representations, matched to different query types:

| Representation | Good at | Bad at |
|---|---|---|
| **Just put everything in** | Small, wholly relevant material | Cost explodes as it grows |
| **Vector index** | "What content is semantically close to this question?" — local factual questions | Global questions that need aggregation across documents |
| **Knowledge graph** | "What is the relationship between these entities?" — questions needing multi-hop reasoning | Expensive index construction |
| **Semantic layer** | Controlled queries over structured data (business records, for example) | Only applies where the data is already structured |

The first three can coexist. Combining hybrid retrieval with a knowledge graph has been shown to beat either alone on documents such as financial filings.

### 3.4 A complete retrieval pipeline

```
documents → parse → chunk → embed → index
                                     ↓
question → (query rewrite) → retrieve → rerank → compress → generate
```

| Stage | The judgement to make |
|---|---|
| Parse | Does a scanned PDF need OCR? How is table structure preserved? |
| Chunk | Fixed length or semantic boundaries? Too small loses context; too large introduces noise |
| Embed | Which embedding model? Does the domain need a fine-tuned one? |
| Query rewrite | A user's phrasing is usually a poor retrieval query; it may need rewriting or expansion |
| Rerank | A dedicated reranking model for a second pass over the candidates usually pays off |
| Compress | Extract only the relevant part of a long chunk, cutting noise and tokens |

### 3.5 Document conversion

PDFs, HTML and images are not model input; they have to be turned into clean text or structured data. This step is routinely underestimated, and its defects propagate all the way to the final answer — **a number lost during parsing cannot be recovered by better retrieval**.

Cases worth watching for: two-column layouts read across instead of down; tables flattened into unintelligible text; headers and footers bleeding into the body; information inside images never extracted at all.

### 3.6 Data freshness

Grounding is only as good as how current the data is. An index is not a one-off job; it needs machinery for incremental indexing of new documents, invalidation and replacement of revised ones, and periodic rebuilds. The judgement here is **how tight the freshness requirement actually is** — policy documents may be fine on a daily refresh, a trading dashboard is not.

### 3.7 Order of diagnosis

When the model answers poorly in some domain, work through this order:

1. Does the question **need external information at all**? If the model already knows, retrieval only adds noise.
2. If it does, is the information **the same every time**? Same → put it in the prompt. Varies → use a tool.
3. If a tool, what is the **query type**? Local factual → vectors. Relational → graphs. Structured records → a semantic layer.
4. How **accurate is retrieval**? Check retrieval metrics before blaming generation.

## 4. Going deeper

### 4.1 Retrieval quality is the ceiling

This is the most consistent finding in the field: **if retrieval does not bring the relevant material back, no amount of generator strength will produce the right answer.**

The original RAG paper (Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, [arXiv:2005.11401](https://arxiv.org/abs/2005.11401), NeurIPS 2020) established exactly this two-memory paradigm: knowledge splits into a parametric part (pretrained weights) and a non-parametric part (a retrievable external index), and the latter can be updated at any time without retraining. That design addresses three problems at once — knowledge that cannot be updated, thin coverage of long-tail knowledge, and answers that cannot be traced to a source.

One design choice in the paper is easy to miss: the retriever and the generator were **trained jointly**, not optimised separately. That points at a principle still true today: **the goal of retrieval is not "find documents semantically similar to this question" but "find the passages the generator actually needs."** Those are not always the same set.

### 4.2 Chunking is the most underrated stage

Chunking sets the unit of retrieval, and it is almost always a **hand-tuned trade-off**:

| Smaller chunks | Larger chunks |
|---|---|
| More precise retrieval, less noise | Vaguer retrieval, more irrelevant content |
| A chunk is incomplete on its own; several must be combined | A chunk is self-contained, but carries a lot that is irrelevant |
| Better for precise factual lookup | Better for explanatory questions needing surrounding context |

In practice chunks usually also need metadata attached (source, title, section path) — otherwise a retrieved chunk is hard to interpret once it is separated from its surroundings. **Chunk boundaries should follow semantic structure, not a character count** — splitting on sections and paragraphs generally beats splitting on a fixed token count.

### 4.3 Long context is not a substitute for retrieval

As context windows grow, it is tempting to conclude that you can simply put the whole knowledge base in the prompt. RETRO offers a useful counter-datapoint:

> Borgeaud et al., *Improving Language Models by Retrieving from Trillions of Tokens*, [arXiv:2112.04426](https://arxiv.org/abs/2112.04426), NeurIPS 2022

By conditioning an autoregressive language model on documents retrieved from a two-trillion-token corpus, this work matched GPT-3 and Jurassic-1 on the Pile benchmark with roughly **1/25 of their parameters**. The value of retrieval is therefore not only "added knowledge" but **reaching comparable performance with a much smaller model** — a structural cost advantage that a larger context window does not reproduce.

Conversely, long context has a place retrieval cannot fill: when the task requires global reasoning over an entire document, retrieval's fragmentation gets in the way. The two are complementary.

### 4.4 The trouble with global questions, and the price of a graph index

Vector retrieval exposes its fundamental limit here: it retrieves **independent chunks**, so it handles "what is X" well but cannot answer "what themes run through this document set" or "how do these teams relate to one another" — questions requiring aggregation across documents.

GraphRAG answers by building a knowledge graph of entities and relations, then doing community detection and hierarchical summarisation:

> Edge et al., *From Local to Global: A Graph RAG Approach to Query-Focused Summarization*, [arXiv:2404.16130](https://arxiv.org/abs/2404.16130), Microsoft Research, 2024

The paper reports roughly **30–70%** improvement in answer quality over naive RAG on global questions. But the cost has to be read alongside it: index construction requires many LLM calls to extract entities and relations, making it substantially more expensive than a vector index, and document updates often force a rebuild of the graph. **This is not "a better RAG" — it is an expensive supplement aimed at one class of query.**

### 4.5 Retrieval can succeed and the answer still be wrong

Getting the relevant material back into the context is no guarantee the model uses it well. The "lost in the middle" effect from manual 1.1 applies directly: the same passage performs best at the very start or very end of the context and degrades significantly in the middle; in the extreme, being given 20–30 documents performed **worse than being given none** (Liu et al., [arXiv:2307.03172](https://arxiv.org/abs/2307.03172)).

The implication for retrieval design is direct: **more recall is not better**. Twenty documents of uneven quality may well be worse than three well-chosen ones. Reducing distractors and placing the most important material at the edges beats raising top-k.

### 4.6 Not every question should trigger retrieval

A counter-intuitive but important finding: **retrieving for every question is harmful.** For some questions the model already knows the answer, and the retrieved content is pure noise — capable of pulling the model off a correct answer.

Self-RAG (Asai et al., [arXiv:2310.11511](https://arxiv.org/abs/2310.11511)) trains the model to emit reflection tokens, letting it judge four things for itself: whether retrieval is needed at all, whether the retrieved documents are relevant, whether the generated content is supported by them, and whether the answer is useful. CRAG (Yan et al., [arXiv:2401.15884](https://arxiv.org/abs/2401.15884)) addresses the other pain point — what to do when retrieval quality is poor — with a lightweight retrieval evaluator that selects different strategies by quality tier, including degrading to a web search.

The shared lesson: **retrieval should be a stage that is allowed to fail, and that can tell when it has failed** — not a fixed first stop in the pipeline.

### 4.7 Evaluate retrieval and generation separately

This connects directly to manual 1.4. A RAG system can fail in two completely different places, and a single blended metric will never let you locate which:

| Layer | Metrics | Question answered |
|---|---|---|
| **Retrieval** | Hit rate, MRR, Recall@K | Was the relevant material brought back? |
| **Generation** | Faithfulness, answer relevance, context precision/recall | Was the retrieved material used correctly? |

Gao et al.'s survey (*Retrieval-Augmented Generation for Large Language Models: A Survey*, [arXiv:2312.10997](https://arxiv.org/abs/2312.10997)) groups the architectural evolution into three generations — naive, advanced and modular RAG — and is the standard reference for this separation.

## 5. Capability checkpoints

1. I can decide whether a given piece of content belongs in the prompt or should be fetched on demand by a tool, and state the basis for that decision.
2. I can describe the trade-offs between sparse, dense and hybrid retrieval, and say when dense retrieval's advantage is largest.
3. I can choose between a vector index, a knowledge graph, a semantic layer and simply putting everything in the context for a given task, and justify the choice.
4. I can draw and explain every stage of a complete retrieval pipeline and name the judgement required at each.
5. I can explain why chunking strategy is a decisive variable in retrieval quality, and say what chunk boundaries should follow.
6. I can explain why a larger context window does not replace retrieval, and cite evidence for that position.
7. I can explain why retrieving more documents can make results worse, and connect it to the lost-in-the-middle effect.
8. I can separate a RAG system's failures into retrieval-layer and generation-layer causes, and name at least two metrics for each.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 2*, 2026-08-21 | [link](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications) |
| Tier 1 — paper | Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, NeurIPS 2020 | [arXiv:2005.11401](https://arxiv.org/abs/2005.11401) |
| Tier 1 — paper | Karpukhin et al., *Dense Passage Retrieval for Open-Domain Question Answering*, EMNLP 2020 | [arXiv:2004.04906](https://arxiv.org/abs/2004.04906) |
| Tier 1 — paper | Guu et al., *REALM: Retrieval-Augmented Language Model Pre-Training*, ICML 2020 | [arXiv:2002.08909](https://arxiv.org/abs/2002.08909) |
| Tier 1 — paper | Borgeaud et al., *Improving Language Models by Retrieving from Trillions of Tokens* (RETRO), NeurIPS 2022 | [arXiv:2112.04426](https://arxiv.org/abs/2112.04426) |
| Tier 1 — paper | Edge et al., *From Local to Global: A Graph RAG Approach*, 2024 | [arXiv:2404.16130](https://arxiv.org/abs/2404.16130) |
| Tier 1 — paper | Asai et al., *Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection*, 2023 | [arXiv:2310.11511](https://arxiv.org/abs/2310.11511) |
| Tier 1 — paper | Yan et al., *Corrective Retrieval Augmented Generation* (CRAG), 2024 | [arXiv:2401.15884](https://arxiv.org/abs/2401.15884) |
| Tier 1 — paper | Liu et al., *Lost in the Middle: How Language Models Use Long Contexts*, TACL 2024 | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) |
| Tier 1 — survey | Gao et al., *Retrieval-Augmented Generation for Large Language Models: A Survey*, 2024 | [arXiv:2312.10997](https://arxiv.org/abs/2312.10997) |
| Tier 2 | Robertson & Zaragoza, *The Probabilistic Relevance Framework: BM25 and Beyond*, 2009 (cited at second hand; original not consulted) | — |
