# no-magic-ai Expansion Strategy

**Status:** Draft for review
**Date:** 2026-04-25
**Scope:** Decision framework for (1) sourcing content from external repos, specifically `dive-into-llms`; (2) expanding `no-magic-ai` from an algorithm catalog to an umbrella learning platform.
**Audience:** no-magic-ai maintainers, future contributors, agentic tooling that operates on this repo set.

---

## 1. Context

`no-magic-ai` is a GitHub organization hosting educational content about AI algorithms. As of 2026-04-25 the org contains:

- `no-magic` — 48 single-file algorithm implementations. Stdlib-only. CPU-only. Train + infer lifecycle per script. Target runtime: 7–10 minutes on consumer hardware.
- `no-magic-viz` — Manim animation scenes for algorithm visualizations.
- `no-magic-ai.github.io` — Static website and portal.
- `apprentice/` — Design-stage agent factory for automating algorithm entry creation (not yet implemented).

The triggering question: a contributor surfaced `github.com/Lordog/dive-into-llms` (a Chinese-language LLM tutorial repo with 34.2k stars) and asked whether to fork-and-translate, cherry-pick, or both. The answer expanded into a broader strategic question about whether `no-magic-ai` should grow from a reference catalog into a general-purpose AI learning platform.

This document records the analysis and the decisions.

---

## 2. Source Repo Analysis: `dive-into-llms`

### 2.1 What it is

- **Maintainers:** SJTU BCMI lab (张倬胜 et al.), with NUS collaboration.
- **Structure:** 11 chapters covering LLM fine-tuning, prompting, knowledge editing, math reasoning, watermarking, jailbreak, steganography, multimodal, GUI agents, agent safety, RLHF.
- **Format per chapter:** one Jupyter notebook + one PDF slide deck + one Chinese-language README.
- **Following:** 34.2k stars, 4.2k forks, active commits through mid-2025.
- **Dependencies:** `torch`, `transformers==4.51.3`, `trl==0.9.4`, `datasets`, plus chapter-specific: `EasyEdit`, `EasyJailbreak`, `LLaMA-Factory`, `NExT-GPT`.
- **GPU expectation:** A100/A800 class for most chapters.
- **Language:** ~100% Simplified Chinese prose; code comments and variable names mixed Chinese/English.

### 2.2 Blockers for direct integration

**License.** `GET /repos/Lordog/dive-into-llms/license` returns 404. No LICENSE file in the tree. Under default copyright law, no redistribution, no derivative works, no translation without explicit written permission. Chapter 11's README additionally states it "翻译并整合了" HuggingFace TRL material (Apache-2.0), mixing upstream licenses opaquely.

**Ideology mismatch.** The source teaches _how to wire HuggingFace components to run experiment X on an A800_. `no-magic` teaches _here is the algorithm, in 200 lines, with stdlib only_. These are orthogonal pedagogies, not translatable ones. Porting any chapter to no-magic style would discard ~95% of the source.

**Coverage overlap.** `no-magic` already ships: GPT, BERT, BPE, attention (MHA/GQA/MQA/sliding-window), RoPE, KV-cache, PagedAttention, Flash Attention, LoRA, QLoRA, DPO, PPO, GRPO, REINFORCE, RAG, MoE, speculative decoding, SSM/Mamba, quantization. The set of `dive-into-llms` topics that are _both_ missing from `no-magic` _and_ algorithmically crisp enough for single-file treatment is narrow.

### 2.3 Decision

**Reject fork-and-translate (Option A).** License forbids it; ideology forbids it even if license allowed it.

**Reject cherry-pick-as-port (Option B).** The source has no from-scratch implementations worth porting; chapter 11's "RLHF implementation" is a `trl.PPOTrainer(...)` call-site walkthrough.

**Accept topic-inspiration-only (Option C, reshaped).** The _topic selection_ is a fact about the field, not copyrightable expression. Use it as a curriculum hint. Implement any selected topic from the _original papers_ cited by each chapter, never by reading the chapter itself.

### 2.4 Genuine gaps identified via topic selection

| Topic                       | Source chapter | Paper (implement from)    | Feasibility                       | Tier              |
| --------------------------- | -------------- | ------------------------- | --------------------------------- | ----------------- |
| Knowledge editing (ROME)    | Ch.3           | Meng et al., 2022         | ~300 LOC, rank-1 FFN update       | `02-alignment/`   |
| Knowledge editing (MEMIT)   | Ch.3           | Meng et al., 2023         | ~400 LOC, batched ROME            | `02-alignment/`   |
| LLM watermarking            | Ch.5           | Kirchenbauer et al., 2023 | ~150 LOC green-list               | `03-systems/`     |
| Self-consistency decoding   | Ch.2           | Wang et al., 2022         | ~100 LOC over existing GPT        | `03-systems/`     |
| Chain-of-thought variants   | Ch.2           | Wei et al., 2022          | ~120 LOC comparison script        | `01-foundations/` |
| Supervised fine-tuning loop | Ch.1           | standard SFT              | ~250 LOC — closes alignment track | `02-alignment/`   |
| Knowledge distillation      | implicit       | Hinton et al., 2015       | ~200 LOC, teacher→student         | `02-alignment/`   |
| Prefix / prompt tuning      | not in source  | Li & Liang, 2021          | ~150 LOC, companion to LoRA       | `02-alignment/`   |

---

## 3. Clean-Room vs Paper-First

### 3.1 Why clean-room is the wrong tool here

Clean-room reverse engineering (Phoenix BIOS, Compaq's AMI clone) exists to produce non-infringing implementations of code _whose specification is not publicly available_. Team A reads the target, writes an English spec; Team B implements from the spec, never seeing the target.

`dive-into-llms` does not meet that precondition. Every chapter cites academic papers. The algorithms themselves are public: ROME is Meng et al., watermarking is Kirchenbauer et al., DPO is Rafailov et al. Copyright protects _expression_, not _ideas_ or _algorithms_. There is nothing to reverse-engineer that isn't already in the papers.

### 3.2 Paper-first is sharper and simpler

- **Legal:** No derivative-works exposure. Papers describe techniques; techniques aren't copyrightable.
- **Pedagogical:** Primary sources beat tutorial summaries. Tutorials add a translation layer that loses precision.
- **Independent:** No dependency on the maintainer's license posture, future or present.
- **Attribution:** A single line in each script's docstring — "Implementation from Meng et al., 2022 (arXiv:2202.05262). Curriculum position informed by dive-into-llms ch.3." — attributes fairly without creating legal entanglement.

### 3.3 Operational rule

**Never open a `dive-into-llms` file while writing a `no-magic` script.** Use the repo's table of contents (visible on the public GitHub page) as a topic list, nothing more. Implement exclusively from the papers cited on that page.

---

## 4. Umbrella Platform Vision

### 4.1 Target audience

Two primary personas:

1. **ML engineers using frameworks daily** who want to understand internals. Comfortable with Python, surface-level ML familiarity, bored by black-box abstractions.
2. **Career switchers / students** moving from theory to practice. Need prose-first explanation alongside code.

Current `no-magic` serves persona 1 well. It does not yet serve persona 2 — code is the primary teaching artifact, and not every learner extracts intuition from reading code.

### 4.2 Target shape

Not a learning management system. Not a chatbot. Not a reimplementation of fast.ai, Karpathy's `nn-zero-to-hero`, or the HuggingFace course. The umbrella adds content, not infrastructure.

**Defining constraint of the umbrella:** every repo under `no-magic-ai` enforces a single-sentence constraint. Repos do not bleed into each other. The website federates them.

### 4.3 Proposed repo structure

| Repo                    | Constraint                                                                               | Role                     | Status              |
| ----------------------- | ---------------------------------------------------------------------------------------- | ------------------------ | ------------------- |
| `no-magic`              | one file, stdlib, CPU, <10 min runtime                                                   | reference implementation | exists (48 scripts) |
| `no-magic-viz`          | Manim scene per algorithm                                                                | animation                | exists              |
| `no-magic-ai.github.io` | static HTML, no JS framework                                                             | portal & search          | exists              |
| `no-magic-papers`       | paper cards in `papers/`, optional prose lessons in `lessons/`; single repo, shared slug | primary-source + prose   | **next**            |
| `no-magic-paths`        | curated reading orders, prereqs, tracks                                                  | curriculum structure     | planned             |
| `no-magic-labs`         | exercises, problem sets, applied projects                                                | practice                 | later               |
| `apprentice`            | agent pipeline for algorithm entry creation                                              | automation               | design-stage        |

**Merge decision (2026-04-25):** lessons and papers were previously scoped as two separate repos. They are consolidated into a single `no-magic-papers` repo with `papers/` and `lessons/` subdirectories sharing the canonical slug. Rationale: the maintainer is solo, both artifacts are prose-only markdown, and both are reviewed manually. Forking into two repos can happen later if maintenance burden justifies it; for now, a single review surface is the correct default. Paper cards are primary content; lessons are optional companions that may not exist for every card.

### 4.4 Federation model

Slugs are split by artifact type:

- **Paper slug** — the paper's canonical name, lowercased and hyphenated. Used as the filename in `no-magic-papers`. Examples: `rome`, `turboquant`, `deepseek-r1`, `titans-2501`.
- **Script slug** — the `no-magic` file's basename (minus `.py`), which keeps the `micro*` prefix as a claim of pedagogical-miniature status. Examples: `microrome`, `microturboquant`, `micror1`.

Location map:

- `no-magic-papers/papers/{paper-slug}.md` — paper card.
- `no-magic-papers/lessons/{paper-slug}.md` — optional prose lesson, same key as the paper card.
- `no-magic/{tier}/{script-slug}.py` — implementation.
- `no-magic-viz/scenes/{script-slug}.py` — Manim scene, keyed on the implementation (the miniature is what gets animated, not the paper).
- `no-magic-paths` references mix paper slugs and script slugs as needed.

Cross-linking is explicit in paper-card frontmatter (`implementation.path` names the script file) and in the `no-magic` script's thesis docstring (`Paper card: no-magic-papers/papers/{paper-slug}.md`). There is no convention-inferred mapping — every link is typed out. CI enforces that every script has a paper card and every card's `implementation.path`, if set, resolves to an existing file.

**Collision rule:** if a paper's canonical name is ambiguous, generic, or already taken, append `-{year}` or `-{first-author}` until unique. Decided case-by-case by the maintainer at Stage 2 of the ingestion SOP.

This keeps each repo independently buildable. A reader who wants only the code still clones `no-magic`. A reader who wants paper summaries and prose clones `no-magic-papers`. The portal unifies without coupling.

---

## 5. Risks and Mitigations

| Risk                                                                                          | Mitigation                                                                                                                                                                                                               |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Crowded market.** fast.ai, Karpathy, HuggingFace, Raschka, mlabonne.                        | Differentiation is the single-file stdlib-only constraint. Losing that constraint means losing the moat. Constraint is non-negotiable for `no-magic`; adjacent repos can relax it but must declare their own constraint. |
| **Scope dilution.** "Learning platform" pulls toward LMS features.                            | Static + GitHub Discussions only. Defer any stateful feature (accounts, progress tracking, auth) indefinitely. Reject contributors who want to add them.                                                                 |
| **AI-tutor scope creep.** A RAG chatbot over the catalog is weeks of infra plus ongoing cost. | Defer until content depth is ≥3× current, at which point the tutor becomes obviously useful rather than gimmicky.                                                                                                        |
| **Double maintenance burden.** `no-magic-lessons` doubles per-algorithm work.                 | Commit to the discipline or don't start. A half-populated lessons repo is worse than no lessons repo. Gate new repo creation on a visible contributor pipeline.                                                          |
| **License contamination via contributors.** New contributors porting from other tutorials.    | Add `CONTRIBUTING.md` section: "Implementations must cite original papers. Do not copy code or prose from existing tutorials or textbooks. Cite topic inspiration only."                                                 |
| **Translation debt.** 6-language translation scaffolding exists; 0 languages populated.       | Hold translation work until English content stabilizes. Publish one language at a time. Do not open translation contributions for algorithms newer than 90 days.                                                         |

---

## 6. Coverage Gap Map

### 6.1 LLM topics currently in `no-magic`

Tokenizers (BPE), embeddings, all transformer variants (GPT, BERT, attention mechanisms, MHA/GQA/MQA/sliding), positional encoding (RoPE), Flash Attention, KV-cache, PagedAttention, quantization, LoRA, QLoRA, full RLHF suite (DPO, PPO, GRPO, REINFORCE), RAG, speculative decoding, SSM/Mamba, vector search, MoE.

### 6.2 LLM topics missing

Instruction tuning / SFT, constitutional AI, chain-of-thought reasoning, in-context learning mechanics, prefix/prompt tuning, adapter modules beyond LoRA, knowledge distillation, model merging (DARE / linear interpolation), curriculum learning, continual learning, position interpolation for long context, data filtering / deduplication, knowledge editing (ROME / MEMIT), watermarking, self-consistency decoding.

### 6.3 Priority batch (informed by dive-into-llms topic selection, implemented from papers)

1. `microrome.py` — rank-1 knowledge editing. Meng et al., 2022.
2. `microsft.py` — supervised fine-tuning loop. Closes the alignment track.
3. `microwatermark.py` — Kirchenbauer green-list watermark.
4. `microdistill.py` — teacher-student knowledge distillation. Hinton et al., 2015.
5. `microcot.py` — chain-of-thought + self-consistency comparison script.

Stretch (post-batch):

6. `microprefix.py` — prefix tuning, companion to LoRA.
7. `micromemit.py` — batched ROME.
8. `micromerge.py` — model merging (linear + DARE).

---

## 7. Recommended Sequence

### Phase 0 — Strategic alignment (1 week)

1. Publish an **umbrella manifesto** at `no-magic-ai.github.io/about` (or equivalent). One page. Content: what `no-magic-ai` is, what repos exist, what each repo's constraint is, what it is _not_ (not a LMS, not a chatbot).
2. Update `no-magic` CONTRIBUTING.md with the paper-first rule and the no-tutorial-copying rule.
3. Add a `FURTHER_READING.md` at `no-magic` root citing `dive-into-llms` as one curriculum reference among several.

### Phase 1 — Close LLM gaps in `no-magic` (2–4 weeks)

1. Ship the priority batch (`microrome`, `microsft`, `microwatermark`, `microdistill`, `microcot`). Implement from papers, one at a time. Standard review cycle.
2. Do not start adjacent repos until this batch lands.

### Phase 2 — Paper + lesson repo (4–6 weeks)

1. Create `no-magic-papers` repo with layout: `papers/`, `lessons/`, `INDEX.md`, `SCHEMA.md`, `THEMES.md`, `CONTRIBUTING.md`, `.github/ISSUE_TEMPLATE/`.
2. Seed with 5 diverse themed paper cards (see `seed-papers-proposal.md` in this directory). Each card uses the canonical template in `paper-ingestion-process.md` §10.
3. Constraint: one markdown per paper in `papers/`; one markdown per optional lesson in `lessons/` (lesson cap: 1500 words; structure fixed as paper → intuition → code walkthrough → exercises).
4. Ship `no-magic-papers` v0.1 with 5 seed cards; ship 2 lessons alongside to validate the combined-repo layout before scaling.

### Phase 3 — Curriculum paths (post Phase 2)

1. Create `no-magic-paths` repo. Each path is a markdown file listing an ordered sequence of algorithm slugs with prerequisites and motivation.
2. Seed with 3 paths: "LLM internals from zero", "RLHF in practice", "Inference optimization tour".

### Phase 4 — Labs (later, gated on demand)

1. `no-magic-labs` when exercises in `no-magic-papers/lessons/` outgrow their host.

### Explicitly deferred

- AI tutor / RAG chatbot
- Account system, progress tracking
- Translation rollout beyond English
- `apprentice` automation — remains at design stage until `no-magic` catalog stabilizes (≥60 algorithms) and the per-algorithm template is locked.

---

## 8. Legal and Attribution Operating Rules

1. **Paper-first always.** Every `no-magic` script cites the original paper(s) in its thesis docstring.
2. **No tutorial contact.** Do not read `dive-into-llms`, Karpathy's `nn-zero-to-hero`, fast.ai notebooks, or similar resources while writing a `no-magic` script. Topic inspiration from their tables of contents is allowed; prose and code contact is not.
3. **Attribution block.** Each script's thesis docstring includes: primary paper citation; optional curriculum-inspiration line ("Topic selection informed by dive-into-llms ch.X").
4. **License vigilance.** Before adopting any new upstream asset (dataset, image, text), verify the license. Record provenance in `ASSETS.md`.
5. **Contributor pledge.** `CONTRIBUTING.md` requires contributors to affirm they implemented from papers, not from tutorials.

---

## 9. Decisions Locked by This Document

1. No fork of `dive-into-llms`.
2. No translation of `dive-into-llms` content.
3. No code copied from `dive-into-llms`.
4. Topic selection inspired by `dive-into-llms` is acceptable with visible attribution.
5. `no-magic-ai` expands into an umbrella via new repos, each with a single-sentence constraint, federated through the website.
6. Priority gap-fill batch for `no-magic`: ROME, SFT, watermarking, distillation, CoT + self-consistency.
7. `no-magic-papers` is the next new repo. Lessons and paper cards live in the same repo under `papers/` and `lessons/` subdirectories, sharing the canonical slug.
8. Deferred: AI tutor, stateful platform features, translation rollout, `apprentice` activation.
9. **Umbrella manifesto lives in two places:** `no-magic-ai.github.io/about.html` (long-form design) and `.github/profile/README.md` (markdown mirror on the org landing page). Content parity is maintained manually.
10. **Discovery and triage are maintainer-only, on demand.** No weekly cadence. Papers enter the pipeline when the maintainer personally surfaces them.
11. **Contributions to papers, lessons, and implementations are manually reviewed by the maintainer.** No automated merges. CI validates invariants; humans validate content.
12. **Papers are themed using the canonical taxonomy** in `paper-ingestion-process.md` §3.5 (12 primary themes). Cards tag 1 primary theme and 0–2 secondary themes.
13. Lesson cap: 1500 words per file in `no-magic-papers/lessons/`. Longer than that signals the lesson should be split or demoted to a lab exercise.
14. **Slug convention (Option C).** Paper cards and lessons use paper-canonical slugs (`rome`, `turboquant`, `deepseek-r1`). `no-magic` scripts keep the `micro*` prefix (`microrome.py`, `microturboquant.py`) as a claim of pedagogical miniature. Cross-repo references are explicit, not convention-inferred. Collision fallback: append `-{year}` or `-{first-author}`.
15. **Seed set finalized.** Five papers: TurboQuant, DeepSeek-R1, Titans-2501, NSA, Mamba-2. Mamba-2 simultaneously occupies slot 5 and satisfies the "tied to existing script" demonstration via the `microssm.py` family. FeatureBench was triaged out as a benchmark per SOP §3.1 criterion #1.
16. **One paper card per paper, list-valued implementations.** When a paper introduces multiple distinct algorithms, the card declares a list of `implementations:` entries — one per script — rather than splitting into multiple cards. Bibliographic metadata is the unit of citation; algorithms are the units of implementation.
17. **`INDEX.md` is generated, committed, and CI-verified.** A `scripts/generate_index.py` reads paper-card frontmatter, groups by primary theme, emits sorted markdown. Pre-commit hook regenerates; CI fails on drift. Hand-edits to `INDEX.md` are reverted.
18. **Website `/papers` page deferred.** v0.1 of `no-magic-papers` ships without a dedicated website page; `INDEX.md` on github.com is the navigation surface. A `/papers` page is built only when card count exceeds 30 or a curriculum path requires it.
19. **First lesson target: TurboQuant.** Lesson template is validated on a mechanically straightforward, math-dense concept before the marquee DeepSeek-R1 lesson is written.
20. **Release sequence: v2.1 → v0.1 → v3.0.** `no-magic` v2.1 ships the gap-fill batch (ROME, SFT, watermarking, distillation, CoT) as additive content. `no-magic-papers` v0.1 ships the 5 seed cards + 2 lessons (TurboQuant, DeepSeek-R1). `no-magic` v3.0 is reserved for the schema change that makes paper cards mandatory per script and adds `paper_slug` to `catalog.json`, with backfill of paper cards for all existing scripts.

---

## 10. Open Questions for Maintainer

None currently open. All previously-listed questions resolved per locked decisions §9.1–§9.20.

**Resolved batch (2026-04-25):**
- Manifesto location → both `about.html` and `.github/profile/README.md` (#9).
- Lesson word cap → 1500 words (#13).
- Review gate for papers and lessons → maintainer-only (#11).
- Contributor pledge → manual review (#11).
- Slug convention → Option C, paper-canonical for cards, `micro*` prefix for scripts (#14).
- Seed set → TurboQuant, DeepSeek-R1, Titans-2501, NSA, Mamba-2 (#15).
- Multi-algorithm papers → one card, list-valued implementations (#16).
- `INDEX.md` → generated, CI-verified (#17).
- Website `/papers` page → deferred (#18).
- First lesson target → TurboQuant (#19).
- Release sequence → v2.1 → v0.1 → v3.0 (#20).

---

## Appendix A: Rejected Approaches

- **Full fork + translation of dive-into-llms.** License blocker, ideology blocker.
- **Cherry-pick notebooks and port to stdlib Python.** ~95% of each source notebook would be discarded. Topic-inspiration path dominates.
- **Clean-room reverse engineering.** Spec is public via papers; clean-room procedure is overkill for public-spec content.
- **AI tutor over existing catalog.** Infrastructure and ongoing cost not justified until content is 3× current depth.
- **Algorithm-per-PR LMS platform with accounts and progress tracking.** Scope creep; violates "static + GitHub Discussions only" guardrail.

## Appendix B: Source Repo Quick Facts (`dive-into-llms`)

- URL: https://github.com/Lordog/dive-into-llms
- Stars/forks: 34.2k / 4.2k (as of 2026-04-25)
- Chapters: 11
- License: none (default copyright)
- Maintainer contact: SJTU BCMI lab, Zhang Zhuosheng et al.
- Citation position in `no-magic`: `FURTHER_READING.md`, topic-inspiration line in relevant script docstrings.
