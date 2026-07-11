---
name: ai-research
description: Research technologies, APIs, frameworks, and best practices
shortcut: /research
---

# AI Research Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-07

Conduct systematic research on technologies, APIs, and development topics using the AI Research Assistant.

## Usage

```bash
# Quick research
/research <topic>

# Detailed research with context
/research <topic> --context="Current project: ES-1048, Stack: Go, Kubernetes"

# Comparative research
/research "Compare <option1> vs <option2> for <use case>"

# Implementation research
/research "How to implement <feature> in <language/framework>"
```

## Examples

### Technology Research
```bash
/research "Kubernetes storage interface development best practices"
```

**Output**:
- Official Storage Interface spec documentation
- Best practice guides
- Example implementations
- Common pitfalls
- Testing strategies

### API Research
```bash
/research "gRPC health checking in Go"
```

**Output**:
- gRPC health checking protocol
- Go implementation examples
- Integration with Kubernetes liveness/readiness
- Code snippets

### Comparative Research
```bash
/research "Compare MapStruct vs manual mapping in Java"
```

**Output**:
| Feature | MapStruct | Manual |
|---------|-----------|--------|
| Performance | Compile-time | Runtime |
| Type Safety | Yes | Partial |
| Boilerplate | Minimal | High |

**Recommendation**: MapStruct for ES-1048 (matches existing patterns)

### Problem-Solving Research
```bash
/research "Solutions for Docker build failures on ARM64 Mac"
```

**Output**:
- Root causes (platform mismatch, base image incompatibility)
- Solutions (multi-platform builds, platform-specific images)
- Best practices for cross-platform Docker
- Example Dockerfile configurations

## Research Scopes

### Quick (< 5 min)
- Technology overview
- API endpoint summary
- Single best practice

### Standard (10-15 min)
- Comparative analysis
- Implementation guide
- Code examples
- Recommendations

### Deep (30+ min)
- Comprehensive technology evaluation
- Multiple implementation approaches
- Performance analysis
- Production considerations

## Research Output Format

```markdown
# Research: [Topic]
**Author**: <AUTHOR_NAME>  
**Date**: YYYY-MM-DD  
**Scope**: [Quick/Standard/Deep]

## Executive Summary
[2-3 sentence key findings]

## Findings

### 1. [Key Finding]
**Description**: [What it is]
**Use Case**: [When to use]
**Example**:
```language
[Code example]
```

### 2. [Key Finding]
...

## Recommendations

**Primary**: [Recommended approach]
**Rationale**: [Why]
**Implementation**: [How]

## References
- [Source 1]
- [Source 2]

## Next Steps
1. [Action 1]
2. [Action 2]
```

## Integration with ES-1048

### Research Kubernetes Storage Interface Patterns
```bash
/research "storage interface volume lifecycle patterns for distributed filesystems"
```

**Relevant to**:
- `pkg/driver/controller.go`: CreateVolume implementation
- `pkg/driver/node.go`: NodeStageVolume, NodePublishVolume
- `deploy/kubernetes/`: Deployment manifests

### Research Go Concurrency
```bash
/research "Go concurrency patterns for high-throughput storage interfaces"
```

**Apply to**:
- Optimize volume creation operations
- Improve health check performance
- Enhance logging efficiency

### Research Testing Strategies
```bash
/research "Integration testing for Kubernetes storage interfaces"
```

**Use for**:
- `pkg/driver/*_test.go`: Improve test coverage
- Add Testcontainers for Kubernetes
- Mock distributed filesystem interactions

## Common Research Topics

### Go Development
- "Go error handling best practices 2026"
- "Go context usage patterns"
- "Go module dependency management"

### Kubernetes
- "Kubernetes operator patterns"
- "Storage Interface sidecar containers configuration"
- "Kubernetes RBAC for storage interfaces"

### Storage
- "distributed filesystem mount options"
- "Project quota management in Distributed Filesystem"
- "Storage class parameters for Storage Interface"

### DevOps
- "Docker multi-stage builds for Go"
- "Kubernetes deployment strategies"
- "GitOps workflows for storage interfaces"

## Tips for Effective Research

1. **Be Specific**: "Storage Interface NodePublishVolume idempotency" > "Storage Interface best practices"
2. **Include Context**: Mention current stack, constraints, project phase
3. **Set Scope**: Specify quick/standard/deep to manage time
4. **Ask for Examples**: Request code snippets and real-world usage
5. **Request Comparisons**: Ask for pros/cons of different approaches

## Reference Files
- **AI Research Assistant Agent**: `.ai-config/agents/ai-research-assistant.md`
- **Research Methodology Skill**: `.ai-config/skills/research-methodology/SKILL.md`
- **Research Report Template**: `.ai-config/templates/research-report-template.md`
