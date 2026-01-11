---
allowed-tools: Bash(gadd:*), Bash(gcommit:*), Bash(gs:*), Bash(gdiff:*), Bash(gdiffs:*)
description: Stage and commit changes using dotfile aliases
---

## Context

- Git status: !`gs`
- Staged changes: !`gdiffs`
- Unstaged changes: !`gdiff`

## Task

Stage and commit changes using the dotfile git aliases:

1. Run `gadd` to stage all changes
2. Run `gcommit` to commit (auto-generates message with AI) or `gcommit "message"` if a message was provided

Follow these commit conventions:
- Use conventional commit format (feat:, fix:, refactor:, docs:, chore:, etc.)
- Keep the first line under 72 characters
- Add details after a blank line if needed

If a message was provided: $ARGUMENTS
