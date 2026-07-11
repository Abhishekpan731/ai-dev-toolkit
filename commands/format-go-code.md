---
name: format-go-code
description: Format and organize Go code using gofmt and goimports
shortcut: /fmt
---

# Format Go Code Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Automatically format and organize Go source code using standard tools.

## Usage

```bash
# Format all Go files in project
/fmt

# Format specific file
/fmt <file.go>

# Format specific package
/fmt ./pkg/driver

# Dry run (show changes without applying)
/fmt --dry-run
```

## Command Execution

### Complete Formatting Script
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

set -e

TARGET=${1:-.}
DRY_RUN=${2:-false}

echo "=== Go Code Formatting ==="
echo "Target: $TARGET"
echo ""

# 1. Run gofmt
echo "Step 1: Running gofmt..."
if [ "$DRY_RUN" = "true" ]; then
    gofmt -l $TARGET
else
    gofmt -w $TARGET
    echo "✅ Code formatted"
fi

# 2. Run goimports
echo ""
echo "Step 2: Running goimports..."
if command -v goimports &> /dev/null; then
    if [ "$DRY_RUN" = "true" ]; then
        goimports -l $TARGET
    else
        goimports -w $TARGET
        echo "✅ Imports organized"
    fi
else
    echo "⚠️  goimports not found. Install with: go install golang.org/x/tools/cmd/goimports@latest"
fi

# 3. Run go mod tidy
echo ""
echo "Step 3: Running go mod tidy..."
go mod tidy
echo "✅ Dependencies cleaned"

# 4. Verify formatting
echo ""
echo "Step 4: Verifying formatting..."
UNFORMATTED=$(gofmt -l $TARGET)
if [ -n "$UNFORMATTED" ]; then
    echo "❌ Unformatted files found:"
    echo "$UNFORMATTED"
    exit 1
else
    echo "✅ All files properly formatted"
fi

echo ""
echo "=== Formatting Complete ==="
```

## Formatting Tools

### gofmt (Standard Formatter)
```bash
# Format specific file
gofmt -w file.go

# Format all files in directory
gofmt -w ./pkg/driver

# Show diff without applying
gofmt -d file.go

# List files that need formatting
gofmt -l .
```

### goimports (Import Organizer)
```bash
# Format and organize imports
goimports -w file.go

# Format all Go files
find . -name "*.go" -exec goimports -w {} \;

# Local package prefix (for sorting)
goimports -local github.com/storage-provider/storage-interface-driver -w .
```

### gci (Import Formatter)
```bash
# Format imports with custom sections
gci write -s standard -s default -s "prefix(github.com/ddn)" .
```

## Editor Integration

### VS Code (settings.json)
```json
{
  "go.formatTool": "goimports",
  "go.lintOnSave": "package",
  "go.vetOnSave": "package",
  "[go]": {
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": true
    }
  }
}
```

### Vim/Neovim
```vim
" Auto-format on save
autocmd BufWritePre *.go :silent! !gofmt -w %
autocmd BufWritePre *.go :silent! !goimports -w %
```

### GoLand/IntelliJ
```
Settings → Editor → Code Style → Go
✅ Enable "Run gofmt on save"
✅ Enable "Optimize imports on save"
```

## Pre-Commit Hook

### Install Hook
```bash
#!/bin/bash
# .git/hooks/pre-commit
# @author <AUTHOR_NAME>

# Format staged Go files
for file in $(git diff --cached --name-only | grep '\.go$'); do
    if [ -f "$file" ]; then
        gofmt -w "$file"
        goimports -w "$file"
        git add "$file"
    fi
done

# Verify no unformatted files are committed
UNFORMATTED=$(gofmt -l $(git diff --cached --name-only | grep '\.go$'))
if [ -n "$UNFORMATTED" ]; then
    echo "❌ Unformatted files found:"
    echo "$UNFORMATTED"
    exit 1
fi
```

## Formatting Rules

### Standard Go Format
- Tabs for indentation (not spaces)
- No trailing whitespace
- One declaration per line
- Consistent spacing around operators
- Consistent brace placement

### Import Organization
```go
import (
    // Standard library
    "context"
    "fmt"
    "os"
    
    // Third-party packages
    "google.golang.org/grpc"
    "k8s.io/api/core/v1"
    
    // Local packages
    "github.com/storage-provider/storage-interface-driver/pkg/driver"
    "github.com/storage-provider/storage-interface-driver/pkg/utils"
)
```

## Expected Output

```
=== Go Code Formatting ===
Target: .

Step 1: Running gofmt...
✅ Code formatted

Step 2: Running goimports...
✅ Imports organized

Step 3: Running go mod tidy...
✅ Dependencies cleaned

Step 4: Verifying formatting...
✅ All files properly formatted

=== Formatting Complete ===
```

## Common Issues

### Issue: goimports not found
```bash
# Install goimports
go install golang.org/x/tools/cmd/goimports@latest
```

### Issue: Module cache errors
```bash
# Clean module cache
go clean -modcache
go mod download
```

## CI/CD Integration

### GitHub Actions
```yaml
- name: Check Formatting
  run: |
    gofmt -l . | tee /tmp/gofmt.out
    [ ! -s /tmp/gofmt.out ]
```

## Reference Files

- **Go Standards**: `.ai-config/rules/golang-standards.md`
- **Code Quality**: `.ai-config/rules/code-quality.md`
- **Linter Agent**: `.ai-config/agents/go-linter.md`
