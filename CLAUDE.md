# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is not a software codebase — it's a set of Claude Code subagent definitions that make up a 6-stage revenue pipeline for a specific business: **By Celebration**, a Shopify store selling bachelorette/wedding/birthday party kits (117 products, 23 collections, 13 "Ready-to-Go" bundles). There is no build, lint, or test tooling; the only "code" is Markdown files defining agent roles and one shared business context file.

## The pipeline

Six subagents run in this sequence, each consuming the previous ones' output:

1. **market-signal-researcher** — validates real demand (search trends, competitor data); writes findings back into `business-brief.md`'s `[bracket]` placeholders
2. **offer-architect** — turns validated demand into a specific offer (bundle, price, guarantee)
3. **content-angle-strategist** — picks the messaging angle/story for that offer, not the platform plan
4. **reel-script-writer** — turns the angle into a finished, filmable short-form script (hook variants, shot list, caption)
5. **distribution-lead** — turns finished scripts into an actual posting calendar, flags production bottlenecks
6. **conversion-system-builder** — designs the landing page / email / checkout flow traffic lands on; runs in parallel with steps 4-5 since it only depends on step 2's offer, not on the scripts or calendar

Each agent's ground rule #1 is to read the outputs of the agents before it in the chain (and `business-brief.md`) rather than re-deciding upstream decisions — an angle strategist doesn't redesign the offer, a distribution lead doesn't rewrite the script. Preserve that separation when adding or editing agents.

See `RUNBOOK-UPDATE.md` for the reasoning behind this ordering and for By Celebration's current recommended execution order (conversion infrastructure — abandoned cart flow, published blog posts, confirmed domain — should be finished before reel-script-writer/distribution-lead start driving traffic at it).

## `business-brief.md` is the shared source of truth

Every agent reads this file first. It holds the offer, audience, pricing, brand voice, and current store state, including explicit `[Confirm: ...]` placeholders for data that hasn't been validated yet (competitor pricing, margins, revenue targets). Rules for this file:

- Only fill in a `[bracket]` placeholder with real, sourced data (e.g. from market-signal-researcher's research or actual Shopify product data) — never invent numbers to make a placeholder look complete.
- Update the "Current state" and "Immediate priority" sections whenever store state actually changes; stale state here produces generic or wrong output from every downstream agent.
- Discount codes and other concrete store facts live here — treat them as the single source, don't duplicate/fork them into individual agent files.

## Agent file conventions

- Agent definitions use YAML frontmatter (`name`, `description`, `tools`) followed by `# Role`, `# Ground Rules`, `# Process`, and `# Output Format` sections. Follow this structure for any new agent.
- Wired subagents live in `.claude/agents/` (market-signal-researcher, offer-architect, content-angle-strategist, distribution-lead, conversion-system-builder). **`agents/reel-script-writer.md` is the exception** — despite being part of the same pipeline and having identical frontmatter format, it lives at the top-level `agents/` directory rather than `.claude/agents/`, so it is not actually registered as a Claude Code subagent the way the other five are. Be aware of this when adding new agents or if reel-script-writer isn't behaving like a subagent — moving it into `.claude/agents/` is likely the fix, but confirm with the user before relocating it since it may be intentional staging.
- Each agent's `# Output Format` section is a contract for the next agent in the chain — if you change one agent's output structure, check whether a downstream agent's "Process" step depends on that structure.

## Editing `RUNBOOK-UPDATE.md`-style docs

`RUNBOOK-UPDATE.md` is itself an instruction to merge specific steps into a separate (not-yet-present) `revenue-agent-runbook.md`, not a standalone runbook. If asked to apply it, insert the described steps into the sequence rather than replacing an existing runbook wholesale.
