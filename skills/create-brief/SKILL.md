---
name: create-brief
description: Plan a multi-round, parallel-agent deep-research project. Use when starting a new deep-research project, scoping a multi-topic research effort, or planning parallel research on a domain.
---

# create-brief

Interactive skill that produces a research brief at `{output_dir}/brief.md`. The brief is the single source of truth that the deep-research skill reads to dispatch parallel research agents.

## Why this matters

The deep-research workflow runs in rounds: pick a domain, plan one parallel agent per topic, each producing an exhaustive markdown report (1500–4000 words, dense external citations) by doing fresh web research, then synthesise across all topics into a tight executive summary. The first round is typically 5–15 topics; later rounds add coverage based on what the synthesis surfaces. Rounds are append-only — earlier rounds are never rewritten.

The brief is the difference between research that produces tight, comparable topic files and a sprawl. **An agent reading a single topic line in the brief should be able to start working immediately, with no follow-up questions.** That bar is what the wizard exists to enforce.

## Workflow

### Step 1 — Load AskUserQuestion

`AskUserQuestion` is a deferred tool. Before the first question, load it:

```
ToolSearch(query: "select:AskUserQuestion", max_results: 1)
```

### Step 2 — Resolve output root folder

Before opening the wizard, determine the root folder where research output lives in this repo. Run the cascade below and stop at the first match. The cascade resolves the *root* only — the per-project subfolder is `{root}/{domain-slug}/`, confirmed in the wizard.

1. **Consumer-defined skill.** If a skill named `sagan-load-output-folder` is registered in the user's environment, read its SKILL.md body for a line of the form `output_folder: <path>` and use that path. The plugin ships a template at `sagan-load-output-folder/SKILL.template.md` that consumers copy into their own repo (or `.claude/skills/`, or any plugin) and edit.
2. **Harness instructions.** If the harness instructions in your context document an output folder for AI output or research output (any naming the local convention uses — e.g. `ai-output-folder`, `research-output`, `docs-research-folder`), use that path. Read the instructions and infer from context; don't insist on a specific marker or filename.
3. **Repo heuristic.** Scan the repo for evidence of an existing research folder: directories containing `brief.md` files, or common patterns like `meta/research/`, `docs/research/`, `research/`, `ai/research/`. If you find a clear single candidate, use it. If multiple plausible candidates exist, list them and ask the user to pick.
4. **Default.** Fall back to `meta/research/`.

Don't persist the resolved root anywhere. Each invocation runs the cascade fresh — the skill is run infrequently enough that token efficiency isn't critical.

### Step 3 — Structured wizard

Ask the following seven questions one at a time via `AskUserQuestion`. Use freeform-text questions (no multiSelect — these are open-ended). Wait for each answer before asking the next. Between Q1 and Q2, derive the domain slug so Q2 can show the full proposed path.

1. **Domain** — "What's the domain or project you're researching? A few words is fine; I'll convert it into a kebab-case slug for the directory name. Examples: 'desktop power controls for a Mac app', 'design system foundations', 'mobile auth patterns'."
2. **Output folder** — "Output will go to `{resolved-root}/{domain-slug}/`. Press enter to accept, or give a different path."
3. **Goal** — "What's the goal of this research, in one sentence? What decision or design will it inform?"
4. **Inspirations** — "What inspirations or reference material should the research draw on? Apps, products, companies, design systems, frameworks, articles — whatever applies. One per line, or 'none'."
5. **Constraints / scope** — "What's explicitly OUT of scope? Platforms, eras, technologies, audiences — anything that should bound the research."
6. **Initial topics** — "List your initial research topics, one per line. Aim for 5–10. Rough phrasing is fine; we'll sharpen each one in the next step."
7. **Source types** — "What's the source preference for this research? Pick which types apply and rank them. Type 1 — papers / peer-reviewed work (scientific research). Type 2 — official docs, specs, white papers, renowned domain blogs (tech / industry research). Type 3 — social media, small blogs, podcasts, conference videos (cultural pulse, trends). E.g. 'Type 2 primary, Type 3 secondary, Type 1 not relevant'."

After answering, derive:

- **Domain slug** — already derived between Q1 and Q2. Confirm in plain text if the user's domain phrase was ambiguous.
- **Topic slugs** — kebab-case, lowercase, ASCII, unique within the brief. Used as filenames.
- **Output dir** — Q2's answer (the proposed default or the user's override).

### Step 4 — One-question-at-a-time follow-up

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

### Step 5 — Write the brief

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

For this project, source preference is: {user's confirmed preference verbatim from the wizard}.

The three types are defined below regardless of priority. When citing a source, use the format and tag for its type. No quotas — cover whichever types the topic actually justifies, weighted by the priority above.

- **Type 1** — published research papers, peer-reviewed work, conference proceedings. Cite as: source + page + paragraph. Example: `[T1] Smith et al. 2024, p. 12, ¶3`.
- **Type 2** — official documentation, technical specs, design system docs, white papers, books, renowned domain blogs. Cite as: source + subsection (if any) + line number. Example: `[T2] ESP32 Reference Manual, GPIO chapter, line 42`.
- **Type 3** — Twitter/X, Hacker News, Reddit, small blogs, podcasts, conference videos. Cite as: source + URL. Example: `[T3] @username on X, https://...`.

Tag every inline citation with its type (`[T1]`, `[T2]`, `[T3]`) so the synthesis can spot coverage gaps relative to the stated preference.

## Conventions

- Each topic is researched by a parallel agent and saved to `{output_dir}/{topic-slug}.md`.
- Each topic agent's primary mode is external web research — fresh sources, not training data.
- Synthesis lives at `{output_dir}/synthesis.md` and is fully rewritten after each round.
- Rounds are appended below as `## Round 2`, `## Round 3`, etc. Earlier rounds and their topic files are never rewritten.

## Round 1

### {Topic 1 title}

File: `{topic-1-slug}.md`

{1–2 sentence research prompt. Specific. Actionable. Reference inspirations if relevant.}

### {Topic 2 title}

File: `{topic-2-slug}.md`

{1–2 sentence research prompt.}

...
```

**Topic prompts are the load-bearing part of the brief.** Calibration:

> Good — "Research cross-platform command palette implementations (VS Code, Linear, Superhuman, Raycast, Notion). Compare architecture (search-first vs. action-first), interaction model (keyboard vs. mouse emphasis), visual layout (width, positioning, backdrop, animations), search algorithm and latency perception, and customization/extensibility. Identify patterns that could inform our app's palette design."

> Weak — "Research command palettes."

The good prompt names exemplars, lists comparison axes, and ties back to the project goal. The weak prompt forces the agent to invent its own scope.

### Step 6 — Hand off

Tell the user:

1. The full path to the brief.
2. The number of Round 1 topics.
3. The exact next command: `/deep-research {brief-path}`.
4. That they can edit the brief directly before running research — in particular the Source types preference can be reweighted and topic prompts can be tuned.

## Notes

- **Output dir is resolved once per invocation** via Step 2's cascade and confirmed in the wizard. The resolved root is not persisted between runs.
- **`sagan-load-output-folder` is a documented hook**, not a skill sagan ships. Consumers who want custom output-folder logic define this skill in their own repo; sagan invokes it if present.
- **Domain slug must be kebab-case, lowercase, ASCII, and unique** within the resolved root (no clash with existing project subfolders).
- **Topic slugs must be kebab-case, lowercase, ASCII, and unique** within the brief.
- **Don't write topic files yourself.** Only write `brief.md`. The `deep-research` skill writes the topic files.
- **Don't commit.** Leave the brief uncommitted. The user reviews and runs `/deep-research`.
- **No emojis.** Skill outputs should not contain emojis.
