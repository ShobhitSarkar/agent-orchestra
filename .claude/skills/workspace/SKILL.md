---
name: workspace
description: Use the workspace skill to clone repositories, create feature branches, and manage git operations for tasks. Invoke this when setting up a repo to work on, committing completed work, or pushing changes to a feature branch.
allowed-tools: Bash(git *)
---

# Workspace Management

All work on external repositories happens inside `.workspaces/` within the agent-orchestra project. This directory is gitignored — cloned repos live here and never pollute agent-orchestra's own git history.

## Setting Up a Repo

```bash
# Clone a repo into the workspace
git clone <repo-url> .workspaces/<repo-name>
```

If the repo is already cloned, pull latest main instead:

```bash
git -C .workspaces/<repo-name> checkout main
git -C .workspaces/<repo-name> pull origin main
```

## Creating a Feature Branch

```bash
git -C .workspaces/<repo-name> checkout -b <branch-name>
```

Use descriptive branch names: `feat/<feature>`, `fix/<issue>`, `chore/<description>`.

## Committing and Pushing Work

```bash
# Stage specific files — never use `git add .` or `git add -A`
git -C .workspaces/<repo-name> add <specific-files>

# Commit with a descriptive message
git -C .workspaces/<repo-name> commit -m "commit message"

# Push to the feature branch
git -C .workspaces/<repo-name> push -u origin <branch-name>
```

Always push to the feature branch after committing so work is available for review.

## Checking Status

```bash
# See what's changed
git -C .workspaces/<repo-name> status
git -C .workspaces/<repo-name> diff

# Review commit history
git -C .workspaces/<repo-name> log --oneline -10
```

## Important Rules

1. **Never push to main/master directly** — always work on a feature branch.
2. **Never force-push or delete remote branches.**
3. **Stage specific files by name** — avoid `git add .` which can catch unintended files.
4. **Pull latest main before branching** — keeps feature branches up to date.
5. **All repos live in `.workspaces/`** — nowhere else.
