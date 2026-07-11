# Test Generator Skill

**Purpose**: Generate comprehensive test suites for functions, classes, and modules.

## Test Generation Workflow

### 1. Analyze the Code
- Identify the function/class/module to test
- Understand inputs and outputs
- Identify dependencies
- Understand business logic
- Note edge cases

### 2. Identify Test Cases
Generate tests for:
- **Happy path**: Normal, expected inputs
- **Edge cases**: Boundary values, empty inputs, null/undefined
- **Error cases**: Invalid inputs, exceptions, failures
- **Integration**: Interaction with dependencies
- **Performance**: Large inputs, stress testing (if relevant)

### 3. Choose Test Framework
Based on language:
- **JavaScript/TypeScript**: Jest, Vitest, Mocha
- **Python**: pytest, unittest
- **Go**: testing package
- **Rust**: cargo test
- **Java**: JUnit

### 4. Generate Test Structure
Follow AAA pattern:
- **Arrange**: Set up test data and mocks
- **Act**: Execute the code being tested
- **Assert**: Verify the result

### 5. Generate Mocks
Mock external dependencies:
- Database calls
- API requests
- File system operations
- Third-party services
- Time-dependent functions

### 6. Write Test Cases

#### Happy Path Tests
```javascript
test('should create user with valid data', async () => {
  const userData = { email: 'test@example.com', name: 'Test User' };
  const user = await createUser(userData);
  expect(user.email).toBe('test@example.com');
  expect(user.name).toBe('Test User');
});
```

#### Edge Case Tests
```javascript
test('should return empty array when no users exist', async () => {
  const users = await getAllUsers();
  expect(users).toEqual([]);
});

test('should handle null input gracefully', () => {
  expect(() => processData(null)).toThrow('Input cannot be null');
});
```

#### Error Case Tests
```javascript
test('should throw error when email is invalid', async () => {
  const userData = { email: 'invalid-email', name: 'Test' };
  await expect(createUser(userData)).rejects.toThrow('Invalid email');
});
```

### 7. Add Test Data Factories
Create reusable test data:
```javascript
function createTestUser(overrides = {}) {
  return {
    id: '123',
    email: 'test@example.com',
    name: 'Test User',
    createdAt: new Date('2024-01-01'),
    ...overrides
  };
}
```

### 8. Verify Coverage
- Run coverage report
- Ensure all branches are tested
- Check for untested edge cases
- Add missing tests

## Test Categories

### Unit Tests
- Test individual functions in isolation
- Mock all dependencies
- Fast execution
- Focus on logic

### Integration Tests
- Test component interactions
- May use real database (test DB)
- Test critical workflows
- Slower than unit tests

### Snapshot Tests
- Test component rendering (React, Vue)
- Capture output and compare
- Detect unintended changes

### Property-Based Tests
- Generate random inputs
- Verify properties hold
- Find edge cases automatically

## Test Naming Conventions

```javascript
// Pattern: should_<expected>_when_<condition>
test('should return 404 when user not found', () => {});
test('should calculate discount when user is premium', () => {});
test('should throw error when input is invalid', () => {});

// Pattern: test_<what>_<when>_<then>
test('getUserById returns user when valid ID provided', () => {});
test('createOrder fails when inventory is insufficient', () => {});
```

## Mock Examples

### JavaScript (Jest)
```javascript
const mockDb = {
  users: {
    findById: jest.fn(),
    create: jest.fn()
  }
};

test('should fetch user from database', async () => {
  mockDb.users.findById.mockResolvedValue({ id: '123', name: 'Test' });
  const user = await userService.getUser('123');
  expect(mockDb.users.findById).toHaveBeenCalledWith('123');
  expect(user.name).toBe('Test');
});
```

### Python (pytest)
```python
from unittest.mock import Mock, patch

def test_sends_email_on_signup(mocker):
    email_service = mocker.Mock()
    user_service = UserService(email_service)
    
    user_service.signup('test@example.com')
    
    email_service.send_welcome_email.assert_called_once_with('test@example.com')
```

## Output Structure

Generate test files with:
1. **Imports**: Test framework, code under test, mocks
2. **Setup/Teardown**: beforeEach, afterEach hooks
3. **Test suites**: Grouped by functionality
4. **Individual tests**: Clear, focused tests
5. **Helper functions**: Reusable test utilities

## Usage
Invoke when writing tests for new code or improving test coverage.

## Example Invocation
- "Generate tests for the UserService class"
- "Write tests for this authentication function"
- "Create test cases for edge cases"
