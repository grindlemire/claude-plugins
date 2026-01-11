---
allowed-tools: Bash(git:*), Bash(gh:*)
description: Commit, push, and create a pull request
---

## Context

- Current branch: !`git branch --show-current`
- Git status: !`git status --short`
- Commits not yet pushed: !`git log @{u}..HEAD --oneline 2>/dev/null || git log --oneline -5`
- Diff from main: !`git diff main...HEAD --stat 2>/dev/null || git diff HEAD~5 --stat`

## Task

1. If there are uncommitted changes, stage and commit them
2. Push the current branch to origin
3. Create a pull request using `gh pr create`

Use a clear PR title and description based on the commits and changes.

Additional instructions: $ARGUMENTS
