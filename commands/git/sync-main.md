---
description: Sync local repository with remote main branch
allowed-tools: [Bash, TodoWrite]
---

Synchronize the local repository with the remote main branch to get the latest changes.

**Note:** Execute all commands from your project root directory.

## Before Starting

1. **Create TodoWrite list** to track sync process (optional for simple sync):
   - "Check for uncommitted changes"
   - "Sync with remote main"
   - "Verify sync successful"
   - "Review latest commits"

2. **Check for uncommitted changes** (critical safety step):
   ```bash
   # Check if working tree has uncommitted changes
   if ! git diff-index --quiet HEAD --; then
     echo "⚠️  Warning: You have uncommitted changes"
     echo ""
     git status --short
     echo ""
     echo "Options:"
     echo "1. Commit changes: git add . && git commit -m 'WIP: Save before sync'"
     echo "2. Stash changes: git stash save 'Before sync'"
     echo "3. Discard changes: git reset --hard HEAD (CAUTION: loses all changes)"
     exit 1
   fi

   echo "✅ Working tree is clean - safe to proceed"
   ```

---

## Sync Process

### Option 1: Using GitHub CLI (Recommended)

```bash
# Sync using gh CLI
gh repo sync

# Check exit code
if [ $? -eq 0 ]; then
  echo "✅ Successfully synced with remote"
else
  echo "❌ Sync failed - see error above"
  exit 1
fi
```

**Expected Outcomes:**
- ✅ "Already up to date" - No new changes from remote
- ✅ "Successfully synced" - Remote changes merged successfully
- ⚠️  Merge conflicts - Need manual resolution (see troubleshooting)

### Option 2: Using Git Directly

```bash
# Fetch latest changes from remote main
git fetch origin main

# Check if behind main
BEHIND_COUNT=$(git rev-list --count HEAD..origin/main)

if [ "$BEHIND_COUNT" -eq 0 ]; then
  echo "✅ Already up to date with origin/main"
else
  echo "Syncing $BEHIND_COUNT commits from origin/main..."

  # Merge remote main into current branch
  git merge origin/main

  if [ $? -eq 0 ]; then
    echo "✅ Successfully merged $BEHIND_COUNT commits"
  else
    echo "❌ Merge failed - conflicts need resolution"
    exit 1
  fi
fi
```

---

## Handle Merge Conflicts (If They Occur)

If merge conflicts are detected:

### Step 1: View Conflicted Files

```bash
# List all files with conflicts
echo "Conflicted files:"
git diff --name-only --diff-filter=U

# View conflict markers
git status | grep "both modified"
```

### Step 2: Resolve Each Conflict

For each conflicted file:

```bash
# Open file and look for conflict markers:
# <<<<<<< HEAD           (your changes)
# =======                (separator)
# >>>>>>> origin/main    (remote changes)

# Edit the file to resolve conflicts
# Remove conflict markers
# Keep the correct version or merge both changes
```

**Example conflict:**
```
<<<<<<< HEAD
const API_URL = 'http://localhost:3000';
=======
const API_URL = 'https://api.production.com';
>>>>>>> origin/main
```

**Resolved:**
```
const API_URL = process.env.API_URL || 'http://localhost:3000';
```

### Step 3: Stage Resolved Files

```bash
# Stage each resolved file
git add <resolved-file>

# Or stage all resolved files
git add .
```

### Step 4: Complete the Merge

```bash
# Get current branch name for commit message
CURRENT_BRANCH=$(git branch --show-current)

# Complete the merge commit
git commit -m "Merge origin/main into $CURRENT_BRANCH

Resolved conflicts in:
- <file1>
- <file2>"

# Verify merge completed
if [ $? -eq 0 ]; then
  echo "✅ Merge conflicts resolved and committed"
  git status
else
  echo "❌ Commit failed"
  exit 1
fi
```

---

## Verification

After successful sync, verify the state:

```bash
# View latest commits to confirm sync
echo "Latest 5 commits:"
git log --oneline -5

# Check current branch status
echo ""
echo "Branch status:"
git status

# Verify no uncommitted changes
if git diff-index --quiet HEAD --; then
  echo "✅ Working tree is clean"
else
  echo "⚠️  There are uncommitted changes"
  git status --short
fi

# Check if ahead of remote (if on feature branch)
CURRENT_BRANCH=$(git branch --show-current)
if [ "$CURRENT_BRANCH" != "main" ]; then
  AHEAD_COUNT=$(git rev-list --count origin/$CURRENT_BRANCH..$CURRENT_BRANCH 2>/dev/null || echo "0")
  if [ "$AHEAD_COUNT" -gt 0 ]; then
    echo "⚠️  Your branch is $AHEAD_COUNT commits ahead of origin/$CURRENT_BRANCH"
    echo "Consider pushing: git push origin $CURRENT_BRANCH"
  fi
fi
```

---

## Troubleshooting

**If sync fails with "uncommitted changes" error:**
- View uncommitted changes: `git status`
- **Option 1 - Commit changes:**
  ```bash
  git add .
  git commit -m "WIP: Save before sync"
  # Then retry sync
  ```
- **Option 2 - Stash changes:**
  ```bash
  git stash save "Before syncing with main"
  # Sync
  # Then restore: git stash pop
  ```
- **Option 3 - Discard changes (CAUTION):**
  ```bash
  git reset --hard HEAD  # Loses all uncommitted changes!
  ```

**If sync fails with authentication error:**
- Check GitHub authentication: `gh auth status`
- View auth details: `gh auth status --show-token`
- Re-authenticate: `gh auth login --web`
- Check SSH keys (if using SSH): `ssh -T git@github.com`

**If conflicts are too complex to resolve:**
- **Abort the merge:**
  ```bash
  git merge --abort
  echo "Merge aborted - back to pre-sync state"
  ```
- **Create backup branch:**
  ```bash
  BACKUP_BRANCH="backup-$(date +%Y%m%d-%H%M%S)"
  git branch "$BACKUP_BRANCH"
  echo "Created backup branch: $BACKUP_BRANCH"
  ```
- **Seek help from team:**
  - Share conflicted files with team member
  - Pair program to resolve conflicts
  - Consider rebasing instead: `git rebase origin/main`

**If accidentally synced wrong branch:**
- **Reset to previous state:**
  ```bash
  git reset --hard ORIG_HEAD
  echo "Reset to state before sync"
  ```
- **Verify:**
  ```bash
  git log --oneline -5
  git status
  ```

**If gh CLI is not available:**
- **Install gh CLI:**
  - Mac: `brew install gh`
  - Linux: `sudo apt install gh` or download from https://cli.github.com/
  - Windows: `winget install GitHub.cli`
- **Or use git directly** (see Option 2 above)

**If sync brings unexpected changes:**
- **Review changes:**
  ```bash
  git log origin/main..HEAD  # Your changes
  git log HEAD..origin/main  # Remote changes before sync
  git diff origin/main...HEAD  # Full diff
  ```
- **If changes are problematic:**
  ```bash
  git reset --hard ORIG_HEAD  # Undo sync
  # Investigate with team before re-syncing
  ```

**If merge conflicts keep recurring:**
- **Rebase instead of merge:**
  ```bash
  git fetch origin main
  git rebase origin/main
  # Resolve conflicts as they appear
  # git add <resolved-files>
  # git rebase --continue
  ```
- **Update .gitattributes** for consistent line endings
- **Coordinate with team** on conflicting changes

---

## Success Criteria

✅ No uncommitted changes before sync
✅ Local main branch synchronized with remote
✅ No merge conflicts remaining (or all resolved)
✅ Working tree is clean after sync
✅ Recent commits visible in git log
✅ Ready to create feature branch or continue development

---

## When to Use This Command

**Before starting new work:**
- Ensures you have latest code
- Prevents conflicts later
- Good practice at start of day

**Before creating a pull request:**
- Keeps PR up to date
- Reduces merge conflicts
- Shows latest code in PR

**After extended development periods:**
- Stay in sync with team changes
- Catch breaking changes early
- Easier conflict resolution

**As part of team workflow:**
- Daily syncs recommended
- After major merges to main
- Before deployments

---

## Related Commands

- `/start-task` - Syncs main before creating feature branch
- `/create-pr` - Should sync before creating PR
- `/complete-task` - May sync as part of completion process

---

## Example Sync Flow

```bash
# 1. Check current state
git status
# Output: On branch main, nothing to commit

# 2. Sync with remote
gh repo sync
# Output: ✓ Synced the "main" branch from "origin"

# 3. Verify sync
git log --oneline -5
# Shows latest 5 commits including new ones from remote

# 4. Ready to work
echo "Sync complete - ready to start new work"
```

**With uncommitted changes:**
```bash
# 1. Check status
git status
# Output: modified: src/app.ts

# 2. Stash changes
git stash save "WIP before sync"
# Output: Saved working directory state

# 3. Sync
gh repo sync
# Output: ✓ Synced

# 4. Restore changes
git stash pop
# Output: Changes restored

# 5. Verify
git status
# Output: modified: src/app.ts (changes back)
```

**With merge conflicts:**
```bash
# 1. Sync attempts
gh repo sync
# Output: CONFLICT (content): Merge conflict in src/config.ts

# 2. View conflicts
git diff --name-only --diff-filter=U
# Output: src/config.ts

# 3. Resolve conflict
# Edit src/config.ts, remove <<<, ===, >>> markers

# 4. Stage resolved file
git add src/config.ts

# 5. Complete merge
git commit -m "Merge origin/main into main

Resolved conflict in src/config.ts"

# 6. Verify
git status
# Output: nothing to commit, working tree clean
```
