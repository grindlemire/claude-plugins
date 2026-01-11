---
allowed-tools: Bash(git:*)
description: Rollback recent changes or commits
---

## Context

- Git status: !`git status --short`
- Recent commits: !`git log --oneline -10`
- Reflog: !`git reflog --max-count=10`

## Task

Help rollback changes. Options:
- Discard uncommitted changes: `git checkout -- .` or `git restore .`
- Undo last commit (keep changes): `git reset --soft HEAD~1`
- Undo last commit (discard changes): `git reset --hard HEAD~1`
- Revert a specific commit: `git revert <sha>`

Ask for confirmation before any destructive operation.

Specific request: $ARGUMENTS
