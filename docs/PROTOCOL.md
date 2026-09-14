# The Protocol

This document defines what this repository is, how it should be used, and the rules every manual in it must follow. Read this before contributing or before starting to work through the material.

---

## 1. Purpose

Turn a high-level skills taxonomy into a set of manuals that can actually be studied. The source map says *what* to learn; this protocol specifies *how each manual must be written* so that the result is both approachable and technically credible.

Two failure modes it is designed to avoid:

- **Too shallow** — restating the source letters. They are roughly 3,000 words total across five letters; restating them adds nothing. Every manual must add real depth from first-hand sources.
- **Too dense** — dumping primary literature without orientation. Depth without an on-ramp is not learnable.

## 2. Scope

**Belongs here:** concepts, techniques, principles, known pitfalls, and the sources that establish them.

**Does not belong here:** personal project details, employer-specific information, unpublished internal data, or anything that would not survive review by the authors of the cited sources. This repository is public; write only what can be public.

## 3. The unit of work

One manual per sub-skill. There are 20 sub-skills, therefore 20 manuals (each in two languages). No manual covers two sub-skills, and no sub-skill is split across two manuals.

## 4. Template specification

Every manual has exactly these six sections, in this order.

### 4.1 Overview
**Purpose:** the on-ramp. A reader with no background should finish this section knowing what the skill is and why it matters.
**Include:** a one-sentence definition; the problem the skill solves; what goes wrong without it.
**Exclude:** jargon, acronyms, and citations. If a term must appear, gloss it inline.
**Length:** 1–2 short paragraphs.

### 4.2 In the map
**Purpose:** anchor the manual to the source. Prevents drift from what the map actually claims.
**Include:** a faithful paraphrase of what the source letter says about this sub-skill; **at least one direct quotation from the letter**; a link to the letter. Quoting the source is mandatory — it anchors the manual to what the map actually claims and lets the reader verify it.
**Exclude:** long quotation. See §5.3.
**Length:** 1–2 paragraphs of paraphrase plus one or two quotations.

### 4.3 Core concepts
**Purpose:** working knowledge. The menu of things you need to know in order to act.
**Include:** concepts, categories, and criteria of choice — typically as tables or short lists. Where a concept has a decision attached ("use X when…"), state the decision rule explicitly.
**Exclude:** historical context; general background that does not change what the reader does.
**Length:** the bulk of the manual's structure; length varies by sub-skill.

### 4.4 Going deeper
**Purpose:** the principles, mechanisms, and non-obvious findings. This is where the manual earns its credibility.
**Include:** why things work the way they do; measured results with numbers where they exist; documented biases and their mitigations; claims that contradict common practice.
**Exclude:** speculation presented as fact. If a claim is contested or empirical only in a narrow setting, say so.
**Length:** the manual's longest section.

### 4.5 Capability checkpoints
**Purpose:** make it self-testable. This is what turns a set of articles into a protocol.
**Include:** 5–8 statements, each beginning "I can…", each verifiable by producing something (an explanation, a design decision, a list). Every checkpoint must be answerable from §4.3 or §4.4.
**Exclude:** vague items ("understand X", "be familiar with Y"). If you cannot tell whether you have met it, rewrite it.
**Length:** 5–8 items.

### 4.6 Sources
**Purpose:** auditability.
**Include:** every source cited in the body, as a table with type, reference, and link.
**Exclude:** sources you did not actually use.
**Length:** as needed.

---

## 5. Evidence rules

### 5.1 Source hierarchy
| Tier | What counts | Use for |
|---|---|---|
| **1. Primary** | Peer-reviewed papers; official vendor or standards documentation; the author's own writing | All factual claims that matter |
| **2. Practitioner** | Write-ups by named engineers grounded in production experience | Method, workflow, trade-offs |
| **3. Secondary** | Surveys, course material, summary blog posts | Orientation only — verify against tier 1 before stating as fact |

### 5.2 Citation rules
- Every non-obvious factual claim carries a source.
- **Quoting the source letter is mandatory.** Every manual must contain at least one direct quotation from its source letter — in the original English, with a link — regardless of how the rest of the manual is sourced. This is the one sourcing requirement that is never relaxed.
- Cite inline as a Markdown link, and list the source again in §4.6.
- Never cite a source you have not read. Never cite a secondary source for a claim it is itself only summarising.
- Where a number is quoted (an accuracy figure, an agreement rate), give the source that measured it.
- There is no fixed minimum number of tier-1 sources. Research-led sub-skills should carry several; practitioner-led sub-skills (much of Part 4) may legitimately rest mainly on tier-2 material, provided it is labelled as such.

### 5.3 Quotation and copyright
The source letters are © DeepLearning.AI. This repository is unofficial and must stay clearly on the right side of that line:
- Quote at most one or two sentences per letter per manual, always attributed and linked.
- Paraphrase everything else.
- Never reproduce a letter in full, in either language, including as a translation. Translation is not a defence.
- Never imply endorsement by, or affiliation with, Andrew Ng or DeepLearning.AI.
- Original prose written for this repository is licensed CC BY-NC-SA 4.0 (see `LICENSE`). Quoted third-party text is excluded from that licence and keeps its own attribution, so it can never be relicensed along with the rest of the repository.

### 5.4 Uncertainty
State the strength of a claim. If something is a reasonable inference, say "likely" or "in practice". If the evidence is limited to one setting, say so.

---

## 6. Terminology rules

- The bilingual glossary in [`GLOSSARY.md`](GLOSSARY.md) is authoritative. If a manual needs a term that is not there, add it to the glossary in the same change.
- First mention in a manual uses the Chinese term with the English original, formatted as 中文（English）. Thereafter use the Chinese term alone.
- Where no Chinese translation is standard in practice, do **not** invent one. Keep the English term and record that decision in the glossary. `eval`, `grader`, and `trace` are current examples.

## 7. Bilingual rules

- `docs/en/` and `docs/zh/` mirror each other. Every manual exists in both.
- The two versions must be semantically equivalent. They need not be literal translations, but neither may add or omit substance.
- Terminology must match the glossary in both languages.
- A change to one language version requires the corresponding change in the other.

## 8. File naming and placement

```
docs/{en|zh}/part-{n}-{part-slug}/{nn}-{sub-skill-slug}.md
```

- `nn` is the sub-skill number from [`MAP.md`](MAP.md), zero-padded, so that alphabetical order matches study order.
- Slugs are lowercase ASCII with hyphens. No spaces, no non-ASCII characters in paths.

Example: `docs/en/part-1-ai-applications/04-evaluation-driven-development.md`

## 9. Definition of done

A manual is complete when all of the following hold:

- [ ] All six sections present, in order, in both languages
- [ ] 5–8 capability checkpoints, each verifiable
- [ ] At least one direct quotation from the source letter, in the original English, with a link
- [ ] Sources appropriate to the sub-skill: tier-1 wherever a factual claim depends on one, tier-2 labelled as such
- [ ] Every factual claim has a source
- [ ] Terminology consistent with `GLOSSARY.md`, newly needed terms added there
- [ ] Quotation limits in §5.3 respected
- [ ] Entry in `MAP.md` marked complete

## 10. Adding a manual

1. Check the sub-skill is listed in `MAP.md` and not already done.
2. Gather sources first — tier-1 where the sub-skill has a research or standards base, tier-2 where it is practitioner-led. If a claim cannot be sourced at all, do not make it.
3. Draft in one language, following the template.
4. Write the counterpart version.
5. Add any new terms to `GLOSSARY.md`.
6. Update `MAP.md`.
