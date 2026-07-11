---
mode: auto
name: ai-assisted-development
description: Best practices for AI pair programming and code generation
tags: [ai, development, coding, pair-programming, code-generation]
version: 1.0.0
---

# AI-Assisted Development

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-07

Best practices and patterns for effective AI-assisted software development.

## Core Principles

### 1. AI as Pair Programmer
- **Human**: Architect, design, review
- **AI**: Implement, test, document
- **Collaboration**: Iterative refinement

### 2. Verify Everything
- AI-generated code is a **draft**, not final
- Always review before committing
- Run tests to validate functionality
- Check for security vulnerabilities

### 3. Incremental Development
- Break tasks into small chunks
- Generate code incrementally
- Validate each step before proceeding

## Effective AI Prompting

### Be Specific
```markdown
❌ BAD: "Write authentication code"

✅ GOOD: "Write a Go function AuthenticateUser(username, password string) 
that validates credentials against PostgreSQL, uses bcrypt for password 
hashing, returns User struct on success, and appropriate error types 
(ErrInvalidCredentials, ErrUserNotFound, ErrDatabaseError). Follow 
existing error handling patterns in auth_service.go."
```

### Provide Context
```markdown
✅ Include in every prompt:
- Current project/tech stack
- Related files and functions
- Existing patterns to follow
- Constraints (line limits, dependencies)
```

### Request Examples
```markdown
✅ "Include usage example showing typical call with error handling"
✅ "Generate test cases covering success and error scenarios"
✅ "Provide example configuration for production deployment"
```

## AI-Generated Code Workflow

### Step 1: Generate
```bash
# Use AI to generate initial implementation
"Implement CreateVolume function in pkg/driver/controller.go following 
Storage Interface spec. Include validation, idempotency check, and error handling."
```

### Step 2: Review
```markdown
**Review Checklist**:
- [ ] Function length <50 lines
- [ ] @author <AUTHOR_NAME> present
- [ ] Error handling complete
- [ ] Input validation included
- [ ] Follows existing patterns
- [ ] No hardcoded values
- [ ] Comments explain complex logic
```

### Step 3: Test
```go
// AI generates tests alongside implementation
func TestCreateVolume_Success(t *testing.T) {
    // Arrange
    driver := NewFakeDriver()
    req := &csi.CreateVolumeRequest{...}
    
    // Act
    resp, err := driver.CreateVolume(context.Background(), req)
    
    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, resp)
}
```

### Step 4: Refine
```markdown
# Iteratively improve based on review
"Refactor CreateVolume to extract validation into separate function. 
Each function should be <50 lines."
```

### Step 5: Document
```go
// AI generates documentation
// CreateVolume creates a new persistent volume for the storage interface.
// It validates the request, provisions the volume on the distributed filesystem,
// and returns volume metadata.
//
// @author <AUTHOR_NAME>
// ...
```

## Common Patterns

### Pattern 1: Interface First
```markdown
1. "Define interface for [component]"
2. "Generate mock implementation for testing"
3. "Implement production version"
4. "Add integration tests"
```

### Pattern 2: Test-Driven
```markdown
1. "Write test cases for [feature]"
2. "Generate minimal implementation to pass tests"
3. "Refactor for production quality"
4. "Add edge case tests"
```

### Pattern 3: Incremental Feature
```markdown
1. "Add basic [feature] without error handling"
2. "Add input validation and error handling"
3. "Add logging and metrics"
4. "Add configuration options"
5. "Add documentation"
```

## Quality Assurance

### Automated Checks
```bash
# Before accepting AI-generated code
make test              # Run all tests
make lint              # Run linters (golangci-lint)
make vet               # Run go vet
make fmt               # Check formatting
make security-scan     # Scan for vulnerabilities
```

### Manual Review
```markdown
**Code Quality**:
- [ ] Readable and maintainable
- [ ] Follows project conventions
- [ ] No code duplication
- [ ] Appropriate abstractions

**Functionality**:
- [ ] Solves the stated problem
- [ ] Handles edge cases
- [ ] Graceful error handling
- [ ] No obvious bugs

**Performance**:
- [ ] No obvious inefficiencies
- [ ] Appropriate data structures
- [ ] Reasonable complexity

**Security**:
- [ ] Input validation
- [ ] No SQL injection vulnerabilities
- [ ] No hardcoded credentials
- [ ] Proper error messages (no leak sensitive data)
```

## Anti-Patterns

### ❌ Accepting Code Blindly
```go
// DON'T: Copy-paste AI code without review
// AI might generate syntactically correct but logically flawed code
```

### ❌ Over-Relying on AI
```go
// DON'T: Use AI for everything including trivial tasks
// Some tasks are faster to do manually
```

### ❌ Ignoring Context
```go
// DON'T: Generate code without understanding existing codebase
// AI-generated code must fit project architecture
```

### ❌ Skipping Tests
```go
// DON'T: Trust AI-generated code without tests
// Always validate with comprehensive tests
```

## Best Practices

### ✅ Start Small
- Generate small functions first
- Validate before moving to next function
- Build incrementally

### ✅ Provide Examples
- Show AI existing code patterns
- Reference similar implementations
- Share error handling approaches

### ✅ Iterate
- First generation rarely perfect
- Refine through multiple iterations
- Gradually improve quality

### ✅ Document Intent
- Explain why code exists
- Document design decisions
- Add usage examples

## ES-1048 Project Examples

### Generate Storage Interface Method
```markdown
"Implement NodePublishVolume in pkg/driver/node.go. 
Follow Storage Interface 1.5 spec. Use existing NodeStageVolume as reference 
for error handling and logging patterns. Include idempotency check."
```

### Generate Tests
```markdown
"Write table-driven tests for NodePublishVolume covering:
- Success case
- Volume not staged
- Target path already mounted (idempotency)
- Permission errors
Use testify/assert. Target 80%+ coverage."
```

### Generate Documentation
```markdown
"Generate GoDoc for pkg/driver/controller.go. Include:
- Package overview
- Function descriptions with parameters
- Usage examples
- Common errors
Add @author <AUTHOR_NAME> to all functions."
```

## Reference Files
- **AI Research Assistant**: `.ai-config/agents/ai-research-assistant.md`
- **Prompt Engineer**: `.ai-config/agents/prompt-engineer.md`
- **AI Development Workflow**: `.ai-config/guides/AI_DEVELOPMENT_WORKFLOW.md`
- **AI-Assisted Coding Rules**: `.ai-config/rules/ai-assisted-coding.md`
