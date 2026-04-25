# Paper Ingestion Process

**Status:** Draft for review
**Date:** 2026-04-25
**Companion to:** `no-magic-ai-expansion-strategy.md`
**Scope:** The end-to-end process by which a new research paper enters the `no-magic-ai` ecosystem — from discovery through summary, routing, implementation, and cross-repo synchronization.
**Audience:** Maintainers, contributors, and agentic tooling (notably `apprentice`) that operates on the `no-magic-ai` repos.

---

## 0. Why this document exists

The `no-magic-ai` org is paper-first by charter. Every implementation cites a paper; every lesson references a paper; every curriculum path is a sequence of papers rendered as code. Without an explicit ingestion process, the ecosystem accumulates inconsistent, unranked, and silently-duplicated content.

This SOP makes the process deterministic enough for a human to run in an afternoon and legible enough for an agent to execute with human review gates. It governs how papers move through the following stages:

```
Discovery → Triage → Summary → Route → Implement → Sync → Lifecycle
```

The pipeline lives in the **`no-magic-papers`** repo (which becomes the system of record). That single repo hosts both paper cards (`papers/{slug}.md`) and optional prose lessons (`lessons/{slug}.md`) sharing the canonical slug. Other repos reference paper cards by slug; none of them owns the authoritative copy.

**Governance mode:** discovery and triage are performed by the maintainer personally, on demand, whenever an interesting paper surfaces. There is no weekly cadence, no SLA clock, no intake queue to drain. All contributions to paper cards, lessons, and implementations are manually reviewed by the maintainer before merge.

---

## 1. System diagram

```
                 ┌──────────────────────────────────────────┐
                 │           SOURCES (Stage 0)              │
                 │  arXiv · NeurIPS · ICML · ICLR · blog    │
                 │  contributor suggestions · citations     │
                 └───────────────┬──────────────────────────┘
                                 ▼
                 ┌──────────────────────────────────────────┐
                 │            TRIAGE (Stage 1)              │
                 │    in / out / parking lot                │
                 └───────────────┬──────────────────────────┘
                                 ▼
                 ┌──────────────────────────────────────────┐
                 │           SUMMARY (Stage 2)              │
                 │    paper card in no-magic-papers         │
                 │    status: summarized                    │
                 └───────────────┬──────────────────────────┘
                                 ▼
                 ┌──────────────────────────────────────────┐
                 │             ROUTE (Stage 3)              │
                 │    catalog? viz? lesson? path?           │
                 │    status: backlog-implement | reference │
                 └───────────────┬──────────────────────────┘
                                 ▼
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
      ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐
      │  IMPLEMENT   │   │  VISUALIZE   │   │    LESSONIFY     │
      │  (no-magic)  │   │(no-magic-viz)│   │ (no-magic-papers │
      │              │   │              │   │     /lessons/)   │
      └───────┬──────┘   └───────┬──────┘   └────────┬─────────┘
              └──────────────────┼────────────────────┘
                                 ▼
                 ┌──────────────────────────────────────────┐
                 │              SYNC (Stage 5)              │
                 │  catalog.json · paper ↔ script refs      │
                 │  path membership · status updates        │
                 └───────────────┬──────────────────────────┘
                                 ▼
                 ┌──────────────────────────────────────────┐
                 │           LIFECYCLE (Stage 6)            │
                 │  deprecate · replace · archive           │
                 └──────────────────────────────────────────┘
```

---

## 2. Stage 0 — Discovery

### 2.1 Intake model

Papers are surfaced by the maintainer when they encounter interesting work — reading, conference proceedings, citations, social media, colleagues' recommendations, or following threads from existing paper cards. There is no automated scanner, no queue target, no intake SLA. The ingestion pipeline runs when the maintainer decides a paper is worth the maintainer's time.

Contributor suggestions are welcome via GitHub issue and are treated as one of many surfacing channels — the maintainer still personally reviews and triages each suggestion before the paper enters the pipeline.

| Channel | Typical frequency |
|---|---|
| Maintainer-surfaced (reading, conferences, social) | Irregular, as-encountered |
| Contributor suggestions via issue | Irregular, maintainer-reviewed |
| Citations followed from existing paper cards | As-encountered during card authoring |

### 2.2 Intake record

When the maintainer decides a paper is worth summarizing, they either open a lightweight tracking issue in `no-magic-papers` or proceed directly to drafting the paper card (Stage 2). The tracking issue is optional — useful when triage is not yet decisive or when a contributor is suggesting the paper.

Optional intake issue template:

```yaml
title: "[Triage] {paper-title}"
body:
  - paper_title
  - authors
  - arxiv_id_or_doi
  - venue_and_year
  - discovered_via     # reading, conference, contributor, citation-chain
  - why_relevant       # 2-3 sentences
  - tentative_paper_slug   # e.g. rome, grpo, turboquant (paper-canonical, no micro* prefix)
  - suggested_themes   # from §3.5 taxonomy
  - suggested_tier     # 01-foundations | 02-alignment | 03-systems | 04-agents | n/a
```

### 2.3 Intake hygiene

- **No paper is ingested without a human-readable `why_relevant`.** If the maintainer cannot articulate why this paper belongs in `no-magic-ai`, it does not enter the funnel.
- **No paper is re-summarized.** Before writing a new card, grep `no-magic-papers/papers/` and `no-magic/docs/catalog.json` by title, arxiv ID, and slug.
- **Pre-print vs published:** pre-prints are eligible, but the paper card records `version` so updates to the published version can supersede.

---

## 3. Stage 1 — Triage

### 3.1 Gate criteria

A paper passes triage if and only if **all** of the following are true. Any failure routes to `out` or `parking`.

| # | Criterion | Fails if |
|---|---|---|
| 1 | **Algorithmic contribution.** The paper proposes a specific method, not only an empirical study or benchmark. | Paper is a pure benchmark, position paper, or survey. |
| 2 | **Publicly accessible.** Full text is available without paywall or permissions barrier. | Paywalled without pre-print mirror. |
| 3 | **Reproducible in principle.** Method is specified in enough detail to implement without access to proprietary systems. | Requires undocumented internal infrastructure or non-public data. |
| 4 | **Single-file tractability (pedagogical, not production).** The core algorithm can be expressed in ≤500 lines of stdlib Python on CPU, even at toy scale. | Inherently requires GPU cluster, TB-scale data, or multi-component distributed systems. |
| 5 | **Not already covered.** Neither `no-magic` nor `no-magic-papers` has an equivalent entry (checked by slug, title, and first author). | Duplicate. |
| 6 | **Licence of upstream materials permits summarization.** Paper text is quotable under fair use; author's reference code (if relied upon) is permissively licensed. | Code is unlicensed or restrictive, and summary would require code contact beyond the paper. |

### 3.2 Triage outcomes

- **in** — all six criteria pass. Label `triage-in`. Proceed to Stage 2.
- **out** — at least one criterion fails definitively. Label `triage-out`. Close issue with a one-sentence rationale.
- **parking** — a criterion fails but might be resolved later (e.g. pre-print not yet published, single-file feasibility unclear pending reading). Label `triage-parking`. Stays open with target date for re-review.

### 3.3 Who triages

Triage is performed by the maintainer personally, on demand, with no SLA. A paper surfaces, the maintainer runs it through the six criteria, and the decision is recorded either in the tracking issue or directly in the paper card being drafted.

Agent assistance (via `apprentice`, once activated) will be limited to producing draft triage assessments for the maintainer to confirm. Triage recommendations from agents are never binding; the maintainer signs off every decision.

Triage outcomes are binding for 6 months. Re-raising a `triage-out` paper requires new evidence cited in the new triage record.

### 3.4 Lesson-authoring gate

For each summarized paper, the maintainer separately decides whether the paper warrants a prose lesson in `no-magic-papers/lessons/`. The lesson gate is independent of the implementation gate: a paper can have an implementation without a lesson, a lesson without an implementation, both, or neither. Lessons are authored when:

- The paper's algorithm is already implemented in `no-magic`, and code comments alone are insufficient for the audience who wants prose.
- The paper is conceptually dense enough that a walkthrough adds value beyond the paper's own exposition.
- A curriculum path (`no-magic-paths`) is planned that requires this concept as a teaching beat.

Lessons are cap-bound at 1500 words and follow the structure `paper summary → intuition → code walkthrough → exercises`.

### 3.5 Themes taxonomy

Every paper card tags one primary theme and zero to two secondary themes from this canonical list. Themes drive batching, navigation, and curriculum paths. Themes can be extended only via an amendment to this document — drift is prevented by keeping the list small and the definitions sharp.

| Theme | Definition |
|---|---|
| `efficient-inference` | Quantization, pruning, distillation, speculative decoding, KV-cache optimization. |
| `long-context` | Attention variants, memory architectures, RoPE extensions, sparse/hierarchical attention. |
| `alignment` | RLHF, DPO, PPO, GRPO, KTO, SimPO, preference optimization, reward modeling. |
| `reasoning` | Chain-of-thought, self-consistency, process reward, test-time compute, self-play, tree-search reasoning. |
| `architecture` | New model designs — SSMs, MoE, hybrid attention, tokenization-free, differential attention. |
| `training-dynamics` | Optimizers, scaling laws, curriculum, pretraining recipes, data ordering. |
| `parameter-efficient` | LoRA, DoRA, adapters, prefix tuning, prompt tuning. |
| `interpretability` | Mechanistic interpretability, knowledge editing, probing, sparse autoencoders. |
| `retrieval` | RAG, vector search, reranking, dense retrieval, hybrid search. |
| `safety-robustness` | Jailbreak, watermarking, red-teaming, bias, adversarial robustness. |
| `agents` | Planning, tool use, memory-augmented agents, multi-agent coordination, computer use. |
| `multimodal` | Vision-language, audio, video, cross-modal fusion. |

Additional free-form `tags:` are permitted in the paper card frontmatter for finer-grained categorization (e.g. `grpo`, `rope-extension`, `green-list`). Tags do not need to be pre-registered.

---

## 4. Stage 2 — Summary

### 4.1 Create the paper card

Upon `triage-in`, open a PR to `no-magic-papers` that creates a single markdown file:

```
no-magic-papers/papers/{paper-slug}.md
```

**Slug rules (Option C):**

- Lowercase ASCII, hyphen-separated.
- Paper slugs reflect the paper's **canonical name**, not the future script filename. Do not apply the `micro*` prefix to paper slugs — that prefix belongs exclusively to `no-magic` implementations, where the "miniature" claim is accurate.
- Preferred form: the paper's short name as it appears in citations and community discussion.
- Examples: `rome`, `memit`, `lora`, `dpo`, `grpo`, `turboquant`, `deepseek-r1`, `nsa`.
- **Collision fallback:** if the canonical name is generic or already taken, append `-{year}` or `-{first-author-lastname}` until unique. Examples: `titans-2501` (generic name), `mamba-gu` (first-author tiebreaker), `moe-switch` vs `moe-aux-free`.

The corresponding implementation, if one is planned, uses a separate `micro*`-prefixed filename in `no-magic` (e.g. paper `rome` → script `microrome.py`). The mapping is recorded explicitly in `implementation.path` in the card frontmatter — never inferred by string manipulation.

### 4.2 Paper card template

See §10 for the full template. At summary time, the following fields are **required**:

- `title`, `authors`, `venue`, `year`, `arxiv_id_or_doi`, `url`
- `tldr` (one-sentence contribution in plain English)
- `problem` (what's broken without this paper)
- `contribution` (what this paper adds)
- `method_summary` (3–7 bullets describing the mechanism)
- `key_results` (what it achieves; cite specific numbers from the paper)
- `relation_to_existing` (what it builds on, what it replaces)
- `implementation_notes` (pedagogical guidance for anyone writing `no-magic` code from this paper)
- `status: summarized`

Optional at this stage: `slug_for_implementation`, `suggested_tier`, `dependencies_on_other_papers`.

### 4.3 Summary quality bar

- **Maximum 800 words** for the paper card body. Summaries longer than that signal that the paper should become a multi-card entry or a lesson in `no-magic-lessons`.
- **Quote sparingly.** Fair-use quoting is permitted for a few sentences at most; paraphrase everything else.
- **No tutorial contact.** The summary is written from the paper alone, not from any existing blog post or tutorial about the paper. Subsequent ecosystem links (e.g. "see also: blog post X") may be added after the summary is written, never during.
- **No screenshots of paper figures.** Regenerate diagrams or describe them in prose.

### 4.4 Review checklist for summary PRs

- [ ] All required fields filled.
- [ ] Word count ≤ 800.
- [ ] No verbatim copying beyond quoted fragments.
- [ ] Status set to `summarized`.
- [ ] File named by canonical slug.
- [ ] Listed in `no-magic-papers/INDEX.md`.

---

## 5. Stage 3 — Route

Once the card lands with `status: summarized`, a routing decision determines which downstream repos should ingest it. Routing is recorded inside the paper card under `routing:` and closes with one of these statuses:

| Route status | Meaning |
|---|---|
| `backlog-implement` | Will be implemented in `no-magic` — paper becomes the source of a future script. |
| `reference-only` | Not implementing. Paper remains a reference cited by existing or future scripts. |
| `visualize-only` | Warrants an animation in `no-magic-viz` without a code implementation (rare — typically for conceptual papers where code already exists under a different slug). |
| `deferred` | Eligible but queued behind higher-priority items. Review date set. |

### 5.1 Routing questions

For each paper, the router (human or agent) answers in order:

1. **Is the algorithm within the single-file constraint?** If no: `reference-only`.
2. **Does the algorithm already exist under another name in `no-magic`?** If yes: `reference-only`; cross-link from the existing script's docstring.
3. **Is there a `no-magic-viz` scene opportunity, independent of a script?** If yes and no script planned: `visualize-only`.
4. **Is this a candidate for the next implementation batch?** If yes: `backlog-implement` + assign to batch.
5. Otherwise: `deferred` with a re-review date.

### 5.2 Batching rule

- `no-magic` implementation batches are sized 3–6 papers each, released as a minor version bump.
- A paper stays in `backlog-implement` until a batch picks it up.
- Batches are organized by **primary theme** from the taxonomy in §3.5, not by cadence. A batch ships when its theme has enough papers to tell a coherent story (e.g. "knowledge-editing batch" = ROME + MEMIT + KE; "efficient-inference batch" = TurboQuant + smoothquant variant + speculative decoding). Themes with only one or two papers wait until related work joins them, unless the single paper is independently significant.

### 5.3 Routing output

Routing updates the card with:
```yaml
routing:
  decision: backlog-implement | reference-only | visualize-only | deferred
  target_repo: no-magic | no-magic-viz | n/a
  target_script_slug: microrome     # if applicable — with micro* prefix
  target_path: 02-alignment/microrome.py   # full relative path in target_repo
  target_tier: 02-alignment         # if applicable
  batch_label: interpretability-batch-1    # if applicable
  review_date: 2026-07-01           # if deferred
```

---

## 6. Stage 4 — Implement

Only cards with `routing.decision: backlog-implement` reach this stage.

### 6.1 Hard rules

- **Paper-first.** The implementer reads the paper and this paper card. The implementer does not read tutorials, blog posts, other repos' implementations, or framework source for the algorithm.
- **Stdlib only.** The implementation is a single `.py` file under the routed tier directory in `no-magic`.
- **Train + infer lifecycle.** The script trains a tiny model and runs inference end to end, under the repo's runtime budget.
- **Docstring attribution.** The thesis docstring cites: primary paper (full bibliographic form), arXiv or DOI URL, optional curriculum-inspiration note.

### 6.2 Implementation sub-process

1. **Draft issue.** Open a tracking issue in `no-magic` referencing the paper card. Label `impl`.
2. **Scaffold the file.** Start from the `no-magic` script template (constants → data loader → model → training loop → inference → demo).
3. **Reduce to toy scale.** Choose hyperparameters so runtime is ≤ 10 minutes CPU.
4. **Comment density.** Maintain 30–40% comment ratio; include math-to-code mappings for every non-trivial derivation in the paper.
5. **Self-review.** Run the script top-to-bottom as if reading a tutorial. Remove anything that breaks that flow.
6. **Open PR.** Link to paper card, issue, and batch label.

### 6.3 Review gates

- Constraint check: single file, stdlib only, runs in budget.
- Correctness check: training loss decreases, inference produces sensible output, seeds deterministic.
- Provenance check: docstring cites the correct paper; no unattributed fragments.
- Commenting check: 30–40% density, required comment types present.
- Paper-card sync: status updates pending (handled in Stage 5).

---

## 7. Stage 5 — Sync

Implementation is not done until the cross-repo state is consistent. This stage runs as the final commit(s) of the implementation PR, or as a follow-up PR no later than 24 hours after merge.

### 7.1 Sync checklist

- [ ] **`no-magic/docs/catalog.json`** — add new entry with slug, display name, tier, thesis, paper reference, and dependency list. From `no-magic` v3.0 onward, the entry must include `paper_slug` pointing to the paper card.
- [ ] **Paper card** (`no-magic-papers/papers/{paper-slug}.md`) — append (or update) the entry in `implementations`:
  ```yaml
  status: implemented
  implementations:
    - repo: no-magic
      path: 02-alignment/microrome.py
      script_slug: microrome
      commit: <sha>
      release: v2.1.0
  ```
- [ ] **`no-magic-papers/INDEX.md`** — regenerate via `scripts/generate_index.py`. Do not hand-edit.
- [ ] **`no-magic/README.md`** — if the script introduces a new tier category, update the top-level catalog section.
- [ ] **`no-magic-viz`** — if a visualization is planned, open a `viz` issue tagged with the same slug.
- [ ] **`no-magic-paths`** (once it exists) — if the script slots into an existing path, update that path's YAML.
- [ ] **`no-magic-papers/lessons/{slug}.md`** — if a lesson is planned, open a `lesson` issue in `no-magic-papers` tagged with the slug. Lesson authoring is a separate PR in the same repo.
- [ ] **Website** — no action required; `no-magic-ai.github.io` reads `catalog.json` at runtime.

### 7.2 Cross-reference rules

- Paper card → script is **forward reference** via the `implementations[].path` list, with one entry per implementing script (typically a list of length 1). Paper slug and script slug differ by convention; never attempt to derive one from the other.
- Script → paper card is **back reference** in the docstring: `Paper card: no-magic-papers/papers/rome.md` (paper slug).
- Lesson file uses the same paper slug as the paper card: `no-magic-papers/lessons/{paper-slug}.md`. Lesson references both the paper card (by paper slug) and the script (by path).
- Paths → mix of paper slugs and script slugs as appropriate; each entry is labeled with its artifact type.

### 7.3 Atomic integrity invariants

At any point in time, the following must hold:

1. If `catalog.json` has an entry for a script `S`, exactly one paper card in `no-magic-papers/papers/` has an `implementations[]` entry whose `path` points to `S`, and the card's `status` is `implemented`.
2. If a paper card has `status: implemented`, every entry in its `implementations[]` list resolves to a file that exists on `main` in the named `repo`.
3. From `no-magic` v3.0 onward: no script exists in `no-magic` without a paper card whose `implementations[]` references it. (Pre-v3.0: this invariant is advisory; v0.1–v2.x scripts may temporarily lack paper cards during the backfill window.)
4. Paper slugs and script slugs are disjoint namespaces — a paper slug never appears as a script filename, and vice versa. The `micro*` prefix is reserved for script slugs; paper slugs never carry it. (Lint rule: reject paper cards whose filename starts with `micro`; reject new scripts whose filename does not start with `micro`.)
5. `INDEX.md` byte-for-byte matches the output of `scripts/generate_index.py` against the current frontmatter. Hand-edits are reverted by CI.

A CI script in `no-magic-papers` enforces these invariants on every PR, by traversing `catalog.json` in `no-magic` and `papers/*.md` in `no-magic-papers`.

---

## 8. Stage 6 — Lifecycle

Papers and their implementations are not static.

### 8.1 Status transitions

```
  triaged ──► summarized ──► backlog-implement ──► implemented
                  │                  │                   │
                  ▼                  ▼                   ▼
            reference-only      deferred            deprecated
                                                        │
                                                        ▼
                                                     replaced
                                                        │
                                                        ▼
                                                    archived
```

- **deprecated:** the paper is still accurate but the implementation no longer reflects best practice or has been superseded. The script gets a `DEPRECATED.md` banner; paper card is updated.
- **replaced:** a newer paper supersedes this one. The card links forward to the replacement card; the implementation may be retired or kept as a historical reference.
- **archived:** content moved out of the active catalog but preserved in git history and paper card (`status: archived` with rationale).

### 8.2 Lifecycle review cadence

- **Every minor release of `no-magic`:** sweep for `deprecated` candidates (tests failing, runtime blowing up, tier boundaries changed).
- **Annually:** sweep `reference-only` cards — does the field now consider this method baseline? Does it deserve implementation after all?
- **On paper retraction or correction:** immediate review; status → `replaced` or `archived` with note.

---

## 9. Governance, SLAs, and cadence

### 9.1 Roles

| Role | Phase 0 (current) | Phase 1 (agent-assisted) | Phase 2 (agent-led) |
|---|---|---|---|
| Discoverer | Maintainer | Maintainer (agent may flag candidates) | Maintainer (agent may flag candidates) |
| Triager | Maintainer | Agent proposes, maintainer confirms | Agent proposes, maintainer confirms |
| Summarizer | Maintainer or contributor | Agent drafts, maintainer revises | Agent drafts, maintainer revises |
| Router | Maintainer | Maintainer | Maintainer (human-only gate) |
| Implementer | Contributor or maintainer | Contributor (agent drafts scaffold) | Contributor + agent |
| Lesson author | Maintainer or contributor | Agent drafts, maintainer revises | Agent drafts, maintainer revises |
| Reviewer | Maintainer | Maintainer | Maintainer |
| Syncer | Implementer | Agent | Agent |

**Discovery, triage, routing, and all content review remain human-only decisions at every automation phase.** Automation may draft and suggest; the maintainer always signs off.

### 9.2 Cadence

There is no SLA clock. The ingestion pipeline runs on the maintainer's initiative. When a paper is surfaced, triage, summary, routing, and implementation proceed at the maintainer's pace until the paper card reaches a terminal state (`implemented`, `reference-only`, `deferred`, or `triage-out`).

The only rhythmic obligation is the **annual lifecycle sweep**: once per year, the maintainer reviews all cards for deprecation candidates, replacement by newer papers, and archival of obsolete content.

Batch releases of `no-magic` happen when a theme has 3–6 implemented papers ready to ship, not on a fixed schedule. Theme coherence drives timing, not the calendar.

---

## 10. Paper card template

This is the canonical template for every file in `no-magic-papers/papers/`. Fields are YAML frontmatter; body is markdown.

```markdown
---
slug: rome                         # paper slug — paper-canonical, no micro* prefix
title: "Locating and Editing Factual Associations in GPT"
authors:
  - Kevin Meng
  - David Bau
  - Alex Andonian
  - Yonatan Belinkov
venue: NeurIPS
year: 2022
arxiv_id: 2202.05262
doi: null
url: https://arxiv.org/abs/2202.05262
discovered_via: dive-into-llms-ch3
discovered_date: 2026-04-25
status: summarized                 # triaged | summarized | backlog-implement | implemented | deprecated | replaced | archived | reference-only
themes:                            # from §3.5 canonical taxonomy
  primary: interpretability
  secondary:
    - alignment
tags:                              # free-form, finer-grained
  - knowledge-editing
  - rank-1-update
  - causal-tracing
routing:
  decision: null                   # set at Stage 3
  target_repo: null
  target_script_slug: null         # e.g. microrome (with micro* prefix)
  target_tier: null
  batch_label: null
  review_date: null
implementations:                   # list — set at Stage 5; one entry per script
  - repo: null                     # e.g. no-magic
    path: null                     # e.g. 02-alignment/microrome.py
    script_slug: null              # e.g. microrome (basename minus .py)
    commit: null
    release: null
  # Add additional entries when one paper introduces multiple distinct algorithms.
  # Common case is a list of length 1.
lesson:
  path: null                       # no-magic-papers/lessons/{paper-slug}.md if authored
  status: null                     # none | planned | drafted | published
dependencies_on_other_papers:
  - slug: transformer              # paper slugs, not script slugs
---

## TL;DR

One sentence. What's new in one breath.

## Problem

What breaks without this paper. Frame the gap in the field before the contribution.

## Contribution

What this paper adds. Precise; no marketing prose.

## Method summary

- Bullet 1: step or component
- Bullet 2: step or component
- Bullet 3: ...

## Key results

What it achieves, in numbers. Cite figures and tables.

## Relation to existing work

What it builds on. What it replaces. Where it fits in the field's arc.

## Implementation notes

Guidance specifically for someone writing a `no-magic` script from this paper:

- Minimum toy dataset shape.
- Which hyperparameters matter; which can be scaled down.
- Which parts of the paper are implementation vs ablation.
- Pedagogical pitfalls.

## Open questions

What the paper leaves unresolved. Optional.

## Further reading

- Follow-up papers (linked by slug if they exist in `no-magic-papers`).
- Survey papers that contextualize.
- Explicitly NOT: tutorials, blog explainers, or course material.
```

---

## 11. Anti-patterns

| Anti-pattern | Why it's forbidden |
|---|---|
| Summarizing from a blog post instead of the paper. | Blog posts hallucinate; they also import an unclear copyright chain. |
| Triage based on citation count alone. | Citation count lags relevance by years and over-rewards early publishers. |
| Implementing without a matching paper card. | Breaks the atomic integrity invariant; creates orphan code. |
| Routing decided by the implementer. | Concentration of concern: the person volunteering effort should not be the person rationing attention. |
| Cross-posting paper content (figures, code) from the paper PDF into the card. | Copyright ambiguity; regenerate or describe in prose. |
| Auto-merging summary PRs. | Summaries are the system's factual layer; human review is load-bearing. |
| Paper card statuses changed by anyone other than the syncer. | Causes drift between card state and implementation state. |
| Introducing a new tier because a paper doesn't fit the existing four. | Tier structure is org-level taxonomy; change via strategy doc, not in response to one paper. |
| Applying the `micro*` prefix to a paper slug. | The prefix is a pedagogical-miniature claim. A paper is not a miniature; its slug must be the paper's canonical name (see §4.1 and §7.3 invariant 4). |
| Inferring a paper slug from a script slug by stripping `micro`, or vice versa. | The mapping is never convention-based. Cross-references are explicit fields in frontmatter or docstrings. |

---

## 12. Worked example — ROME

A concrete walk-through for paper slug `rome` / script slug `microrome` (knowledge editing via rank-1 FFN update).

1. **Discovery (Stage 0).** Maintainer surfaces ROME while working on a knowledge-editing thread. Paper slug: `rome`. Intake issue is optional — maintainer proceeds straight to the card.
2. **Triage (Stage 1).** Maintainer runs the six criteria: algorithmic (rank-1 weight update), public (arXiv), reproducible (full method in paper), feasible (≤300 LOC projected), not covered, permissive use of paper text. Outcome: `triage-in`.
3. **Summary (Stage 2).** PR adds `papers/rome.md` with all required fields. Themes tagged `primary: interpretability, secondary: alignment`. Word count ~650. Status `summarized`. Maintainer reviews and merges.
4. **Route (Stage 3).** Maintainer answers: single-file tractable → yes. Existing in `no-magic` → no. Assigns `backlog-implement`, target_script_slug `microrome`, target_path `02-alignment/microrome.py`, batch label `interpretability-batch-1`.
5. **Implement (Stage 4).** Contributor opens `no-magic` issue referencing paper card `rome`. Reads the paper only. Drafts `microrome.py` using the standard template: loads a small pretrained LM, causal-traces a factual association, computes the rank-1 update, re-queries to verify. Docstring back-references `Paper card: no-magic-papers/papers/rome.md`. Runtime 6 minutes CPU. PR opened.
6. **Review (Stage 4).** Maintainer reviews manually. Gates pass. Merged in `no-magic` v2.1.0.
7. **Sync (Stage 5).** `catalog.json` updated with `microrome` entry (and `paper_slug: rome` from v3.0 onward). Paper card `rome` status → `implemented`; `implementations[]` gains an entry with `path: 02-alignment/microrome.py`. `scripts/generate_index.py` re-runs and updates `no-magic-papers/INDEX.md` under the `interpretability` theme section. `no-magic-viz` issue opened for a future Manim scene keyed on `microrome`.
8. **Lesson gate (Stage 3.4).** Maintainer decides ROME warrants a prose lesson because causal tracing is dense. Opens lesson-proposal issue; contributor drafts `no-magic-papers/lessons/rome.md` (lesson slug matches paper slug, not script slug); maintainer reviews and merges.
9. **Lifecycle (Stage 6).** Marked for annual review; flagged for possible replacement card when `memit` paper lands (anticipated, batched with `rome`).

---

## 13. Automation roadmap

The SOP is written so it can be executed manually today and automated in phases without structural change.

### Phase 0 — manual (now)

Everything runs by hand. Maintainer surfaces a paper, triages it, either drafts the card directly or opens an issue, routes it, manually reviews every PR. No `apprentice` dependency. No SLA. No queue.

### Phase 1 — agent-drafted, maintainer-confirmed

`apprentice` produces on demand (when the maintainer requests or when a contributor suggestion arrives):
- Triage recommendations with cited reasoning against each of the six criteria.
- Draft paper cards from paper PDF (using `to-markdown` skill + structured summarization). Includes suggested themes from §3.5 taxonomy.
- Draft lesson outlines from paper card + existing `no-magic` script.
- Sync automation (invariant checks, cross-repo references).

Maintainer confirms every triage decision, every card merge, and every routing decision. Agents do not auto-merge.

### Phase 2 — agent-led with strict gates

`apprentice` proposes high-confidence triage with evidence. Draft cards and lessons enter PRs that the maintainer reviews on their own cadence. Routing remains a human-only gate — always. Implementation scaffolding is agent-drafted; maintainer and contributors review and merge.

### Phase 3 — not planned

Fully autonomous paper-to-implementation is off-charter. The maintainer review gate at summary, routing, implementation, and lesson authoring remains human for the duration of the project.

---

## 14. Decisions locked by this document

1. The paper card is the system of record. No script without a card; no card without at least a status.
2. `no-magic-papers` is the next new repo (after Phase 1 gap-fill batch in `no-magic`). Paper cards live in `papers/`, optional prose lessons in `lessons/`, both keyed on canonical slug.
3. Triage criteria in §3.1 are binding; they can only be changed via an amendment to this document.
4. Discovery, triage, routing, and all content review are human-only decisions at every automation phase. Agents may draft; the maintainer always signs off.
5. Paper summaries are written from the paper, without tutorial contact.
6. Cross-repo sync follows the checklist in §7.1; CI enforces the three invariants in §7.3.
7. Batching is theme-based per §3.5 taxonomy; no fixed cadence.
8. Ingestion runs on the maintainer's initiative with no SLA. The only rhythmic obligation is the annual lifecycle sweep (§9.2).
9. The paper card template in §10 is canonical; fields can be extended, not removed. Every card carries one primary theme from §3.5.
10. All contributions to paper cards, lessons, and implementations are manually reviewed by the maintainer before merge.
11. **Slug convention (Option C).** Paper cards and lessons use paper-canonical slugs without a `micro*` prefix (e.g. `rome`, `turboquant`, `deepseek-r1`). `no-magic` scripts keep the `micro*` prefix as a pedagogical-miniature claim. Cross-repo linkage is explicit via `implementations[].path` in the paper card and a back-reference in the script docstring. CI enforces the disjoint-namespace invariant (§7.3).
12. **One paper card per paper, list-valued `implementations:`.** When a paper introduces multiple distinct algorithms, the card declares multiple entries in `implementations[]`. Bibliographic metadata is the unit of citation; algorithms are the units of implementation.
13. **`INDEX.md` is generated.** Built by `scripts/generate_index.py` from card frontmatter, grouped by primary theme. Pre-commit hook regenerates; CI fails on drift (§7.3 invariant 5). Hand-edits are reverted.
14. **Release sequence:** `no-magic` v2.1 (gap-fill batch, additive) → `no-magic-papers` v0.1 (5 seed cards + 2 lessons) → `no-magic` v3.0 (mandatory `paper_slug` in `catalog.json`, full backfill, §7.3 invariant 3 becomes enforced).

---

## 15. Open questions for maintainer

None currently open. All previously-listed questions resolved per locked decisions §14.1–§14.14.

**Resolved batch (2026-04-25):**
- Triage ownership → maintainer-only, on demand (#9.1, #14.4).
- Batching strategy → theme-based (#7).
- Initial seed → 5 papers: TurboQuant, DeepSeek-R1, Titans-2501, NSA, Mamba-2 (see `seed-papers-proposal.md`).
- Slug convention → Option C (#11).
- Multi-algorithm cards → list-valued `implementations:` (#12).
- `INDEX.md` → generated, CI-verified (#13).
- Website page → deferred until card count > 30.
- Release sequence → v2.1 → v0.1 → v3.0 (#14).
- First lesson author → TurboQuant.

---

## Appendix A — File layout for `no-magic-papers`

```
no-magic-papers/
├── README.md                      # repo charter (single-sentence constraint)
├── CONTRIBUTING.md                # how to add/update a card or lesson
├── INDEX.md                       # human-readable status table, grouped by theme
├── SCHEMA.md                      # paper card schema (canonical)
├── THEMES.md                      # taxonomy mirror of §3.5 for in-repo reference
├── papers/                        # paper-canonical slugs, no micro* prefix
│   ├── rope.md
│   ├── lora.md
│   ├── dpo.md
│   ├── grpo.md
│   ├── rome.md
│   └── ...
├── lessons/                       # optional prose walkthroughs, same paper slugs
│   ├── dpo.md
│   ├── grpo.md
│   └── ...
├── scripts/
│   ├── validate_invariants.py    # CI enforcement of §7.3
│   └── generate_index.py          # optional INDEX.md generator
└── .github/
    ├── ISSUE_TEMPLATE/
    │   ├── triage.yml             # §2.2
    │   └── lesson-proposal.yml
    └── workflows/
        └── validate.yml
```

## Appendix B — Relationship to other documents

- `no-magic-ai-expansion-strategy.md` (sibling, in `.github/docs/`) — the parent document. Authorizes the existence of `no-magic-papers` as a new repo under the umbrella.
- `seed-papers-proposal.md` (sibling, in `.github/docs/`) — the proposed 5 seed papers for `no-magic-papers` v0.1.
- `no-magic/CONTRIBUTING.md` — inherits the "paper-first, no tutorial contact" rule from §6.1.
- `no-magic-papers/README.md` (to be written when repo is created) — the public-facing entry point; references this SOP by link.
- `no-magic-papers/THEMES.md` (to be written) — in-repo mirror of the taxonomy in §3.5.
- `apprentice/design.md` — must be updated to reference this SOP as the primary pipeline `apprentice` will execute.
- `.github/profile/README.md` and `no-magic-ai.github.io/about.html` — the umbrella manifesto. Both surfaces are kept in content parity.
