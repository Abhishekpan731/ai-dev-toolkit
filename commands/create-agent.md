---
name: create-agent
description: Create a new custom AI agent with proper configuration
shortcut: /newagent
---

# Create Agent Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-07

Create a new custom AI agent using the standard template and best practices.

## Usage

```bash
# Create agent from template
/newagent <agent-name> --description="<description>"

# Create agent with specific tools
/newagent <agent-name> --tools="view,codebase-retrieval,launch-process"

# Create agent with color
/newagent <agent-name> --color="cyan"
```

## Agent Creation Workflow

### Step 1: Define Agent Purpose
```markdown
**Agent Name**: performance-analyzer
**Description**: Analyze code performance and suggest optimizations
**Primary Use Case**: Identifying performance bottlenecks in Go code
**Target Users**: Developers optimizing storage interface performance
```

### Step 2: Select Tools
```markdown
**Required Tools**:
- view: Read code files
- codebase-retrieval: Find performance-critical code
- launch-process: Run benchmarks

**Optional Tools**:
- web-search: Research optimization techniques
```

### Step 3: Choose Model & Color
```markdown
**Model**: sonnet4.5 (for complex analysis)
**Color**: orange (high visibility)
```

### Step 4: Create Agent File
```yaml
---
name: performance-analyzer
description: Analyze code performance and suggest optimizations
model: sonnet4.5
color: orange
tools:
  - view
  - codebase-retrieval
  - launch-process
  - web-search
---

# Performance Analyzer Agent

**Author**: <AUTHOR_NAME>  
**Date**: YYYY-MM-DD

You are a Performance Analyzer specializing in Go code optimization.

## Your Role

1. **Analyze Code**: Identify performance bottlenecks
2. **Benchmark**: Run performance tests
3. **Optimize**: Suggest concrete improvements
4. **Validate**: Verify optimizations don't break functionality

## Analysis Workflow

### Step 1: Profile
```bash
go test -bench=. -cpuprofile=cpu.prof
go tool pprof cpu.prof
```

### Step 2: Identify Bottlenecks
- CPU-intensive operations
- Memory allocations
- I/O operations
- Lock contention

### Step 3: Suggest Optimizations
- Algorithm improvements
- Caching strategies
- Goroutine usage
- Memory pooling

### Step 4: Validate
- Run benchmarks before/after
- Verify functionality unchanged
- Check memory usage

## Example Analysis

```go
// Before: O(n²) complexity
func FindDuplicates(items []string) []string {
    var duplicates []string
    for i := 0; i < len(items); i++ {
        for j := i + 1; j < len(items); j++ {
            if items[i] == items[j] {
                duplicates = append(duplicates, items[i])
            }
        }
    }
    return duplicates
}

// After: O(n) complexity
func FindDuplicates(items []string) []string {
    seen := make(map[string]bool)
    var duplicates []string
    for _, item := range items {
        if seen[item] {
            duplicates = append(duplicates, item)
        }
        seen[item] = true
    }
    return duplicates
}
```

## Reference Files
- **Agent Template**: `.ai-config/templates/agent-template.md`
- **Agent Creation Guide**: `.ai-config/guides/AGENT_CREATION_GUIDE.md`
```

## Agent Template Structure

```markdown
---
name: <agent-name>
description: <one-line description>
model: sonnet4.5
color: <cyan|purple|green|blue|orange|red>
tools:
  - <tool1>
  - <tool2>
---

# <Agent Name> Agent

**Author**: <AUTHOR_NAME>  
**Date**: YYYY-MM-DD

You are a <role> specializing in <expertise>.

## Your Role

1. **<Capability 1>**: <Description>
2. **<Capability 2>**: <Description>
3. **<Capability 3>**: <Description>

## <Primary Workflow>

### Step 1: <Phase Name>
<Description and code examples>

### Step 2: <Phase Name>
<Description and code examples>

## Examples

<Concrete usage examples>

## Reference Files
- <Related agents, commands, skills>
```

## Available Tools

| Tool | Purpose | Use When |
|------|---------|----------|
| `view` | Read files | Agent needs file content |
| `codebase-retrieval` | Search codebase | Agent needs to find code |
| `save-file` | Create files | Agent generates output files |
| `str-replace-editor` | Edit files | Agent modifies existing files |
| `launch-process` | Run commands | Agent executes builds/tests |
| `web-search` | Search web | Agent needs external info |
| `web-fetch` | Fetch URLs | Agent needs specific docs |
| `github-api` | GitHub operations | Agent manages PRs/issues |
| `render-mermaid` | Create diagrams | Agent visualizes architecture |

## Model Selection

| Model | Use For | Characteristics |
|-------|---------|------------------|
| `sonnet4.5` | Complex reasoning | High quality, slower |
| `sonnet4` | Standard tasks | Balanced performance |
| `haiku` | Simple tasks | Fast, lower cost |

## Color Palette

| Color | Meaning | Use For |
|-------|---------|---------|
| `cyan` | Technical | Code-focused agents |
| `purple` | Research | Research agents |
| `green` | Creation | Documentation/generation |
| `blue` | Analysis | Code analysis agents |
| `orange` | Performance | Optimization agents |
| `red` | Critical | Debugging/troubleshooting |

## Agent Best Practices

### DO
- ✅ Give agents clear, specific roles
- ✅ Provide step-by-step workflows
- ✅ Include concrete examples
- ✅ Link to related resources
- ✅ Add author attribution

### DON'T
- ❌ Make agents too broad ("do everything")
- ❌ Omit tool specifications
- ❌ Forget YAML frontmatter
- ❌ Skip usage examples
- ❌ Leave undefined workflows

## Testing Your Agent

```bash
# 1. Create agent file
vim .ai-config/agents/my-agent.md

# 2. Use agent in conversation
"@my-agent <task description>"

# 3. Verify:
- Agent activates correctly
- Has access to specified tools
- Follows defined workflow
- Produces expected output
```

## Reference Files
- **Agent Template**: `.ai-config/templates/agent-template.md`
- **Agent Creation Guide**: `.ai-config/guides/AGENT_CREATION_GUIDE.md`
- **Existing Agents**: `.ai-config/agents/`
