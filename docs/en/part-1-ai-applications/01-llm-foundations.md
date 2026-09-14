# 1.1 LLM Foundations

> **Part 1** · Building and deploying AI applications
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

LLM foundations means understanding **what units a large language model actually reads and writes**, **how it produces output one step at a time**, and how that machinery determines which tasks it can be trusted with and which it will fail at in predictable ways.

It matters because almost every engineering decision about a model rests on this: how large a model does this task need? How much can I fit in the context? Why did changing the wording change the answer? Why is it more expensive in some languages? Why did it still get the wrong answer when the relevant document was right there in the context? None of these questions is answered reliably by trial and error — the answers live in the mechanism.

Without this, the characteristic failures run in two directions. First, **fighting the model where it structurally cannot help**: asking it to count letters in a word, do exact arithmetic, or retrieve a number buried in the middle of a hundred pages — then attributing the failure to insufficient model capability and trying a bigger one. Second, **over-engineering where the model was already fine**: building a retrieval pipeline for a simple classification task, or reaching for fine-tuning first, spending cost and complexity on a problem that did not need it.

## 2. In the map

This is the first of the six sub-skills of Part 1, and the one Ng lists first when he expands the Part. His scope for the skill is broad:

- Understanding how a model tokenizes input and generates output
- Using that to judge **when to count on it and when it may fail**
- Knowing **when to use a multimodal model**
- Deciding what to put in the context window and what to leave out
- Reasoning about **cache hits, knowledge cutoff, reasoning effort and sampling parameters**, and when to use special features such as tool calling
- Choosing the right model, or mix of models
- Applying specialised techniques where needed, such as **fine-tuning or self-hosting**

On why it pays off, Ng is direct:

> "Understanding how large language models tokenize input and generate output allows you to understand when to count on them and when they may fail."

Source: [AI Engineering Skills Map Part 2, 2026-08-21](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications)

## 3. Core concepts

### 3.1 One generation step: autoregression

A large language model does not compose a whole reply and then emit it. It predicts **one next token**, appends that token to the input, and predicts again — repeatedly.

| Step | What happens |
|---|---|
| 1. Tokenize | The input text is split into a sequence of tokens |
| 2. Forward pass | The model computes internal representations and produces raw scores (logits) over the whole vocabulary |
| 3. Sample | The sampling parameters turn those logits into a probability distribution, and one token is chosen |
| 4. Append | The chosen token is added to the end of the sequence |
| 5. Repeat | Back to step 2, until a stop condition or a length limit is hit |

Two consequences worth committing to memory:

- **There is no global plan for the output.** Each step optimises only the next token. Output that reads well locally but drifts as a whole is normal behaviour of the mechanism, not a bug.
- **The recomputation is heavy.** Producing each token requires reprocessing everything before it, unless caching is in play (§3.5). This is why long context is both slow and expensive.

### 3.2 Tokenization

Models do not read characters, and they do not read words. They read **tokens** — subword units produced by a tokenizer against a fixed vocabulary. The dominant method is **BPE (byte-pair encoding)**: start from bytes or characters and repeatedly merge the most frequent adjacent pair into a new unit, until the vocabulary reaches its target size.

The thing to remember is that **a token is not a character and not a word**, and the exchange rate varies by language:

| Input | Approximate tokens | Why |
|---|---|---|
| The English word `the` | 1 | Frequent words often occupy a single token |
| The English word `tokenization` | 2–3 | Rare words get split into fragments |
| A Chinese sentence | Roughly one per character, sometimes more | Chinese is less efficient under most English-dominated vocabularies |
| A random number string `84930517` | Several | Digits tend to be split irregularly |

### 3.3 Context window

The context window is the maximum number of tokens a model can handle in one pass. Input and output **share** that budget.

| Point to understand | Implication |
|---|---|
| Input and output share the budget | A longer answer leaves less room for input |
| Attention cost is quadratic | Doubling sequence length roughly quadruples the attention computation ([Vaswani et al., 2017](https://arxiv.org/abs/1706.03762)) |
| Fitting is not the same as using | See §4.2, lost in the middle |
| Window size is a hard constraint on model choice | But a bigger window does not automatically mean better results |

### 3.4 Sampling parameters

The model outputs a probability distribution; the sampling parameters decide **how a token is drawn from it**.

| Parameter | What it does | Practical effect |
|---|---|---|
| **Temperature** | Scales the logits, controlling how peaked the distribution is | Lower → more deterministic and conservative; higher → more varied and more prone to drift |
| **top-p** (nucleus sampling) | Samples only from the smallest set of tokens whose cumulative probability reaches *p* | Candidate set expands and contracts with the model's confidence |
| **top-k** | Samples only from the *k* highest-probability tokens | Candidate set is a fixed size, regardless of the distribution's shape |
| **Greedy decoding** | Always takes the single most probable token | Deterministic and fast, but prone to repetitive loops |
| **seed** | Fixes the random seed | Improves reproducibility, but does not guarantee it across versions |

The motivation behind top-p is worth remembering: Holtzman et al. showed that for one and the same well-trained model, **changing only the decoding strategy** dramatically changes output quality — maximisation-based decoding (greedy, beam search) produces bland and repetitive text. Human writing does not follow a maximum-likelihood trajectory.

### 3.5 Prompt caching and the economics of context

If every generated token requires reprocessing everything before it, a repeated prefix is pure waste. Prompt caching addresses this by **storing the computed state of a prefix — the KV cache, the key-value states of the attention mechanism** — and reusing it when a later request shares an identical prefix.

The mechanism is **exact prefix matching**:

| Point | Detail |
|---|---|
| Matching is on the prefix | Comparison runs from the start, token by token; the first difference invalidates the rest |
| Static content must come first | System instructions, tool definitions and long documents at the beginning; user input and timestamps at the end |
| There is a minimum length | OpenAI's implementation requires 1,024 tokens before automatic caching applies ([official docs](https://platform.openai.com/docs/guides/prompt-caching)) |
| Caches expire | Typically minutes; late in a long session the cache may already be cold |
| Providers differ in configuration | OpenAI caches automatically and reports `cached_tokens`; Anthropic requires explicit `cache_control` markers |

### 3.6 Knowledge cutoff and external information

A model knows only what was in its training data, up to a cutoff date. The world after that point does not exist for it. This is not something you can talk your way around with a clearer question.

Any **time-sensitive fact** — today's price, the current version number, the latest policy — must therefore be injected from outside, through retrieval or directly by the application. It is equally important to note that a model does not know where its own knowledge ends: it will answer questions past its cutoff in exactly the same confident register.

### 3.7 Reasoning effort

Newer models expose an adjustable reasoning effort: before answering, they generate an internal chain of reasoning (thinking tokens), trading more compute for higher accuracy.

| Setting | When |
|---|---|
| High | Multi-step maths, complex logic, debugging — the gain is real |
| Low | Simple extraction, format conversion, classification — high effort only adds latency and cost |

This is a routine lever in model and parameter selection: not every request deserves maximum effort.

### 3.8 Multimodal input and tool calling

Both are "special features" whose use depends on whether the task genuinely needs them.

- **Multimodal:** use it when the information itself lives in an image, an audio clip or a video (reading a chart, looking at a screenshot, listening to speech). The cost is that token consumption is usually far higher than for equivalent text, and reliability on visual detail varies widely between models.
- **Tool calling:** use it when the model needs **an external action or live data**. Its significance is not only data retrieval — it changes the model from "must already know" to "can go and look", which is one of the main defences against hallucination.

### 3.9 Choosing and adapting a model

Work through these in order; do not skip steps.

| Decision | The question to ask first |
|---|---|
| Which model | How much reasoning does the task need? What is the latency budget? What cost is acceptable? How large a context? |
| A mix of models | Routing simple requests to a small model and hard ones to a large one is usually the most direct cost reduction available |
| Add retrieval or add a tool | Is the need "more context" or "an external action / live data"? |
| Fine-tune or not | Have prompting, context and tools all been exhausted first? |
| Self-host or not | Is there a hard data-compliance requirement, or enough volume to amortise the operational cost? |

## 4. Going deeper

### 4.1 Tokenization is the root of many "inexplicable" failures

Model reasoning operates on tokens; the operations people care about operate on characters or words. When the two do not line up, failures appear that look arbitrary:

| Symptom | Mechanistic cause |
|---|---|
| Cannot count letters in a word, or characters in a sentence | The model sees tokens, not characters; the letters in `strawberry` are not separate countable units |
| Spelling and letter-level transformations are unreliable | Same cause: letters are not the model's atomic units |
| Exact arithmetic is error-prone | Digits are split into irregular fragments — `1234` may not be the number one thousand two hundred and thirty-four, but several semantically empty pieces |
| Non-English languages cost more | The same meaning often needs more tokens under an English-dominated vocabulary; cost and latency rise together |
| A different space changes the answer | The token boundaries moved, so the sequence the model sees is different |

These are limitations of **representation**, not of capability. Knowing this tells you to route around them — make the model split the characters itself, use a tool or write code to do the arithmetic — rather than trying a stronger model on the same task.

### 4.2 Lost in the middle

A repeatedly confirmed effect: putting the same relevant passage at different positions in the context changes performance systematically.

> Liu et al., *Lost in the Middle: How Language Models Use Long Contexts*, [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) (published in TACL 2024)

| Finding | Detail |
|---|---|
| Performance follows a **U-shaped curve** | Best when the information sits at the very beginning or very end of the context, degrading significantly in the middle |
| The drop can be large | In the extreme case, GPT-3.5-Turbo with 20–30 retrieved documents performed **worse than with no documents at all** (closed-book) |
| Longer-context models did not fix it | When the input fits inside both the short and long variants, their position curves nearly coincide — **a larger window is not the same as using it well** |
| It is not purely a semantics problem | The same mid-context dip appears on a semantics-free UUID key-value lookup task, so part of the cause is retrieval rather than reasoning |

The engineering implication is direct: **do not dump everything you retrieved into the context.** Curate, or place the most important material at the start or the end, or reduce the number of distractors. More documents is not automatically better, and can be worse.

### 4.3 Sampling: why temperature 0 is still not reproducible

Even with temperature set to 0 (equivalent to greedy decoding), the same request can return different results at different times. Among the reasons: the floating-point accumulation order in parallel computation is not fixed; server-side batching and hardware differ; model versions are silently updated; and some implementations introduce randomness regardless.

The practical conclusion: **do not build reproducibility on decoding parameters.** Where you need stability, measure a fixed eval set repeatedly under varying conditions and look at the distribution rather than any single result.

One counter-intuitive point alongside it: greedy decoding, despite being "most deterministic", is the most prone to repetitive loops and bland output — which is precisely why nucleus sampling was proposed (Holtzman et al., *The Curious Case of Neural Text Degeneration*, [arXiv:1904.09751](https://arxiv.org/abs/1904.09751), ICLR 2020; their experiments used p = 0.95 as the main setting).

### 4.4 Why your cache hit rate is zero

This is the most common and most easily missed trap in prompt caching, and it deserves its own heading:

**If a single token of the prompt's prefix differs, everything after it is invalidated.**

So all of the following, each of which looks reasonable, drives the cache hit rate straight to zero:

- Inserting the current date or time at the **very start** of the system prompt
- Injecting a user name, session ID or request ID near the beginning
- Placing a configuration value that changes per request in an early position
- Appending every tool-call result to an unbounded conversation, so the prefix keeps changing

The fix is structural. Split the prompt into three layers ordered by **increasing rate of change**: long-lived static instructions first, semi-static documents and tool definitions in the middle, this request's dynamic input last. The same ordering is portable across OpenAI and Anthropic — OpenAI's documentation states it plainly: "place static content like instructions and examples at the beginning of your prompt, and put variable content, such as user-specific information, at the end."

### 4.5 Fine-tuning is usually the last resort, not the first

Fine-tuning looks like the serious option, but on most projects the order should be: **prompting → context/retrieval → tools → evaluation to locate the failure → and only then fine-tuning.**

The reason is that the cost is not only the forward pass. You have to construct paired training data, maintain a training pipeline, re-validate after every base-model update, and accept the risk of capability regression (catastrophic forgetting). And if the failure mode actually came from not supplying the right context, fine-tuning will not fix it.

When it is genuinely warranted, a parameter-efficient method such as LoRA ([Hu et al., 2021](https://arxiv.org/abs/2106.09685)) is more common than full fine-tuning.

### 4.6 Hallucination and calibration

When a model does not know an answer, it usually does not say so — it produces a formally complete but factually wrong response. This follows from the training objective: the model is optimised to generate **a plausible continuation**, and "I don't know" is often not the most probable continuation in context.

Research indicates that models internally **encode information about their own correctness**, but do not express it by default:

> Kadavath et al., *Language Models (Mostly) Know What They Know*, [arXiv:2205.14334](https://arxiv.org/abs/2205.14334)

Three engineering responses are common. First, **give the model something to ground on** — retrieval, reference material, a requirement to cite sources. Second, **give it a way out** — explicitly permit and demonstrate an "uncertain" answer format, for example by requiring a structured confidence field. Third, **verify externally** — use a tool or code to check the model's conclusion rather than adopting it directly.

## 5. Capability checkpoints

1. I can describe one full loop of autoregressive generation, and explain why locally fluent output that drifts overall is normal behaviour of the mechanism rather than an anomaly.
2. I can explain the difference between tokens, characters and words, and use it to account for why models are unreliable at letter counting, spelling and exact arithmetic.
3. I can state that the context window budget is shared between input and output, and explain why a larger window does not mean more accurate use of it.
4. I can explain the trade-offs between `temperature`, `top-p`, `top-k` and greedy decoding, and choose a set of sampling parameters for a given task.
5. I can explain the prefix-matching mechanism of prompt caching, and name at least three practices that reduce the cache hit rate to zero.
6. I can restate the core finding of "lost in the middle" and say what it implies for how a retrieval system should be designed.
7. I can lay out the decision chain — prompting → context → tools → evaluation → fine-tuning — in order, and explain why fine-tuning should not come first.
8. I can describe at least three engineering techniques for reducing hallucination, and state the mechanism each one relies on.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 2*, 2026-08-21 | [link](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications) |
| Source letter | Andrew Ng, *The AI Engineering Skills Map* (overview), 2026-08-14 | [link](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map) |
| Tier 1 — paper | Vaswani et al., *Attention Is All You Need*, 2017 | [arXiv:1706.03762](https://arxiv.org/abs/1706.03762) |
| Tier 1 — paper | Holtzman et al., *The Curious Case of Neural Text Degeneration*, ICLR 2020 | [arXiv:1904.09751](https://arxiv.org/abs/1904.09751) |
| Tier 1 — paper | Liu et al., *Lost in the Middle: How Language Models Use Long Contexts*, TACL 2024 | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) |
| Tier 1 — paper | Kadavath et al., *Language Models (Mostly) Know What They Know*, 2022 | [arXiv:2205.14334](https://arxiv.org/abs/2205.14334) |
| Tier 1 — paper | Hu et al., *LoRA: Low-Rank Adaptation of Large Language Models*, 2021 | [arXiv:2106.09685](https://arxiv.org/abs/2106.09685) |
| Tier 1 — official | OpenAI, *Prompt caching* (official documentation) | [platform.openai.com](https://platform.openai.com/docs/guides/prompt-caching) |
| Tier 2 — practitioner | PromptHub, *Prompt Caching with OpenAI, Anthropic, and Google Models* (cross-provider comparison) | [prompthub.us](https://www.prompthub.us/blog/prompt-caching-with-openai-anthropic-and-google-models) |
