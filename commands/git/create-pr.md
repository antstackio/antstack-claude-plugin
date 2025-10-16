---
description: Create a pull request with comprehensive description
allowed-tools: [Bash]
---

Create a pull request for the current feature branch:

## Pre-PR Verification:

Before creating the PR, ensure:
1. ✅ All changes committed and pushed to remote
2. ✅ Validation checks passed (use /validate command)
3. ✅ Temporary files cleaned up
4. ✅ Branch is up to date with main

## PR Creation Process:

### 1. Analyze Changes

Review all changes that will be included in the PR:

```bash
# View all commits in this branch
git -C /app/workspace log --oneline main..HEAD

# View full diff from main branch
git -C /app/workspace diff main...HEAD
```

### 2. Generate PR Summary

Analyze the changes and create a comprehensive summary:

**PR Title Format:** `<type>: <brief description>`
- Types: feat, fix, refactor, docs, test, chore, enhancement

**PR Body Structure:**
```markdown
## Summary
- Bullet point 1: Key change or feature
- Bullet point 2: Important implementation detail
- Bullet point 3: Notable improvement or fix

## Changes Made
- List specific files/components modified
- Highlight architectural decisions
- Note any breaking changes

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests passing
- [ ] Manual testing completed
- [ ] Edge cases verified

## Validation Checklist
- [ ] TypeScript check passed
- [ ] Linting passed
- [ ] All tests passing
- [ ] Build successful

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### 3. Create Pull Request

Execute the GitHub CLI command with HEREDOC for proper formatting:

```bash
gh pr create --cwd /app/workspace \
  --title "PR Title Here" \
  --body "$(cat <<'EOF'
## Summary
- Key change 1
- Key change 2

## Changes Made
- Detailed change description

## Testing
- [ ] Tests completed

## Validation Checklist
- [x] All checks passed

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 4. Output PR URL

After successful creation:
- Extract and display the PR URL from the command output
- Provide the URL to the user
- Confirm task completion

## PR Review Guidelines:

**For Reviewers:**
- Check code quality and adherence to project standards
- Verify test coverage is adequate
- Review security implications
- Ensure documentation is updated
- Validate CI/CD checks pass

## Success Criteria:

✅ PR created successfully on GitHub
✅ PR includes comprehensive description
✅ All validation checks marked as completed
✅ PR URL provided to user
✅ PR is ready for review

## Common Issues:

**No commits to create PR:**
- Verify changes are committed and pushed
- Check you're on the correct branch

**PR creation fails:**
- Verify GitHub authentication (GITHUB_TOKEN)
- Check repository permissions
- Ensure branch is pushed to remote

**Conflicts detected:**
- Sync with main branch first
- Resolve conflicts before creating PR
