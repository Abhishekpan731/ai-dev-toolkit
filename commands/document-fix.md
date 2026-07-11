---
title: "AI-Optimized Documentation Standards - Complete Reference"
description: "Master guide for production-grade, AI-optimized documentation across repository. Mandatory YAML frontmatter, Mermaid diagrams, structured tables, comprehensive examples. Achieves 70-90% token reduction, 35% RAG accuracy improvement, 97.3% AI format accuracy."
version: 3.0.0
author: <AUTHOR_NAME>
last_updated: 2026-07-03
status: production
category: documentation-standards
tags:
  - documentation
  - ai-optimization
  - standards
  - yaml-frontmatter
  - mermaid-diagrams
  - markdown
  - best-practices
keywords:
  - documentation standards
  - AI-optimized docs
  - YAML frontmatter
  - Mermaid diagrams
  - markdown best practices
  - documentation automation
  - technical writing
search_keywords:
  - how to write AI-friendly documentation
  - YAML frontmatter template
  - Mermaid diagram examples
  - documentation upgrade guide
  - AI documentation standards 2026
  - markdown documentation best practices
related_commands:
  - /test
  - /review
  - /fix
  - /generate-docs
related_files:
  - .ai-config/DOCUMENTATION_UPGRADE_TEMPLATE.md
  - .ai-config/DOCUMENTATION_FILE_FORMAT_GUIDE.md
  - .ai-config/AI_OPTIMIZED_DOCUMENTATION_GUIDE.md
  - .ai-config/DOCUMENTATION_UPGRADE_PLAN.md
  - .ai-config/scripts/add-frontmatter.sh
  - .ai-config/commands/fix.md (fully upgraded example)
  - .ai-config/commands/test.md (upgraded example)
dependencies:
  - Markdown (.md format)
  - YAML parser
  - Mermaid diagram renderer (optional)
  - Markdown linter (optional)
inputs:
  - Existing documentation files (.md)
  - Content to document
  - Frontmatter metadata
outputs:
  - AI-optimized documentation
  - YAML frontmatter blocks
  - Mermaid workflow diagrams
  - Structured data tables
  - Comprehensive examples
platform:
  - All (platform-agnostic)
  - Works with any Markdown renderer
  - Compatible with GitHub, GitLab, Bitbucket
audience:
  - developers
  - technical writers
  - documentation maintainers
  - AI agents
  - LLMs
ai_optimized: true
human_friendly: true
shortcut: /doc-standards
---

# AI-Optimized Documentation Standards - Complete Reference

> **TL;DR**: This is the **single source of truth** for all documentation in `.ai-config/` repository. ALL files MUST have: (1) YAML frontmatter, (2) Mermaid workflow diagram, (3) structured tables for parameters/errors, (4) comprehensive examples, (5) troubleshooting section. Target metrics: 70-90% token reduction, 35% RAG accuracy improvement, 97.3% AI format accuracy. See `.ai-config/commands/fix.md` for complete example.

---

## Purpose

This document defines **mandatory standards** for all documentation across `.ai-config/commands`, `.ai-config/rules`, `.ai-config/hooks`, and `.ai-config/skills` directories.

**Why these standards exist**:

- **AI Retrieval**: 35% higher RAG accuracy, 97.3% AI format accuracy
- **Token Efficiency**: 70-90% reduction vs. traditional docs
- **Human Readability**: Visual diagrams, structured tables, clear examples
- **Maintainability**: Versioned, searchable, cross-linked, future-proof
- **Automation**: Enables automated documentation generation and validation

---

## Critical Success Metrics (Research-Backed)

| Metric                   | Target                     | Source                  |
| ------------------------ | -------------------------- | ----------------------- |
| **Token Reduction**      | 70-90% vs HTML             | FormatArc, Beam.ai 2026 |
| **RAG Accuracy**         | 35% improvement            | Cloudflare 2026         |
| **AI Format Accuracy**   | 97.3%                      | Pactify production 2026 |
| **Table Extraction**     | 60.7% (MD) vs 53.6% (HTML) | Beam.ai GPT benchmarks  |
| **Frontmatter Coverage** | 100% of files              | Mandatory               |
| **Mermaid Diagrams**     | ≥1 per command/skill       | Mandatory               |

---

## MANDATORY RULE 1: YAML Frontmatter

**ALL documentation files MUST begin with YAML frontmatter**. No exceptions.

### Minimum Required Template

```yaml
---
title: "Descriptive Title"
description: "One-sentence AI-optimized summary"
version: 1.0.0
author: <AUTHOR_NAME>
last_updated: 2026-07-03
status: production
category: command|skill|rule|hook
ai_optimized: true
human_friendly: true
---
```

### Complete Frontmatter Template (Recommended)

See `.ai-config/DOCUMENTATION_UPGRADE_TEMPLATE.md` for the full 40+ field template including:

- `tags`, `keywords`, `search_keywords`
- `related_commands`, `related_skills`, `related_rules`, `related_files`
- `dependencies`, `inputs`, `outputs`
- `platform`, `audience`
- `shortcut` (for commands)

### Why Frontmatter Matters

**80% context token savings**: AI reads metadata (20 tokens) vs. full page (2000 tokens)

**Example**:

```yaml
---
title: "Fix Command"
description: "Debug and fix issues systematically"
category: command
tags: [debugging, bug-fix, testing]
related_commands: [/test, /review]
---
```

3. **Purpose Section** (why it exists, what problem it solves)
4. **When to Use / When NOT to Use** (with ✅ ❌ emojis)
5. **Workflow/Architecture** (with Mermaid diagram - MANDATORY)
6. **Usage Examples** (≥2 complete examples with output)
7. **Parameters/Configuration** (structured table)
8. **Error Handling** (error code table)
9. **Troubleshooting** (common problems + solutions)
10. **Best Practices** (3-5 key recommendations)
11. **Related Components** (cross-links)
12. **Changelog** (version history table)

**See `.ai-config/commands/fix.md` for perfect example of this structure.**

---

## MANDATORY RULE 3: Mermaid Workflow Diagram

Every command and skill MUST include at least ONE Mermaid diagram showing workflow/architecture.

### Example: Flowchart (Most Common)

\`\`\`mermaid
flowchart TD
Start([User Action]) --> Step1[Step 1: Validate]
Step1 --> Decision{Valid?}
Decision -->|Yes| Step2[Step 2: Execute]
Decision -->|No| Error[Error State]
Step2 --> End([Complete])
Error --> End

    style Start fill:#4CAF50,color:#fff
    style End fill:#4CAF50,color:#fff
    style Step2 fill:#2196F3,color:#fff
    style Error fill:#F44336,color:#fff

\`\`\`

**Use Cases**:

- Commands: Execution flow
- Skills: Process workflow
- Integrations: Component interaction
- Decision logic: Branching paths

---

## MANDATORY RULE 4: Structured Tables

All parameters, errors, and configuration MUST use Markdown tables.

### Parameters Table Template

```markdown
| Parameter   | Type    | Required | Default | Description             |
| ----------- | ------- | -------- | ------- | ----------------------- |
| `--name`    | string  | Yes      | -       | Resource identifier     |
| `--timeout` | number  | No       | 30      | Max wait time (seconds) |
| `--retry`   | boolean | No       | false   | Enable automatic retry  |
```

### Error Handling Table Template

```markdown
| Error Code | Condition         | Root Cause          | Resolution                | Prevention          |
| ---------- | ----------------- | ------------------- | ------------------------- | ------------------- |
| ERR_001    | File not found    | Missing input       | Verify path exists        | Validate before run |
| ERR_002    | Invalid format    | Wrong type          | Convert to correct format | Type checking       |
| ERR_003    | Permission denied | Insufficient rights | Grant permissions         | Check access first  |
```

### Compatibility Matrix Template

```markdown
| Platform   | Version | Status       | Notes               |
| ---------- | ------- | ------------ | ------------------- |
| Kubernetes | 1.24+   | ✅ Supported | Fully tested        |
| Kubernetes | 1.20-23 | ⚠️ Limited   | Basic features only |
| OpenShift  | 4.10+   | ✅ Supported | Requires SCC config |
```

---

## MANDATORY RULE 5: Complete Examples

Every file MUST include ≥2 complete, copy-pasteable examples.

### Example Template

```markdown
### Example 1: Common Use Case

**Scenario**: User wants to authenticate with JWT

**Command**:
\`\`\`bash
/fix "TypeError: Cannot read property 'name' of undefined"
\`\`\`

**Process**:

1. Gather context: Stack trace shows error in getUserName()
2. Reproduce: Call API endpoint that triggers error
3. Analyze: User object is undefined when no auth
4. Fix: Add null check before accessing property
5. Test: Write test for unauthenticated scenario

**Expected Output**:
\`\`\`text
✅ Bug fixed
✅ Test added (87% coverage)
✅ No regressions detected
\`\`\`

**Result**: Bug resolved, future occurrence prevented
```

**Rules**:

- Include scenario, command, process, output, result
- Show BOTH success AND failure cases
- Use real, tested commands
- Explain WHY it works

---

## Writing Style Rules

### ✅ DO: High Semantic Density

**Good Example** (71% density):

```markdown
**Authentication**: JWT

**Process**: Server signs → Client includes → Server validates

**Security**: RS256, 15-min expiry, httpOnly cookies
```

**28 tokens, packed with information**

### ❌ DON'T: Low Semantic Density

**Bad Example** (6.7% density):

```markdown
It is worth noting that when implementing authentication, you should probably consider using JSON Web Tokens, which are really great for modern applications. Basically, what happens is...
```

**45 tokens, minimal information**

### Remove These Words (Zero Value)

- Filler: "basically", "essentially", "actually", "simply"
- Pleasantries: "Great question!", "Let's dive in"
- Hedging: "you might want to", "probably", "perhaps"
- Redundancy: "in order to" → "to", "due to the fact that" → "because"

### Keep These (High Value)

- Exact function names: `createAuthenticatedUser()` not `create()`
- Type annotations: `Promise<User>`, `string[]`, `error`
- Error messages: Exact text from actual errors
- Code examples: Complete, runnable snippets

---

## Automation Tools

### Add Frontmatter to All Files

```bash
# Upgrade all commands
bash .ai-config/scripts/add-frontmatter.sh .ai-config/commands

# Upgrade all skills
bash .ai-config/scripts/add-frontmatter.sh .ai-config/skills

# Single file
bash .ai-config/scripts/add-frontmatter.sh .ai-config/commands/fix.md
```

### Validate Documentation

```bash
# Check markdown lint
npx markdownlint-cli2 ".ai-config/**/*.md"

# Fix automatically
npx markdownlint-cli2-fix ".ai-config/**/*.md"

# Validate frontmatter
python .ai-config/scripts/validate-frontmatter.py .ai-config/
```

---

## Quality Checklist

Before marking documentation complete, verify ALL items:

### Structure ✅

- [ ] YAML frontmatter present (all required fields)
- [ ] TL;DR in first 200 tokens
- [ ] Purpose section explains WHY and WHAT
- [ ] When to use / NOT use documented
- [ ] Logical heading hierarchy (H1 → H2 → H3)

### Content ✅

- [ ] ≥1 Mermaid diagram (workflow/architecture)
- [ ] ≥2 complete examples (scenario + command + output)
- [ ] Parameters in table format
- [ ] Error handling in table format
- [ ] Troubleshooting section (≥2 problems + solutions)
- [ ] Best practices (3-5 recommendations)
- [ ] Changelog table

### Technical ✅

- [ ] All code blocks have language tags
- [ ] Tables properly formatted (aligned)
- [ ] Related files cross-linked
- [ ] No broken links
- [ ] Mermaid diagrams render correctly
- [ ] Examples tested and working

### AI Optimization ✅

- [ ] High semantic density (filler removed)
- [ ] Searchable keywords in frontmatter
- [ ] Complete, not implied information
- [ ] Front-loaded important content
- [ ] Terminology consistent across files

---

## Upgrade Process

### For Existing Files (Step-by-Step)

1. **Backup**: `cp file.md file.md.bak`
2. **Add Frontmatter**: Copy template, fill all fields
3. **Add TL;DR**: First paragraph after title
4. **Add Mermaid Diagram**: Workflow or architecture
5. **Convert to Tables**: Parameters, errors, config
6. **Add Examples**: ≥2 complete, tested examples
7. **Add Troubleshooting**: Common issues + solutions
8. **Add Changelog**: Document the upgrade
9. **Validate**: Run quality checklist
10. **Test**: Verify AI readability

### For New Files

1. Start with `.ai-config/DOCUMENTATION_UPGRADE_TEMPLATE.md`
2. Fill frontmatter completely
3. Follow mandatory structure
4. Add Mermaid diagram
5. Add ≥2 examples
6. Run quality checklist

---

## Reference Files (Examples)

### Fully Upgraded (Use as Templates)

- **`.ai-config/commands/fix.md`** - Perfect example, all standards applied
- **`.ai-config/commands/test.md`** - Complete command documentation
- **`.ai-config/commands/multi-agent-orchestrate.md`** - Complex workflow

### Guides

- **`.ai-config/DOCUMENTATION_FILE_FORMAT_GUIDE.md`** - File format comparison
- **`.ai-config/AI_OPTIMIZED_DOCUMENTATION_GUIDE.md`** - AI optimization techniques
- **`.ai-config/DOCUMENTATION_UPGRADE_TEMPLATE.md`** - Blank template

### Tools

- **`.ai-config/scripts/add-frontmatter.sh`** - Automated frontmatter
- **`.ai-config/DOCUMENTATION_UPGRADE_PLAN.md`** - Full upgrade plan (69+ files)

---

## Common Mistakes (AVOID)

### ❌ Mistake 1: No Frontmatter

**Problem**: AI cannot index or retrieve document efficiently
**Fix**: Add YAML frontmatter block (mandatory)

### ❌ Mistake 2: Vague Examples

**Problem**: Examples don't show complete workflow
**Fix**: Include scenario, command, process, output, explanation

### ❌ Mistake 3: No Mermaid Diagrams

**Problem**: Complex workflows not visualized
**Fix**: Add flowchart, sequence diagram, or architecture diagram

### ❌ Mistake 4: Prose-Heavy Tables

**Problem**: Parameters buried in paragraphs
**Fix**: Convert to structured Markdown table

### ❌ Mistake 5: Low Token Density

**Problem**: Filler words waste AI context window
**Fix**: Remove "basically", "simply", "it's worth noting"

---

## Success Criteria

Documentation is complete when:

1. ✅ **100% Frontmatter Coverage**: Every file has complete YAML block
2. ✅ **Visual Workflows**: Every command/skill has ≥1 Mermaid diagram
3. ✅ **Structured Data**: All parameters/errors in table format
4. ✅ **Complete Examples**: ≥2 real, tested examples per file
5. ✅ **Cross-Linked**: Related files properly linked
6. ✅ **AI-Optimized**: High semantic density, searchable metadata
7. ✅ **Lint Clean**: Zero markdown lint warnings
8. ✅ **Validated**: Quality checklist 100% passed

---

## Next Steps

1. **Read Examples**: Study `.ai-config/commands/fix.md` (fully upgraded)
2. **Use Template**: Copy `.ai-config/DOCUMENTATION_UPGRADE_TEMPLATE.md`
3. **Follow Checklist**: Run quality checklist on every file
4. **Automate**: Use scripts for bulk frontmatter addition
5. **Validate**: Test AI readability with ChatGPT/Claude

**Questions?** See working examples or run automation scripts.

---

**This is a living document. Version 3.0.0 reflects 2026 AI documentation research and production standards. All contributors MUST follow these guidelines.**

AI now knows:

- What it is (command)
- What it does (debugging)
- Related tools (/test, /review)
- WITHOUT reading 200+ lines

---

## MANDATORY RULE 2: Document Structure

Every file MUST follow this structure in order:

1. **YAML Frontmatter** (lines 1-N)
2. **Title + TL;DR** (first 200 tokens)
