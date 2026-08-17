---
name: create-brief
description: Plan a multi-round, parallel-agent deep-research project. Use when starting a new deep-research project, scoping a multi-topic research effort, or planning parallel research on a domain.
---

# create-brief

Interactive skill that produces a research brief at `{output_dir}/brief.md`. The brief is the single source of truth that the deep-research skill reads to dispatch parallel research agents.

A research project has this layout:

```
{output_dir}/
├── brief.md              # this skill's only output
├── research/             # one file per topic, written by /deep-research
│   └── {topic-slug}.md
└── synthesis/            # syntheses across the corpus
    ├── general.md        # the whole-corpus synthesis, rewritten each round
    └── {axis-slug}.md    # per-axis cuts, written later by /synthesize
```

## Why this matters

The deep-research workflow runs in rounds: pick a domain, plan one parallel agent per topic, each producing an exhaustive markdown report (1500–4000 words, dense external citations) by doing fresh web research, then synthesise across all topics into a tight executive summary. The first round is typically 5–15 topics; later rounds add coverage based on what the synthesis surfaces. Rounds are append-only — earlier rounds are never rewritten.

The brief is the difference between research that produces tight, comparable topic files and a sprawl. **An agent reading a single topic line in the brief should be able to start working immediately, with no follow-up questions.** That bar is what this skill exists to enforce.

## Workflow

### Step 1 — Resolve output root folder

Before opening the wizard, determine the root folder where research output lives in this repo. Run the cascade below and stop at the first match. The cascade resolves the *root* only — the per-project subfolder is `{root}/{domain-slug}/`, confirmed in Step 2 Q7.

1. **Consumer-defined skill.** If a skill named `sagan-load-output-folder` is registered in the user's environment, read its SKILL.md body for a line of the form `output_folder: <path>` and use that path. The plugin ships a template at `sagan-load-output-folder/SKILL.template.md` that consumers copy into their own repo (or `.claude/skills/`, or any plugin) and edit.
2. **Harness instructions.** If the harness instructions in your context document an output folder for AI output or research output (any naming the local convention uses — e.g. `ai-output-folder`, `research-output`, `docs-research-folder`), use that path. Read the instructions and infer from context; don't insist on a specific marker or filename.
3. **Repo heuristic.** Scan the repo for evidence of an existing research folder: directories containing `brief.md` files, or common patterns like `meta/research/`, `docs/research/`, `research/`, `ai/research/`. If you find a clear single candidate, use it. If multiple plausible candidates exist, list them and ask the user to pick.
4. **Default.** Fall back to `meta/research/`.

Don't persist the resolved root anywhere. Each invocation runs the cascade fresh — the skill is run infrequently enough that token efficiency isn't critical.

### Step 2 — Brief interview

#### Phase 1 — ask the user in plain text

These two are open-ended foundations. Don't try `AskUserQuestion` here — it requires 2–4 preset options per question and will reject a freeform-only setup.

**Question 1 — Domain**

"What's the domain or project you're researching? A few words is fine; I'll convert it into a kebab-case slug for the directory name. Examples: 'desktop power controls for a Mac app', 'design system foundations', 'mobile auth patterns'."

**Question 2 — Goal**

"What's the goal of this research, in one sentence? What decision or design will it inform?"

Derive the domain slug from Q1's answer before moving to Phase 2 — Q7 needs it for the proposed default path.

#### Phase 2 — structured wizard

Load `AskUserQuestion`:

```
ToolSearch(query: "select:AskUserQuestion", max_results: 1)
```

For each question below, generate preset options grounded in the user's Q1+Q2 answers (and any prior Phase 2 answers). Use `multiSelect: true` for list-type questions (Q3, Q4, Q5). Always include an "Other (specify)" option so the user can override your suggestions.

**Question 3 — Inspirations** *(multiSelect)*

Ask: "What inspirations / references should the research draw on?"

Generate 4–8 contextually relevant options based on Domain + Goal — apps, products, companies, frameworks, design systems, well-known articles. Always include "None — start clean" and "Other (specify)".

Example: for a domain about command-palette patterns, options might include Apple HIG, Raycast, Arc browser, VS Code, Linear, Notion.

**Question 4 — Constraints / scope** *(multiSelect)*

Ask: "What's explicitly OUT of scope?"

Generate 4–6 plausible constraints based on context (platform, era, technology, audience). Always include "No constraints — broad coverage" and "Other (specify)".

**Question 5 — Initial topics** *(multiSelect)*

Ask: "Which topics should Round 1 cover?"

Generate 8–12 candidate topics for the domain, structured around the user's goal and informed by inspirations + constraints. The user picks which ones to keep; the follow-up loop in Step 3 sharpens each into a concrete research prompt. Always include "Other (add a custom topic)".

**Question 6 — Source types** *(single select)*

Ask: "What's the source preference for this research?"

Options:
- "Type 2 primary, Type 3 secondary; Type 1 not relevant (best for tech / industry research)"
- "Type 1 primary, Type 2 secondary; Type 3 not relevant (best for scientific research)"
- "Type 3 primary, Type 2 secondary; Type 1 not relevant (best for cultural pulse / trends)"
- "All three balanced"
- "Other (custom ranking)"

**Question 7 — Output folder** *(single select)*

Ask: "Where should research output live?"

Options:
- "Use default: `{resolved-root}/{domain-slug}/`"
- "Other (specify path)"

#### After Q7, derive

- **Topic slugs** — kebab-case, lowercase, ASCII, unique within the brief. Used as filenames. Derive from the topics the user picked in Q5.
- **Output dir** — Q7's answer (default or override).

### Step 3 — One-question-at-a-time follow-up

Stop using `AskUserQuestion`. From here, ask plain-text questions one at a time until you can write a brief that justifies launching parallel agents.

This phase exists to:

- Sharpen each rough topic into a concrete 1–2 sentence research prompt — specific enough that an agent can act on it without re-asking.
- Surface topics you'd expect for the domain that the user didn't list (suggest, don't impose).
- Resolve ambiguity in goal, inspirations, or constraints.
- Flag overlapping topics for merge or split.

Examples of useful follow-ups:

- "For 'command palette patterns' — should this focus on open-source implementations (VS Code, Linear, Raycast) or include proprietary case studies too?"
- "Your goal mentions a Mac app but inspirations include Raycast and Arc. Should we look at cross-platform patterns or stay Mac-only?"
- "I notice you have no topic on accessibility. Should I add one, or is that out of scope?"
- "Topic 4 ('typography') and Topic 7 ('text styles') seem to overlap. Combine, or split with clear boundaries?"

Stop when:

- Every Round 1 topic has a 1–2 sentence research prompt that's actionable on its own.
- Goal, inspirations, constraints, and source-type preference are unambiguous.
- The user has nothing more to add ("ready", "done", "go").

### Step 4 — Write the brief

Write to `{output_dir}/brief.md`. Create the directory if it doesn't exist.

The structure below is exact — `deep-research` parses these headers. Don't deviate.

```markdown
# {Domain Title} — Research Brief

**Domain**: `{domain-slug}`
**Output dir**: `{output_dir}`
**Created**: {YYYY-MM-DD}

## Goal

{One-sentence goal verbatim from the user.}

## Inspirations & references

- {Inspiration 1}
- {Inspiration 2}
...

(or "None.")

## Constraints & scope

- {Constraint 1}
- {Constraint 2}
...

(or "None.")

## Source types

For this project, source preference is: {user's confirmed preference verbatim from Q6}.

The three types are defined below regardless of priority. When citing a source, use the format and tag for its type. No quotas — cover whichever types the topic actually justifies, weighted by the priority above.

- **Type 1** — published research papers, peer-reviewed work, conference proceedings. Cite as: source + page + paragraph. Example: `[T1] Smith et al. 2024, p. 12, ¶3`.
- **Type 2** — official documentation, technical specs, design system docs, white papers, books, renowned domain blogs. Cite as: source + subsection (if any) + line number. Example: `[T2] ESP32 Reference Manual, GPIO chapter, line 42`.
- **Type 3** — Twitter/X, Hacker News, Reddit, small blogs, podcasts, conference videos. Cite as: source + URL. Example: `[T3] @username on X, https://...`.

Tag every inline citation with its type (`[T1]`, `[T2]`, `[T3]`) so the synthesis can spot coverage gaps relative to the stated preference.

## Conventions

- Each topic is researched by a parallel agent and saved to `{output_dir}/research/{topic-slug}.md`.
- Each topic agent's primary mode is external web research — fresh sources, not training data.
- The whole-corpus synthesis lives at `{output_dir}/synthesis/general.md` and is fully rewritten after each round. Other files in `{output_dir}/synthesis/` are per-axis cuts across the same corpus, written by `/synthesize` and never touched by the research loop.
- Rounds are appended below as `## Round 2`, `## Round 3`, etc. Earlier rounds and their topic files are never rewritten.
- **No research-process mechanics in the output.** Topic files and the synthesis must read as standalone research, not as a log of the research process. Do not write things like "in round 1 we found X; round 2 confirmed Y", "this round added coverage of Z", "the previous agent missed…", etc. The only structural leak allowed is the topic/synthesis split itself. State findings directly with their citations; the reader should not be able to tell from the prose how many rounds produced this document.

## Round 1

### {Topic 1 title}

File: `research/{topic-1-slug}.md`

{1–2 sentence research prompt. Specific. Actionable. Reference inspirations if relevant.}

### {Topic 2 title}

File: `research/{topic-2-slug}.md`

{1–2 sentence research prompt.}

...
```

**Topic prompts are the load-bearing part of the brief.** Calibration:

> Good — "Research cross-platform command palette implementations (VS Code, Linear, Superhuman, Raycast, Notion). Compare architecture (search-first vs. action-first), interaction model (keyboard vs. mouse emphasis), visual layout (width, positioning, backdrop, animations), search algorithm and latency perception, and customization/extensibility. Identify patterns that could inform our app's palette design."

> Weak — "Research command palettes."

The good prompt names exemplars, lists comparison axes, and ties back to the project goal. The weak prompt forces the agent to invent its own scope.

### Step 5 — Hand off

Tell the user:

1. The full path to the brief.
2. The number of Round 1 topics.
3. The exact next command: `/deep-research {brief-path}`.
4. That they can edit the brief directly before running research — in particular the Source types preference can be reweighted and topic prompts can be tuned.

## Notes

- **Output dir is resolved once per invocation** via Step 1's cascade and confirmed in Step 2 Q7. The resolved root is not persisted between runs.
- **`sagan-load-output-folder` is a documented hook**, not a skill sagan ships. Consumers who want custom output-folder logic define this skill in their own repo; sagan invokes it if present.
- **Domain slug must be kebab-case, lowercase, ASCII, and unique** within the resolved root (no clash with existing project subfolders).
- **Topic slugs must be kebab-case, lowercase, ASCII, and unique** within the brief.
- **`File:` paths are relative to the output dir** and always live under `research/`.
- **Don't write topic files yourself, and don't create `research/` or `synthesis/`.** Only write `brief.md`. The `deep-research` skill creates those directories and writes the files.
- **Don't commit.** Leave the brief uncommitted. The user reviews and runs `/deep-research`.
- **No emojis.** Skill outputs should not contain emojis.
