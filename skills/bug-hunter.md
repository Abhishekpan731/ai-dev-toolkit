# Bug Hunter Skill

**Purpose**: Systematically debug issues using a structured approach.

## Debugging Workflow

### 1. Reproduce the Bug
- Understand the expected behavior
- Understand the actual behavior
- Identify steps to reproduce
- Confirm the bug is reproducible

### 2. Gather Information
- Read error messages carefully
- Check logs (application, server, database)
- Examine stack traces
- Check recent changes (git history)
- Identify affected components

### 3. Form Hypotheses
Based on symptoms, generate possible causes:
- Configuration issues
- Code logic errors
- Race conditions
- Missing error handling
- Dependency issues
- Environment differences

### 4. Test Hypotheses
For each hypothesis:
- Add logging/debugging statements
- Use debugger breakpoints
- Write a failing test
- Check related code paths
- Verify assumptions

### 5. Isolate the Root Cause
- Binary search through code
- Comment out suspicious sections
- Test with minimal reproduction
- Check input/output at each step
- Verify data flow

### 6. Fix the Bug
- Implement the minimal fix
- Ensure fix doesn't break other functionality
- Add error handling if missing
- Update related code if needed

### 7. Write Tests
- Write a failing test that reproduces the bug
- Verify the test passes with the fix
- Add edge case tests
- Ensure regression tests exist

### 8. Verify the Fix
- Run all tests
- Test in development environment
- Check for similar bugs elsewhere
- Review code for related issues

### 9. Document
- Update comments if needed
- Add to changelog
- Document workarounds if any
- Note any technical debt created

## Debugging Techniques

### For Runtime Errors
1. Read the full error message
2. Check the stack trace
3. Identify the failing line
4. Examine variables at that point
5. Work backwards to find root cause

### For Logic Errors
1. Add print/log statements
2. Verify input values
3. Check intermediate calculations
4. Verify output values
5. Compare with expected behavior

### For Performance Issues
1. Profile the application
2. Identify bottlenecks
3. Check database queries (N+1 problem?)
4. Look for inefficient algorithms
5. Check for memory leaks

### For Intermittent Bugs
1. Check for race conditions
2. Look for timing-dependent code
3. Examine shared state
4. Check for external dependencies
5. Add extensive logging

## Common Bug Patterns

### Null/Undefined Errors
- Check for null/undefined before accessing
- Use optional chaining or safe navigation
- Provide default values

### Off-by-One Errors
- Check loop conditions
- Verify array indices
- Check boundary conditions

### Type Errors
- Verify data types at boundaries
- Use type checking/validation
- Check for implicit conversions

### Async Issues
- Check for race conditions
- Verify promise handling
- Check for missing await
- Verify callback execution order

## Bug Report Template

```markdown
# Bug Report

## Description
[What's broken?]

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Behavior
[What should happen?]

## Actual Behavior
[What actually happens?]

## Error Messages
```
[Paste error messages]
```

## Root Cause
[What caused the bug?]

## Fix
[How was it fixed?]

## Tests Added
- Test 1
- Test 2

## Related Issues
[Link to related bugs/issues]
```

## Usage
Invoke when debugging issues, investigating errors, or hunting down elusive bugs.

## Example Invocation
- "Help me debug this error"
- "Why is this function not working?"
- "Find the root cause of this issue"
