# Fable 5 Prompting - The Missing Manual

A distilled, plain-English guide to prompting Claude Fable 5 - based on [Anthropic's official playbook](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5).

> **Naming note:** Claude Fable 5 (`claude-fable-5`) and Claude Mythos 5 (`claude-mythos-5`) are two separate model IDs sharing the same underlying model. Fable 5 is generally available (with additional safety measures); Mythos 5 is available only to approved organizations through Project Glasswing. This guide applies to both.

## Why This Exists

Fable 5 is fundamentally different from every Claude model before it. It's designed for **autonomous, multi-hour work** with minimal iteration. The prompting rules you learned for Opus/Sonnet actively work against you here.

This guide translates Anthropic's dense developer documentation into a practical skill you can load into any AI tool.

## What's Different About Fable 5

| Previous Claude | Claude Fable 5 |
|----------------|----------------|
| Short bursts of work | Sustained multi-hour/multi-day runs |
| Needs detailed instructions | Figures out approach on its own |
| You iterate many times | Often gets well-specified problems right first time |
| Hesitant to delegate | Reliably dispatches and sustains parallel subagents |
| Instructions-first | Context-first (performs better with the "why") |

## Quick Install

### Claude Code

Skills are folders containing a `SKILL.md`, placed under `.claude/skills/` (project) or `~/.claude/skills/` (personal):

```bash
mkdir -p .claude/skills/fable5-prompting
curl -o .claude/skills/fable5-prompting/SKILL.md \
  https://raw.githubusercontent.com/byhartvig/fable5-prompting/main/SKILL.md
```

Or clone the repo straight into the skills directory:

```bash
git clone https://github.com/byhartvig/fable5-prompting.git .claude/skills/fable5-prompting
```

### Hermes Agent

```bash
hermes skills install https://raw.githubusercontent.com/byhartvig/fable5-prompting/main/SKILL.md
```

### Manual

Copy `SKILL.md` into your project or agent's skills directory.

## How to Use

Once installed, the skill works as an active prompt coach - it does two things with any draft prompt you give it:

1. **Feedback:** a short bullet list assessing your draft - which of the 4 components are missing (context/why, request, output format, constraints), whether checkpoints are defined, and any anti-patterns present.
2. **Rewrite:** an improved version of your prompt in the 4-component structure, preserving your intent and wording where it already works, with a brief note on what changed and why.

Ways to trigger it in Claude Code:

```
# Invoke directly with your draft prompt as argument
/fable5-prompting Audit my GitHub Actions workflows and fix the flaky ones

# Or just ask naturally - the skill loads automatically
"Can you improve this prompt for Fable 5: ..."
"Review my prompt before I run it"
```

If you invoke it without a prompt, it will ask what you're trying to get Fable 5 to do and draft one for you.

## What's Inside

### The 4-Component Prompt Structure

Every prompt should have: Context → Request → Output Format → Constraints

```
"I'm working on [larger task] for [who it's for].
They need [what the output enables].

Request: [one clear sentence]

Output format: [exactly how you want the result]

Constraints: [what must NOT happen]"
```

### Effort Level Guide

Set via `output_config: {effort: "..."}` in the API. Levels: `low | medium | high | xhigh | max`.

| Level | When |
|-------|------|
| `low` / `medium` | Routine, latency- or cost-sensitive work (still strong on Fable 5) |
| `high` | **Default** - most tasks |
| `xhigh` | The most capability-sensitive workloads - complex coding/agentic work |
| `max` | Correctness matters more than cost or latency |

### Autonomous Patterns

- **Checkpoints:** "Pause only when the work genuinely requires my input"
- **Loops:** `/loop 15 minutes, check build, notify on failure`
- **Memory:** "Store one lesson per file, one-line summary at top"

### Anti-Patterns to Avoid

- ❌ Over-engineering prompts with excessive instructions
- ❌ Reusing old prompts built for Opus 4.8 unchanged (review and de-prescribe instead)
- ❌ Skipping checkpoint definitions
- ❌ Giving instructions without context/why
- ❌ Asking the model to reproduce its internal reasoning in the response (can trigger a refusal)
- ❌ Micro-managing every step

## Example: Before & After

**❌ Old way (works on Opus, degrades Fable 5):**
```
1. Read the file at src/pipeline.py
2. Find all functions related to deployment
3. Check for error handling in each function
4. Document missing error handlers
5. Format output as a table
6. Sort by severity
7. Add code examples for each fix
```

**✅ Fable 5 way:**
```
I'm working on a deployment pipeline audit for our DevOps team.
They need reliable deployments that don't silently fail.

Request: Audit src/pipeline.py for error handling gaps.

Output format: Markdown table (gap, severity, impact, fix)
with code snippets.

Constraints: Do not modify any files.
Pause before any recommendation that changes infrastructure.
```

## Source

This guide is based on [Anthropic's official Fable 5 prompting playbook](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5).

## License

MIT - use it, share it, improve it. This is community knowledge.
