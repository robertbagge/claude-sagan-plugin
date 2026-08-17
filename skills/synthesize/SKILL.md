---
name: synthesize
description: Synthesise an existing research corpus along one axis, in a chosen shape (briefing, recommendation, implementation guide, comparison, primer, timeline). Use when cutting a finished research corpus along a seam or topic, or asking what the research says about one specific question.
argument-hint: "[axis]"
---

# synthesize

Cut an existing research corpus along one axis and write a synthesis in the shape the reader needs.

`/deep-research` produces breadth: one file per topic, plus `synthesis/general.md` covering the whole corpus. `/synthesize` produces depth on a seam that runs *across* those topics — what the corpus knows about one question, assembled into one document.

```
{output_dir}/
├── brief.md
├── research/                 # read by this skill, never written
│   └── {topic-slug}.md
└── synthesis/
    ├── general.md            # owned by /deep-research — never write this
    └── {axis-slug}.md        # written by this skill
```

## Why this matters

Two properties make a synthesis trustworthy, and both are easy to lose:

- **Citations survive the trip.** Material passes through an extraction agent before it reaches the synthesis. If that agent paraphrases a claim and drops its `[T2]` tag, the synthesis inherits an unsourced assertion that reads as sourced. Extraction is a copying job, not a writing job.
- **Nothing is silently skipped.** Every topic file gets an agent, even ones that look unrelated from their one-line summary. A seam that runs across a corpus surfaces in files nobody would have triaged into scope.

This skill is **corpus-only**. It never runs web searches and never introduces material that isn't already in `research/`. When the axis needs something the corpus lacks, that goes in the Coverage gaps section — which is the input to the next `/deep-research` round, not something to paper over.

## Modes

The mode determines the document's skeleton. Six in the catalog:

| Mode | For a reader who wants | Skeleton |
| --- | --- | --- |
| **Briefing** | What the corpus knows about the axis, densely, as a peer | Sub-axes of the topic, 3–6 H2s |
| **Recommendation** | A call, with the reasoning exposed | Options → Recommendation → Rationale → Risks → What would change this |
| **Implementation guide** | To build the thing, decision already made | Prerequisites → Sequenced steps → Pitfalls → Checkpoints |
| **Comparison** | To weigh contenders against criteria | Criteria → Per-option assessment → Trade-offs |
| **Primer** | To learn the axis from zero | Layered: foundations → mechanics → current state, terms defined on first use |
| **Timeline** | How the axis got here | Chronological phases, each with what changed and what forced it |

Section skeletons are in Step 6.

## Workflow

### Step 1 — Resolve the corpus

The argument may carry an axis, a mode, a corpus path, or any combination. Pull out a path if one is present; the rest is axis/mode text.

Resolve the corpus:

- **Path given** — accept either a project dir or a `brief.md` inside it.
- **No path** — if exactly one `meta/research/*/brief.md` exists in or under cwd, use it. This only checks the default location; corpora under a custom output dir won't be found this way.
- **Ambiguous or nothing found** — list what you found and ask. Don't guess.

Then check the layout:

- `{output_dir}/research/` must exist and hold at least one topic file. If it's empty, stop: there's nothing to synthesise, and the user needs `/deep-research` first.
- **Legacy flat corpus** (topic files directly in `{output_dir}/`, or a `{output_dir}/synthesis.md`) — stop and offer to migrate: `git mv` topic files into `research/`, `synthesis.md` to `synthesis/general.md`, then fix the relative links inside it. Don't synthesise across a half-migrated corpus.

### Step 2 — Read the map, not the corpus

Read only these:

- `brief.md` — Goal, Constraints & scope, Source types, and the round topic prompts.
- `synthesis/general.md` if it exists — its Contents section gives you every topic title with a one-line summary; its Cross-cutting themes, Tensions, and Open questions sections are the best available list of candidate axes.
- A directory listing of `research/` — the authoritative file list. `general.md` can be stale; the directory isn't.

**Do not read the topic files here.** The extraction agents do that in parallel in Step 5. Reading them in the orchestrator burns the context budget the synthesis itself needs.

### Step 3 — Resolve axis and mode

**Mode inference.** If the argument explicitly names a shape, take it and skip the mode question — state the inferred mode in your first reply so a wrong read is cheap to correct. Signals:

| Argument wording | Mode |
| --- | --- |
| "implementation guide", "how do we build/implement/ship", "how would we do" | Implementation guide |
| "should we", "recommend", "which should", "make the call" | Recommendation |
| "compare", "X vs Y", "weigh", "trade-offs between" | Comparison |
| "primer", "explain to someone new", "intro to", "from scratch" | Primer |
| "history of", "timeline", "how did X evolve", "over time" | Timeline |
| "brief me on", "what does the research say about", bare noun phrase | *no signal — ask* |

A bare noun phrase is not a signal. "Power scaling" tells you the axis and nothing about the shape.

**The dialog.** Load `AskUserQuestion`:

```
ToolSearch(query: "select:AskUserQuestion", max_results: 1)
```

Send **one call** carrying only the questions still open — the axis question if no axis came in the argument, the mode question if inference didn't resolve it. If both are already resolved, skip the dialog entirely.

*Axis question* — "Which axis should the synthesis cut along?" Generate 4 candidate seams from the map you read in Step 2. The strongest candidates come from `general.md`'s Cross-cutting themes (already proven to run across topics), Tensions and trade-offs (already proven to be contested), and Open questions. Name each candidate concretely — "how power tiers are actually adjudicated in-story", not "power scaling". Always include "Other (specify)".

*Mode question* — "What shape should the synthesis take?" The catalog has six entries and the dialog shows four, so pick the four that fit this axis and corpus. **Always include Briefing.** Choose the other three on fit: Timeline only when the corpus carries dated material along this axis; Implementation guide only when the axis is about building something; Comparison only when the corpus covers multiple contenders. Describe each option by what the reader gets, not by its name. "Other (specify)" covers the rest of the catalog.

**One sharpening question.** If the mode is Recommendation or Implementation guide and the axis is a bare noun phrase, ask one plain-text question about the decision at stake — "recommendation for whom, and what are they choosing between?" These two modes are worthless without a decision to aim at; the other four are fine on a topic alone.

Close the step by writing the **axis intent**: 1–2 sentences stating what this synthesis has to answer. It goes verbatim into every extraction prompt, and it is what stops twenty agents from each interpreting the axis differently.

### Step 4 — Derive the output path

`{output_dir}/synthesis/{axis-slug}.md`, slug kebab-case, lowercase, ASCII, derived from the axis.

- **`general.md` is reserved.** If the slug lands there, use a more specific one — that file belongs to `/deep-research` and this skill never writes it.
- **File already exists** — ask: overwrite, or write to `{axis-slug}-{mode}.md`? Default the suggestion to the suffixed name when the existing file's mode differs from this one, since two shapes of the same axis are both worth keeping.

Tell the user the axis, mode, output path, and how many extraction agents you're about to dispatch. Then go.

### Step 5 — Dispatch parallel extraction agents

**One agent per topic file in `research/`. Single message, multiple tool calls.** Splitting them across messages runs them serially. Dispatch to *every* topic file — do not pre-triage from the one-line summaries in `general.md`. A file that returns `NO RELEVANT MATERIAL` costs one cheap agent; a file wrongly triaged out costs a hole in the synthesis that nothing downstream will catch.

Use `subagent_type: "general-purpose"` for every call — it's the only universally-available agent type, and the skill has to work in any repo or plugin context.

Build each prompt from this template, substituting everything in `{...}`:

```
You are extracting material from ONE file of an existing research corpus. Your extract will be combined with extracts from {N-1} sibling agents, each reading a different file, into a single synthesis document. You are not writing that synthesis — you are supplying sourced raw material for it.

SOURCE FILE (read this file, and only this file): {output_dir}/research/{topic-slug}.md
CORPUS GOAL: {goal verbatim from brief}
SYNTHESIS AXIS: {axis}
AXIS INTENT: {axis intent from Step 3, verbatim}
SYNTHESIS MODE: {mode} — {mode's "For a reader who wants" text from the catalog}

WHAT TO EXTRACT
- Every passage in the source file that bears on the axis, including passages that complicate, qualify, or contradict it. Contradictions are signal. Never drop a passage because it conflicts with the rest of the file or with what you expect the synthesis to conclude.
- Material the MODE specifically needs: {mode-specific extraction guidance from the table below}
- When in doubt about relevance, include it and mark Relevance: low. Over-inclusion is cheap; a silent omission is not recoverable downstream.

HARD RULES
1. Read only the SOURCE FILE. Do not read other files in the corpus — sibling agents are covering them, and the orchestrator has the brief.
2. Do NOT use web search or any other external source. Do not add anything from your own knowledge. If it is not in the source file, it does not go in your extract.
3. Do NOT write, create, or modify any file. Your entire output is your final message.
4. Carry citations verbatim. When you extract a claim, copy its inline citation exactly as written in the source file — type tag ([T1]/[T2]/[T3]) and every piece of metadata inside it. Never reformat, shorten, renumber, merge, or split a citation.
5. Never move a citation between claims. If a claim in the source file carries no inline citation, you may still extract it, but prefix that bullet with [UNCITED]. Attaching a neighbouring citation to a claim it did not belong to is the worst failure available to you.
6. Copy numbers, dates, names, versions, and quoted text exactly. Paraphrase connective prose only.
7. Extract; do not reason. No inference, no extrapolation, no conclusions, no "this suggests". Synthesis-level reasoning is the orchestrator's job and it needs your material unmixed with your opinions.
8. If the source file contains nothing bearing on the axis, your entire reply is exactly: NO RELEVANT MATERIAL

OUTPUT FORMAT — return exactly this in your final message, no preamble, no sign-off:

## Extract: {topic title}

Source file: research/{topic-slug}.md
Relevance: high | medium | low — {one sentence on how this topic bears on the axis}

### Findings

- {Claim, with its inline citation copied verbatim.}
- {...}

### Contradictions and caveats

- {Passages that cut against the grain of the findings above, with citations. Or "None."}

### Sources cited above

- {For every citation appearing anywhere above, copy its full entry from the source file's ## Sources section, verbatim. If a citation has no matching entry in that section, write: "{citation} — no Sources entry in source file".}

LENGTH: as long as the material warrants, typically 300–1500 words. Do not pad. A file with two relevant paragraphs should produce a short extract, not an inflated one.
```

Mode-specific extraction guidance to substitute in:

| Mode | Guidance |
| --- | --- |
| Briefing | Established facts, mechanisms, and current state along the axis, with their strength of evidence. |
| Recommendation | Options and alternatives, evidence for and against each, stated costs, risks, failure cases, and any explicit recommendations the source file already makes. |
| Implementation guide | Concrete procedures, sequences, prerequisites, configuration, thresholds, gotchas, and anything reported to have gone wrong in practice. |
| Comparison | Named contenders, the dimensions they're compared on, and any head-to-head evidence — including where the source declines to pick a winner. |
| Primer | Definitions, first principles, the minimum background a newcomer needs, and worked examples or analogies the source uses. |
| Timeline | Dated events, sequence, causes and precipitating forces, and turning points — with dates copied exactly. |

Send every Task call in **one** assistant message. Wait for all to return.

### Step 6 — Write the synthesis

First, check what came back:

- Count the extracts that returned material. If **zero or one** topic file had anything relevant, stop and tell the user: the corpus doesn't cover this axis. Offer to narrow the axis, pick a different one, or run a `/deep-research` round to cover it. Don't write a synthesis off one file and present it as a cross-corpus view.
- Note any agent that failed outright or returned something other than the specified format. Surface those files to the user before writing — a missing extract is a hole in the synthesis.

Then write `{output_dir}/synthesis/{axis-slug}.md`. Every mode shares this frame:

```markdown
# {Axis title} — {Mode}

**Corpus**: {Domain Title}
**Last updated**: {YYYY-MM-DD}
**Drawn from**: {M} of {N} topic files

## Contents

- [Brief](../brief.md) — research brief that drove this corpus
- [General synthesis](./general.md) — whole-corpus overview
- Topic files drawn on: [{title}](../research/{slug}.md), [{title}](../research/{slug}.md), ...

{mode-specific body — see skeletons below}

## Coverage gaps

{What this axis needs that the corpus doesn't have. Be specific enough to become a topic prompt in the next round — "no material on X" beats "coverage is thin". Include claims that arrived [UNCITED], since those are gaps in sourcing rather than gaps in coverage.}

## Sources

{Every source cited in the body, assembled from the extracts' "Sources cited above" sections. Deduplicate across extracts — the same source cited by three topics appears once. Keep type tags. Keep the citation format the brief defines.}
```

Mode skeletons for the body:

- **Briefing** — 3–6 H2s along the axis's natural sub-axes. No generic template; let the material set the structure. Dense, peer-level, no ramp.
- **Recommendation** — `## Options` (each with its evidence), `## Recommendation` (the call, stated plainly in the first sentence), `## Rationale`, `## Risks and unknowns`, `## What would change this`.
- **Implementation guide** — `## Prerequisites`, `## Steps` (sequenced, each with what it depends on), `## Pitfalls` (drawn from what the corpus reports going wrong), `## Checkpoints` (how to tell each stage worked).
- **Comparison** — `## Criteria` (derived from the axis, stated before any option is assessed), `## {Option}` per contender against those criteria, `## Trade-offs`. End on trade-offs, not a verdict — that's what Recommendation is for.
- **Primer** — `## Foundations`, `## Mechanics`, `## Current state`, `## Where to go deeper` (pointing into the topic files). Define every term on first use.
- **Timeline** — one H2 per phase in chronological order, each covering what changed and what forced it, then `## Where it stands now`.

Rules that hold across all modes:

- **Length follows the material.** 1000–4000 words is typical. Don't pad a thin axis to hit a number; don't truncate a rich one.
- **Every non-obvious claim carries its citation**, with the type tag, in the format the brief defines. A claim you can't trace to an extract doesn't go in the document.
- **`[UNCITED]` material stays marked** or gets dropped. Never promote it to sourced.
- **Contradictions get surfaced, not smoothed.** If topics disagree, say so and attribute both sides. A synthesis that reads as more settled than the corpus is a broken synthesis.
- **Link back to topic files** as `[Topic title](../research/{slug}.md)` when pointing at where something is treated in depth.
- **No research-process mechanics.** Write standalone prose — no "the extraction agents found", "topic 4 covers", "this round". The header states the axis and mode; that's the only structural acknowledgement allowed.

### Step 7 — Hand off

Tell the user:

1. The output path.
2. Axis and mode (and, if the mode was inferred from the argument, that it was inferred).
3. How many topic files contributed, and which returned nothing — an unexpected `NO RELEVANT MATERIAL` is often more informative than the synthesis.
4. The coverage gaps, as a short list, with the note that they can go straight into a `/deep-research` round.

Then stop. Don't start another synthesis, and don't modify the brief — this skill never touches `brief.md`.

## Notes

- **Corpus-only, always.** No web search, in the orchestrator or in any agent. New external evidence enters through `/deep-research`, not here.
- **Single-message parallel dispatch** is the difference between a minute and an hour.
- **One agent per topic file, no pre-triage.** Silent omissions are the failure this design exists to prevent.
- **`synthesis/general.md` belongs to `/deep-research`.** Read it, link to it, never write it.
- **`research/` is read-only** to this skill, as is `brief.md`. The only file written is `synthesis/{axis-slug}.md`.
- **Single agent type only.** Always `subagent_type: "general-purpose"`.
- **Don't commit.** Leave the synthesis uncommitted for review.
- **No emojis.** Skill outputs should not contain emojis.
