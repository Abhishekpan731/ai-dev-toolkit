---
name: code-review
description: Perform comprehensive code review for Go/Java code with storage provider quality standards
shortcut: /review
---

# Code Review Checklist

Review the selected code or current file and provide comprehensive analysis covering:

## 1. Code Quality (Functions <50 lines per .ai-config/rules)
- Check function length compliance (<50 lines)
- Identify functions that need refactoring
- Verify @author <AUTHOR_NAME> tags on new functions (Go: Godoc style)

## 2. Error Handling
- Verify all errors are properly wrapped with context
- Check for silent error failures
- Ensure proper error propagation in Go (fmt.Errorf with %w)
- For Java: verify exception handling patterns

## 3. Testing Coverage
- Identify missing test cases
- Verify table-driven tests for Go
- Check test file naming conventions (file_test.go, FileTest.java)
- Suggest edge cases to test

## 4. Documentation
- Verify Godoc/Javadoc comments on public functions
- Check package-level documentation
- Identify missing inline comments for complex logic

## 5. Security & Best Practices
- Check for hardcoded credentials
- Verify input validation
- Look for potential nil pointer panics (Go)
- Check for resource leaks (unclosed connections, file handles)

## 6. Storage Interface Driver Specific (if applicable)
- Verify proper GRPC error codes
- Check idempotency for Storage Interface operations
- Validate volume lifecycle management
- Review mount/unmount cleanup

## 7. Kubernetes Integration
- Verify proper label/annotation usage
- Check resource cleanup on deletion
- Review pod security context handling

Provide specific line numbers and actionable recommendations for each finding.
