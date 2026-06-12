---
name: fable5-prompting
description: "Optimal prompting for Claude Fable 5 - structure, checkpoints, memory, loops, anti-patterns. Use when the user asks to write, improve, or review a prompt for Fable 5, or shares a draft prompt."
version: 1.2.0
author: Anthropic playbook + community translation
---

# Fable 5 Prompt Engineering

## How to Use This Skill

When the user shares a draft prompt (or invokes this skill with a prompt as argument), do both of the following:

1. **Feedback first:** briefly assess the draft against this guide - which of the 4 components are missing (context/why, request, output format, constraints), whether checkpoints are defined, and any anti-patterns present (over-engineering, micro-managed steps, missing intent). Keep it to a short bullet list.
2. **Then rewrite:** provide an improved version of the prompt using the 4-component structure below, preserving the user's intent and wording where it already works. Note in one or two sentences what you changed and why.

If the user hasn't shared a prompt yet, ask what they're trying to get Fable 5 to do, then draft one for them. If the rest of their request is in another language, give feedback in that language but keep the rewritten prompt in the language the user's prompt was written in.

Fable 5 (API model ID: `claude-fable-5`) is fundamentally different from previous Claude models. It's designed for autonomous, multi-hour/multi-day work with minimal iteration. The way you prompt it must change.

> **Fable 5 vs. Mythos 5:** these are two separate model IDs (`claude-fable-5` and `claude-mythos-5`) sharing the same underlying model. Fable 5 is the generally available version with additional safety measures; Mythos 5 is available only to approved organizations through Project Glasswing. Everything in this guide applies to both.

## Core Principles

1. **Tell it WHY, not just WHAT** - context enables it to connect the task to relevant information rather than inferring intent
2. **Keep instructions short** - over-engineering constrains a model that would figure out the right approach on its own
3. **Define checkpoints explicitly** - tell it when to stop and ask, otherwise it runs autonomously
4. **Build it a memory system** - a markdown file where it records lessons from previous runs

## Prompt Structure (The 4 Components)

Every high-quality Fable 5 prompt:

```
"I'm working on [the larger task] for [who it's for].
They need [what the output enables].

Request: [your specific ask in one clear sentence]

Output format: [exactly how you want the result structured and delivered]

Constraints: [what must NOT happen on the way to the result]"
```

## Effort Level Guide

Effort is the primary intelligence/latency/cost control on Fable 5. In the API it's set via `output_config: {effort: "..."}`; in Claude Code via the effort setting.

| Level | When to use |
|-------|-------------|
| **low** | Quick questions, simple rewrites, latency-sensitive tasks, subagents |
| **medium** | Routine or cost-sensitive work, simple code changes |
| **high** | DEFAULT - most tasks, builds, analysis |
| **xhigh** | The most capability-sensitive workloads - complex coding/agentic work |
| **max** | When correctness matters more than cost or latency |

Note: even `low` on Fable 5 often exceeds the `xhigh`/`max` performance of previous models - don't reflexively run everything at the top.

## Checkpoint Pattern

When you want Fable 5 to run autonomously but stop at the right moments:

```
"Pause for me only when the work genuinely requires my input:
a destructive or irreversible action, a real scope change,
or something only I can provide. Otherwise, keep going
and report back when done."
```

## Memory System Pattern

```
"Store one lesson per file with a one-line summary at the top.
Record corrections and confirmed approaches alike, including
why they mattered. Don't save what the repo or chat history
already records. Update an existing note rather than creating
a duplicate. Delete notes that turn out to be wrong."
```

## Loop Pattern

```
/loop <time interval> + goal

Examples:
  /loop 15 minutes, check if my build is passing, notify me if it fails
  /loop 1 hour, review PRs in repo X, summarize new ones
  /loop 30 minutes, check server health, alert if down

To stop: ask Claude to stop the loop.
```

## What NOT To Do (Anti-patterns)

| Don't | Why |
|-------|-----|
| Over-engineer prompts with excessive instructions | Constrains the model from finding its own approach |
| Reuse old prompts/skills built for Opus 4.8 or earlier unchanged | They're often too prescriptive and can degrade output - review, de-prescribe, and A/B test |
| Skip defining checkpoints | Fable will define its own, possibly in wrong places |
| Give instructions without context/why | Fable performs better when it understands the intent behind a request |
| Ask it to reproduce its internal reasoning in the response | Can trigger a `reasoning_extraction` refusal - read the summarized thinking blocks instead |
| Expect short execution | Fable runs longer - single requests can run many minutes; autonomous runs can extend for hours |
| Micro-manage every step | Fable is a consultant/partner, not a task-executor - let it lead |

## Caveats to Expect

- **Runs longer than expected** - hard tasks at high effort can run many minutes per request
- **Can go beyond what you asked** - proactive by design; use checkpoints to bound it
- **May ask clarifying questions** before autonomous runs - answer them
- **Can occasionally stop early** - say "go ahead and do it end-to-end" to resume
- **Higher token costs** - priced above Opus tier; budget accordingly
- **May decline requests** - safety classifiers target offensive cybersecurity, biology/life sciences, and reasoning extraction; benign adjacent work (defensive security, beneficial bio) can occasionally trigger false positives. API users can configure fallback to Opus 4.8 for declined requests.

## Complete Example

```
"I'm working on a CI/CD pipeline overhaul for our DevOps team.
They need reliable, fast deployments that don't break production.

Request: Audit our current GitHub Actions workflows, identify
the top 5 reliability gaps, and propose concrete fixes with
code examples.

Output format: A markdown report with:
- Executive summary (3 sentences)
- Gap analysis table (gap, severity, impact, fix)
- Code snippets for each fix
- Migration plan in phases

Constraints:
- Do NOT modify any live workflow files
- Do NOT deploy anything
- Pause for me before any recommendation that would
  require changing our infrastructure

Memory: Save lessons learned to .fable/memory/pipeline-audit.md"
```
