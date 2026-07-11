---
mode: manual
name: research-methodology
description: Systematic approach to developer research and technology evaluation
tags: [research, methodology, evaluation, decision-making]
version: 1.0.0
---

# Research Methodology

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-07

Systematic methodology for conducting effective developer research and technology evaluation.

## Research Framework

### 1. Define Scope (5-10% of time)
```markdown
**Research Question**: [Specific question to answer]
**Context**: [Current situation, constraints]
**Success Criteria**: [What defines complete research]
**Time Box**: [Maximum time to spend]
**Deliverable**: [Expected output format]
```

### 2. Gather Information (40-50% of time)
**Sources** (in priority order):
1. **Official Documentation**: Authoritative, up-to-date
2. **GitHub Repositories**: Real implementations, issues
3. **Technical Blogs**: In-depth explanations
4. **Stack Overflow**: Common problems and solutions
5. **Academic Papers**: Theoretical foundations

### 3. Analyze & Synthesize (30-40% of time)
**Analysis Methods**:
- Feature comparison matrices
- Pros/cons lists
- Performance benchmarks
- Complexity assessments

### 4. Document Findings (10-15% of time)
**Output Format**:
- Executive summary
- Detailed findings
- Recommendations
- Action items

## Research Types

### Type 1: Technology Evaluation
```markdown
**Goal**: Choose between competing technologies

**Process**:
1. List requirements (must-have, nice-to-have)
2. Identify candidates
3. Create comparison matrix
4. Evaluate each against requirements
5. Score and rank
6. Recommend top choice

**Example**: "storage interface storage backend: NFS vs Distributed Filesystem vs CephFS"

**Output**:
| Feature | NFS | Distributed Filesystem | CephFS |
|---------|-----|--------|--------|
| Performance | 3 | 5 | 4 |
| Scalability | 2 | 5 | 4 |
| Complexity | 5 | 2 | 3 |
| Maturity | 5 | 4 | 3 |

Recommendation: Distributed Filesystem (highest performance, proven scalability)
```

### Type 2: Best Practices Research
```markdown
**Goal**: Find industry standards for a technique

**Process**:
1. Search official documentation
2. Find reference implementations
3. Identify common patterns
4. Extract best practices
5. Adapt to project context

**Example**: "Go error handling best practices 2026"

**Output**:
- Wrap errors with context (fmt.Errorf with %w)
- Define sentinel errors for expected cases
- Use custom error types for rich error info
- Don't panic except in truly exceptional cases
```

### Type 3: API/Library Research
```markdown
**Goal**: Understand how to use specific API

**Process**:
1. Read official API reference
2. Find code examples
3. Identify common usage patterns
4. Test key features
5. Document edge cases

**Example**: "Kubernetes Storage Interface spec volume lifecycle methods"

**Output**:
- CreateVolume: Provision storage
- NodeStageVolume: Mount filesystem (global)
- NodePublishVolume: Bind mount to pod
- Sequence diagram showing call order
- Error handling requirements
```

### Type 4: Troubleshooting Research
```markdown
**Goal**: Find solution to specific problem

**Process**:
1. Reproduce issue
2. Search error messages
3. Check GitHub issues
4. Review Stack Overflow
5. Test proposed solutions
6. Document working fix

**Example**: "Docker build fails ARM64 Mac: exec format error"

**Output**:
- Root cause: Base image not ARM64-compatible
- Solution: Use multi-platform base image or native ARM64 image
- Prevention: Always specify --platform in Dockerfile
```

## Evaluation Criteria

### Technical Criteria
```markdown
**Performance**:
- Throughput (ops/sec)
- Latency (p50, p95, p99)
- Resource usage (CPU, memory)

**Scalability**:
- Horizontal scaling support
- Vertical scaling limits
- Bottlenecks

**Reliability**:
- Error handling
- Failure modes
- Recovery mechanisms

**Security**:
- Authentication/authorization
- Data encryption
- Vulnerability history
```

### Non-Technical Criteria
```markdown
**Maturity**:
- Years in production
- Adoption rate
- Stability of API

**Community**:
- GitHub stars, forks
- Active maintainers
- Issue response time

**Documentation**:
- Quality of official docs
- Availability of examples
- Community resources

**Licensing**:
- License compatibility
- Commercial restrictions
- Patent concerns
```

## Research Report Template

```markdown
# Research: [Topic]

**Author**: <AUTHOR_NAME>  
**Date**: YYYY-MM-DD  
**Research Time**: [X hours]  
**Scope**: [Quick/Standard/Deep]

## Executive Summary
[2-3 sentences summarizing key findings and recommendation]

## Research Question
[Original question this research aimed to answer]

## Methodology
- Sources consulted: [List]
- Evaluation criteria: [List]
- Time spent: [Breakdown]

## Findings

### Finding 1: [Title]
**Description**: [What is it]
**Pros**:
- [Pro 1]
- [Pro 2]

**Cons**:
- [Con 1]
- [Con 2]

**Code Example**:
```language
[Example code]
```

### Finding 2: [Title]
...

## Comparison Matrix
| Criterion | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| [Criterion 1] | [Score] | [Score] | [Score] |
| [Criterion 2] | [Score] | [Score] | [Score] |

## Recommendations

### Primary Recommendation: [Option]
**Rationale**: [Why this is best choice]

**Implementation Approach**:
1. [Step 1]
2. [Step 2]

**Estimated Effort**: [Time estimate]

**Risks**:
- [Risk 1]: [Mitigation]
- [Risk 2]: [Mitigation]

### Alternative Options
- **[Alternative 1]**: Use if [condition]
- **[Alternative 2]**: Use if [condition]

## References
- [Link 1]: [Description]
- [Link 2]: [Description]

## Next Steps
1. [Action 1]
2. [Action 2]

## Appendix
[Additional details, benchmarks, full code examples]
```

## Research Techniques

### Technique 1: Breadth-First Search
```
1. Quick scan of many sources
2. Identify most relevant
3. Deep dive on top 2-3
4. Synthesize findings
```

### Technique 2: Depth-First Search
```
1. Pick most authoritative source
2. Thoroughly study it
3. Identify gaps
4. Fill gaps with targeted research
```

### Technique 3: Comparative Analysis
```
1. List options to compare
2. Define comparison criteria
3. Research each option against criteria
4. Create comparison matrix
5. Score and rank
```

## Quality Checklist

- [ ] **Clear Question**: Research question well-defined
- [ ] **Multiple Sources**: Consulted 3+ reliable sources
- [ ] **Authoritative**: Prioritized official docs
- [ ] **Current**: Information is up-to-date (within 1-2 years)
- [ ] **Verified**: Tested claims when possible
- [ ] **Objective**: Considered multiple viewpoints
- [ ] **Actionable**: Includes concrete recommendations
- [ ] **Documented**: Findings clearly recorded

## Common Pitfalls

### ❌ Research Paralysis
```
Spending too much time researching, not enough implementing
Solution: Set strict time boxes
```

### ❌ Confirmation Bias
```
Only seeking information that supports preconceived opinion
Solution: Actively seek contradictory evidence
```

### ❌ Outdated Information
```
Using old blog posts, deprecated APIs
Solution: Check publication dates, verify current docs
```

### ❌ Insufficient Depth
```
Relying on single source or superficial understanding
Solution: Cross-reference multiple sources
```

## Reference Files
- **AI Research Assistant**: `.ai-config/agents/ai-research-assistant.md`
- **AI Research Command**: `.ai-config/commands/ai-research.md`
- **Research Report Template**: `.ai-config/templates/research-report-template.md`
