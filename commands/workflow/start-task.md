---
description: Initialize a new task with proper git workflow and branch creation
argument-hint: <task-description>
allowed-tools: [Bash, TodoWrite, AskUserQuestion]
---

Initialize a new development task with proper git workflow and feature branch creation.

**Note:** Execute all commands from your project root directory.

## Before Starting

1. **Gather task information** using AskUserQuestion:
   - Task description/summary
   - Task type (feature, bugfix, or enhancement)
   - Expected deliverables
   - Any specific requirements or constraints

2. **Create TodoWrite list** to track initialization progress:
   - "Sync with main branch"
   - "Generate and validate branch name"
   - "Create local feature branch"
   - "Push branch to remote"
   - "Verify branch on GitHub"
   - "Set up task tracking"

## Prerequisites Check

Before proceeding, verify:
- Git repository initialized: `git status`
- GitHub CLI authenticated: `gh auth status`
- No uncommitted changes on current branch: `git diff-index --quiet HEAD --`

---

## Step 1: Sync with Main Branch

Ensure you're working with the latest code:

```bash
# Switch to main branch if not already there
git checkout main

# Sync with remote
gh repo sync

# Verify sync succeeded
if [ $? -eq 0 ]; then
  echo "✅ Successfully synced with remote main"
else
  echo "❌ Sync failed - check network and authentication"
  exit 1
fi
```

**Verify**: Check for "Already up to date" or successful merge message

**Alternative** if gh CLI not available:
```bash
git fetch origin main
git merge origin/main
```

---

## Step 2: Generate Feature Branch Name

Create a properly formatted branch name from the task description.

```bash
# Get task description from user (via AskUserQuestion)
TASK_DESC="[task description from user]"

# Determine branch type prefix based on task type
# Options: "feature", "bugfix", "enhancement"
TYPE_PREFIX="feature"  # Adjust based on task type

# Convert task description to kebab-case
# - Convert to lowercase
# - Replace spaces with hyphens
# - Remove special characters
# - Trim multiple hyphens to single hyphen
BRANCH_SUFFIX=$(echo "$TASK_DESC" | \
  tr '[:upper:]' '[:lower:]' | \
  tr ' ' '-' | \
  sed 's/[^a-z0-9-]//g' | \
  sed 's/--*/-/g' | \
  sed 's/^-//; s/-$//')

# Combine prefix and suffix
BRANCH_NAME="${TYPE_PREFIX}/${BRANCH_SUFFIX}"

echo "Generated branch name: $BRANCH_NAME"
```

**Examples:**
- "Add User Authentication" → `feature/add-user-authentication`
- "Fix Login Bug" → `bugfix/fix-login-bug`
- "Improve Performance" → `enhancement/improve-performance`

**Verify**: Branch name follows pattern `(feature|bugfix|enhancement)/[a-z0-9-]+`

---

## Step 3: Create and Checkout Feature Branch

Create the new branch locally:

```bash
# Create and checkout new branch
git checkout -b "$BRANCH_NAME"

# Verify branch creation
if git branch | grep -q "* $BRANCH_NAME"; then
  echo "✅ Branch '$BRANCH_NAME' created and checked out"
else
  echo "❌ Branch creation failed"
  exit 1
fi
```

**Verify**:
- `git branch` shows asterisk (*) next to your new branch
- `git branch --show-current` outputs your branch name

---

## Step 4: Push Branch to Remote

Push the new branch to GitHub immediately:

```bash
# Push to remote and set upstream
git push -u origin "$BRANCH_NAME"

# Verify push succeeded
if git ls-remote --heads origin "$BRANCH_NAME" | grep -q "$BRANCH_NAME"; then
  echo "✅ Branch pushed to remote successfully"
else
  echo "❌ Push failed - check permissions and network"
  exit 1
fi
```

**Verify**:
- Command outputs "Branch '$BRANCH_NAME' set up to track remote branch"
- Check on GitHub: Branch appears in repository branches list

---

## Step 5: Set Up Task Tracking

Create TodoWrite list for the actual task implementation:

```bash
# The TodoWrite list should include specific subtasks based on the task description
# Example for "Add User Authentication":
# - "Design authentication flow"
# - "Implement login endpoint"
# - "Add JWT token generation"
# - "Create authentication middleware"
# - "Add unit tests"
# - "Update documentation"
```

**Create TodoWrite with specific subtasks** based on the task requirements gathered earlier.

---

## Verification Checklist

After completing all steps, verify:

```bash
# Check current branch
echo "Current branch: $(git branch --show-current)"

# Verify remote tracking
git branch -vv | grep "* $BRANCH_NAME"

# Check remote branch exists
gh browse --branch "$BRANCH_NAME" --no-browser 2>/dev/null && echo "✅ Branch exists on GitHub"

# Verify clean working tree
git status
```

---

## Troubleshooting

**If sync fails:**
- Check network connection: `ping github.com`
- Verify GitHub authentication: `gh auth status`
- If auth expired: `gh auth login`
- Try manual sync: `git fetch origin && git merge origin/main`

**If branch already exists:**
- List all branches: `git branch -a`
- Check if it's your branch: `git log <branch-name> --oneline`
- Delete if abandoned: `git branch -D <branch-name>`
- Delete remote if needed: `git push origin --delete <branch-name>`

**If branch name is invalid:**
- Check for special characters: Branch names cannot contain spaces, ~, ^, :, *, ?, [, \
- Ensure starts with letter or number
- Re-generate with corrected task description

**If push fails:**
- Check remote exists: `git remote -v`
- Verify repository write permissions
- Check if remote branch exists: `git ls-remote --heads origin`
- Try verbose push: `git push -v -u origin "$BRANCH_NAME"`

**If authentication issues:**
- Check GitHub token: `echo $GITHUB_TOKEN`
- Re-authenticate with gh CLI: `gh auth login --web`
- Verify SSH keys: `ssh -T git@github.com`
- Check remote URL format: `git remote get-url origin`

**If uncommitted changes block checkout:**
- View changes: `git status`
- Stash changes: `git stash save "Before starting new task"`
- Or commit them: `git add . && git commit -m "WIP: Save before new task"`

**If main branch is not up to date:**
- Force sync from remote: `git fetch origin main && git reset --hard origin/main`
- **Warning**: This discards local changes on main branch

---

## Success Criteria

✅ Main branch synchronized with remote
✅ Feature branch created with proper naming convention
✅ Branch pushed to remote with upstream tracking
✅ Branch visible on GitHub repository
✅ TodoWrite list created with task breakdown
✅ Clean working tree (`git status` shows no changes)
✅ Ready to begin implementation

---

## Next Steps

After successful initialization:
1. Begin implementing the task requirements
2. Make small, focused commits as you progress
3. Update TodoWrite list as subtasks complete
4. Run `/validate` before each commit
5. Use `/complete-task` when all work is finished

---

## Project Workflow Integration

This command is part of the standard development workflow:
1. **→ /start-task** - Initialize new task (you are here)
2. Development work (implement features, fix bugs)
3. /validate - Run checks before committing
4. /complete-task - Commit, push, and create PR
5. Review and merge

---

## Example Usage

```bash
# User provides via AskUserQuestion:
# Task: "Add user profile page"
# Type: "feature"

# Commands executed:
git checkout main
gh repo sync
# Generated branch: feature/add-user-profile-page
git checkout -b feature/add-user-profile-page
git push -u origin feature/add-user-profile-page

# TodoWrite created with subtasks:
# - Design profile page layout
# - Create Profile component
# - Add API endpoint for user data
# - Integrate with existing auth
# - Add tests
# - Update documentation
```

**IMPORTANT:** Do not proceed with implementation until all success criteria are met and branch is confirmed on remote.
