# Seed Papers Proposal — `no-magic-papers` v0.1

**Status:** Proposal for maintainer approval
**Date:** 2026-04-25
**Companion to:** `no-magic-ai-expansion-strategy.md`, `paper-ingestion-process.md`
**Purpose:** Propose 5 diverse, high-profile papers from 2025 to seed the `no-magic-papers` repo. Each paper exercises a different theme from the canonical taxonomy (SOP §3.5), a different tier of `no-magic`, and shows a different way the paper card + optional lesson pattern works in practice. The set functions as both content and reference implementation of the ingestion process.

---

## 0. Selection criteria

Each seed paper satisfies:

1. **Published in 2025** (or very late 2024). Fresh enough to demonstrate the repo tracks current research; old enough that the community has consensus on significance.
2. **High profile.** Widely discussed, influential, or from a named lab with credibility. Reduces risk of seeding with papers that age badly.
3. **Algorithmic core suitable for single-file treatment.** Not a survey, benchmark, or infrastructure paper. Has a discrete method a `no-magic` script could implement.
4. **Theme diversity.** The 5 papers cover 5 distinct primary themes from the SOP §3.5 taxonomy.
5. **Tier diversity.** The 5 papers span at least 3 of the 4 `no-magic` tiers (foundations, alignment, systems, agents).
6. **Permissive paper text.** arXiv or open-venue publication, allowing paraphrase and quotation under fair use.

---

## 1. The seed set

Slugs follow Option C: paper slugs are paper-canonical (no `micro*` prefix); script slugs keep the `micro*` prefix as a pedagogical-miniature claim.

| # | Paper | Year | Primary theme | Paper slug | Target tier | Script slug (if implemented) |
|---|-------|------|---------------|-----------|-------------|------------------------------|
| 1 | TurboQuant | 2025 | `efficient-inference` | `turboquant` | 03-systems | `microturboquant` |
| 2 | DeepSeek-R1 | 2025 | `reasoning` | `deepseek-r1` | 02-alignment | `micror1` |
| 3 | Titans: Learning to Memorize at Test Time | 2025 | `long-context` | `titans-2501` | 01-foundations | `microtitans` |
| 4 | Native Sparse Attention | 2025 | `architecture` | `nsa` | 03-systems | `micronsa` |
| 5 | Mamba-2 (Transformers are SSMs / Structured State Space Duality) | 2024 | `architecture` | `mamba-2` | 03-systems | `micromamba2` |

Slot 5 was originally proposed as DeepSeek-V3, then briefly considered for FeatureBench (rejected — benchmark fails SOP §3.1 criterion #1). Mamba-2 selected as final seed.

Slug choices explained:
- `turboquant`, `nsa` — unambiguous paper names, used as-is.
- `deepseek-r1` — paper name already hyphenated and lab-prefixed; natural to use directly. Avoids collision with any future R1 from a different lab.
- `titans-2501` — "Titans" is a generic noun and has been used as a paper name before. Disambiguator is the arXiv year-month (`2501` = January 2025).
- `mamba-2` — paper-canonical name. The hyphenated number is part of how the community refers to this paper; collision-safe against future Mamba-3.
- Script slugs keep existing `no-magic` naming pattern with no hyphens or separators (`micror1`, `micromamba2`).

---

## 2. Paper-by-paper rationale

### 2.1 TurboQuant

- **arXiv:** [2504.19874](https://arxiv.org/abs/2504.19874) (April 2025)
- **Primary theme:** `efficient-inference`
- **Secondary themes:** (none)
- **Target tier:** `03-systems`
- **Paper slug:** `turboquant`
- **Script slug (if implemented):** `microturboquant`

**Why it's in the seed set.** User-surfaced. A recent quantization method with a crisp algorithmic core — adaptive random rotations to decorrelate weight distributions before low-bit quantization, with theoretical guarantees on error bounds. Fits single-file scope: rotation + quantization + dequantization + inference over a toy linear layer or attention head.

**Single-file feasibility.** High. The method is matrix-multiplication level; a pedagogical CPU implementation on a small pretrained model (e.g. GPT-2 small or a toy transformer) is ~300–400 LOC.

**Companion lesson (optional).** Worth a lesson because quantization math intimidates more readers than it should; a walkthrough of rotation → quantize → dequantize → measure MSE has high pedagogical value.

---

### 2.2 DeepSeek-R1

- **arXiv:** [2501.12948](https://arxiv.org/abs/2501.12948) (January 2025)
- **Primary theme:** `reasoning`
- **Secondary themes:** `alignment`
- **Target tier:** `02-alignment`
- **Paper slug:** `deepseek-r1`
- **Script slug (if implemented):** `micror1`

**Why it's in the seed set.** The most influential LLM paper of early 2025. Demonstrates that reasoning capability can emerge from pure reinforcement learning (GRPO over rule-based rewards), without supervised fine-tuning as a prerequisite. Reshaped the field's approach to post-training. Ties into the existing `microgrpo.py` script (the algorithmic backbone) while adding the full recipe around it.

**Single-file feasibility.** Medium-hard. Full R1 needs a base model and a verifier; toy version is feasible — a tiny transformer + rule-based reward on a synthetic task (e.g. simple arithmetic or string manipulation where correctness is programmatically checkable) + GRPO loop. Can reuse patterns from `microgrpo.py`.

**Companion lesson (optional).** Strongly recommended. The conceptual shift — from "SFT then PPO" to "pure RL from a base" — is the lesson. Walkthrough shows why the training curve has its characteristic shape, what the "aha moment" is, and how rule-based rewards avoid reward-model distributional drift.

---

### 2.3 Titans: Learning to Memorize at Test Time

- **arXiv:** [2501.00663](https://arxiv.org/abs/2501.00663) (January 2025)
- **Primary theme:** `long-context`
- **Secondary themes:** `architecture`
- **Target tier:** `01-foundations`
- **Paper slug:** `titans-2501` (disambiguator: "Titans" is a generic noun; year-month tiebreaker)
- **Script slug (if implemented):** `microtitans`

**Why it's in the seed set.** Google Research paper proposing a neural long-term memory module that is updated at test time, combined with short-term attention. A novel architectural pattern that generalizes recurrent memory beyond the standard transformer. Speaks to a core `no-magic` value: exposing an architectural idea with minimum scaffolding.

**Single-file feasibility.** Medium. The memory module is a small MLP with gradient-based test-time updates; the combining mechanism with attention is mechanical. A toy version on a synthetic copying or retrieval task fits in 400–500 LOC.

**Companion lesson (optional).** Worth a lesson. Test-time learning is confusing to most readers; prose can separate "learns during training" from "learns during inference" more clearly than comments alone.

---

### 2.4 Native Sparse Attention

- **arXiv:** [2502.11089](https://arxiv.org/abs/2502.11089) (February 2025)
- **Primary theme:** `architecture`
- **Secondary themes:** `long-context`, `efficient-inference`
- **Target tier:** `03-systems`
- **Paper slug:** `nsa`
- **Script slug (if implemented):** `micronsa`

**Why it's in the seed set.** DeepSeek paper introducing natively-trainable sparse attention with hardware-aware design. Addresses long-context scaling without the usual accuracy cliff. Demonstrates that sparse attention can be a first-class training target, not a post-hoc optimization. Complements the existing Flash Attention and PagedAttention scripts in `no-magic`.

**Single-file feasibility.** Medium. Core idea is compressed + selective + sliding attention paths with learned gating. Toy CPU implementation loses the hardware-aware story but keeps the algorithmic one.

**Companion lesson (optional).** Recommended. The three-path structure (compressed, selective, sliding) benefits from a comparative diagram and prose walkthrough that code comments don't deliver as cleanly.

---

### 2.5 Mamba-2 — Structured State Space Duality

- **arXiv:** [2405.21060](https://arxiv.org/abs/2405.21060) (May 2024)
- **Authors:** Tri Dao, Albert Gu
- **Venue:** ICML 2024
- **Primary theme:** `architecture`
- **Secondary themes:** `efficient-inference`, `long-context`
- **Target tier:** `03-systems`
- **Paper slug:** `mamba-2`
- **Script slug (if implemented):** `micromamba2`

**Why it's in the seed set.** The Tri Dao / Albert Gu follow-up to Mamba that establishes the State Space Duality (SSD) framework — a theoretical bridge between SSMs and attention via structured semiseparable matrices. The paper produces a refined selective-SSM layer that is 2–8× faster than Mamba-1 while remaining competitive with Transformers on language modeling. Demonstrates that architectural innovation in 2024–2025 is happening via mathematical reframing, not just compute scaling. Ties into existing `no-magic` SSM scripts (`microssm`, `microcomplexssm`, `microdiscretization` per the 03-systems tier), making this card a natural test of the back-reference pattern (paper card → existing implementation, not just future implementation).

**Date note.** Mamba-2 is May 2024, slightly outside the strict "2025 or very late 2024" recency criterion. Relaxed for this seed because (a) the SSD framework crystallized only through 2024–2025 with follow-up implementations, (b) the paper anchors the architecture theme alongside NSA, and (c) it lets the seed set demonstrate the back-reference pattern against existing code on day one.

**Single-file feasibility.** High. The existing `microssm.py` (or analogue) already establishes the SSM scaffolding. Mamba-2 adds the SSD algorithm — a structured-matrix reformulation of the recurrence that admits efficient batched computation. CPU-pedagogical version compares Mamba-1 selective-SSM vs Mamba-2 SSD on a small synthetic task in 400–500 LOC.

**Companion lesson (optional).** Strongly recommended. The SSD theoretical bridge (attention ⟷ SSM duality) is the lesson — it explains why Mamba-2 exists at all, and why the field stopped treating SSMs and attention as separate paradigms. This is the kind of conceptual lesson that prose delivers better than code comments.

---

## 3. What the seed set demonstrates about the ecosystem

| Aspect | Demonstrated by |
|---|---|
| Recent papers get first-class treatment | 4 of 5 are 2025; Mamba-2 is May 2024 (relaxation explained in §2.5). |
| Theme diversity across 12-theme taxonomy | 4 primary themes (efficient-inference, reasoning, long-context, architecture) + multiple secondaries. |
| Tier spread | 01-foundations (1), 02-alignment (1), 03-systems (3). |
| User-surfaced papers are welcome | TurboQuant and Mamba-2 were user-proposed. |
| Major lab output is ingested | DeepSeek (×2), Google Research (×1), Princeton/CMU (Mamba-2), user curation (×1). |
| Paper and script slug conventions are distinct | `turboquant` (paper) ↔ `microturboquant` (script); `nsa` ↔ `micronsa`; `mamba-2` ↔ `micromamba2`. |
| Collision disambiguator is applied when needed | `titans` is generic → `titans-2501`. |
| Lessons are optional but recommended for dense concepts | All 5 warrant lessons. |
| Ties to existing `no-magic` content | R1 ↔ `microgrpo.py`; NSA ↔ Flash/Paged attention; Mamba-2 ↔ existing SSM scripts (`microssm`, `microcomplexssm`). |
| Triage gate is load-bearing | FeatureBench (originally suggested for slot 5) was correctly rejected as a benchmark per §3.1 criterion #1. |

---

## 4. Gaps the seed set does NOT cover

Seed diversity is deliberate but not total. Themes not represented in this 5-paper set:

- `alignment` (as primary) — DeepSeek-R1 uses it as secondary only. Next batch should include a primary-alignment paper (e.g. KTO, SimPO successor, or a 2025 preference-optimization variant).
- `interpretability` — covered by the separately-planned gap-fill batch (ROME, MEMIT).
- `parameter-efficient` — existing `no-magic` has LoRA and QLoRA; a DoRA or 2025 variant would be a good next card.
- `retrieval` — no 2025 retrieval paper in seed; existing `no-magic` has RAG and BM25.
- `safety-robustness` — covered by gap-fill (watermarking).
- `agents` — no dedicated agent paper in seed; existing `no-magic` has ReAct and MCTS.
- `multimodal` — intentionally deferred. Multimodal rarely fits single-file stdlib scope.
- `training-dynamics` — no seed paper. Muon optimizer (2024, viral in 2025) is the obvious next candidate.

Post-seed batches should close these gaps in a theme-coherent way.

---

## 5. Rollout plan for `no-magic-papers` v0.1

Locked sequence per strategy §9.20 (release sequence) and SOP §14.10 (first lesson target).

1. **Create the repo** with the layout in SOP Appendix A.
2. **Write `README.md`** with the single-sentence constraint: *"Paper cards in `papers/`, optional prose lessons in `lessons/`, paper-canonical slugs."*
3. **Write `THEMES.md`** mirroring SOP §3.5.
4. **Write `SCHEMA.md`** mirroring SOP §10 (with the list-valued `implementations:` shape).
5. **Write `CONTRIBUTING.md`** with the paper-first, no-tutorial-contact rules.
6. **Implement `scripts/generate_index.py`** plus the CI workflow that enforces SOP §7.3 invariants (1, 2, 4, 5; invariant 3 stays advisory until `no-magic` v3.0).
7. **Draft 5 paper cards** in this order, one PR each, maintainer-reviewed: `turboquant` → `deepseek-r1` → `titans-2501` → `nsa` → `mamba-2`.
8. **Generate initial `INDEX.md`** via the script. Commit the output.
9. **Draft 2 lessons** in this order: `lessons/turboquant.md` first (validates the lesson template on a mechanically straightforward concept), `lessons/deepseek-r1.md` second.
10. **Tag `v0.1`.** Update `about.html` and `.github/profile/README.md` repo tables to flip `no-magic-papers` from `next` to `live`.
11. **Stop.** Do not pursue gap-fill paper cards, additional lessons, or curriculum paths until v0.1 has absorbed feedback.

---

## 6. Review checklist for this proposal

- [ ] All 5 papers are correctly identified (title, authors, arXiv ID, year).
- [ ] Each paper matches the maintainer's taste and the ecosystem's direction.
- [ ] Proposed themes align with SOP §3.5 taxonomy.
- [ ] Proposed slugs don't conflict with existing `no-magic` scripts.
- [ ] Tier assignments are appropriate.
- [ ] Substitution candidates in §7 are worth considering.
- [ ] Rollout plan in §5 is realistic for solo-maintainer bandwidth.

---

## 7. Substitution candidates

If any of the five proposed papers don't fit maintainer taste, swap from this shortlist. Candidates are grouped by theme so substitutions preserve the diversity property.

### Replace TurboQuant (`efficient-inference`)
- **SpinQuant** (2405.16406, 2024) — rotation-based quantization with learned rotation matrices.
- **QuaRot** (2404.00456, 2024) — similar rotation family, earlier.

### Replace DeepSeek-R1 (`reasoning`)
- **Absolute Zero Reasoner** (2505.03335, May 2025) — self-play reasoning without external data. *Verify arXiv ID before use.*
- **Process Reward Model papers** of 2025 (e.g. Math-Shepherd successor work).

### Replace Titans (`long-context`)
- **Differential Transformer** (2410.05258, 2024) — attention variant that subtracts noise.
- **LongRoPE successors** of 2025.

### Replace Native Sparse Attention (`architecture`)
- **Byte Latent Transformer (BLT)** (2412.09871, December 2024) — tokenization-free architecture.
- **Differential Transformer** (2410.05258) — attention-noise subtraction.

### Replace Mamba-2 (`architecture`)
- **DeepSeek-V3 (MLA + aux-free MoE)** (2412.19437, late 2024 / 2025) — original slot-5 candidate.
- **Tülu 3** (2411.15124, late 2024) — alignment recipe; would shift the paper to primary `alignment`.
- **Muon optimizer** write-up — primary `training-dynamics`; fills a theme gap.
- **Phi-4** (2412.08905) — small-model training recipe.

---

## 8. Open questions

None currently open. All seed-related questions resolved.

**Resolved batch (2026-04-25):**
- Slot 5 → Mamba-2. DeepSeek-V3 moved to substitution shortlist; FeatureBench rejected at triage (benchmark — fails SOP §3.1 criterion #1).
- Tied-to-existing-script demonstration → satisfied by Mamba-2 ↔ `microssm.py` family on day one. No separate backfill batch needed for v0.1.
- Multi-script-per-card schema → list-valued `implementations:` field (SOP §10, §14.12).
- First lesson author target → TurboQuant (strategy §9.19, SOP §14.10).

