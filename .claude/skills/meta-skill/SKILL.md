---
name: meta-skill
description: Create new reusable skills from experience gained during a task. Invoke this after completing a task where you learned a repeatable workflow, discovered a useful tool pattern, or built domain expertise that future agents would benefit from.
disable-model-invocation: true
argument-hint: "[skill-name] [brief purpose]"
allowed-tools: Bash(ls *), Read, Write, Glob
---

# Meta-Skill: Write Skills from Experience

When you've completed a task and identified a repeatable pattern worth capturing, use this skill to create a new skill that future agents can use.

## When to Create a Skill

Create a skill when you notice:
- You followed a **multi-step workflow** that others will repeat (e.g., deploying, migrating data, onboarding a new service)
- You learned a **tool's CLI or API** and can document the essential commands (like the beads skill)
- You discovered **project-specific conventions** that aren't obvious from the code alone
- You built up **domain knowledge** during a task that would save future agents significant exploration time
- You developed a **debugging or troubleshooting pattern** for a recurring class of issues

Do NOT create a skill for:
- One-off tasks that won't recur
- General programming knowledge (agents already know how to code)
- Information that belongs in CLAUDE.md (project-wide context) rather than an invocable workflow

## How to Write a Skill

### Step 1: Choose a name and scope

- Name: lowercase, hyphenated, max 64 chars (e.g., `db-migration`, `deploy-staging`, `api-testing`)
- Scope: one clear purpose — if it does two unrelated things, split into two skills

### Step 2: Create the skill file

```
.claude/skills/<skill-name>/SKILL.md
```

### Step 3: Write the frontmatter

```yaml
---
name: skill-name
description: One-sentence description of WHEN to use this skill. Be specific — Claude uses this to decide whether to auto-invoke it.
allowed-tools: Tool1, Tool2, Bash(command-prefix *)
---
```

**Frontmatter fields to consider:**

| Field | When to use it |
|-------|---------------|
| `description` | Always. Write it from the perspective of "invoke this when..." |
| `allowed-tools` | When the skill needs specific tool access. Scope `Bash()` narrowly (e.g., `Bash(docker *)` not `Bash(*)`) |
| `disable-model-invocation: true` | For destructive or side-effect-heavy skills (deploy, delete, send) that should only run when explicitly requested |
| `user-invocable: false` | For background knowledge that Claude should absorb but users don't invoke directly |
| `argument-hint` | When the skill accepts arguments — shows in autocomplete |
| `context: fork` | For long-running or exploratory skills that shouldn't pollute the main context |

### Step 4: Write the body

Follow this structure:

```markdown
# Title

One paragraph: what this skill does and why it exists.

## Quick Reference

The most common commands/operations as copy-paste-ready examples.
Cover the 80% use case first.

## Detailed Guide

### Section per workflow or concept

Deeper explanation with examples.
Use concrete, realistic examples — not abstract placeholders.

## Workflow Patterns

### Pattern: [Named scenario]

Step-by-step for a specific real-world use case.
Map patterns to agent roles (planner/worker/supervisor) when relevant.

## Important Rules

Numbered list of gotchas, constraints, and best practices.
Only include rules that an agent would plausibly get wrong.
```

### Step 5: Validate the skill

Before finishing, verify:

- [ ] The `description` clearly states WHEN to invoke (not just WHAT it does)
- [ ] `allowed-tools` is scoped to the minimum needed
- [ ] Examples use realistic values, not just `PLACEHOLDER`
- [ ] The skill is self-contained — an agent can follow it without reading other files
- [ ] No sensitive information (secrets, internal URLs, credentials) is hardcoded
- [ ] The file is under ~200 lines — move reference material to supporting files if longer

## Quality Checklist

A good skill:
- **Saves real time** — captures knowledge that took significant effort to discover
- **Is specific** — "how to use our CI pipeline" beats "how to do CI/CD"
- **Has concrete examples** — show exact commands with realistic arguments
- **Covers failure modes** — what goes wrong and how to recover
- **Matches agent roles** — indicate which agents (planner/worker/supervisor) use which parts
- **Scopes permissions tightly** — `Bash(bd *)` not `Bash(*)`

A bad skill:
- Restates what's already in the tool's `--help` output
- Is so generic it applies to any project
- Contains only theory with no actionable commands
- Requires reading 3 other files to understand

## Example: Creating a Skill from Task Experience

Suppose you just finished setting up a Python project with uv and learned the key commands. You'd run `/meta-skill uv-workflow "manage Python deps and scripts with uv"` and then create:

```
.claude/skills/uv-workflow/SKILL.md
```

```yaml
---
name: uv-workflow
description: Use uv to manage Python dependencies, virtual environments, and scripts. Invoke when adding/removing packages, creating new Python projects, or running Python tools.
allowed-tools: Bash(uv *)
---

# uv Workflow

uv is the package manager for this project. Use it instead of pip/poetry/pipenv.

## Quick Reference

...commands learned during the task...

## Workflow Patterns

### Adding a new dependency
...steps you actually followed...

## Important Rules
...gotchas you hit during the task...
```

The key: write the skill you WISH you had when you started the task.
