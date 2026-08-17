# Sagan

A Claude Code plugin for multi-round, parallel-agent deep research. Plan a brief, fan out one agent per topic, synthesise across rounds, propose follow-up topics — then cut the finished corpus along whatever axis you need. Citations are mandatory and tier-tagged.

## Installation

```bash
claude plugin marketplace add robertbagge/claude-registry
claude plugin install sagan@claude-registry
```

## `/create-brief` — plan a research project

Interactive wizard that captures the research domain, goal, inspirations, constraints, initial topics, and source-type preference, then writes a structured `brief.md` that drives `/deep-research`. The brief is the single source of truth for everything the parallel agents need.

```
/create-brief                   # start a new research project
```

The wizard runs a six-question pass via `AskUserQuestion`, then a one-question-at-a-time follow-up loop that sharpens each topic into a concrete research prompt. Output is left uncommitted — review and edit the brief directly before running research.

## `/deep-research` — run a research round

Reads a brief, dispatches one parallel Task agent per missing topic, fully rewrites the synthesis across all rounds, then proposes gap topics for the next round and appends them to the brief. The user re-invokes to execute each new round.

```
/deep-research meta/research/{domain}/brief.md
/deep-research                  # auto-detects single brief in cwd
```

Each invocation is one round. Rounds are append-only — earlier rounds and their topic files are never rewritten. The synthesis is fully rewritten each round so it integrates everything the corpus knows so far.

## `/synthesize` — cut the corpus along an axis

Takes a finished corpus and writes a synthesis of one seam that runs across the topic files, in the shape the reader needs. Corpus-only: it never runs web searches, and material the axis needs but the corpus lacks is reported as a coverage gap rather than filled in.

```
/synthesize                                     # reads the corpus, proposes candidate axes
/synthesize how power tiers get adjudicated     # axis given, asks for the shape
/synthesize implementation guide for retrieval  # shape inferred from wording, no dialog
```

One extraction agent runs per topic file — every file, no pre-triage — each carrying its citations across verbatim. The orchestrator writes from those extracts to `synthesis/{axis-slug}.md`.

Six modes: **briefing** (dense peer-level overview), **recommendation** (a call with reasoning exposed), **implementation guide** (prerequisites, steps, pitfalls), **comparison** (contenders against criteria), **primer** (from zero, terms defined on first use), **timeline** (how the axis evolved). The mode is inferred when the argument names it, otherwise asked.

## Project layout

A research project is a folder with three parts:

```
{output_dir}/
├── brief.md              # written by /create-brief; the contract both skills read
├── research/             # one file per topic, 1500–4000 words, densely cited
│   ├── {topic-a}.md
│   └── {topic-b}.md
└── synthesis/
    ├── general.md        # whole-corpus synthesis, fully rewritten each round
    └── {axis}.md         # written by /synthesize, one per axis
```

`brief.md` is the only file at the top level. Topic `File:` paths in the brief are relative to the output dir and always live under `research/`. `general.md` is owned by `/deep-research` and rewritten every round; every other file in `synthesis/` is an axis cut written by `/synthesize` and left alone by the research loop.

## Output folder resolution

`/create-brief` runs a four-rung cascade to find where research output should live. It stops at the first match.

1. **`sagan-load-output-folder` skill** — consumer-defined hook (see below).
2. **Harness instructions** — if the harness in context documents an AI/research output folder under any naming, use it.
3. **Repo heuristic** — scans for directories containing existing `brief.md` files, or common patterns (`meta/research/`, `docs/research/`, `research/`, `ai/research/`).
4. **Default** — `meta/research/`.

The wizard then confirms the resolved path with the user; the brief stores it as `**Output dir**:`.

### `sagan-load-output-folder` skill

Sagan documents `sagan-load-output-folder` as a public extension point but does not ship it as an active skill. To pin the output folder for your repo, copy the template at the plugin root into your environment and edit the path:

```
sagan-load-output-folder/SKILL.template.md   →   <your-target>/sagan-load-output-folder/SKILL.md
```

Valid targets include:

- `.claude/skills/sagan-load-output-folder/SKILL.md` — repo-scoped (this repo only).
- `~/.claude/skills/sagan-load-output-folder/SKILL.md` — user-scoped (all your repos).
- Inside another plugin's `skills/` directory — distribute it to a team.

The template is intentionally minimal:

```markdown
---
name: sagan-load-output-folder
description: Loads output folder for the sagan plugin
---

output_folder: <add your folder here>
```

Replace `<add your folder here>` with the absolute or repo-relative path you want sagan to use (e.g. `meta/research/`, `docs/research/`, `ai-output/research/`). When `/create-brief` runs, it reads this skill's body, picks up the `output_folder:` line, and uses that as the root.

## Source types

Briefs declare a per-project source preference using three types:

- **Type 1 (`[T1]`)** — papers / peer-reviewed work / conference proceedings. Best for scientific research. Cite as: source + page + paragraph.
- **Type 2 (`[T2]`)** — official documentation, technical specs, white papers, books, renowned domain blogs. Best for tech and industry research. Cite as: source + subsection (if any) + line number.
- **Type 3 (`[T3]`)** — Twitter/X, Hacker News, Reddit, small blogs, podcasts, conference videos. Best for cultural pulse and trends. Cite as: source + URL.

The brief specifies which types apply and in what priority for the project (e.g. "Type 2 primary, Type 3 secondary, Type 1 not relevant"). Every inline citation is tagged with its type so the synthesis can spot coverage gaps relative to the stated preference.

## Brief contract

`/deep-research` parses these headers from the brief:

- `**Output dir**:` — project root; holds `brief.md`, `research/`, and `synthesis/`.
- `**Domain**:` — kebab-case slug.
- `## Goal` — one-sentence goal of the research.
- `## Inspirations & references` — reference material the agents draw on.
- `## Constraints & scope` — what's out of scope.
- `## Source types` — passed verbatim to each topic agent.
- `## Round N` sections — each contains topic blocks with title, `File:` line (`research/{topic-slug}.md`, relative to the output dir), and prompt.

Don't deviate from the structure if you edit the brief by hand.

## Examples

Two complete research corpuses are checked in under `meta/` to show what a finished project looks like — brief, per-topic files, and synthesis.

- [`meta/marvel-universe/`](meta/marvel-universe/) — fictional-world research across cosmology, factions, power scaling, continuities, and crossovers. Demonstrates Type 2/3 source mix on a sprawling, well-documented domain.
- [`meta/natural-language-processing/`](meta/natural-language-processing/) — technical history of NLP from symbolic era through transformers, RLHF, and frontier directions. Demonstrates Type 1/2-heavy sourcing on a scientific domain.

Read the `brief.md` in each folder to see how the project was scoped, then `synthesis/general.md` for the integrated output.

## Files

```
claude-sagan-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── create-brief/
│   │   └── SKILL.md
│   ├── deep-research/
│   │   └── SKILL.md
│   └── synthesize/
│       └── SKILL.md
├── sagan-load-output-folder/
│   └── SKILL.template.md   # copy into your env to pin the output folder
├── meta/
│   ├── marvel-universe/             # example research corpus
│   │   ├── brief.md
│   │   ├── research/
│   │   └── synthesis/general.md
│   └── natural-language-processing/ # example research corpus
├── README.md
└── LICENSE
```

## Why "Sagan"

Carl Sagan as the patron saint of deep research grounded in rigorous external sources — curiosity, exhaustive coverage, citations not claims. The plugin doesn't have an opinion on what you should research, just on how. Find the primary sources, tag them by type, and write the synthesis as if someone might actually act on it.
