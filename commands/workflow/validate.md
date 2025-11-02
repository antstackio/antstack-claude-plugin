---
description: Run comprehensive validation checks (typecheck, lint, test, build)
allowed-tools: [Bash, TodoWrite]
---

Run comprehensive validation checks before committing code.

**Note:** Execute all commands from your project root directory.

## Before Starting

**Create TodoWrite list** to track validation progress:
- "Run TypeScript type check"
- "Run linting"
- "Execute test suite"
- "Run production build"
- "Run security scan"
- "Generate validation report"

---

## Validation Sequence (Must Pass in Order)

All checks must pass before proceeding with commits. Fix errors immediately after each step.

---

### 1. TypeScript Type Check

Execute the type check command:

```bash
npm run typecheck || npx tsc --noEmit
```

**Expected Output:**
- ✅ No errors: Continue to next check
- ❌ Errors found: Fix immediately (see troubleshooting)

**If fails:**
- Read error messages carefully
- Fix type errors incrementally, one at a time
- **NEVER** proceed with TypeScript errors
- Use `@ts-expect-error` only with detailed justification comment
- Use Context7 MCP if library-related type issues occur

**Common Type Errors:**
- Missing types: Install @types packages
- Implicit any: Add explicit types
- Incompatible types: Check function signatures
- Missing properties: Update interfaces

---

### 2. Linting

Execute the linting command:

```bash
npm run lint || npx eslint .
```

**Expected Output:**
- ✅ 0 errors, 0 warnings: Continue to next check
- ⚠️  Warnings only: Review and fix if possible
- ❌ Errors found: Fix immediately

**If fails:**
- Fix all linting errors (don't disable rules)
- Try auto-fix first:
  ```bash
  npm run lint:fix || npx eslint . --fix
  ```
- Address remaining errors manually
- Review warnings and fix critical ones

**Common Linting Errors:**
- Unused variables: Remove or prefix with underscore
- Missing dependencies in hooks: Add to dependency array
- Console statements: Remove or use proper logging
- Missing return statements: Add returns or void type

---

### 3. Testing

Execute the test suite:

```bash
npm run test || npm test
```

**Expected Output:**
- ✅ All tests passing: Continue to next check
- ❌ Test failures: Debug and fix immediately

**If fails:**
- Debug the specific failing test
- **NEVER** skip or comment out tests
- Ensure minimum 80% coverage for new code
- Run integration tests if applicable

**Testing Best Practices:**
- Run specific test: `npm test -- <test-file>`
- Run with coverage: `npm test -- --coverage`
- Watch mode: `npm test -- --watch`
- Debug mode: `node --inspect-brk node_modules/.bin/jest --runInBand`

**Common Test Failures:**
- Async timing issues: Use proper async/await
- Mock not working: Verify mock setup
- State pollution: Ensure test isolation
- Missing test data: Check fixtures and setup

---

### 4. Build

Execute the build command:

```bash
npm run build || npm run compile
```

**Expected Output:**
- ✅ Build successful: Continue to security scan
- ❌ Build failed: Fix errors immediately

**If fails:**
- Check build error messages
- Verify all imports resolve correctly
- Check for missing dependencies
- For CDK projects: Run `cdk synth` to validate stack
- For infrastructure issues, consult AWS CDK MCP

**Common Build Errors:**
- Module not found: Check import paths
- TypeScript errors: Run typecheck first
- Memory issues: Increase Node memory: `NODE_OPTIONS=--max_old_space_size=4096`
- Circular dependencies: Refactor imports

---

### 5. Security Scan

Check for security vulnerabilities:

```bash
npm audit --audit-level=high
```

**Security Scan Results:**
- ✅ 0 vulnerabilities: **Safe to proceed**
- ⚠️  Low/moderate only: Review and document (acceptable to proceed)
- ❌ High/critical found: **Must fix before proceeding**

**If high/critical issues found:**

```bash
# View detailed vulnerability report
npm audit

# Try automatic fix (may update packages)
npm audit fix

# For breaking changes (use with caution!)
npm audit fix --force

# Manual fix: Update specific package
npm update <package-name>

# If vulnerability is in transitive dependency
# Check if parent package has update available
npm outdated
npm update <parent-package>
```

**Security Scan Actions:**

1. **High/Critical Vulnerabilities:**
   - **MUST FIX** before proceeding
   - Update vulnerable packages
   - If no fix available, consider alternatives
   - Document accepted risks in security review

2. **Moderate Vulnerabilities:**
   - Review severity and exploitability
   - Fix if easily addressable
   - Document decision if accepting risk

3. **Low Vulnerabilities:**
   - Can proceed but note for future fix
   - Address in next maintenance cycle

**Alternative Security Tools:**

```bash
# Snyk (if installed)
npx snyk test

# OWASP Dependency Check
npm install -g dependency-check
dependency-check . --project my-project

# Check for outdated packages
npm outdated
```

---

## Error Recovery Process

Follow this process for any validation failure:

1. **Incremental Fixes**: Address one error at a time
2. **No Shortcuts**: Never ignore errors or disable checks
3. **Use Context7**: Consult Context7 for dependency-related errors
4. **AWS MCP**: Consult AWS CDK MCP for infrastructure issues
5. **Rerun After Each Fix**: Validate after each correction
6. **Never Mark Complete**: Task is incomplete if validation fails

**Recovery Commands:**

```bash
# Clear caches if issues persist
npm cache clean --force
rm -rf node_modules package-lock.json
npm install

# Reset TypeScript build
rm -rf dist/ build/ .tsbuildinfo

# Clear test cache
npm test -- --clearCache

# Verify Node version
node --version  # Should match project requirements
```

---

## Validation Status Report

After all checks, provide a clear summary:

### ✅ ALL PASSED - Ready to Commit

```
Validation Summary:
✅ TypeScript check: 0 errors
✅ Linting: 0 errors, 0 warnings
✅ Testing: All 42 tests passing, 87% coverage
✅ Build: Successful
✅ Security: No high/critical vulnerabilities

Status: READY TO COMMIT
```

### ❌ FAILED - Cannot Proceed

```
Validation Summary:
❌ TypeScript check: 3 errors
✅ Linting: 0 errors
⚠️  Testing: 2 tests failing
❌ Build: Failed
✅ Security: 0 vulnerabilities

Status: BLOCKED - Fix errors before committing

Action Required:
1. Fix 3 TypeScript errors
2. Debug and fix 2 failing tests
3. Resolve build failure
4. Rerun validation
```

---

## Success Criteria

✅ TypeScript check: 0 errors
✅ Linting: 0 errors (warnings acceptable with review)
✅ Testing: All tests passing, 80%+ coverage
✅ Build: Successful build with no errors
✅ Security: No high/critical vulnerabilities

**IMPORTANT:** Do not proceed with `/complete-task` if validation fails.

---

## Integration with Workflow

This command integrates with the development workflow:

1. **During Development**: Run validation before each commit
2. **Before Complete Task**: Run as part of `/complete-task`
3. **Before PR**: Ensure validation passes before creating PR
4. **CI/CD**: Same checks run automatically on GitHub

---

## Continuous Validation

**Best Practices:**

```bash
# Run validation in watch mode during development
npm test -- --watch         # Tests
npm run lint -- --watch     # Linting (if supported)
npx tsc --watch --noEmit   # TypeScript

# Pre-commit hook (if configured)
# Validates automatically before each commit

# Pre-push hook (if configured)
# Runs full validation before pushing
```

---

## Troubleshooting

**If TypeScript check fails:**
- Check `tsconfig.json` configuration
- Verify all @types packages installed
- Use Context7 to check library type definitions
- Check for conflicting type definitions

**If linting fails:**
- Check `.eslintrc` configuration
- Verify ESLint and plugins installed
- Check for conflicting rules
- Use `--debug` flag for more info: `npx eslint . --debug`

**If tests fail:**
- Run single test to isolate: `npm test -- path/to/test.spec.ts`
- Check test environment setup
- Verify mocks and fixtures
- Use `--verbose` for more details

**If build fails:**
- Clear build artifacts: `rm -rf dist/ build/`
- Check webpack/vite/rollup configuration
- Verify all dependencies installed
- Check for circular dependencies

**If security scan fails:**
- Check if vulnerability is exploitable in your context
- Look for available patches or updates
- Consider alternative packages
- Document accepted risks

**If validation is too slow:**
- Run checks in parallel (if supported)
- Use incremental TypeScript: `--incremental`
- Skip tests for quick check: Only run typecheck and lint
- Use cache: `npm test -- --cache`

**If Context7 MCP is needed:**
- Query for library documentation: "Latest API for [library]"
- Check for breaking changes: "[library] migration guide"
- Verify compatibility: "[library] version compatibility"

**If AWS CDK MCP is needed:**
- For CDK syntax errors: "CDK construct [name] usage"
- For deployment issues: "CDK deployment troubleshooting"
- For Nag issues: "CDK Nag rule [rule-id] explanation"

---

## Related Commands

- `/complete-task` - Runs validation before committing
- `/create-pr` - Should validate before creating PR
- `/start-task` - Sets up validation expectations for task

---

## Example Validation Run

```bash
# Full validation sequence
echo "Running validation checks..."

# 1. TypeScript
echo "1/5: Type checking..."
npm run typecheck
# ✅ No errors

# 2. Linting
echo "2/5: Linting..."
npm run lint
# ✅ 0 errors, 0 warnings

# 3. Tests
echo "3/5: Running tests..."
npm test
# ✅ Tests: 42 passed, 42 total
# ✅ Coverage: 87.3%

# 4. Build
echo "4/5: Building..."
npm run build
# ✅ Build completed successfully

# 5. Security
echo "5/5: Security scan..."
npm audit --audit-level=high
# ✅ Found 0 vulnerabilities

echo ""
echo "✅ ALL CHECKS PASSED"
echo "Ready to commit and create PR"
```

---

## Post-Validation Actions

After successful validation:

1. **Commit your changes**: Use `/complete-task`
2. **Create PR**: Use `/create-pr`
3. **Update TodoWrite**: Mark validation task as complete
4. **Document any warnings**: Note any accepted warnings in commit message

After failed validation:

1. **Fix errors incrementally**: One error at a time
2. **Rerun validation**: After each fix
3. **Do NOT commit**: Until all checks pass
4. **Seek help if stuck**: Use Context7, AWS CDK MCP, or ask team
