# Code Review Skill

**Purpose**: Perform thorough code review of changes with security, quality, and best practices analysis.

## Workflow

### 1. Understand the Changes
- Read the git diff or modified files
- Identify what changed and why
- Understand the context and purpose

### 2. Check Code Quality
- **Function length**: Any functions > 50 lines?
- **Complexity**: High cyclomatic complexity?
- **Naming**: Clear, descriptive names?
- **DRY**: Any code duplication?
- **Comments**: Are they necessary and up-to-date?
- **Error handling**: Proper error handling and logging?

### 3. Security Review
- **Input validation**: All user inputs validated?
- **SQL injection**: Using parameterized queries?
- **XSS prevention**: Proper output escaping?
- **Authentication/Authorization**: Proper checks in place?
- **Secrets**: No hardcoded credentials?
- **Logging**: No sensitive data in logs?

### 4. Testing Review
- **Test coverage**: Are there tests for the changes?
- **Edge cases**: Are edge cases covered?
- **Test quality**: Are tests meaningful and well-named?
- **Mocking**: Are external dependencies mocked?

### 5. Performance Review
- **N+1 queries**: Any database query issues?
- **Inefficient algorithms**: Big-O concerns?
- **Resource leaks**: Proper cleanup?
- **Caching**: Should results be cached?

### 6. Best Practices
- **Following conventions**: Consistent with codebase style?
- **SOLID principles**: Single responsibility, etc.?
- **Language idioms**: Using language best practices?
- **Dependencies**: Any new dependencies necessary?

### 7. Generate Review Report
Provide a structured review with:
- **Summary**: Overall assessment
- **Critical Issues**: Must fix before merge
- **Suggestions**: Nice-to-have improvements
- **Praise**: What's done well
- **Security Concerns**: Any security issues found
- **Testing Gaps**: Missing test coverage

## Output Format

```markdown
# Code Review Report

## Summary
[Overall assessment of the changes]

## Critical Issues ❌
- [ ] Issue 1
- [ ] Issue 2

## Security Concerns 🔒
- [ ] Concern 1
- [ ] Concern 2

## Suggestions 💡
- Suggestion 1
- Suggestion 2

## What's Good ✅
- Good thing 1
- Good thing 2

## Testing Gaps 🧪
- Missing test 1
- Missing test 2

## Overall Recommendation
[APPROVE / REQUEST CHANGES / COMMENT]
```

## Usage
Invoke this skill when reviewing code changes before committing, creating PRs, or during code review process.

## Example Invocation
- "Review the changes I just made"
- "Can you review this PR?"
- "Check if my code follows best practices"
