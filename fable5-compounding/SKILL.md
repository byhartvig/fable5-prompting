---
name: fable5-compounding
description: "Run Claude Fable 5 as a compounding agent system - state files, independent verifier subagents, skills that accumulate lessons, model routing, refusal fallbacks. Use when setting up long-running or autonomous agent workflows, when sessions should build on previous runs, or when the user asks about agent memory, verification loops, or self-improving systems."
version: 1.0.0
---

# Fable 5 Compounding System

The model is stateless; the system around it doesn't have to be. A compounding setup means every run leaves the next run smarter: state files accumulate verified facts, skills accumulate lessons, and an independent verifier keeps quality honest. This skill defines the operating procedure.

When this skill is active, follow the session discipline below on every run, and apply the other sections when the task calls for them.

## Session Discipline (every run)

1. **Read at session start.** Before doing anything else, read `STATE.md` (if present) and any skills relevant to the task. Consult recorded rules instead of re-deriving facts from scratch.
2. **Write before walking away.** End every session by updating `STATE.md`: what was tried, what passed, what failed, which new rules survived. A session that ends without a write forces the next session to restart from zero.

## The State File

Keep a `STATE.md` in the project root (in Claude Managed Agents, use the mounted memory store the same way). Use the structure in [STATE-template.md](STATE-template.md). Five sections, each with a job:

| Section | Job |
|---------|-----|
| Verified facts | Things checked against reality (a query, a test, a doc) - stop guessing about these |
| General rules | Distilled lessons that apply beyond the specific case - consult before re-deriving |
| Open failures | Documented failures awaiting investigation, with reproduction steps |
| Lessons learned | Post-mortem distillations worth keeping |
| Last session | Resume pointer: what happened, what's next |

Promotion rules: a guess stays in *Open failures* until verified; a verified fact only becomes a *General rule* when it generalizes beyond the case it came from. Don't record what the repo or chat history already records. Update existing notes rather than duplicating. Delete notes that turn out to be wrong.

## Independent Verification

Don't grade your own work - a maker evaluating its own output sees its own reasoning trail and prefers conclusions consistent with what it already wrote. Use a separate verifier with fresh context that sees only the artifact and the acceptance criteria, never the maker's reasoning.

- **In Claude Code:** spawn a verifier subagent after substantial work, with the spec/rubric and the output - nothing else. For long builds, establish a checking method up front and run it at an interval, verifying against the specification with subagents.
- **In Claude Managed Agents:** use Outcomes - send a `user.define_outcome` event with a gradeable rubric; an independent grader runs an iterate-grade-revise loop (default 3 iterations, max 20).
- **For visual output (UI, dashboards, charts):** verify with vision. Render a screenshot, have the verifier compare it against the goal and any design constraints. Text-only verification misses the failure modes that matter for visual work.

Write rubrics as explicit, independently gradeable criteria ("CSV has a numeric price column"), not vibes ("data looks good").

## Skills That Compound

`STATE.md` is project memory; skills are procedural memory that travels across projects. The contract: **after any non-trivial failure or confirmed correction, write the lesson into the relevant skill itself** - typically as entries under "Known failure modes" or "Anti-patterns" sections. A skill that never gets written to is wasted scaffolding; one that accumulates two weeks of real lessons outperforms anything derived from scratch.

When updating a skill, prefer removing over adding: Fable 5 performs worse with over-prescriptive instructions, so distill lessons into short rules and delete steps the model no longer needs spelled out.

## Model Routing

Not every step needs the top tier. Route by task complexity:

| Role | Model | Why |
|------|-------|-----|
| Orchestrator / long-horizon work | `claude-fable-5` | Planning, delegation, self-verification over long runs |
| Hard-but-bounded subtasks | `claude-opus-4-8` | Architecture decisions, complex debugging; also the refusal fallback |
| High-volume worker tasks | `claude-sonnet-4-6` | Lint passes, simple refactors, doc updates |
| Graders / cheap classifiers | `claude-haiku-4-5` | Fresh-context verifier role at low cost |

Run subagents at low effort unless the subtask is genuinely hard.

## The Safety Boundary

Fable 5 runs safety classifiers targeting offensive cybersecurity, biology/life sciences, and extraction of its internal reasoning. Benign adjacent work (defensive security tooling, beneficial bio, "explain your reasoning" instructions) can trigger false positives. Design for it explicitly:

- A declined request returns HTTP 200 with `stop_reason: "refusal"` - in autonomous loops, treat this as a distinct branch, never as a generic error. A loop that fails silently on a refusal looks identical to a real bug until you debug it.
- Configure fallback to `claude-opus-4-8` (server-side `fallbacks` parameter on the API, or explicit routing in your harness) for tasks likely to hit the boundary.
- Don't instruct the model to echo or transcribe its internal reasoning in responses - that can trigger `reasoning_extraction` refusals. Read the summarized thinking blocks instead.
- Note which of a skill's tasks may hit the classifier and document the expected fallback behavior in the skill.

## Common Mistakes

- Using Fable 5 like a chat model: 5-minute prompt-and-close sessions get no compound effect
- Self-critique instead of an independent verifier
- No state file, or a state file that's written but never read at session start
- Skills that never accumulate lessons after real failures
- Fable 5 on tasks Sonnet would handle - route by complexity
- Loops without an objective stop condition checked by an independent grader - they stop at "handled enough" instead of done
- Ignoring the safety boundary, producing silent regressions in autonomous runs
