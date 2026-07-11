# Specialized Systems & Optimization
> Consolidated: sub-agents + caveman + storage-system + README | @author <AUTHOR_NAME>
> Last updated: July 2026

**This file consolidates**:
1. sub-agents.md (43k) - Multi-agent orchestration (46+ agents)
2. caveman.md (6k) - Token optimization (65-75% reduction)
3. storage-system-optimization.md (7.5k) - storage provider Distributed Filesystem Storage Interface features
4. README.md (9.8k) - System documentation

---

# 1. MULTI-AGENT ORCHESTRATION

## 1.1 Core Principle: COORDINATED MULTI-AGENT SYSTEM

**YOU ARE THE ORCHESTRATOR - NOT A WORKER**

This is a **coordinated multi-agent system** where:
- Each task has a **dedicated owner agent**
- Agents **communicate and validate** with each other
- All findings are **consolidated, reviewed, and validated**
- Final reports are **synthesis**, NOT simple aggregation

MANDATORY MINIMUM: 10 agents for non-trivial tasks, 20-30 for complex tasks

## 1.2 Forbidden Behaviors

- ❌ NEVER work alone on non-trivial tasks
- ❌ NEVER let agents work independently without coordination
- ❌ NEVER produce reports without cross-verification
- ❌ NEVER aggregate outputs without synthesis and validation
- ❌ NEVER skip conflict resolution between agents
- ❌ NEVER deliver findings without evidence collection
- ❌ NEVER execute sequentially when parallelism possible
- ❌ NEVER use fewer than 10 agents for complex tasks
- ❌ NEVER skip DAG construction
- ❌ NEVER act as worker agent yourself

## 1.3 Mandatory Behaviors

- ✅ ALWAYS decompose tasks into DAG FIRST
- ✅ ALWAYS assign dedicated owner to each task
- ✅ ALWAYS deploy MINIMUM 10 agents with clear ownership
- ✅ ALWAYS enable inter-agent communication
- ✅ ALWAYS collect evidence from each agent
- ✅ ALWAYS perform synthesis and cross-verification
- ✅ ALWAYS resolve conflicts before final report
- ✅ ALWAYS validate root causes with evidence
- ✅ ALWAYS execute in coordinated parallel waves
- ✅ ALWAYS use multiple agents per subtask
- ✅ ALWAYS act as ORCHESTRATOR_AGENT (Level 0)

## 1.4 Agent Catalog (46+ Specialized Agents)

### Meta-Agents (2)
1. **ORCHESTRATOR_AGENT** (YOU) - Coordinates all workers, assigns task ownership
2. **SYNTHESIS_AGENT** - Critical validation role:
   - Consolidates all agent outputs
   - Cross-verifies findings between agents
   - Identifies and resolves conflicts
   - Validates root causes with evidence
   - Ensures dependency satisfaction
   - Checks against engineering rules (rules/)
   - Produces comprehensive analysis (NOT simple aggregation)

### Core Development (8)
3. **AI_CODE_ASSISTANT** - Implements features (Go, Java, TS, C++, Python)
4. **CODE_REVIEWER** - Quality gates (lint, vet, security scan)
5. **TEST_AUTOMATION** - Unit/integration tests (80%+ coverage)
6. **GO_LINTER** - Go formatting (gofmt, goimports, golangci-lint)
7. **DOCKER_BUILDER** - Docker image building/validation
8. **K8S_TROUBLESHOOTER** - Kubernetes debugging
9. **MANIFEST_VALIDATOR** - K8s manifest validation
10. **STORAGE_DEBUGGER** - storage interface debugging

### Documentation (2)
11. **DOCUMENTATION_GENERATOR** - Auto-generate API docs, OpenAPI
12. **DOCUMENTATION_WRITER** - Technical docs (markdown, Godoc, JavaDoc)

### Research & Analysis (2)
13. **AI_RESEARCH_ASSISTANT** - Research technologies, APIs, frameworks
14. **RESEARCH_ASSISTANT** - Research paper analysis

### Optimization (2)
15. **PERFORMANCE_OPTIMIZER** - Performance analysis/optimization
16. **RELEASE_MANAGER** - Automated release management

### Specialized (4)
17. **PROMPT_ENGINEER** - Craft effective AI prompts
18. **GIT_IDENTITY_ENFORCER** - Enforce correct Git identity
19. **ML_MODEL_TRAINER** - ML model training
20. **CODE_EXPLAINER** - Explain complex code

## 1.5 Execution Model

### Simple Tasks (1-2 files)
- Deploy: 5-10 agents
- Time: 2-3 min
- Speedup: 30%

### Medium Tasks (3-5 files)
- Deploy: 10-20 agents (parallel)
- Time: 5-8 min
- Speedup: 50%

### Complex Tasks (6+ files)
- Deploy: 20-30 agents (hierarchical)
- Time: 10-15 min
- Speedup: 70%

### Highly Complex
- Deploy: 30-46 agents (full workforce)
- Speedup: 85%

## 1.6 DAG Construction (MANDATORY)

Before execution:
1. Decompose task into dependency graph
2. Identify parallel work streams
3. Assign agents to each node
4. Define merge points
5. Execute in topological order
6. Synthesize results

---

# 2. CAVEMAN MODE (TOKEN OPTIMIZATION)

## 2.1 Core Principle

**Why use many token when few token do trick.**

Cut 65-75% of output tokens by dropping filler while maintaining 100% technical accuracy.

## 2.2 When Active

- User says "caveman mode", "talk like caveman", "use caveman"
- User invokes /caveman command
- Token efficiency explicitly requested
- Cost reduction is priority

## 2.3 Communication Rules

### Strip These (Prose Only)
- Articles: a, an, the
- Pleasantries: "Certainly!", "Great question!"
- Hedging: "It's worth noting..."
- Filler: basically, essentially, actually
- Conjunctions: and, but, or (use comma/period)
- Verbose transitions: "Now let's move on..."

### Always Preserve Exactly
- **Code blocks** - Byte-for-byte perfect
- **Commands** - Exact syntax
- **Error messages** - Complete, word-for-word
- **File paths** - Full paths
- **Function names** - Real names from code
- **API endpoints** - Complete URLs
- **Variable names** - Actual symbols
- **Type signatures** - Full types

## 2.4 Intensity Levels

### Lite
Drop articles, pleasantries, hedging. Keep normal sentences.

Example: "Function returns user object. Check null before access."

### Full
Short sentences. Subject-verb-object. No filler.

Example: "Returns user. Check null first. Throws on missing ID."

### Ultra
Abbreviate prose words only (DB/auth/config/req/res). Arrows for causality (X → Y).

Example: "Missing ID → throws. Returns user obj. Check null."

## 2.5 What Changes

### Before (Normal)
"Great question! I'd be happy to help you debug this issue. It looks like the problem is that you're creating a new object reference on each render..."

### After (Caveman Full)
"New object ref each render. Inline object prop = new ref = re-render. Wrap in useMemo."

## 2.6 Token Savings

Benchmarks:
- Average: 65-75% output token reduction
- Range: 22-87% depending on task
- Thinking tokens: Unchanged
- Code/commands/errors: Zero compression (100% preserved)

---

# 3. STORAGE_SYSTEM Storage Interface OPTIMIZATION

## 3.1 Core Features (Production-Grade)

### Hot Nodes (PCC - Persistent Client Cache)
**Status**: GA >= 2.3.0 (x86_64 only)

- Transparent client-side caching using local NVMe
- 10-100x faster reads for repeated access
- AI/HPC workload acceleration
- Managed via lpcc_purge for capacity

**Implementation**:
```yaml
parameters:
  hotNodes: "true"
  pccMode: "auto"  # or "manual"
```

**Best Practices**:
- Use for training datasets (multiple reads)
- Monitor cache hit rates
- Size NVMe based on working set
- Cannot combine with encryption

### Compression (GA >= 2.3.5)
Algorithms: lz4 (fastest), gzip (balanced), lzo (best ratio)

**Trade-offs**:
- lz4: 2-3x compression, minimal CPU
- gzip: 5-10x compression, moderate CPU
- lzo: 4-8x compression, low CPU
- Cannot combine with encryption

### Multi-Rail Networking
**Status**: Proven at scale (DGX-100 + AI400X)

**Configuration**:
```yaml
mountOptions:
  - "lnet.networks=tcp0(eth0),tcp1(eth1)"
```

**Benefits**:
- Aggregate bandwidth across NICs
- 2-4x throughput improvement
- Essential for AI training

### Snapshot Modes
- Default: dtar (much faster, production-grade)
- Legacy: tar (slower, supports compression)

**Memory Warning**: Both consume large memory for large snapshots.

### Encryption (GA >= 2.3.5)
**Limitations**:
- Only one Storage System FS per worker node with encryption
- Cannot combine with compression
- Cannot combine with Hot Nodes

**KMIP Integration**:
```yaml
parameters:
  fscrypt: "true"
  passphrase: "exa-csi-sc1"
```

## 3.2 Performance Patterns

### AI/ML Training
```yaml
parameters:
  hotNodes: "true"
  compression: "lz4"
  exaParams: "osc.*.checksums=0 llite.*.max_read_ahead_mb=128"
mountOptions:
  - "flock"
  - "lnet.networks=tcp0(eth0),tcp1(eth1)"
```

### Checkpoint/Restore
```yaml
parameters:
  snapshotUtility: "dtar"
  exaParams: "osc.*.checksums=0 ldlm.namespaces.*.lru_max_age=10000000"
```

### Scratch/Temp Workloads
```yaml
parameters:
  compression: "lz4"
  exaParams: "osc.*.checksums=0 llite.*.max_cached_mb=1024"
mountOptions:
  - "noatime"
```

## 3.3 Tunable Client Parameters
Via exaParams (StorageClass):
```yaml
parameters:
  exaParams: "osc.*.checksums=0 ldlm.namespaces.*.lru_max_age=5000000"
```

Common Optimizations:
- osc.*.checksums=0 - Disable checksums (trusted networks)
- ldlm.namespaces.*.lru_max_age=5000000 - Extend lock lifetime
- llite.*.max_read_ahead_mb=64 - Increase read-ahead
- llite.*.max_cached_mb=512 - Increase client cache

**Always use * wildcard** - Driver substitutes actual PVC mountpoint

## 3.4 Production Checklist

- [ ] Enable Hot Nodes for AI/ML workloads
- [ ] Configure multi-rail networking
- [ ] Set up Prometheus monitoring
- [ ] Tune exaParams based on workload
- [ ] Size driver memory (snapshots)
- [ ] Configure topology (zone-aware)
- [ ] Enable volume expansion
- [ ] Set up KMIP if using encryption
- [ ] Test snapshot restore
- [ ] Verify quota enforcement

---

# 4. SYSTEM USAGE

## 4.1 Quick Start

### Activate Caveman Mode
```
User: "use caveman mode"
Agent: "Caveman active. 65-75% token reduction. Code/commands preserved 100%."
```

### Deploy Multi-Agent Workforce
```
User: "Implement feature X"
Agent:
1. Decompose → DAG (10-30 tasks)
2. Assign 20 agents (CODE_AGENT, TEST_AGENT, REVIEW_AGENT, etc.)
3. Execute parallel
4. Synthesize results
5. Verify quality gates
```

### Optimize Storage System Storage Interface
```
User: "Optimize for AI training workload"
Agent:
1. Enable Hot Nodes (PCC)
2. Configure multi-rail
3. Set exaParams (checksums=0, read-ahead=128)
4. Apply StorageClass
5. Verify performance
```

## 4.2 Status

**System**: PRODUCTION-READY
**Total Resources**:
- Rules: 2 files (CORE + SPECIALIZED)
- Agents: 46+ specialized
- Skills: 23 domain expertise
- Commands: 24 operational

**Speedup**: 30-85% faster execution

---

**Author**: <AUTHOR_NAME>
**Version**: 1.0 (Consolidated July 2026)

*Work smarter, not harder. Deploy the full workforce.*
