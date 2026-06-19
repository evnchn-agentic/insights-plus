# insights-plus

**`/insights` describes you well, then advises a stranger.** insights-plus makes the advice yours.

Claude Code's `/insights` fans out ~8 tiny prompts (96–737 tokens each) on a fast model with
no thinking budget. It mirrors your transcripts fine. But its *suggestions* have no model of
where you sit relative to the tool's own frontier — so they regress an advanced user to the median.
Author your own skills, ship hooks, wire a dozen MCP servers, curate a 150-entry memory? It
will still tell you to "try Custom Skills" and `claude mcp add github`.

The miss isn't the model being dumb. It's a missing step: **nobody diffed the advice against
what you already do.**

## What insights-plus adds

One load-bearing step the stock pipeline skips — for every candidate suggestion, **diff it
against your actual config and memory, and suppress anything already covered.** Then:

- **Mine for net-new only** — search your session history for the pain that *isn't* yet
  codified (tensions, repeated workarounds, contradictions in your own system).
- **Fan out, don't single-pass** — dispatch a fresh-context miner; a cold run finds what the
  primary agent's context-bias buries.
- **Gate through your own persistence rules** — a single-incident overfit is noise, not a rule.
- **Propose, never auto-write** — keep the descriptive mirror; replace the coaching with
  candidates for your approval. *"Already covered, no action"* is a valid, high-value result.

## What that looks like

- **Suppressed** — stock said *"add a CLAUDE.md rule: always quote shell variables."*
  insights-plus: already covered, and you're on zsh, where you'd derived the sharper, opposite
  rule. → *No action.*
- **Net-new** — mined from history: *"the agentic loop is the ceiling — you keep capping
  outputs at a self-imposed limit (monochrome, a DSL) instead of the medium's real ceiling."*
  Not in any existing rule. → *Candidate for approval.*

## Does it actually work? (the origin story)

We tested it the honest way. A fresh agent was given the skill as its *only method* — no human
critique to copy — plus the real 164-session corpus and the stock report as its data. Cold, it:

- **suppressed all 10** stock prescriptions, each with the specific file that already covered
  it (9 redundant, 1 single-incident overfit);
- **reclassified** a reported "tool bug" as a deliberate probe *succeeding* — the exact misread
  the skill warns about;
- and **surfaced one genuinely new insight the skill's own author had missed.**

That last line is the whole philosophy. No single pass — not even this one — is the ceiling.
**The loop is.** So insights-plus tells you to fan out cold-context miners, because one agent
ships its own blind spots as "complete." The skill learned its most important rule *by being
run*, then was rewritten to obey it.

## Install

```bash
git clone https://github.com/evnchn-agentic/insights-plus ~/.claude/skills/insights-plus
```

Then invoke it when generating, re-reading, or critiquing a `/insights`-style report. The full
method lives in [SKILL.md](./SKILL.md). The fan-out step uses a fresh-context subagent — it'll
use `superpowers:dispatching-parallel-agents` if you have it, otherwise it spawns a plain one.

## Status: working beta (拋磚引玉)

This is a **brick thrown to attract jade** —
[拋磚引玉](https://github.com/evnchn-agentic/chengyu-skills) is the discipline of shipping a
deliberately-early *working* pass to draw out refinement. It runs, and it outperformed the stock
report in that one case study. It is also young:

**Known gaps**
- Validated on **one** user's corpus (a homelab / OSS-maintainer profile) and **one** cold run.
  The calibration heuristics are tuned to that profile.
- The frontier-diff assumes you *have* a curated config/memory to diff against. Sparse setups
  get less lift.
- The memory-gate step assumes you keep persistence rules to gate candidates through.

Issues and PRs welcome — especially calibration data from other user profiles. That's the jade.

---

Part of the [evnchn](https://github.com/evnchn) project family.
