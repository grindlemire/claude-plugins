---
allowed-tools: Bash(gpull:*), Bash(git:*), Bash(gs:*), Bash(gdiff:*), Read, AskUserQuestion
description: Pull latest changes from upstream or main branch, handling conflicts interactively
---

## Context

- Git status: !`gs`

## Task

Pull the latest changes, handling any conflicts that arise:

### Step 1: Determine the pull target

- If the current branch has an upstream tracking branch, pull from that using `gpull`
- If no upstream exists, pull from the main/master branch using `git pull origin <base-branch>`

### Step 2: Execute the pull

Run the appropriate pull command and check the result.

### Step 3: Handle conflicts

If there are merge conflicts:

1. Run `git status` to identify the conflicting files
2. For each conflicting file, read the file and describe the conflict to the user:
   - What the incoming changes are trying to do
   - What the local changes are trying to do
   - Where the conflict markers are located

3. Use AskUserQuestion to ask the user how they want to resolve each conflict:
   - Accept incoming changes (theirs)
   - Keep local changes (ours)
   - Manually merge (show both and let user decide)
   - Abort the merge entirely

4. After resolving conflicts based on user input:
   - Stage the resolved files with `git add`
   - Complete the merge with `git commit`

### Step 4: Report result

- If successful with no conflicts: Report the commits that were pulled
- If conflicts were resolved: Summarize what was merged and how conflicts were resolved
- If merge was aborted: Confirm the abort and show current state

Additional instructions: $ARGUMENTS
