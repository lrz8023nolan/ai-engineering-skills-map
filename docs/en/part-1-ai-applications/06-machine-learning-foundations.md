# 1.6 Machine Learning Foundations

> **Part 1** · Building and deploying AI applications
> **Status** · Complete
> **Last updated** · 2026-09-14

---

## 1. Overview

Machine learning foundations means understanding **how a model learns from data**, and how that learning process determines what it can do, what it cannot, and which direction to push next.

Its value is not the ability to derive algorithms. It is a set of **mental frameworks for making decisions about systems with uncertain output**. Faced with "accuracy is 83%, what now?", machine learning foundations do not hand you a technique — they hand you a diagnostic order: is this a data problem, a model-capacity problem, or something you can only see by looking at the errors? Without that, improvement is guesswork.

Without the skill, the characteristic failure is **spending effort on the wrong axis**: tuning architecture when the data volume is nowhere near sufficient, training beautifully and performing badly once deployed, or staring at 83% with no idea what to change. At a deeper level, you may not even know which trade-offs exist — accuracy, training speed, inference speed and data requirements pull against one another, and optimising one usually costs another.

## 2. In the map

This is the last of the six sub-skills of Part 1. Ng frames it in two layers. The first is that almost everything else rests on it:

> "Modern LLMs are built using machine learning techniques including supervised learning and reinforcement learning. Every engineer I know that's good at building with LLMs also understands machine learning and deep learning at some depth."

The second is that many applications still need machine learning itself — whether a model someone else trained or one you train yourself. His specific requirements:

- Knowing the popular machine learning and deep learning models, and the **trade-offs in accuracy, training speed, inference speed** and so on
- Understanding how to **engineer the data needed** to train and evaluate them
- Commanding **bias/variance, error analysis and data engineering** — which he calls the core mental frameworks for navigating decisions on systems with uncertain output

Source: [AI Engineering Skills Map Part 2, 2026-08-21](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications)

## 3. Core concepts

### 3.1 Three paradigms

| Paradigm | What is learned | Data form | Typical use |
|---|---|---|---|
| **Supervised learning** | A mapping from input to labelled answer | Labelled | Classification, regression, detection, scoring |
| **Unsupervised / self-supervised** | Representations from the structure of the data itself | Unlabelled | Pretraining, embeddings, clustering, anomaly detection |
| **Reinforcement learning** | A policy, adjusted through interaction and reward signals | Reward signal | Sequential decision-making; also used to align models to human preference |

One point worth noting: **modern large models are trained in several mixed stages** — self-supervised pretraining on vast text, then supervised fine-tuning on labelled demonstrations, then preference-based reinforcement learning to align output. Understanding that pipeline is what makes sense of why one base model behaves so differently across alignment stages.

### 3.2 The standard shape of one training run

| Split | Purpose | Key discipline |
|---|---|---|
| **Training set** | Fit the parameters | Never use it to report performance |
| **Validation set (dev set)** | Make decisions — model choice, tuning, direction | You use it repeatedly, so it too "overfits" |
| **Test set** | Used once, at the end, to estimate real performance | Use it more than once and it stops meaning anything |

**A discipline that is easy to miss**: the validation set must reflect **the scenario you will actually deploy into**, not the distribution of the data you happen to have. If training data comes from one source and real users come from another, sample the validation set from real users. Ng makes a strong point of this in *Machine Learning Yearning*: the dev and test sets should come from **the same distribution**, and that distribution should be the one you actually care about.

### 3.3 Bias and variance: two failure modes

This is the most practical diagnostic framework available. Read the training error and validation error together and the problem type falls out:

| Situation | Diagnosis | Principal response |
|---|---|---|
| Training error high, validation error high | **High bias (underfitting)** | Larger model, richer features, train longer |
| Training error low, validation error far above it | **High variance (overfitting)** | More data, regularisation, simpler model |
| Both high, with a large gap between them | **High bias and high variance** | Both need work |
| Both low | Good | Try a larger model to see how much headroom remains |

**Diagnose before acting** is the entire point. It converts the vague feeling "it is not working well" into a specific problem type that points at specific actions.

### 3.4 Avoidable bias and human-level performance

Bias alone does not tell you how much room is left. Ng's move is to establish a **reference point**: human-level performance, or the performance of an existing system.

The logic: if humans achieve 1% error and your model is at 10%, then **avoidable bias is about nine percentage points** — substantial opportunity, worth continued investment. If humans are at 7.5% and you are at 8%, avoidable bias is 0.5 points, so further bias reduction has little return and attention should shift to **variance and data distribution**.

The idea still works today: using "the current best model" or "expert performance" as a reference beats inventing a target number.

### 3.5 Learning curves: more data or a bigger model

A learning curve plots error against training-set size and answers one concrete question: **would more data help?**

| Shape | Reading |
|---|---|
| Validation error still falling clearly with more data | More data helps |
| Validation error has flattened while training error rises | Data is not the bottleneck; capacity or features are |
| A persistent wide gap between the two curves | A variance problem, and more data is the most direct way out |

### 3.6 Mismatched training and test distributions

A source of failure independent of bias and variance: **the training distribution and the real-world distribution differ.**

The signature: both training and validation error are low, but performance drops clearly once deployed. This is neither overfitting nor underfitting — the model learned its distribution well, and production is a different one.

The approach is usually to draw a small sample from the real distribution as a dedicated "training-dev" set to localise the mismatch, then gradually add real-distribution data into training. **The data you diagnose with must come from the deployment distribution** — that is the precondition for handling this class of problem at all.

### 3.7 Error analysis: reading the errors by hand

This is the highest-leverage action in the whole manual, and the method is plain:

1. Take the examples the model **got wrong** from the validation set.
2. **Look at them one by one and classify by hand** — build a column per category, and count how many fall into each.
3. Having worked through a batch (in practice around 100 misclassified examples is enough to build intuition), you get the categories **ranked by frequency**.
4. Attack the **largest** category first.

**Why it is worth doing by hand**: it repeatedly overturns your intuitions. People systematically overestimate the importance of the problem they most recently thought of. After one round of error analysis, "what to do next" usually becomes very clear — and often different from what you had assumed.

This is the same lineage as the eval methodology in manual 1.4 — **look at the data before deciding what to measure**. The objects differ (LLM traces there, misclassified examples here), but the order and the discipline are identical.

### 3.8 The model-selection trade-off matrix

| Dimension | Question to ask |
|---|---|
| Accuracy | How much does this task need? How far from human level? |
| Training cost | How long and how much per run? How often will you retrain? |
| Inference cost | Cost and latency per call? What volume? |
| Data requirement | How much labelled data? How expensive to label? |
| Interpretability | Can you localise a failure? Are there compliance requirements? |
| Deployment constraints | Can it be self-hosted? Is there a hard latency target? |

**There is no "best model", only a model that fits the constraints.** A large model wins on accuracy while costing more in inference, latency and deployment complexity — all part of the trade.

### 3.9 Data engineering: the underrated half

A shift Ng has emphasised repeatedly: **from model-centric to data-centric.** When model choice has largely converged — everyone using a handful of strong models — the headroom moves into data quality rather than architecture.

What needs engineering: label consistency (do annotators read the rule the same way?), coverage (which slices are thin?), label noise (how much of the annotation is wrong?), and data versioning. **In real projects, annotation noise is often the accuracy ceiling, not model capacity.**

## 4. Going deeper

### 4.1 The classical U-curve is not the whole story: double descent

The textbook says: as model complexity grows, performance improves then degrades — a U-shape. In modern deep learning **that picture is incomplete.**

> [Belkin, Hsu, Ma & Mandal, *Reconciling modern machine learning practice and the bias-variance trade-off*, PNAS, arXiv:1812.11118](https://arxiv.org/abs/1812.11118)

The paper's observation: in modern practice, models with far more parameters than training points are trained to **interpolate the data almost exactly**. Classically that would be severe overfitting, yet they perform well on test data.

Plotting risk against model capacity, the real curve looks like this:

| Regime | Behaviour |
|---|---|
| Capacity < number of samples | The classical descending arm — the textbook U-curve describes only this part |
| Capacity ≈ samples (interpolation threshold) | **Risk peaks** — the model barely interpolates, with very high variance |
| Capacity ≫ samples | **Risk falls again** — the "second descent" |

This explains why "bigger models are better" keeps being true in deep learning: **it lies outside the range classical theory covers.** The engineering implication: do not use the textbook U-curve to choose model capacity. In practice, capacity is often not the main source of risk.

### 4.2 Capacity is not generalisation: networks can memorise random labels

A sharper experimental result:

> [Zhang, Bengio, Hardt, Recht & Vinyals, *Understanding deep learning requires rethinking generalization*, ICLR 2017, arXiv:1611.03530](https://arxiv.org/abs/1611.03530)

The design is clean: take a network that reaches 94% test accuracy on CIFAR-10, **replace the true labels with entirely random ones**, and train as usual. The results:

- Training accuracy still reaches **100%** — the network memorises an arbitrary labelling.
- **The loss curves look almost identical** — the training process itself gives no sign that the labels are random.
- Explicit regularisation (dropout, weight decay) **does not prevent it**.
- Replacing the images with fully unstructured random noise works just as well.

The implication is sharp: **model capacity alone cannot explain why a network generalises.** The same architecture both memorises random labels and generalises to 94% on real ones — the difference is not in the model, it is in the data. Any capacity-based generalisation bound (VC dimension, Rademacher complexity) is **vacuous** for such networks: the bound exceeds 1 and says nothing.

For practice: **"the model is too big, so it overfits" is an unreliable inference.** If a model can memorise random labels but fails on real data, the problem is much more likely to be **data quality, labelling or distribution** than capacity.

### 4.3 "More data" has a counter-intuitive side too

Double descent also appears along the **number of training samples** axis, not just capacity.

> [Nakkiran et al., *Deep Double Descent: Where Bigger Models and More Data Hurt*, arXiv:1912.02292](https://arxiv.org/abs/1912.02292)

The title is the finding — **in a particular regime, increasing either model size or data volume can make performance worse.** It runs against intuition, but the mechanism matches §4.1: when model and data sit near the interpolation threshold, the system is at its least stable.

The practical implication: **model comparisons on small datasets are unreliable.** With too little data, "A beats B" may only reflect that both sit near the threshold, and the ranking can invert at a different data volume. When doing model selection, compare at a scale close to the real one.

### 4.4 The discipline of error analysis

Expanding §3.7 into an executable procedure, several rules deserve to stand alone:

| Rule | Reason |
|---|---|
| **Look only at the errors, not the correct cases** | Correct examples carry no improvement signal |
| **Record one dominant problem per example** | An example may have several faults; taking the main one keeps the tally convergent |
| **Classify first, diagnose causes later** | Jumping to "why" is an unbounded rabbit hole and biases what you notice |
| **Tally frequencies in a table** | The purpose is ranking, not record-keeping |
| **Use a sufficient sample (~100)** | Too few and you are counting accidents, not patterns |

**The largest category is usually not the one you first thought of.** That is the entire reason to do this by hand.

### 4.5 Data-centric: why the effort moves to data

Where model choice has converged — broadly where we are today — the distribution of available gains has shifted:

| Action | Typical return |
|---|---|
| Switch to a stronger model | Diminishing returns, rising cost |
| **Clean up label noise** | Can raise the accuracy ceiling directly |
| **Add data for a missing slice** | Usually repairs one class of systematic failure |
| **Unify annotation rules** | Removes self-contradiction from the training signal |

Ng calls this the shift from model-centric to data-centric, with the core claim that **once models are strong enough, quality and coverage of data determine system performance, not model choice.**

One precondition deserves emphasis: **you cannot fix a mislabelled example with a better model.** If the label is wrong, any model will be taught wrongly — and it will also look "wrong" on the validation set, leading you to misjudge the model's capability.

### 4.6 Why human-level performance is a good baseline

Three advantages to using human performance as the reference point:

1. **It is measurable**, unlike an imagined ideal.
2. **It splits bias into avoidable and unavoidable parts**, telling you directly how much room is left.
3. **It scales with task difficulty.** Bayesian-optimal error is close to perfect for every task, but how far human performance sits from it varies — speech recognition is high (humans tire), and so is image classification (humans misjudge too, mistaking a husky for a wolf).

One caveat: **in domains where human performance is already below the model's (some large-scale knowledge question answering), the baseline loses its reference value.** Use a stronger existing system instead.

### 4.7 Why these frameworks still apply in the LLM era

A fair question: with large models, are these classical frameworks still needed? Yes — and they map cleanly:

- **Bias/variance** becomes "is the prompt and context sufficient" (bias) versus "is it overfitting to one phrasing and failing on a reworded version" (variance).
- **Error analysis** becomes the loop from manual 1.4 — read traces, open-code, rank by frequency. Structurally identical.
- **Data-centric** becomes "the quality of what retrieval brings in".
- **Distribution mismatch** becomes "the eval set and the real user population differ".

**The root reason has not changed**: as long as output is uncertain, you can only make progress by measuring — and the central question of measurement is what to look at, how, and what to change as a result. Machine learning foundations supply the vocabulary for that judgement.

## 5. Capability checkpoints

1. I can distinguish supervised, unsupervised and reinforcement learning, and describe the multi-stage training pipeline of a modern large model.
2. I can explain the roles of the training, validation and test sets, and why the validation distribution must reflect the real deployment scenario.
3. I can read a combination of training and validation error to classify the problem as high bias, high variance or both, and name the corresponding actions.
4. I can explain avoidable bias and how to quantify it using human-level performance.
5. I can use the shape of a learning curve to judge whether more data would still help.
6. I can explain how train-test distribution mismatch differs from overfitting and underfitting, and how to diagnose it.
7. I can conduct one round of error analysis following the discipline, and explain why categories are ranked by frequency rather than by intuition.
8. I can explain the double descent phenomenon, and why the textbook U-curve is insufficient for choosing model capacity.
9. I can state what the "networks fit random labels" result implies, including its effect on the inference "the model is too big".
10. I can explain the data-centric shift and its precondition, including why a mislabelled example cannot be fixed by the model.

## 6. Sources

| Type | Source | Link |
|---|---|---|
| Source letter | Andrew Ng, *AI Engineering Skills Map Part 2*, 2026-08-21 | [link](https://www.deeplearning.ai/the-batch/he-ai-engineering-skills-map-in-detail-building-and-deploying-ai-applications) |
| Tier 1 — author's own book | Andrew Ng, *Machine Learning Yearning*, 2018 (free) — the full methodology for error analysis, bias and variance, learning curves, human-level performance, distribution mismatch and error analysis by parts | [deeplearning.ai/resources](https://www.deeplearning.ai/resources) |
| Tier 1 — author's own material | Andrew Ng, *MLOps: From Model-centric to Data-centric AI* (slides) | [deeplearning.ai/resources](https://www.deeplearning.ai/resources) |
| Tier 1 — paper | Belkin, Hsu, Ma & Mandal, *Reconciling modern machine learning practice and the bias-variance trade-off*, PNAS 2019 | [arXiv:1812.11118](https://arxiv.org/abs/1812.11118) |
| Tier 1 — paper | Zhang, Bengio, Hardt, Recht & Vinyals, *Understanding deep learning requires rethinking generalization*, ICLR 2017 | [arXiv:1611.03530](https://arxiv.org/abs/1611.03530) |
| Tier 1 — paper | Nakkiran et al., *Deep Double Descent: Where Bigger Models and More Data Hurt*, 2019 | [arXiv:1912.02292](https://arxiv.org/abs/1912.02292) |
| Tier 1 — paper | Sculley et al., *Hidden Technical Debt in Machine Learning Systems*, NeurIPS 2015 | [paper](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf) |

> The description of large-model multi-stage training in §3.1 (self-supervised pretraining → supervised fine-tuning → preference-based reinforcement learning) is a synthesis of common practice rather than a citation to a single source. Verify details against the relevant technical reports.
