---
name: beads
description: Use beads (bd) to create and manage epics, tasks, and subtasks. Invoke this skill when planning work, breaking down features into tasks, tracking progress, discovering new work during implementation, or querying what to work on next.
allowed-tools: Bash(bd *)
---

# Beads Task Management

Beads (`bd`) is a distributed, git-backed issue tracker designed for AI coding agents. Use it to create structured work hierarchies (epics → tasks → subtasks), track dependencies, and manage progress.

## Quick Reference

### Creating Epics and Tasks

```bash
# Create an epic (top-level container for a feature/initiative)
bd create "Epic title" -t epic -p PRIORITY -d "Description of the epic"

# Create a task under an epic
bd create "Task title" -t task -p PRIORITY --parent EPIC_ID -d "What this task accomplishes"

# Create a subtask under a task
bd create "Subtask title" -t task -p PRIORITY --parent TASK_ID -d "Specific subtask detail"

# Create a bug
bd create "Bug title" -t bug -p PRIORITY -d "What's broken and expected behavior"

# Create with labels
bd create "Task title" -t task -p 1 -l backend,auth -d "Description"
```

**Priority scale:** 0 (lowest) to 4 (highest/urgent)

**Types:** `epic`, `task`, `feature`, `bug`, `chore`

### Issue IDs

- Format: `bd-XXXX` (hash-based, e.g., `bd-a3f8`)
- Child issues: `bd-a3f8.1`, `bd-a3f8.2` (parent.number)
- IDs are auto-generated — capture them from command output

### Status Workflow

```
open → in_progress → closed
```

```bash
# Start working on a task (claim it atomically)
bd update ISSUE_ID --status in_progress --claim

# Close a completed task
bd close ISSUE_ID --reason "Brief summary of what was done"

# Reopen if needed
bd reopen ISSUE_ID --reason "Why it needs more work"
```

### Querying Work

```bash
# Find tasks ready to work on (no blockers)
bd ready

# Find tasks ready to work on (JSON for parsing)
bd ready --json --limit 5

# See blocked tasks
bd blocked

# List with filters
bd list --status open
bd list --status in_progress
bd list --type epic
bd list --priority 3
bd list --label backend
bd list --title-contains "authentication"

# Show full details of a specific issue
bd show ISSUE_ID

# View dependency tree for an epic
bd dep tree EPIC_ID
```

### Managing Dependencies

```bash
# Task B is blocked by Task A (A must finish before B can start)
bd dep add TASK_B TASK_A

# Link discovered work back to its origin
bd dep add NEW_TASK ORIGIN_TASK --type discovered-from

# Other dependency types: blocks, blocked-by, relates-to, duplicates, supersedes
```

### Labels

```bash
bd label add ISSUE_ID label-name
bd label remove ISSUE_ID label-name
bd label list ISSUE_ID
bd label list-all
```

## Workflow Patterns

### Pattern 1: Planning — Break Down a Feature into an Epic

When the planner agent designs work, structure it as:

```bash
# 1. Create the epic
bd create "User Authentication System" -t epic -p 2 -d "Implement complete auth flow with login, signup, and session management"

# 2. Create phase/area tasks under the epic (capture the epic ID from step 1)
bd create "Design database schema for users" -t task -p 2 --parent bd-EPIC -d "Create user table with email, password hash, sessions"
bd create "Implement signup endpoint" -t task -p 2 --parent bd-EPIC -d "POST /api/signup with email validation and password hashing"
bd create "Implement login endpoint" -t task -p 2 --parent bd-EPIC -d "POST /api/login with JWT token generation"
bd create "Add session middleware" -t task -p 1 --parent bd-EPIC -d "Middleware to validate JWT and attach user to request"

# 3. Set dependencies (login depends on schema, middleware depends on login)
bd dep add bd-LOGIN bd-SCHEMA
bd dep add bd-MIDDLEWARE bd-LOGIN
```

### Pattern 2: Worker — Pick Up and Complete Tasks

```bash
# 1. Find ready work
bd ready --json --limit 1

# 2. Claim the task
bd update ISSUE_ID --status in_progress --claim

# 3. Do the implementation work...

# 4. If you discover additional work needed:
bd create "Handle edge case for expired tokens" -t task -p 2 --parent PARENT_ID -d "Description"

# 5. Close when done
bd close ISSUE_ID --reason "Implemented signup endpoint with validation and tests"
```

### Pattern 3: Supervisor — Orchestrate and Track Progress

```bash
# Review overall progress
bd list --type epic
bd dep tree EPIC_ID

# Check what's in progress
bd list --status in_progress

# Check what's ready for assignment
bd ready

# Check what's blocked and why
bd blocked

# After all child tasks are closed, close the epic
bd close EPIC_ID --reason "All auth features implemented and tested"
```

## Important Rules

1. **Always use `--json` when parsing output programmatically** — human-readable output may change format.
2. **Capture issue IDs from creation output** — you need them for `--parent`, `dep add`, and `update` commands.
3. **Use `--claim` when starting work** — this atomically assigns the task and prevents conflicts.
4. **Set dependencies explicitly** — don't assume ordering; use `bd dep add` so `bd ready` works correctly.
5. **Close tasks with meaningful `--reason`** — this preserves context for future reference and compaction.
6. **Use epics as containers, tasks as actionable units** — epics should not contain implementation details, only child tasks should.
7. **Check `bd ready` before starting new work** — respect the dependency graph.

## Initialization

If beads hasn't been initialized in the project yet:

```bash
bd init
```

This creates the `.beads/` directory with the Dolt database and sets up git hooks for automatic JSONL sync.
