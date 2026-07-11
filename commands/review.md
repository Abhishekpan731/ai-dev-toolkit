# /review Command

## Description
Perform a comprehensive code review of recent changes or specified files.

## Usage
```
/review [files]
```

## Examples
- `/review` - Review all uncommitted changes
- `/review src/auth/login.ts` - Review specific file
- `/review src/` - Review all files in directory

## What It Does

### 1. Identify Changes
- Check git status for modified files
- Or review specified files
- Show what will be reviewed

### 2. Run Code Review Skill
Invoke the code-review skill with comprehensive analysis:
- Code quality check
- Security review
- Testing coverage
- Performance concerns
- Best practices adherence

### 3. Generate Report
Create a structured review report with:
- Critical issues (must fix)
- Security concerns
- Suggestions for improvement
- What's done well
- Testing gaps

### 4. Provide Actionable Feedback
- Specific line numbers when possible
- Code examples for fixes
- Links to documentation
- Priority levels (Critical, High, Medium, Low)

## Review Checklist

✅ **Code Quality**
- Function length < 50 lines
- Clear naming conventions
- No code duplication
- Proper error handling
- Updated comments

✅ **Security**
- No hardcoded secrets
- Input validation present
- SQL injection prevention
- XSS prevention
- Proper authentication/authorization

✅ **Testing**
- Tests exist for changes
- Edge cases covered
- Tests are meaningful
- No skipped tests

✅ **Performance**
- No N+1 query problems
- Efficient algorithms
- Proper resource cleanup
- Caching where appropriate

✅ **Best Practices**
- Follows project conventions
- SOLID principles
- Language idioms
- Minimal dependencies

## Output Format

```markdown
# Code Review: [File/Feature Name]

## Summary
[Brief overview of changes and overall assessment]

## Critical Issues ❌ (Must Fix)
1. **[File:Line]** - [Issue description]
   - Impact: [What could go wrong]
   - Fix: [How to fix it]
   
## Security Concerns 🔒
1. **[File:Line]** - [Security issue]
   - Risk: [Potential security impact]
   - Recommendation: [How to secure]

## Suggestions 💡 (Nice to Have)
1. [Improvement suggestion]
2. [Refactoring opportunity]

## What's Good ✅
- [Something done well]
- [Good practice used]

## Testing Gaps 🧪
- [ ] Missing test for [scenario]
- [ ] Edge case not covered: [case]

## Overall Recommendation
[APPROVE ✅ / REQUEST CHANGES ❌ / COMMENT 💬]

## Next Steps
1. [First thing to fix]
2. [Second thing to address]
```

## Integration
This command automatically invokes the `code-review` skill and formats the output.
