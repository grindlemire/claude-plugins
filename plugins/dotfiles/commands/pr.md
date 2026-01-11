---
allowed-tools: Bash(git:*), Bash(gh:*), Bash(gs:*), Bash(fixssh:*)
description: Commit, push, and create a pull request with thorough description
---

## SSH Signing Key Error Handling

If any git command fails with an error about "No private key found" or similar SSH signing key errors, run `fixssh` and then retry the failed command.

## Context

- Git status: !`gs`
- Recent commits: !`git log --oneline -10`

## Task

1. Get the current branch name with `git branch --show-current`
2. If there are uncommitted changes, stage and commit them
3. Push the current branch to origin (use `-u` if needed)
4. Get the diff against main/master with `git diff main...HEAD` to understand changes for the PR description
5. Create a pull request using `gh pr create`

## PR Title Guidelines

- Start with a type prefix: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `test:`
- Be specific and descriptive (not "Update code" but "Add user authentication flow")
- Keep under 72 characters
- Use imperative mood ("Add feature" not "Added feature")

## PR Description Format

Structure the description with these sections:

### Summary
2-3 sentences explaining WHAT this PR does and WHY. Focus on the problem being solved or feature being added.

### Changes
Bullet list of specific changes, grouped logically:
- **Category** (e.g., API, UI, Database, Config)
  - Specific change with context
  - Another change

### Technical Notes (if applicable)
- Architecture decisions or trade-offs made
- Dependencies added/removed
- Migration steps or breaking changes
- Performance considerations

### Testing
- How the changes were tested
- Areas that need manual testing
- Edge cases considered

## Formatting Tips

- Use code blocks for file names, functions, commands: \`fileName.ts\`
- Use bullet points for lists, not paragraphs
- Keep each bullet point to one line when possible
- Add blank lines between sections for readability

Additional instructions: $ARGUMENTS
