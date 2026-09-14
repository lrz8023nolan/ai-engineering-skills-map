# AI Engineering Skills Map — An Open Learning Protocol

A structured, source-backed study protocol built on Andrew Ng's **The AI Engineering Skills Map** (August–September 2026).

The goal is not to archive four newsletters. It is to turn a high-level skills taxonomy into something you can actually work through: every sub-skill gets a plain-language entry point, the concepts you need to know, the principles underneath them, and a set of checkpoints to verify you have learned it.

[中文说明 →](README.zh-CN.md)

---

## Why this exists

In August 2026, Andrew Ng and DeepLearning.AI published a map of the most important AI engineering skills, distilled from over 10,000 job postings, dozens of expert interviews, and survey data. It names four top-level skills and breaks them into 20 sub-skills.

The map is a good index but a thin one — each sub-skill gets one or two paragraphs. It tells you *what* to learn, not enough to actually learn it.

This repository fills that gap. Each sub-skill gets its own manual that keeps the map's framing as an anchor and then builds real depth on top of it, with first-hand sources (papers, official docs, practitioner write-ups) cited throughout.

## The pipeline

The original letters form the backbone. Study order follows them:

| Stage | What it covers |
|---|---|
| 0 | The overview letter — the map itself, four skills at a glance |
| 1 | **Building and deploying AI applications** — 6 sub-skills |
| 2 | **Software engineering fundamentals** — 5 sub-skills |
| 3 | **Using coding agents** — 5 sub-skills |
| 4 | **Shaping the build** — 4 sub-skills |

Within each stage, work through the sub-skills in order. Every manual follows the same six-part template, so the format becomes familiar fast and you can focus on the content.

## Repository layout

```
.
├── README.md            # this file (English)
├── README.zh-CN.md      # 中文说明
├── LICENSE
└── docs/
    ├── PROTOCOL.md      # what this protocol is, how to use it, the template spec
    ├── MAP.md           # the 4 × 20 skills map, with links to the source letters
    ├── GLOSSARY.md      # bilingual terminology table (English / 中文)
    ├── en/              # English manuals
    └── zh/              # 中文手册
```

Start with [`docs/PROTOCOL.md`](docs/PROTOCOL.md) to understand the format, then [`docs/MAP.md`](docs/MAP.md) for the full map and current status.

## Document format

Every manual has six parts:

1. **Overview** — what this skill is and why it matters, explained without jargon
2. **In the map** — what the source letter actually says, with a link
3. **Core concepts** — the menu of things you need to know
4. **Going deeper** — the principles, known biases, and non-obvious findings
5. **Capability checkpoints** — statements you should be able to make; use them to self-test
6. **Sources** — everything cited, with links

Parts 1–2 are the on-ramp. Part 3 is the working knowledge. Part 4 is where the depth lives. Parts 5–6 make it a protocol rather than a set of articles.

## Status

Work in progress. See [`docs/MAP.md`](docs/MAP.md) for per-sub-skill completion status.

## Contributing

Issues and pull requests are welcome — especially corrections to sources, or notes where a term is translated imprecisely. Accuracy of sourcing matters more here than volume.

## Using this material

In plain language. The binding terms are in [`LICENSE`](LICENSE).

**You can** read, copy, translate, quote with attribution, fork and adapt this material for **non-commercial** purposes — your own study, teaching, research, or internal evaluation. If you publish a translation or a modified version, it has to carry the same licence and credit this repository.

**You cannot** sell it, put it behind a paywall, bundle it into a commercial product or paid service, or relicense a derivative under more restrictive terms. Nor can you present it as official material from Andrew Ng or DeepLearning.AI — it is not.

**Commercial use** is available separately. Open an issue if you want to discuss it.

One thing to keep in mind if you reuse this: the repository holds two kinds of material. The manuals are the author's original work and carry the licence above. The sentences quoted from the source letters belong to DeepLearning.AI and are **not** covered by this repository's licence — they are attributed in place, and their appearing here does not make them yours to reuse.

## Attribution and license

This is an **independent, unofficial** study resource. It is not affiliated with, endorsed by, or sponsored by Andrew Ng or DeepLearning.AI.

*The AI Engineering Skills Map* and the associated letters are © DeepLearning.AI. This repository paraphrases and briefly quotes them for study purposes and links to each original letter; it does not reproduce them in full. All original material written for this repository is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). See [`LICENSE`](LICENSE) for the full breakdown.
