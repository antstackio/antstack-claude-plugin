---
description: Sync local repository with remote main branch
allowed-tools: [Bash]
---

Synchronize the local repository with the remote main branch:

## Sync Process:

Execute the GitHub CLI sync command:
```bash
gh repo sync --cwd /app/workspace
```

## Expected Outcomes:

**Success Indicators:**
- ✅ "Already up to date" - No new changes from remote
- ✅ "Successfully synced" - Remote changes merged successfully
- ✅ No merge conflicts

**If Merge Conflicts Occur:**
1. Review conflict details in output
2. Manually resolve conflicts in affected files
3. Stage resolved files: `git -C /app/workspace add <resolved-files>`
4. Complete merge: `git -C /app/workspace commit`
5. Verify with: `git -C /app/workspace status`

## Verification:

After successful sync:
```bash
git -C /app/workspace log --oneline -5
```

This shows the latest 5 commits to verify sync status.

## When to Use:

- **Before starting a new task** - Ensure working with latest code
- **Before creating a pull request** - Prevent merge conflicts
- **After extended development periods** - Stay updated with team changes
- **When instructed by team workflow** - Follow project conventions

## Success Criteria:

✅ Local main branch is synchronized with remote
✅ No merge conflicts remaining
✅ Ready to create feature branch or continue development
