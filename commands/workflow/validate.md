---
description: Run comprehensive validation checks (typecheck, lint, test, build)
allowed-tools: [Bash]
---

Run comprehensive validation checks before committing code:

## Validation Sequence (must pass in order):

### 1. TypeScript Type Check

Execute the type check command:
```bash
npm run typecheck || npx tsc --noEmit
```

**If fails:**
- Read error messages carefully
- Fix type errors incrementally, one at a time
- Never proceed with TypeScript errors
- Use Context7 MCP if library-related type issues occur

### 2. Linting

Execute the linting command:
```bash
npm run lint || npx eslint .
```

**If fails:**
- Fix all linting errors (don't disable rules)
- Try auto-fix first: `npm run lint:fix` or `npx eslint . --fix`
- Address remaining errors manually

### 3. Testing

Execute the test suite:
```bash
npm run test || npm test
```

**If fails:**
- Debug the specific failing test
- Never skip or comment out tests
- Ensure minimum 80% coverage for new code
- Run integration tests if applicable

### 4. Build

Execute the build command:
```bash
npm run build || npm run compile
```

**If fails:**
- Check build error messages
- Verify all imports resolve correctly
- Check for missing dependencies
- For infrastructure issues, consult AWS CDK MCP

### 5. Security Scan (Optional)

Check for security vulnerabilities:
```bash
npm audit
```

**If high/critical issues found:**
- Address vulnerabilities before proceeding
- Update vulnerable dependencies
- Document any accepted risks

## Error Recovery Process:

1. **Incremental Fixes**: Address one error at a time
2. **No Shortcuts**: Never ignore errors or disable checks
3. **Use Context7**: Consult Context7 for dependency-related errors
4. **Rerun After Each Fix**: Validate after each correction
5. **Never Mark Complete**: Task is incomplete if validation fails

## Success Criteria:

✅ TypeScript check: 0 errors
✅ Linting: 0 errors
✅ Testing: All tests passing, 80%+ coverage
✅ Build: Successful build with no errors
✅ Security: No high/critical vulnerabilities

## Validation Status Report:

Provide a clear summary:
- ✅ **PASSED**: All checks successful - ready to commit
- ❌ **FAILED**: List specific failures that need attention

**IMPORTANT:** Do not proceed with /complete-task if validation fails.
