---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*), Bash(git diff:*)
description: Stage and commit all changes
---

## Context

- Git status: !`git status --short`
- Staged diff: !`git diff --cached`
- Unstaged diff: !`git diff`

## Task

Stage all changes and create a commit. Follow these conventions:
- Use conventional commit format (feat:, fix:, refactor:, docs:, chore:, etc.)
- Keep the first line under 72 characters
- Add a blank line then details if needed

If a message was provided: $ARGUMENTS
