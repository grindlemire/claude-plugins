---
allowed-tools: Bash(gpull:*), Bash(git:*), Bash(gs:*), Bash(gdiff:*), Bash(fixssh:*), Read, Edit, AskUserQuestion
description: Pull latest changes from upstream or main branch, handling conflicts interactively
---

## SSH Signing Key Error Handling

If any git command fails with an error about "No private key found" or similar SSH signing key errors, run `fixssh` and then retry the failed command.

## Context

- Git status: !`gs`
- Current branch: !`git branch --show-current`
- Tracking branch: !`git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "No upstream configured"`
- Local commits not pushed: !`git log @{u}..HEAD --oneline 2>/dev/null || echo "N/A"`

## Task

Pull the latest changes, handling any conflicts that arise.

### Step 1: Pre-pull checks

- If there are uncommitted changes, warn the user and ask if they want to stash them first or abort
- **Recommend stashing** if the changes are minor/WIP, recommend aborting if changes are significant and should be committed first
- If stashing, run `git stash push -m "Auto-stash before pull"`

### Step 2: Determine the pull target and execute

- If the current branch has an upstream tracking branch, pull from that using `gpull`
- If no upstream exists, pull from the main/master branch using `git pull origin <base-branch>`

### Step 3: Handle merge conflicts

If there are merge conflicts:

1. **Get the full picture first**: Run `git status` to get the complete list of conflicting files. Tell the user how many files have conflicts.

2. **Ask about resolution strategy**: Before diving into individual files, use AskUserQuestion to ask the user their preferred approach. **Provide a recommendation based on the situation:**
   - **Recommend "one at a time"** (default) when conflicts are in different logical areas or you're unsure what the changes do
   - **Recommend "accept incoming"** when pulling from main/upstream and local branch should adopt those changes (e.g., pulling latest main into a feature branch)
   - **Recommend "keep local"** when local changes are intentional divergences that should be preserved
   - **Recommend "abort"** when the conflicts look complex and the user may want to commit local work first or get help

   Options:
   - Resolve conflicts one file at a time (interactive)
   - Accept all incoming changes for all files (`git checkout --theirs .`)
   - Keep all local changes for all files (`git checkout --ours .`)
   - Abort the merge entirely (`git merge --abort`)

3. **If resolving one at a time**: For each conflicting file:
   - Read the file and show the conflict sections (look for `<<<<<<<`, `=======`, `>>>>>>>` markers)
   - Explain what each side is trying to do
   - **Provide a recommendation** for this specific file based on analyzing the conflict:
     - **Recommend "incoming"** if the remote changes look like improvements, bug fixes, or updates the local code was unaware of
     - **Recommend "local"** if the local changes are intentional improvements over what's incoming
     - **Recommend "manual edit"** if both sides have valuable changes that should be combined, or if the conflict is in a complex area where automatic resolution could break things
   - Use AskUserQuestion to ask how to resolve THIS file:
     - Accept incoming (theirs): `git checkout --theirs <file>`
     - Keep local (ours): `git checkout --ours <file>`
     - Manual edit: Use the Edit tool to resolve the conflict markers, keeping the desired code
   - After resolving, stage the file: `git add <file>`

4. **Complete the merge**: After all conflicts are resolved:
   - Run `git status` to verify no conflicts remain
   - Run `git commit` to complete the merge (use the default merge commit message)

### Step 4: Post-pull cleanup

- If changes were stashed in Step 1, ask the user if they want to pop the stash
- **Recommend popping the stash** unless the pull brought in changes to the same files that were stashed (check with `git stash show` vs files changed in pull)
- Run `git stash pop` if user agrees
- If stash pop causes conflicts, handle those the same way as merge conflicts

### Step 5: Report result

- Show `git log --oneline -5` to display recent commits including what was pulled
- If conflicts were resolved: Summarize which files had conflicts and how each was resolved
- If merge was aborted: Confirm the abort and show current state with `gs`

Additional instructions: $ARGUMENTS
