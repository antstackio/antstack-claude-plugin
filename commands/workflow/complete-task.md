---
description: Complete task with validation, commit, and pull request creation
allowed-tools: [Bash, SlashCommand]
---

Complete the current development task with proper validation and git workflow:

## Pre-Completion Validation:

1. **Run validation checklist** using the /validate command:
   - Execute: Use the /validate slash command
   - **CRITICAL:** Do not proceed if any validation fails
   - Fix all errors before continuing

2. **Clean temporary files**:
   - Remove: `*.tmp`, `*.temp`, `debug_*`, `test_*`, `progress.md`
   - Execute: `rm -f /app/workspace/{*.tmp,*.temp,debug_*,test_*,progress.md}`
   - Verify: Run `git status` to confirm clean state

## Commit and Push:

3. **Stage all changes**:
   - Execute: `git -C /app/workspace add .`
   - Verify: Check staged changes with `git -C /app/workspace status`

4. **Create descriptive commit message**:
   - Analyze the changes made
   - Format: `<type>: <description>`
   - Types: feat, fix, refactor, docs, test, chore
   - Include: `🤖 Generated with [Claude Code](https://claude.com/claude-code)`
   - Include: `Co-Authored-By: Claude <noreply@anthropic.com>`

5. **Commit changes**:
   ```bash
   git -C /app/workspace commit -m "$(cat <<'EOF'
   <commit-message>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"
   ```

6. **Push to remote**:
   - Execute: `git -C /app/workspace push origin <current-branch>`
   - Verify: Confirm all commits pushed successfully

## Pull Request Creation:

7. **Create pull request** using the /create-pr command:
   - Execute: Use the /create-pr slash command
   - Ensure PR includes comprehensive description
   - Verify: PR URL is returned

8. **Provide PR URL to user**:
   - Output the complete PR URL
   - Confirm task completion

## Success Criteria:

✅ All validation checks passed
✅ Temporary files cleaned
✅ Changes committed with descriptive message
✅ Branch pushed to remote
✅ Pull request created successfully
✅ PR URL provided to user

**Task is INCOMPLETE without successful PR creation.**
