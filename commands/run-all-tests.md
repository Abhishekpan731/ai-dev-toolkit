---
name: run-all-tests
description: Execute complete test suite with coverage analysis
shortcut: /test-all
---

# Run All Tests Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Execute the complete test suite for the Storage System storage interface with coverage analysis and reporting.

## Usage

```bash
# From project root
/test-all

# With specific coverage threshold
/test-all --threshold=80

# With race detection
/test-all --race

# Quick mode (no coverage)
/test-all --quick
```

## Command Execution

### Full Test Suite
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

set -e

echo "=== Running Complete Test Suite ==="
echo ""

# 1. Run tests with coverage
echo "Step 1: Running tests with coverage..."
go test ./... -v -cover -coverprofile=coverage.out -covermode=atomic

# 2. Generate coverage report
echo ""
echo "Step 2: Generating coverage report..."
go tool cover -func=coverage.out > coverage.txt

# 3. Check coverage threshold
echo ""
echo "Step 3: Checking coverage threshold..."
COVERAGE=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
THRESHOLD=80

if (( $(echo "$COVERAGE < $THRESHOLD" | bc -l) )); then
    echo "❌ Coverage $COVERAGE% is below threshold $THRESHOLD%"
    exit 1
else
    echo "✅ Coverage $COVERAGE% meets threshold $THRESHOLD%"
fi

# 4. Run race detector
echo ""
echo "Step 4: Running race detector..."
go test -race -short ./...

# 5. Run go vet
echo ""
echo "Step 5: Running go vet..."
go vet ./...

# 6. Summary
echo ""
echo "=== Test Summary ==="
echo "Total Coverage: $COVERAGE%"
echo "Status: ✅ ALL TESTS PASSED"
```

## Test Categories

### Unit Tests
```bash
# Run only unit tests (fast)
go test ./pkg/... -v -short
```

### Integration Tests
```bash
# Run integration tests
go test ./tests/integration -v -timeout=10m
```

### E2E Tests
```bash
# Run end-to-end tests (requires cluster)
go test ./tests/e2e -v -timeout=30m
```

### Storage Interface Sanity Tests
```bash
# Run Storage Interface sanity tests
csi-sanity --csi.endpoint=/tmp/csi.sock
```

## Coverage Analysis

### View Coverage Report
```bash
# Generate HTML coverage report
go tool cover -html=coverage.out -o coverage.html

# Open in browser
open coverage.html  # macOS
xdg-open coverage.html  # Linux
```

### Coverage by Package
```bash
# Show coverage per package
go test ./... -coverprofile=coverage.out
go tool cover -func=coverage.out | grep -v "total"
```

### Find Uncovered Code
```bash
# List uncovered functions
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out | grep -E '\s+0\.0%'
```

## Expected Output

```
=== Running Complete Test Suite ===

Step 1: Running tests with coverage...
?       github.com/storage-provider/storage-interface-driver/cmd   [no test files]
ok      github.com/storage-provider/storage-interface-driver/pkg/driver    2.145s  coverage: 92.3% of statements
ok      github.com/storage-provider/storage-interface-driver/pkg/mount     1.234s  coverage: 88.5% of statements
ok      github.com/storage-provider/storage-interface-driver/pkg/utils     0.567s  coverage: 85.1% of statements

Step 2: Generating coverage report...
✅ Coverage report generated: coverage.txt

Step 3: Checking coverage threshold...
✅ Coverage 87.2% meets threshold 80%

Step 4: Running race detector...
ok      github.com/storage-provider/storage-interface-driver/pkg/driver    2.456s
ok      github.com/storage-provider/storage-interface-driver/pkg/mount     1.567s

Step 5: Running go vet...
✅ No issues found

=== Test Summary ===
Total Coverage: 87.2%
Status: ✅ ALL TESTS PASSED
```

## Error Handling

### Failed Tests
```
❌ FAIL: github.com/storage-provider/storage-interface-driver/pkg/driver
    create_volume_test.go:45: Expected error code InvalidArgument, got Internal
```

**Action**: Fix failing test before proceeding

### Low Coverage
```
❌ Coverage 75.3% is below threshold 80%
```

**Action**: Add tests to improve coverage

## Integration with CI/CD

### GitHub Actions
```yaml
- name: Run Tests
  run: |
    go test ./... -v -cover -coverprofile=coverage.out
    go tool cover -func=coverage.out
```

### Pre-Commit Hook
```bash
#!/bin/bash
# Run tests before commit
go test ./... -short || exit 1
```

## Performance Benchmarks

### Run Benchmarks
```bash
# Run all benchmarks
go test -bench=. ./...

# Run specific benchmark
go test -bench=BenchmarkCreateVolume ./pkg/driver

# With memory profiling
go test -bench=. -benchmem ./...
```

## Reference Files

- **TDD Rules**: `.ai-config/rules/tdd.md`
- **Test Automation**: `.ai-config/agents/test-automation.md`
