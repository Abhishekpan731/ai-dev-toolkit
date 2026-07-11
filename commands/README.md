# AI Commands Reference

**Author**: <AUTHOR_NAME>
**Last Updated**: 2026-07-07
**Total Commands**: 28

## 🆕 NEW: MCP Management Command

### `/mcp-connect` - MCP Server Management & Troubleshooting

Comprehensive MCP workspace management - auto-detects issues, repairs configuration, starts services, and verifies all servers.

**Usage**:
```bash
/mcp-connect                    # Full check and auto-repair
/mcp-connect --test             # Test all servers
/mcp-connect --start            # Start required services
/mcp-connect --repair           # Repair configuration
/mcp-connect --status           # Show status only
/mcp-connect --verbose          # Detailed diagnostics
```

**Features**:
- ✅ Automatic service health checks (Ollama, Docker, kubectl)
- ✅ MCP server testing (10 FREE servers)
- ✅ Auto-repair configuration and environment
- ✅ Starts Ollama if stopped
- ✅ Validates all config files
- ✅ Zero cost ($0/month)

**File**: `.ai-config/commands/mcp-connect.md`

---

## 🚀 Multi-Agent Orchestration Commands

### `/orchestrate` - Multi-Agent Workforce Deployment

Deploy 10-50 specialized AI agents for complex tasks with DAG-based parallel execution.

**Usage**:
```bash
/orchestrate "Implement feature X"
/orchestrate "Refactor module Y" --agents=20 --pattern=hierarchical
```

**Features**:
- DAG-based task decomposition
- Wave-based parallel execution (3-15× speedup)
- Dual validation gates (95% quality threshold)
- Stigmergic coordination (shared memory)
- Contextual bandit routing (learns best agent per task)

**Research-Backed**: Based on 2026 multi-agent AI research (46+ papers)

**File**: `.ai-config/commands/multi-agent-orchestrate.md`

---

### `/coordinate` - Agent Cross-Verification & Validation

Coordinate multiple agents with explicit cross-verification for critical tasks.

**Usage**:
```bash
/coordinate "Implement JWT authentication"
/coordinate "Refactor auth system" --pattern=peer-review
```

**Features**:
- Hierarchical agent coordination
- Peer review pattern (3+ independent reviewers)
- Consensus voting
- Integration testing across all agent outputs
- Detailed analysis and synthesis

**When to Use**:
- Security-critical code
- Complex algorithms
- Shared state management
- Architecture decisions
- Database schema changes

**File**: `.ai-config/commands/agent-coordinate.md`

---

### `/dag` - Task Dependency Graph Visualization

Visualize task dependencies with wave analysis and critical path identification.

**Usage**:
```bash
/dag
/dag --critical-path
/dag --format=mermaid --metrics
```

**Features**:
- DAG construction algorithm
- Wave computation (topological sort)
- Critical path analysis
- Parallelism potential metrics
- Mermaid diagram export

**File**: `.ai-config/commands/dag-visualize.md`

---

## 📚 Documentation & Guides

### Main Coordination Guide

**File**: `.ai-config/MULTI_AGENT_COORDINATION_GUIDE.md`

Comprehensive guide answering:
- **Q**: "How should agents coordinate? Are they cross-verifying or working independently?"
- **A**: **BOTH - Depending on task type**

**Patterns**:
1. **Independent** (Parallel) - Disjoint files, minimal coordination, 2-5× speedup
2. **Coordinated** (Cross-Verified) - Shared state, multiple validation layers, 41% fewer defects
3. **Hybrid** (Recommended) - Planning verified, implementation parallel, integration verified

---

### Research Summary

**File**: `MULTI_AGENT_RESEARCH_SUMMARY.md`

- 46+ research papers analyzed (2026)
- Real-world speedup data (3-15×)
- Coordination mechanisms explained
- Decision tree for pattern selection
- Complete examples with metrics

---

## 📊 Coordination Patterns Explained

### Pattern 1: Independent Agents (Stigmergic)

**When**: Tasks have NO dependencies, disjoint files

**Example**: Build 8 UI pages
- Agent 1 → Page1.tsx
- Agent 2 → Page2.tsx
- ...
- Agent 8 → Page8.tsx

**Coordination**: Shared memory (blackboard) only
**Cross-Verification**: None during execution, validation gates only
**Speedup**: 4× (45 min vs. 180 min sequential)

---

### Pattern 2: Coordinated Agents (Cross-Verification)

**When**: Complex integration, shared state, critical paths

**Example**: Distributed transaction system

```
ARCHITECT (design)
    ↓
SECURITY_REVIEWER (validates design) ← Cross-verification
    ↓
ARCHITECT (revises)
    ↓
[BACKEND_1, BACKEND_2, BACKEND_3] (parallel)
    ↓
INTEGRATION_AGENT (validates all 3) ← Cross-verification
    ↓
[REVIEWER_1, REVIEWER_2, REVIEWER_3] (parallel peer review)
    ↓
CONSENSUS_AGENT (voting) ← Cross-verification
```

**Coordination**: Explicit handoffs, hierarchical validation
**Cross-Verification**: Multiple layers (design, integration, peer review, consensus)
**Quality**: 41% fewer defects (0.34 → 0.20 per task)

---

### Pattern 3: Hybrid (Most Common)

**Phases**:
1. **Planning**: Sequential + cross-verified
2. **Implementation**: Parallel (no cross-verification)
3. **Integration**: Sequential + cross-verified
4. **Synthesis**: Single agent aggregates

**Speedup**: 3-10×
**Quality**: <0.20 defects/task

---

## 🎯 Quick Decision Guide

```
Is task complex?
├─ NO → Use `/orchestrate` with 3-8 agents (independent)
└─ YES
   ├─ Files disjoint? → YES → `/orchestrate` with 5-15 agents
   └─ Files overlap? → YES
      ├─ Security/architecture critical? → YES → `/coordinate` (peer review)
      └─ NO → `/orchestrate` (hybrid pattern)
```

---

## 📈 Expected Results

| Pattern | Agents | Speedup | Quality |
|---------|--------|---------|---------|
| Independent | 3-8 | 2-5× | Validation gates |
| Parallel waves | 5-15 | 3-7× | Validation gates |
| Cross-verified | 15-30 | 5-15× | <0.20 defects/task |
| Hybrid | 10-20 | 3-10× | 41% fewer defects |

---

## 🔍 All Available Commands (27)

### Multi-Agent (3)
- `/orchestrate` - Multi-agent workforce deployment
- `/coordinate` - Agent cross-verification
- `/dag` - Dependency graph visualization

### Development (10)
- `/fix` - Fix issues/bugs
- `/review` - Code review
- `/test` - Run tests
- `/test-plan` - Generate test plan
- `/format-go-code` - Format Go code
- `/generate-code-ai` - AI code generation
- `/explain-code` - Explain code
- `/create-agent` - Create new agent
- `/document-fix` - Fix documentation
- `/generate-docs` - Generate documentation

### Storage Interface/Kubernetes (5)
- `/csi-review` - storage interface review
- `/debug-csi-issue` - Debug Storage Interface issues
- `/deploy-driver` - Deploy storage interface
- `/k8s-integration` - K8s integration
- `/validate-k8s-manifests` - Validate manifests
- `/system-diag` - Storage System diagnostics

### DevOps (3)
- `/build-docker-images` - Build Docker images
- `/run-all-tests` - Run all tests
- `/benchmark-performance` - Performance benchmarks

### Research & Tools (4)
- `/ai-research` - Research technologies
- `/remote-server-ops` - Remote server operations
- `/caveman` - Token-optimized mode (65-75% reduction)
- `/read-cv` - Read CV/resume

### Utilities (2)
- `/code-review` - Comprehensive code review
- `/review` - Quick review

---

## 🔗 Related Files

- **Rules**: `.ai-config/rules/SPECIALIZED-SYSTEMS.md` (Multi-agent orchestration)
- **Rules**: `.ai-config/rules/CORE-ENGINEERING-RULES.md` (Quality gates)
- **Agents**: `.ai-config/agents/` (46+ specialized agents)
- **Skills**: `.ai-config/skills/` (23 domain expertise skills)

---

## 📖 Usage Examples

### Example 1: Simple Feature (Independent)

```bash
/orchestrate "Add user profile page with avatar upload"
```

**Result**: 5 agents, 30 min, 3× speedup, zero merge conflicts

---

### Example 2: Complex Feature (Cross-Verified)

```bash
/coordinate "Implement OAuth 2.0 with SSO integration"
```

**Result**: 25 agents, 180 min, 4× speedup, 96.5% quality score

---

### Example 3: Visualize Plan

```bash
/dag --show-plan "Refactor authentication system"
```

**Result**: Mermaid diagram, critical path (65 min), 3 waves, 15 tasks

---

## 🎓 Learn More

1. Read: `.ai-config/MULTI_AGENT_COORDINATION_GUIDE.md` (Complete guide)
2. Read: `MULTI_AGENT_RESEARCH_SUMMARY.md` (Research details)
3. Try: `/orchestrate "Simple task"` (Start small)
4. Analyze: `/dag` (Understand dependencies)
5. Scale: `/coordinate "Complex task"` (Production-ready)
