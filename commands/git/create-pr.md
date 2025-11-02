---
description: Create a pull request with comprehensive description
allowed-tools: [Bash, TodoWrite, SlashCommand]
---

Create a pull request for the current feature branch with a comprehensive description.

**Note:** Execute all commands from your project root directory.

## Before Starting

1. **Create TodoWrite list** to track PR creation:
   - "Verify all changes committed and pushed"
   - "Run validation checks"
   - "Analyze commits and changes"
   - "Generate PR summary"
   - "Create PR with gh CLI"
   - "Verify PR created and get URL"

2. **Verify you're on a feature branch**:
   ```bash
   CURRENT_BRANCH=$(git branch --show-current)
   echo "Creating PR for branch: $CURRENT_BRANCH"

   if [ "$CURRENT_BRANCH" = "main" ] || [ "$CURRENT_BRANCH" = "master" ]; then
     echo "❌ Error: Cannot create PR from main branch"
     exit 1
   fi
   ```

---

## Pre-PR Verification

Before creating the PR, ensure all prerequisites are met:

### 1. Check All Changes Are Committed

```bash
# Check working tree status
if git diff-index --quiet HEAD --; then
  echo "✅ Working tree is clean"
else
  echo "❌ You have uncommitted changes"
  git status --short
  exit 1
fi
```

### 2. Verify Changes Are Pushed

```bash
# Check if local and remote are in sync
LOCAL_COMMIT=$(git rev-parse HEAD)
REMOTE_COMMIT=$(git rev-parse origin/"$CURRENT_BRANCH" 2>/dev/null)

if [ "$LOCAL_COMMIT" = "$REMOTE_COMMIT" ]; then
  echo "✅ All commits pushed to remote"
else
  echo "⚠️  Local commits not pushed to remote"
  echo "Pushing now..."
  git push origin "$CURRENT_BRANCH"
fi
```

### 3. Run Validation Checks

**Execute `/validate` slash command** to ensure all checks pass:

```bash
# The /validate command will run:
# - TypeScript type check: npm run typecheck || npx tsc --noEmit
# - Linting: npm run lint || npx eslint .
# - Tests: npm run test || npm test
# - Build: npm run build || npm run compile
```

**CRITICAL:** Do not create PR if validation fails. Fix all errors first.

### 4. Verify Branch is Up to Date

```bash
# Check if branch is behind main
git fetch origin main

BEHIND_COUNT=$(git rev-list --count HEAD..origin/main)

if [ "$BEHIND_COUNT" -eq 0 ]; then
  echo "✅ Branch is up to date with main"
else
  echo "⚠️  Branch is $BEHIND_COUNT commits behind main"
  echo "Consider syncing with main first"
fi
```

---

## Step 1: Analyze Changes for PR Description

Review all changes that will be included in the PR:

```bash
# View all commits in this branch (since diverging from main)
echo "Commits in this PR:"
git log --oneline main..HEAD

# Count commits
COMMIT_COUNT=$(git rev-list --count main..HEAD)
echo "Total commits: $COMMIT_COUNT"

# View full diff from main branch
echo "Full diff from main:"
git diff main...HEAD --stat

# Get list of changed files
echo "Changed files:"
git diff --name-status main...HEAD
```

**Review the output carefully** to understand what will be in the PR.

---

## Step 2: Generate Comprehensive PR Summary

Based on the changes analyzed, create a PR description with this structure:

### PR Title Format

`<type>: <brief description>`

**Types:**
- `feat`: New feature or functionality
- `fix`: Bug fix
- `refactor`: Code restructuring without behavior change
- `docs`: Documentation updates
- `test`: Test additions or updates
- `chore`: Maintenance tasks
- `perf`: Performance improvements
- `enhancement`: Improvement to existing feature

**Title Examples:**
- `feat: Add user authentication system`
- `fix: Resolve race condition in data fetching`
- `refactor: Restructure API endpoint handlers`

### PR Body Structure

```markdown
## Summary
- [3-5 bullet points summarizing key changes]
- Focus on WHAT changed and WHY

## Changes Made
- [Detailed list of specific changes]
- Include file/component names
- Highlight architectural decisions
- Note any breaking changes

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests passing
- [ ] Manual testing completed
- [ ] Edge cases verified
- [ ] Performance tested (if applicable)

## Validation Checklist
- [x] TypeScript check passed
- [x] Linting passed
- [x] All tests passing
- [x] Build successful
- [x] No security vulnerabilities

## Related Issues
- Closes #[issue-number] (if applicable)
- Related to #[issue-number] (if applicable)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

---

## Step 3: Create Pull Request

Execute the GitHub CLI command with HEREDOC for proper formatting:

```bash
# Set PR title and body variables
PR_TITLE="feat: your descriptive title here"

# Create PR with comprehensive body
gh pr create \
  --title "$PR_TITLE" \
  --body "$(cat <<'EOF'
## Summary
- Key change 1: Brief description
- Key change 2: Brief description
- Key change 3: Brief description

## Changes Made
### Frontend
- Updated UserProfile component with new fields
- Added form validation for email input
- Integrated with authentication context

### Backend
- Created /api/users/profile endpoint
- Added database migration for user_profiles table
- Implemented input sanitization

### Testing
- Added 15 unit tests for profile component
- Created integration tests for profile API
- Tested with various edge cases

## Testing
- [x] Unit tests added/updated (15 new tests)
- [x] Integration tests passing (all 23 tests)
- [x] Manual testing completed
- [x] Edge cases verified (empty profiles, invalid data)
- [x] Performance tested (sub-100ms response time)

## Validation Checklist
- [x] TypeScript check passed (0 errors)
- [x] Linting passed (0 warnings)
- [x] All tests passing (38/38 tests)
- [x] Build successful
- [x] No high/critical security vulnerabilities

## Related Issues
- Closes #123

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**Important**: Replace the placeholder content with actual PR details based on your changes.

---

## Step 4: Verify PR Creation and Retrieve URL

Confirm the PR was created successfully:

```bash
# Get the most recent PR for current branch
CURRENT_BRANCH=$(git branch --show-current)
PR_URL=$(gh pr list --head "$CURRENT_BRANCH" --limit 1 --json url --jq '.[0].url')

if [ -n "$PR_URL" ]; then
  echo "✅ Pull Request created successfully"
  echo ""
  echo "PR URL: $PR_URL"
  echo ""

  # Get PR number
  PR_NUMBER=$(gh pr list --head "$CURRENT_BRANCH" --limit 1 --json number --jq '.[0].number')
  echo "PR Number: #$PR_NUMBER"

  # Show PR status
  gh pr view "$PR_NUMBER" --json title,state,checks --jq '{title,state,checks: .checks[].name}'
else
  echo "❌ Could not retrieve PR URL"
  echo "Listing all PRs:"
  gh pr list
fi
```

**Alternative - Open PR in browser:**
```bash
gh pr view --web
```

**Provide the PR URL to the user** in your response.

---

## Step 5: Monitor PR Status

After creating the PR, monitor its status:

```bash
# Check PR checks status
gh pr checks

# View PR details
gh pr view

# Check if PR is ready to merge
gh pr view --json mergeable,mergeStateStatus --jq '{mergeable,status: .mergeStateStatus}'
```

---

## Troubleshooting

**If "no commits" error occurs:**
- Verify you're on correct branch: `git branch --show-current`
- Check commits exist: `git log main..HEAD` (should show commits)
- Ensure branch diverged from main: `git merge-base main HEAD`
- Push commits if missing: `git push origin $(git branch --show-current)`

**If GitHub authentication fails:**
- Check auth status: `gh auth status`
- View current token: `gh auth token`
- Re-authenticate: `gh auth login --web`
- Verify scopes: Token needs `repo` and `read:org` scopes

**If PR creation fails with "already exists":**
- List existing PRs: `gh pr list --head $(git branch --show-current)`
- View existing PR: `gh pr view`
- Close old PR if it's stale: `gh pr close <number>`
- Or update existing PR: `gh pr edit <number>`

**If PR body format is broken:**
- Verify HEREDOC syntax uses single quotes: `<<'EOF'` not `<<EOF`
- Check for unescaped special characters in body
- Test heredoc separately: `cat <<'EOF' ... EOF`
- Use --body-file alternative: `gh pr create --body-file pr-body.md`

**If push is rejected:**
- Pull latest changes: `git pull origin $(git branch --show-current)`
- Check for conflicts: `git status`
- Resolve conflicts if any, then commit
- Push again: `git push origin $(git branch --show-current)`
- Force push only if safe and necessary: `git push -f`

**If wrong base branch:**
- Check PR base: `gh pr view --json baseRefName --jq .baseRefName`
- Edit PR to change base: `gh pr edit --base main`
- Or close and recreate with correct base

**If validation checks fail on GitHub:**
- View failed checks: `gh pr checks`
- Check logs: `gh pr checks --watch`
- Fix issues and push new commit
- Checks will re-run automatically

**If PR is blocked by required reviews:**
- Request reviews: `gh pr edit --add-reviewer <username>`
- Check required reviewers: `gh pr view --json reviewRequests`
- Notify reviewers via comments or Slack

**If CI/CD pipeline fails:**
- View GitHub Actions runs: `gh run list --branch $(git branch --show-current)`
- Check specific run: `gh run view <run-id>`
- View logs: `gh run view <run-id> --log`
- Re-run failed jobs: `gh run rerun <run-id> --failed`

---

## Success Criteria

✅ All changes committed to feature branch
✅ All commits pushed to remote
✅ Validation checks passed (TypeScript, lint, tests, build)
✅ Branch up to date with main (or conflicts noted)
✅ PR created successfully on GitHub
✅ PR includes comprehensive description
✅ All validation checks/CI marked in PR body
✅ PR URL provided to user
✅ PR is ready for review

---

## PR Review Guidelines

**For PR Authors:**
- Respond to review comments promptly
- Make requested changes in new commits (don't force push)
- Re-request review after addressing feedback
- Keep PR scope focused (one feature/fix per PR)

**For Reviewers:**
- Check code quality and adherence to standards
- Verify test coverage is adequate (80%+ for new code)
- Review security implications (input validation, auth, etc.)
- Ensure documentation is updated
- Validate CI/CD checks pass before approving

---

## Post-PR Actions

After PR is created:

1. **Monitor CI/CD**: Ensure all automated checks pass
2. **Request Reviews**: Tag appropriate reviewers
3. **Respond to Feedback**: Address comments promptly
4. **Update if Needed**: Push additional commits for requested changes
5. **Merge**: After approval (usually requires 1-2 approvals)
6. **Cleanup**: Delete feature branch after merge (optional)

---

## Related Commands

- `/start-task` - Initialize new feature branch before coding
- `/validate` - Run validation checks before creating PR
- `/complete-task` - Complete task with commit and PR creation
- `/sync-main` - Sync branch with latest main before PR

---

## Example PR Creation Flow

```bash
# 1. Verify on feature branch
git branch --show-current
# Output: feature/add-user-profile

# 2. Check working tree is clean
git status
# Output: nothing to commit, working tree clean

# 3. Verify pushed to remote
git push origin feature/add-user-profile

# 4. Run validation
/validate
# All checks pass ✅

# 5. Analyze changes
git log --oneline main..HEAD
git diff main...HEAD --stat

# 6. Create PR
gh pr create \
  --title "feat: Add user profile page" \
  --body "$(cat <<'EOF'
## Summary
- Added user profile page with editable fields
- Integrated with existing authentication system
- Added comprehensive test coverage

## Changes Made
### Frontend
- Created UserProfile component (src/components/UserProfile.tsx)
- Added profile editing functionality
- Implemented real-time validation

### Backend
- Added GET /api/users/profile endpoint
- Added PUT /api/users/profile endpoint
- Created database migration for profile fields

### Testing
- Added 12 unit tests for UserProfile component
- Added 8 integration tests for profile API
- All tests passing

## Testing
- [x] Unit tests added/updated (12 new tests)
- [x] Integration tests passing (all 20 tests)
- [x] Manual testing completed
- [x] Edge cases verified

## Validation Checklist
- [x] TypeScript check passed
- [x] Linting passed
- [x] All tests passing
- [x] Build successful

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"

# 7. Get PR URL
gh pr list --head feature/add-user-profile --json url --jq '.[0].url'
# Output: https://github.com/user/repo/pull/42

# 8. Share PR URL with team
echo "PR created: https://github.com/user/repo/pull/42"
```
