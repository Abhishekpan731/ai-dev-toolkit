---
name: test-plan
description: Generate comprehensive test plan for Go/Java code changes
shortcut: /test
---

# Test Plan Generator

Generate a comprehensive test plan for the code changes with:

## 1. Unit Tests

### For Go Code:
```go
// Table-driven test template
func TestFunctionName_EdgeCases(t *testing.T) {
    tests := []struct {
        name    string
        input   InputType
        want    OutputType
        wantErr bool
    }{
        {"valid input", validInput, expectedOutput, false},
        {"empty input", emptyInput, nil, true},
        {"nil input", nil, nil, true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := FunctionName(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("want error: %v, got: %v", tt.wantErr, err)
            }
            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("want: %v, got: %v", tt.want, got)
            }
        })
    }
}
```

### For Java Code:
```java
@ExtendWith(MockitoExtension.class)
class ServiceTest {
    @Mock Repository repo;
    @InjectMocks Service service;
    
    @Test
    void shouldHandleEdgeCase() {
        // Arrange
        when(repo.method(any())).thenReturn(expected);
        
        // Act
        Result result = service.method(input);
        
        // Assert
        assertThat(result).isEqualTo(expected);
        verify(repo).method(argThat(arg -> arg.equals(input)));
    }
}
```

## 2. Integration Tests
- Test with real dependencies (Kubernetes API, filesystem)
- Use testcontainers for isolated testing
- Verify end-to-end workflows

## 3. Edge Cases to Cover
- Nil/null inputs
- Empty values
- Boundary conditions
- Concurrent operations
- Timeout scenarios
- Resource exhaustion
- Network failures

## 4. Storage Interface-Specific Tests (if applicable)
- Idempotency verification
- Volume lifecycle state transitions
- Concurrent mount requests
- Cleanup on errors
- Metrics collection

## 5. Test Execution
```bash
# Go tests with coverage
go test ./... -v -cover -coverprofile=coverage.out
go tool cover -func=coverage.out

# Specific package
go test ./pkg/driver -v -race
```

Generate specific test cases with expected inputs/outputs for the code being reviewed.
