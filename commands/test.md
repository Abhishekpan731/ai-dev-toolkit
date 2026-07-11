---
title: "Test Command - Comprehensive Test Suite Generation"
description: "Generate complete test suites with unit, integration, and edge case tests for any code target. Achieves 80%+ coverage with happy path, error cases, and mocking strategies."
version: 2.0.0
author: <AUTHOR_NAME>
last_updated: 2026-07-03
status: production
category: testing
tags:
  - testing
  - test-generation
  - unit-tests
  - integration-tests
  - test-coverage
  - tdd
  - test-driven-development
keywords:
  - generate tests
  - write test cases
  - test suite
  - test coverage
  - unit testing
  - integration testing
  - mocking
  - test automation
search_keywords:
  - how to write tests
  - generate test cases
  - test coverage 80 percent
  - unit test examples
  - integration test examples
  - mock dependencies
  - test-driven development
related_commands:
  - /fix
  - /review
  - /code-review
  - /test-plan
related_skills:
  - test-generator.md
  - code-review.md
  - go-best-practices.md
related_rules:
  - CORE-ENGINEERING-RULES.md
dependencies:
  - Test framework (Jest, Mocha, JUnit, pytest, go test)
  - Mocking library (optional)
  - Coverage tool (optional)
inputs:
  - Target (file path, class name, or function name)
  - Existing code to test
  - Dependencies to mock
outputs:
  - Complete test file(s)
  - Test data factories
  - Mock configurations
  - Coverage report
  - Test execution instructions
platform:
  - All (language-agnostic)
  - Supports: JavaScript/TypeScript (Jest), Python (pytest), Go (testing), Java (JUnit)
audience:
  - developers
  - QA engineers
  - test automation engineers
ai_optimized: true
human_friendly: true
shortcut: /test
---

# Test Command - Comprehensive Test Suite Generation

> **TL;DR**: Generate complete test suites with 80%+ coverage automatically. Creates unit tests, integration tests, edge cases, error handling tests, mocking strategies, and test data factories for any code target.

## Purpose

The `/test` command provides **automated, comprehensive test generation** for any code target. It ensures:

- **High test coverage** (80%+ target)
- **Multiple test categories** (happy path, edge cases, errors)
- **Proper test structure** (setup, teardown, isolation)
- **Mocking strategies** (external dependencies isolated)
- **Best practices** (AAA pattern, descriptive names)

## When to Use

### ✅ Use `/test` For

- Generating tests for new code (TDD approach)
- Adding tests to legacy code (improving coverage)
- Creating comprehensive test suites quickly
- Testing complex logic (multiple edge cases)
- Integration testing (database, API, external services)
- Regression testing (prevent bugs from returning)

### ❌ Don't Use `/test` For

- Manual exploratory testing (human judgment required)
- Performance benchmarking (use dedicated tools)
- Security testing (use security scanners)
- End-to-end UI testing (use Selenium, Cypress)
- Load testing (use k6, JMeter, Gatling)

## Usage

```bash
# Generate tests for file
/test src/auth/login.ts

# Generate tests for specific class
/test UserService

# Generate tests for specific function
/test createUser

# Generate tests with coverage target
/test src/utils/helpers.go --coverage=90
```

## Examples

- `/test src/auth/login.ts` - Generate tests for file
- `/test UserService` - Generate tests for class
- `/test createUser` - Generate tests for function

## What It Does

### 1. Analyze Target

- Identify what needs testing
- Understand inputs/outputs
- Find dependencies
- Note edge cases
- Check existing tests

### 2. Generate Test Plan

Create test cases for:

- Happy path scenarios
- Edge cases
- Error cases
- Integration points
- Performance scenarios

### 3. Create Test Suite

Generate:

- Test file structure
- Setup/teardown code
- Mock configurations
- Test data factories
- Individual test cases

### 4. Ensure Coverage

- Test all code paths
- Cover edge cases
- Test error handling
- Verify boundaries
- Check for regressions

## Test Case Categories

### Happy Path

```javascript
test("should create user with valid data", async () => {
  const result = await createUser(validUserData);
  expect(result).toHaveProperty("id");
  expect(result.email).toBe(validUserData.email);
});
```

### Edge Cases

```javascript
test("should handle empty input", () => {
  expect(() => processData("")).toThrow();
});

test("should return null when user not found", async () => {
  const user = await getUser("nonexistent-id");
  expect(user).toBeNull();
});
```

### Error Cases

```javascript
test("should throw error for invalid email", async () => {
  await expect(createUser({ email: "invalid" })).rejects.toThrow(
    "Invalid email format",
  );
});
```

### Integration

```javascript
test("should save user to database", async () => {
  const user = await createUser(validUserData);
  const saved = await db.users.findById(user.id);
  expect(saved).toMatchObject(validUserData);
});
```

## Test Structure

```javascript
describe("TargetName", () => {
  // Setup
  beforeEach(() => {
    // Reset state, create mocks
  });

  // Teardown
  afterEach(() => {
    // Clean up resources
  });

  // Test groups
  describe("method or feature", () => {
    test("should do X when Y", () => {
      // Arrange
      const input = setupTestData();

      // Act
      const result = targetFunction(input);

      // Assert
      expect(result).toBe(expected);
    });
  });
});
```

## Output Format

````markdown
# Test Suite Generated for: [Target]

## Test File Created

`path/to/target.test.ext`

## Test Coverage

- **Happy path**: X tests
- **Edge cases**: Y tests
- **Error cases**: Z tests
- **Total tests**: N tests

## Test Cases

### Happy Path

- ✅ should [expected behavior]
- ✅ should [expected behavior]

### Edge Cases

- ✅ should handle empty input
- ✅ should handle null values
- ✅ should handle boundary conditions

### Error Cases

- ✅ should throw on invalid input
- ✅ should handle database errors
- ✅ should handle network failures

## Mocks Created

- Database service
- API client
- File system
- External services

## Test Data Factories

- `createTestUser()`
- `createTestOrder()`
- `generateTestData()`

## Next Steps

1. Review generated tests
2. Add any missing edge cases
3. Run tests: `[test command]`
4. Check coverage: `[coverage command]`

## Run Tests

```bash
npm test              # Run all tests
npm test -- --watch   # Watch mode
npm run test:coverage # With coverage
```
````

```

## Integration
This command automatically invokes the `test-generator` skill.
```
