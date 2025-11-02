---
description: Complete task with validation, commit, and pull request creation
allowed-tools: [Bash, SlashCommand, TodoWrite]
---

Complete the current development task with proper validation and git workflow.

**Note:** Execute all commands from your project root directory.

## Before Starting

1. **Create TodoWrite list** to track completion steps:
   - "Run comprehensive validation checks"
   - "Clean temporary files"
   - "Review and stage all changes"
   - "Create descriptive commit message"
   - "Commit changes"
   - "Push to remote"
   - "Create pull request"
   - "Verify PR created and get URL"

2. **Ensure you're on the correct feature branch**:
   ```bash
   CURRENT_BRANCH=$(git branch --show-current)
   echo "Completing task on branch: $CURRENT_BRANCH"

   # Verify not on main branch
   if [ "$CURRENT_BRANCH" = "main" ] || [ "$CURRENT_BRANCH" = "master" ]; then
     echo "❌ Error: Cannot complete task on main branch"
     echo "Please switch to your feature branch first"
     exit 1
   fi
   ```

---

## Step 1: Run Comprehensive Validation

**CRITICAL:** Do not proceed if any validation fails.

Execute the validation command:
```bash
# Option 1: Use slash command (recommended)
# Execute: /validate slash command

# Option 2: Run validation directly
npm run typecheck || npx tsc --noEmit
npm run lint || npx eslint .
npm run test || npm test
npm run build || npm run compile
```

**If any validation fails:**
- **STOP** - Do not proceed with commit
- Fix all errors incrementally (one at a time)
- Rerun validation after each fix
- See `/validate` command for detailed error handling
- Use Context7 MCP for library-related issues
- Consult AWS CDK MCP for infrastructure errors

**Only proceed after all checks pass** ✅

---

## Step 2: Clean Temporary Files

Remove temporary and debug files before committing:

```bash
# Remove common temporary files
find . -type f \( \
  -name "*.tmp" -o \
  -name "*.temp" -o \
  -name "*.log" -o \
  -name "debug_*" -o \
  -name "test_output_*" \) \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  -delete

# Remove specific files if they exist
[ -f "progress.md" ] && rm progress.md
[ -f "TODO.txt" ] && rm TODO.txt
[ -f ".DS_Store" ] && find . -name ".DS_Store" -delete

# Verify cleanup - check git status
echo "Files after cleanup:"
git status --short
```

**Verify**: `git status` should only show intentional changes, no *.tmp, *.log, or debug files

---

## Step 3: Review and Stage Changes

Review what will be committed:

```bash
# View summary of changes
git status

# View detailed diff of unstaged changes
git diff

# View list of changed files
git diff --name-only
```

Stage all changes:

```bash
# Stage all changes
git add .

# Verify staged changes
echo "Staged changes:"
git diff --staged --name-status
```

**Review the staged changes carefully** to ensure only intended files are included.

**If you need to unstage specific files:**
```bash
git reset HEAD <file-path>
```

---

## Step 4: Create Descriptive Commit Message

Analyze the changes and create a meaningful commit message.

**Commit Message Format:**
```
<type>: <brief description>

<optional detailed description>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring
- `docs`: Documentation changes
- `test`: Test additions/updates
- `chore`: Maintenance tasks
- `perf`: Performance improvements

**Example Messages:**
```
feat: Add user authentication with JWT tokens

Implements JWT-based authentication system with:
- Login/logout endpoints
- Token generation and validation
- Middleware for protected routes
- User session management
```

```
fix: Resolve race condition in data fetching

Fixes issue where concurrent requests caused state inconsistency.
Added proper request deduplication.
```

---

## Step 5: Commit Changes

Create the commit using HEREDOC for proper formatting:

```bash
# Set your commit message
COMMIT_TYPE="feat"  # Adjust based on your changes
COMMIT_DESC="your brief description"
COMMIT_DETAILS="Optional detailed description

- Key change 1
- Key change 2"

# Create commit with formatted message
git commit -m "$(cat <<EOF
${COMMIT_TYPE}: ${COMMIT_DESC}

${COMMIT_DETAILS}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Verify commit created:**
```bash
# View the commit
git log -1 --pretty=format:"%h - %s%n%b"

# Confirm it's on your branch
git log --oneline -5
```

---

## Step 6: Push to Remote

Push changes to the remote branch:

```bash
# Get current branch name
CURRENT_BRANCH=$(git branch --show-current)
echo "Pushing branch: $CURRENT_BRANCH"

# Push to remote
git push origin "$CURRENT_BRANCH"

# Verify push succeeded
if git ls-remote --heads origin "$CURRENT_BRANCH" | grep -q "$CURRENT_BRANCH"; then
  echo "✅ Successfully pushed to origin/$CURRENT_BRANCH"

  # Check if remote is up to date
  LOCAL_COMMIT=$(git rev-parse HEAD)
  REMOTE_COMMIT=$(git rev-parse origin/"$CURRENT_BRANCH")

  if [ "$LOCAL_COMMIT" = "$REMOTE_COMMIT" ]; then
    echo "✅ Remote branch is up to date"
  else
    echo "⚠️  Warning: Remote and local commits differ"
  fi
else
  echo "❌ Push failed"
  exit 1
fi
```

**Verify**: Check GitHub repository to confirm commits are visible

---

## Step 7: Create Pull Request

Create a PR using the slash command:

```bash
# Execute the create-pr slash command
# This will:
# 1. Analyze all commits in your branch
# 2. Generate comprehensive PR description
# 3. Create PR on GitHub
# 4. Return PR URL
```

**Use SlashCommand:** Execute `/create-pr` or `/antstack-claude-plugin:git:create-pr`

**Alternative** - Create PR manually with gh CLI:
```bash
# Get commits for PR description
COMMITS=$(git log --oneline main..HEAD)

# Create PR
gh pr create --fill --body "$(cat <<'EOF'
## Summary
- Brief summary of changes

## Changes Made
- Detailed changes list

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests passing
- [ ] Manual testing completed

## Validation Checklist
- [x] TypeScript check passed
- [x] Linting passed
- [x] All tests passing
- [x] Build successful

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

---

## Step 8: Verify PR and Provide URL

Retrieve and display the PR URL:

```bash
# Get the most recent PR for current branch
PR_URL=$(gh pr list --head "$CURRENT_BRANCH" --json url --jq '.[0].url')

if [ -n "$PR_URL" ]; then
  echo "✅ Pull Request created successfully"
  echo "PR URL: $PR_URL"
else
  echo "❌ Could not retrieve PR URL"
  echo "Check PRs manually: gh pr list"
fi

# Alternative: Open PR in browser
gh pr view --web
```

**Provide the PR URL to the user** in your response.

---

## Troubleshooting

**If validation fails:**
- **NEVER** proceed with commit if validation fails
- Fix errors incrementally, one at a time
- After each fix, rerun the specific validation step
- For TypeScript errors: Review error messages, check types
- For lint errors: Try auto-fix first: `npm run lint -- --fix`
- For test failures: Debug specific tests, never skip or comment out
- For build errors: Check import paths, dependencies

**If file cleanup fails:**
- Manually review files: `git status`
- Remove unwanted files individually: `rm <file>`
- Check .gitignore is properly configured
- Verify temp files are listed in .gitignore

**If staging fails:**
- Check file permissions: `ls -la <file>`
- Verify files exist: `git status`
- Check for gitignore rules blocking files: `git check-ignore -v <file>`

**If commit fails:**
- Check for merge conflicts: `git status`
- Verify files are staged: `git diff --staged`
- Ensure commit message is non-empty
- Check for commit hooks blocking: Review .git/hooks/
- Bypass hooks only if necessary: `git commit --no-verify` (not recommended)

**If push fails:**
- Verify remote exists: `git remote -v`
- Check network connection: `ping github.com`
- Verify authentication: `gh auth status`
- Check branch tracking: `git branch -vv`
- Pull latest changes if behind: `git pull origin "$CURRENT_BRANCH"`
- Force push only if safe: `git push -f origin "$CURRENT_BRANCH"` (use with caution)

**If PR creation fails:**
- Ensure all commits are pushed: `git log origin/"$CURRENT_BRANCH"..HEAD` (should be empty)
- Verify GitHub authentication: `gh auth status`
- Check if PR already exists: `gh pr list --head "$CURRENT_BRANCH"`
- Check repository permissions (need write access)
- Try creating via web interface: `gh pr create --web`

**If PR already exists:**
- View existing PR: `gh pr view`
- Update PR description if needed: `gh pr edit`
- Check if it's an old PR from same branch name

**If commits are not in PR:**
- Verify you pushed to correct branch
- Check PR base branch is correct (should be main)
- Review commit history: `git log main..HEAD`

---

## Success Criteria

✅ All validation checks passed (TypeScript, lint, tests, build)
✅ Temporary files cleaned from working directory
✅ All intended changes staged and reviewed
✅ Descriptive commit message created following conventions
✅ Changes committed successfully
✅ Branch pushed to remote with all commits
✅ Pull request created successfully on GitHub
✅ PR URL provided to user
✅ Task marked as completed in TodoWrite

**Task is INCOMPLETE without successful PR creation.**

---

## Post-Completion

After PR is created:
1. **Share PR URL** with team for review
2. **Monitor CI/CD checks** - Ensure all checks pass
3. **Respond to review comments** promptly
4. **Update PR** if requested changes
5. **Merge** after approval (typically requires 1-2 approvals)
6. **Delete branch** after merge (optional)

---

## Next Command in Workflow

After PR is merged:
- Start a new task: `/start-task <description>`
- Sync with main: `/sync-main`
- Begin next feature implementation

---

## Related Commands

- `/start-task` - Initialize new feature branch
- `/validate` - Run validation checks independently
- `/create-pr` - Create PR without committing
- `/sync-main` - Sync with main branch
