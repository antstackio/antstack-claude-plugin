---
description: Initialize a new task with proper git workflow and branch creation
argument-hint: <task-description>
allowed-tools: [Bash, TodoWrite]
---

Initialize a new development task with proper git workflow:

**Task Description:** $ARGUMENTS

## Workflow Steps:

1. **Sync with main branch** to ensure you're working with the latest code:
   - Execute: `gh repo sync --cwd /app/workspace`
   - Verify: Check for "Already up to date" or successful merge message

2. **Generate feature branch name** from task description:
   - Analyze the task to determine type: feature/, bugfix/, or enhancement/
   - Convert task description to kebab-case branch name
   - Example: "Add user authentication" → "feature/add-user-authentication"

3. **Create and checkout feature branch**:
   - Execute: `git -C /app/workspace checkout -b <branch-name>`
   - Verify: Confirm "Switched to a new branch" message

4. **Push branch to remote immediately**:
   - Execute: `git -C /app/workspace push origin <branch-name>`
   - Verify: Branch is visible on GitHub remote

5. **Create TodoWrite list** with task breakdown:
   - Break down the task into measurable subtasks
   - Include completion criteria for each subtask
   - Add time estimates where applicable

## Success Criteria:

✅ Main branch is synced
✅ Feature branch created with appropriate naming convention
✅ Branch pushed to remote
✅ TodoWrite list created with task breakdown
✅ Ready to begin implementation

**IMPORTANT:** Do not proceed with implementation until all steps complete successfully.
