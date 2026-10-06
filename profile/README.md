[![no-magic](https://raw.githubusercontent.com/no-magic-ai/no-magic/main/assets/banner.png)](https://no-magic-ai.github.io)

---

**Because `model.fit()` isn't an explanation.**

`no-magic-ai` is an umbrella for studying how modern AI actually works. Not a framework. Not a course platform. Not a chatbot. Every repo under this org enforces a single-sentence constraint. Every algorithm points back to a paper. Everything else is off-charter.

Our audience is two people: the ML engineer who uses frameworks daily and wants the internals, and the career switcher who read the papers and needs a working implementation they can run on a laptop. Everything we build serves one of these two.

Read the full manifesto at [no-magic-ai.github.io/about.html](https://no-magic-ai.github.io/about.html).

---

## Repositories

One constraint per repo. No bleed. Repos do not share build systems and do not import each other's code. Cross-repo coupling is limited to published metadata and links: `no-magic`'s generated `docs/catalog.json` is read by the website and checked against paper cards by `no-magic-papers` CI. The website is the only learner surface that joins them.

| Repo | Single-sentence constraint | Status |
|------|---------------------------|--------|
| [**no-magic**](https://github.com/no-magic-ai/no-magic) | One algorithm per file. Stdlib only. CPU only. Runs in under 10 minutes. | live |
| [**no-magic-viz**](https://github.com/no-magic-ai/no-magic-viz) | One Manim scene per algorithm. Renders to MP4 and GIF. | live |
| [**no-magic-ai.github.io**](https://github.com/no-magic-ai/no-magic-ai.github.io) | Static HTML portal. No JS framework. Federates the ecosystem. | live |
| [**no-magic-papers**](https://github.com/no-magic-ai/no-magic-papers) | One markdown per paper, plus companion lessons. Summary, contribution, status, link to implementation. | live |
| [**no-magic-paths**](https://github.com/no-magic-ai/no-magic-paths) | Curated reading orders with prerequisites across algorithms and lessons. | planned |
| [**no-magic-labs**](https://github.com/no-magic-ai/no-magic-labs) | Exercises and applied projects that exceed one-file scope. | later |
| [**apprentice**](https://github.com/no-magic-ai/apprentice) | Maintainer-side agent pipeline that drafts algorithm entries from papers for human review. Uses its own declared dependencies and LLM providers; not a learner artifact. | implemented (v0.4.0); activation deferred |

---

## What no-magic-ai refuses to become

The value of this org is in what it won't do. Every feature we reject keeps the scope tight enough that the code stays readable and the constraints stay honest. These are off-charter, not deferred.

- **Not an LMS.** No user accounts. No progress tracking. No auth. No server-side state.
- **Not a chatbot.** No RAG over the catalog. No AI tutor. A good single-file implementation is already the explanation.
- **Not a framework.** No shared base classes. No abstract interfaces. No `no_magic.core` package.
- **Not a tutorial mirror.** No translations or ports of fast.ai, Karpathy, HuggingFace, or dive-into-llms. We implement from primary sources only.
- **Not a production library.** The code is pedagogical. Not performant, not distributed, not hardened.
- **Not a benchmark.** We show how algorithms work, not which beats which.

---

## Org-wide principles

- **Paper-first sourcing.** Every implementation cites the original paper. No content is ported from tutorials, courses, or other educational repos.
- **Single-sentence charter per repo.** A new repo requires a one-sentence constraint that governs it. Violating the constraint is grounds for PR rejection.
- **Two explicit slug namespaces.** Script slugs are `no-magic` file basenames (e.g. `microrope`, `attention_vs_none`); paper slugs are canonical paper-card names (e.g. `rope`, `gpt-1`). One paper card may own several scripts. Every script-to-paper link is written out (`no-magic/scripts/generate_catalog.py` and card `implementations[]`); neither slug is ever derived from the other.
- **Learner artifacts run on commodity hardware.** Learner code runs on a laptop CPU: no GPU, no cloud, no dataset larger than a few MB. Some `no-magic` scripts download a small dataset on first run; the catalog records which. This constraint governs learner artifacts, not maintainer tooling: `apprentice` and the repos' CI/render tooling use their own declared dependencies, and `apprentice` calls hosted or local LLM providers.
- **Comments are the curriculum.** Code is the primary teaching artifact. Prose explanations live in `no-magic-papers/lessons/` and remain optional, never authoritative.
- **Provenance over abstraction.** Three similar implementations beat one abstracted parent class. Duplication is honest; abstraction hides.
- **Deletion is a feature.** Content that stops serving the two audiences gets removed. The catalog is curated, not accreted.
- **Static or nothing for learners.** No databases, services or background jobs behind any learner surface. `apprentice` is a maintainer-run CLI that keeps local run records; it is not a hosted service and is not activated for autonomous operation.

---

## Sourcing policy

Binding on every contributor and every agent that operates on these repos.

1. **Paper-first.** Every algorithm implementation cites its sources in a reference comment directly after its thesis docstring and is explicitly mapped to a paper card. Where the cited sources or the implemented method differ from that card, the catalog's adaptation note says so. If no paper exists, the algorithm does not belong in the catalog.
2. **No tutorial contact.** Do not read other educational repos (fast.ai, Karpathy's nn-zero-to-hero, HuggingFace courses, dive-into-llms) while writing a `no-magic` script. Topic inspiration from their tables of contents is permitted.
3. **Attribution is a line, not a link.** Cite sources in docstrings with full bibliographic detail.
4. **License vigilance.** Before adopting any external asset, verify and record the license in `ASSETS.md`.
5. **Contributor pledge.** Every contributor affirms they implemented from papers, not from tutorials. Enforcement is by maintainer review, not by CI.

---

## Philosophy (core repo specifics)

- **One file per algorithm.** No local imports, no `utils.py`, no companion files.
- **Zero external dependencies.** Python standard library only.
- **`python script.py` runs everything.** For most scripts that is train + inference in one command; the exceptions (side-by-side training comparisons, forward-pass mechanism demonstrations and non-learning algorithm demonstrations) are recorded per script in [`docs/catalog.json`](https://github.com/no-magic-ai/no-magic/blob/main/docs/catalog.json).
- **Comments are the curriculum.** 30–40% comment density, math-to-code mappings, intuition over jargon.

---

## Contributing

Every script is written for the person reading it for the first time. Contributions that change algorithms, paper cards, lessons, curriculum explanations or exercise answers require the maintainer's content approval. For the current enhancement phase only, purely technical PRs (tooling, CI, generated metadata, source-state corrections) may be merged by a gated runner after independent review, verification and all required checks, as recorded in [no-magic issue #39](https://github.com/no-magic-ai/no-magic/issues/39); this does not replace scientific review. See [CONTRIBUTING.md](https://github.com/no-magic-ai/no-magic/blob/main/CONTRIBUTING.md) in each repo for the review checklist.

## Governance

Decisions live in versioned documents, not processes:

- [`no-magic-ai-expansion-strategy.md`](https://github.com/no-magic-ai/.github/blob/main/docs/no-magic-ai-expansion-strategy.md) — top-level strategy doc.
- [`paper-ingestion-process.md`](https://github.com/no-magic-ai/.github/blob/main/docs/paper-ingestion-process.md) — how papers enter the ecosystem.
- Per-repo `CONTRIBUTING.md` — encodes each repo's single-sentence constraint.
- [GitHub Discussions](https://github.com/no-magic-ai/no-magic/discussions) — open proposals and curriculum debates.

## Maintainer

Created and maintained by [Tom Mathews](https://github.com/Mathews-Tom) ([LinkedIn](https://www.linkedin.com/in/mathews-tom/)).

---

![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue?style=flat-square&logo=python&logoColor=white)
![License: MIT](https://img.shields.io/github/license/no-magic-ai/no-magic?style=flat-square)
![Zero Dependencies](https://img.shields.io/badge/dependencies-zero-brightgreen?style=flat-square)
![GitHub stars](https://img.shields.io/github/stars/no-magic-ai/no-magic?style=flat-square)
