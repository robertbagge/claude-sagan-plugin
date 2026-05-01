---
name: deep-research
description: Run the next round of a multi-round, parallel-agent deep-research project. Use when continuing a deep-research project, executing a research brief, or doing the next research pass.
argument-hint: "[brief-path]"
---

# deep-research

Execute one round of deep research from a brief produced by `/create-brief`.

Each invocation runs one research round and stages the next:

1. **Run the next pending round** — dispatch one parallel Task agent per missing topic, each writing to `{output_dir}/{topic-slug}.md`.
2. **Fully rewrite `{output_dir}/synthesis.md`** — incorporating every round executed so far. Tight executive synthesis, not a table of contents in the body.
3. **Propose gap topics for the next round** — one-question-at-a-time conversation, then append `## Round N+1` to the brief and stop. The user re-invokes to execute the new round.

## Why this matters

The user's "better too much research than too little" workflow only works if (a) topic agents run in **true parallel** — a single message with N tool calls, since sequential calls across messages run serially and waste hours; (b) each topic agent does **fresh external web research** rather than relying on training data; (c) the synthesis is **rewritten** each round so it integrates everything rather than accreting fragments; and (d) the brief is **append-only** so it stays an audit trail of what was researched and when.

Each topic file produced by an agent has this shape:

- 1500–4000 word Markdown file.
- 4–8 H2 sections, structured around the topic's natural axes (no generic template).
- Dense inline citations to external sources, each tagged with its type (`[T1]`, `[T2]`, `[T3]`) per the brief's Source types section.
- A `## Sources` section at the bottom with full citation metadata.

Each synthesis has this shape:

- 1500–2500 word Markdown file.
- Five H2 sections in this order: Contents (links to brief and topic files), Executive summary, Cross-cutting themes, Tensions and trade-offs, Open questions.
- Executive prose, not a list of facts. Navigation lives in the Contents section so the body can stay synthetic.

## Workflow

### Step 1 — Resolve the brief

Argument: `[brief-path]`. If not provided:

- If exactly one `meta/research/*/brief.md` exists in or under cwd, use it. This auto-detection only checks the default location; briefs written to a custom Output dir (e.g. `design/systems/corpus/brief.md`) won't be found this way.
- Otherwise ask the user for the path or domain slug. Don't guess.

Read the brief and parse:

- **Output dir** — from the `**Output dir**:` line in the brief header.
- **Domain slug** — from the `**Domain**:` line.
- **Goal**, **Inspirations**, **Constraints**, **Source types** sections — these get passed verbatim to each topic agent.
- **All `## Round N` sections** — each contains topic blocks with title, `File:` line, and prompt.

If the brief is missing required fields (output dir, domain, source types, at least one Round with topics), stop and tell the user what's missing. Don't try to fix the brief yourself.

### Step 2 — Find the next pending round

For each `## Round N` section in order:

- List the topic files it expects (one per `File:` line).
- Check which already exist in `{output_dir}/`.
- The first round where **any** topic file is missing is the round to execute.

Special cases:

- **All rounds fully executed, synthesis up to date** → jump to Step 5 (propose new gap topics for a future round).
- **All rounds fully executed, synthesis missing or older than the latest topic file** → jump to Step 4 (rewrite synthesis), then Step 5.
- **Round partially executed** (some files exist, some don't) → only dispatch agents for the missing files.
- **User added a new round to the brief manually between invocations** → Step 2 will detect it as the next pending round and run it. Skip Step 5/6 in that case — the user already chose the topics themselves.

Tell the user which round you're running and how many agents you'll dispatch.

### Step 3 — Dispatch parallel agents

**Single message, multiple tool calls.** This is the only way to get true parallelism. Splitting agents across messages runs them serially and the round will take much longer than necessary.

For each missing topic in the round, dispatch **one Task tool call** with `subagent_type: "general-purpose"`. Don't reach for any custom or environment-specific agent types — the skill needs to work in any plugin or repo context, and `general-purpose` is the only universally-available agent. Build the prompt from this template (substitute everything in `{...}`):

```
You are executing one topic of a multi-round deep-research project. Your output will be synthesised with findings from {N} parallel topic agents into a single corpus.

DOMAIN: {domain-slug}
GOAL: {goal verbatim from brief}
ROUND: {round number}
TOPIC: {topic title}

RESEARCH PROMPT:
{topic prompt verbatim from brief}

INSPIRATIONS (reference material for the project):
{inspirations bulleted, or "None"}

CONSTRAINTS / SCOPE:
{constraints bulleted, or "None"}

SOURCE TYPES (verbatim from the brief — apply the priority stated in the brief's Source types section, and tag every citation with its type):

{Source types section verbatim from the brief}

OUTPUT PATH: {output_dir}/{topic-slug}.md

PRIMARY MODE: external web research.
- Use web search and read primary sources directly. The whole point of this round is to bring fresh external evidence into the corpus.
- Do NOT rely on training data alone — it is stale, generic, and uncitable.
- Every claim that isn't common knowledge needs an inline citation with a type tag.

OUTPUT SHAPE (write to OUTPUT PATH):
- 1500–4000 word Markdown file.
- 4–8 H2 sections, structured around the topic's natural axes — don't impose a generic template.
- Dense inline citations. Each citation prefixed with its type tag (`[T1]`, `[T2]`, `[T3]`) and using the format defined for that type in the SOURCE TYPES section above. The brief is the single source of truth for type definitions and citation formats — don't invent your own.
- End with a `## Sources` section listing every citation with full metadata (URL, publication date, author, type).

Write a single Markdown file to OUTPUT PATH. Do not write anywhere else. Do not write a synthesis or summary across topics — that's the orchestrator's job.

Return a short summary (under 200 words) of your findings in your final message. Note the type mix of your citations and how it matches the brief's stated source preference. The full output is in the file you wrote.
```

Send all Task tool calls in **one** assistant message. Wait for all to complete before proceeding.

#### Verify before synthesising

After all calls return, verify every expected topic file exists at `{output_dir}/{topic-slug}.md`. If any are missing — agent failed outright, or returned a summary but didn't actually write the file — surface the list of failed topics to the user and ask whether to retry, skip, or stop, before moving to Step 4. Don't synthesise an incomplete corpus silently.

### Step 4 — Fully rewrite the synthesis

Once all agents finish:

1. Read **every** topic file in `{output_dir}/` (across all rounds, not just this one).
2. Read the previous `synthesis.md` if any — only as background. You'll overwrite it.
3. Write a new `{output_dir}/synthesis.md` from scratch:

```markdown
# {Domain Title} — Synthesis

**Last updated**: {YYYY-MM-DD}
**Rounds completed**: {N}

## Contents

- [Brief](./brief.md) — research brief that drove this corpus
- [{topic-1-title}](./{topic-1-slug}.md) — {one-line summary}
- [{topic-2-title}](./{topic-2-slug}.md) — {one-line summary}
...

## Executive summary

{2–4 paragraphs. The decision-shaped core of what the research has found. A reader who reads only this section should walk away knowing what the research means for the goal.}

## Cross-cutting themes

{Patterns that appear across multiple topics. Each theme: 1–3 paragraphs. Cite topic files inline as clickable markdown links using the topic title as link text: `[Topic title](./topic-slug.md)`.}

## Tensions and trade-offs

{Where topics disagree, where evidence is mixed, where the user will need to make a judgement call.}

## Open questions

{What's unresolved. What would benefit from a future round.}
```

The synthesis is the **executive summary across all rounds**. The Contents section handles navigation so the body can stay synthetic — don't duplicate it as a list of topics in the body. Assume readers won't open the topic files. Target 1500–2500 words. If you're listing facts, stop and synthesise.

### Step 5 — Propose gap topics

Drop into a one-question-at-a-time conversation with the user:

1. Open with a recap, depending on what just happened:
   - **A round was just executed** — show 3–5 bullets covering what that round produced and the type mix of citations across the new topic files, comparing against the brief's stated source preference (e.g. "mostly T2, matches the brief's 'Type 2 primary' preference", or "mostly T2 but the brief said Type 1 primary — papers coverage is thin").
   - **No round was executed** (you arrived here via Step 2's "all rounds fully executed" path) — recap the corpus as a whole instead: top themes across all topic files, overall type mix, and the strongest coverage gaps so far.
2. Propose gap topics — angles that came up but weren't covered, weaknesses in coverage, themes that warrant a deeper second pass. As many as the research surfaces. Don't pad. Frame them as suggestions, not decisions.
3. Ask the user one question at a time — confirm, reject, refine, or add. For each accepted topic, sharpen it into a 1–2 sentence research prompt (same bar as Round 1 prompts).
4. Stop when the user says "done", "no more", or similar.

If the user accepts zero new topics, the research is finished. Tell them so and exit without modifying the brief.

### Step 6 — Append the next round

If the user accepted at least one new topic, append (don't rewrite) a new `## Round N+1` section to the brief, with the same per-topic structure as Round 1.

Tell the user:

- The path to the updated brief.
- The number of topics in Round N+1.
- The exact next command: `/deep-research {brief-path}` to execute it.

Then stop. **Do not execute Round N+1 in the same invocation** — the user re-invokes when ready.

## Notes

- **Single-message parallel dispatch is the most important rule** in this skill. Sequential Task tool calls run serially and waste hours.
- **Single agent type only.** Always `subagent_type: "general-purpose"`. Don't reach for repo-specific or plugin-specific agent types — they may not exist when the skill runs in a different environment.
- **External research is the primary mode** for every topic agent. Web search and primary sources first; training data alone is not acceptable output.
- **Source type tagging is mandatory.** Every inline citation gets `[T1]`, `[T2]`, or `[T3]`. The synthesis uses the distribution to spot coverage gaps relative to the brief's stated source preference.
- **Skip topics that already have files.** If a round was partially executed, only dispatch for missing files.
- **Synthesis is always a full rewrite.** Never patch or append.
- **The brief is append-only.** Never rewrite earlier rounds.
- **Don't commit.** Leave changes uncommitted. The user reviews before committing.
- **No emojis.** Skill outputs should not contain emojis.
