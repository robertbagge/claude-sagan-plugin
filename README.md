# Sagan

A Claude Code plugin for multi-round, parallel-agent deep research. Plan a brief, fan out one agent per topic, synthesise across rounds, propose follow-up topics. Citations are mandatory and tier-tagged.

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

## Output folder resolution

`/create-brief` runs a four-rung cascade to find where research output should live. It stops at the first match.

1. **`sagan-load-output-folder` skill** — consumer-defined hook (see below).
2. **Harness instructions** — if the harness in context documents an AI/research output folder under any naming, use it.
3. **Repo heuristic** — scans for directories containing existing `brief.md` files, or common patterns (`meta/research/`, `docs/research/`, `research/`, `ai/research/`).
4. **Default** — `meta/research/`.

The wizard then confirms the resolved path with the user; the brief stores it as `**Output dir**:`.

### `sagan-load-output-folder` hook

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

- `**Output dir**:` — where topic files and synthesis live.
- `**Domain**:` — kebab-case slug.
- `## Goal` — one-sentence goal of the research.
- `## Inspirations & references` — reference material the agents draw on.
- `## Constraints & scope` — what's out of scope.
- `## Source types` — passed verbatim to each topic agent.
- `## Round N` sections — each contains topic blocks with title, `File:` line, and prompt.

Don't deviate from the structure if you edit the brief by hand.

## Files

```
claude-sagan-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── create-brief/
│   │   └── SKILL.md
│   └── deep-research/
│       └── SKILL.md
├── sagan-load-output-folder/
│   └── SKILL.template.md   # copy into your env to pin the output folder
├── README.md
└── LICENSE
```

## Why "Sagan"

Carl Sagan as the patron saint of deep research grounded in rigorous external sources — curiosity, exhaustive coverage, citations not claims. The plugin doesn't have an opinion on what you should research, just on how. Find the primary sources, tag them by type, and write the synthesis as if someone might actually act on it.
