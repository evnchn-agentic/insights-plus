---
name: insights-plus
description: Use when generating, re-reading, or critiquing a Claude Code /insights-style usage report for an advanced or heavily-customized user (authored skills, custom hooks, multiple MCP servers, large curated memory) — to calibrate findings against the user's actual setup instead of emitting median advice.
---

# insights-plus

> Status: working beta (拋磚引玉) — validated once, cold; refine against more runs.

## Overview

Stock `/insights` fans out ~8 tiny per-facet prompts (96–737 tokens) on a fast model with no thinking budget. It **describes** a transcript adequately, but in observed runs its prescriptive half has no step that checks the user's position relative to the tool's frontier — so its advice can regress an advanced user to the median and recommend features/rules they've already mastered.

**Core principle: diff every suggestion against what the user already does; classify the coverage; propose, never auto-prescribe.** Keep the descriptive mirror; replace the coaching.

**No single pass is complete — the ceiling is the iterative loop, not one run of this skill.** A single agent carries context-bias that hides findings; fan out cold-context runs. (This skill is its own proof: a cold-eval subagent found a net-new insight its author missed.)

## When to use

- Generating insights for an advanced/heavily-customized user (authored skills, hooks, multiple MCP servers, large curated memory, multi-node setup).
- Re-reading a stock `/insights` report critically — its prescriptive half is suspect for these users.
- Any "what should I change about my setup" ask where median advice would insult the user's actual sophistication.

Not for a genuine novice — stock `/insights` is fine there.

## The failure this fixes (observed)

On an advanced user, the stock report has produced: `features_to_try` they already built; CLAUDE.md "additions" that are redundant, single-incident overfits, or wrong for their shell/OS; and friction items that misread deliberate tool-probes as defects. All trace to one missing step — **no coverage check against the user's real setup.**

## Method

1. **Load ground truth FIRST.** Inventory the user's actual setup, handling missing sources gracefully: `CLAUDE.md` (+ project-local), memory store, skill catalog, **hooks & settings, MCP servers, custom commands/plugins**, and observable behavior in recent sessions. This is the diff baseline; skip it and you become stock `/insights`.
2. **Ingest the raw signal** (stock facets/report or the session corpus) as *input data*, not conclusions.
3. **Frontier-relative diff — classify coverage (the load-bearing step).** For every candidate, grep ground truth and bucket it:
   - *covered and working* → **suppress** (cite the covering file/rule so it reads as checked, not missed);
   - *covered but recurring / not enforced* → surface as an **enforcement or design gap** (not a new rule);
   - *partially covered* → propose a **refinement**;
   - *not covered* → carry to step 4.
4. **Mine for net-new, fan out, synthesize.** Search the session archive (Claude Code: `~/.claude/projects/<encoded-path>/*.jsonl`; otherwise your platform's session store — sample recent sessions if the corpus is large) for *uncodified recurring pain* — tensions, repeated workarounds, contradictions in the user's own system. Dispatch **≥1 fresh-context subagent** to mine independently (use superpowers:dispatching-parallel-agents / chengyu-listen-all-sides-see-clearly if installed; otherwise spawn a plain fresh-context agent). **Merge** the findings, resolve disagreements against evidence, and **stop** when a pass yields nothing materially new.
5. **Classify each proposal by action type, then gate by the right bar.** A persistence/memory proposal → the user's persistence rules (durable, hard-won, not single-incident, no rotating state). A config / hook / MCP / bug-report / workflow proposal → its own bar (reproducible, actionable). Don't gate everything as if it were a memory entry.
6. **Read friction with intent.** Classify each anomaly as *expected probe result* / *defect exposed by probe* / *ambiguous* — testing a tool doesn't make odd output automatically fine; require evidence of intent and expected outcome before dismissing it.
7. **Output: propose-don't-write, don't pad.** Per finding emit: *finding | classification | evidence (file or session count) | confidence | proposed action*. Keep the descriptive mirror; deliver net-new items as **candidates for approval**, never auto-applied. **Zero net-new is a valid, high-value result** — forced counts pad the tail with confabulation. Render in whatever medium suits the user; reveal when ready.

## Quick reference

| Stock /insights | insights-plus |
|---|---|
| `user does X → tip for X` | Diff X against ground truth; classify coverage |
| Binary suggest / don't | covered-working / covered-unenforced / partial / net-new |
| Prescribe / add rules | Propose candidates; user approves before save |
| Single-incident → standing rule | Gate by action type + the user's persistence bar |
| Friction = anything unexpected | probe-result / defect-exposed / ambiguous |
| One pass | Fan out cold miners; synthesize; stop when dry |

## Common mistakes

- **Skipping step 1** → you become stock `/insights`. The ground-truth load is the whole point.
- **Binary suppression.** "Covered" isn't always "handled" — a covered-but-unenforced rule is a real finding (a gap), not a non-finding.
- **Padding findings to a count.** Zero net-new is valid; confidence-label and let the count be real.
- **Single-passing it.** One agent ships its own blind spots as "complete." Fan out (step 4).
- **Auto-writing to memory/CLAUDE.md.** Always propose; the user's persistence policy owns the write.

## Testing / eval protocol

Validate against real history as ground truth:

1. Pick N past sessions (or an existing stock report) as the dataset.
2. Run stock-style facets AND insights-plus over the same data.
3. **Pass** = stock noise correctly classified (every suppression cites a real cover; no probe misread as a defect; covered-but-unenforced items flagged as gaps) AND any net-new findings are evidence-backed. **Zero net-new is a pass** when classification is correct.
4. **Fail** = reproduces stock output (the coverage diff isn't biting) OR emits unsupported/padded findings.
5. Also spot-check the inverse errors: false suppression (a real, uncovered need wrongly dismissed) and false net-new (a "finding" already covered).
