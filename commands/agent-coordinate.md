---
name: agent-coordinate
description: Coordinate multiple agents with cross-verification, validation, and detailed analysis
shortcut: /coordinate
---

# Agent Coordination Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-07-03

**ANSWERS YOUR QUESTION**: This command demonstrates proper agent coordination with:
- ✅ Dedicated agent assignment per task
- ✅ Cross-verification between agents
- ✅ Validation gates and quality checks
- ✅ Detailed analysis and synthesis

## The Question Answered

**Q**: "Are agents coordinating and cross-verifying, or working independently and generating reports without analysis?"

**A**: **BOTH PATTERNS EXIST** - Here's when to use each:

### Pattern 1: Independent Agents (Parallel Execution)

**When to Use**: Tasks with NO dependencies, disjoint file ownership

```yaml
scenario: "Build 5-page dashboard"
coordination: MINIMAL (stigmergic only)

agents:
  - UI_AGENT_1: HomePage.tsx       # Independent
  - UI_AGENT_2: Dashboard.tsx      # Independent
  - UI_AGENT_3: Profile.tsx        # Independent
  - UI_AGENT_4: Settings.tsx       # Independent
  - UI_AGENT_5: Analytics.tsx      # Independent

communication: Shared memory only (pattern library, color scheme)
cross_verification: NO (files disjoint, no conflicts possible)
validation: YES (each agent's output validated independently)

speedup: 4.5× (5 pages in 45 min vs. 180 min sequential)
```

**Benefits**:
- Maximum parallelism
- No coordination overhead
- Zero merge conflicts
- Clean git history

### Pattern 2: Coordinated Agents (Cross-Verification)

**When to Use**: Complex integration, shared state, critical paths

```yaml
scenario: "Implement distributed transaction system"
coordination: INTENSIVE (explicit handoffs)

workflow:
  phase_1_planning:
    - ARCHITECT_AGENT:
        role: Design transaction protocol
        output: architecture.md
        
    - REVIEWER_AGENT_1:
        role: Review architecture
        validates: [correctness, scalability, fault_tolerance]
        feedback_to: ARCHITECT_AGENT
        
    - SECURITY_AGENT:
        role: Security audit
        validates: [isolation, consistency, auth]
        blocks_next_phase: true  # Must pass before coding
  
  phase_2_implementation:
    - BACKEND_AGENT_1:
        role: Transaction coordinator
        depends: [ARCHITECT_AGENT.output]
        
    - BACKEND_AGENT_2:
        role: State machine
        depends: [ARCHITECT_AGENT.output]
        
    - BACKEND_AGENT_3:
        role: Persistence layer
        depends: [ARCHITECT_AGENT.output]
        
    - INTEGRATION_AGENT:
        role: Cross-agent integration
        validates: [BACKEND_AGENT_1, BACKEND_AGENT_2, BACKEND_AGENT_3]
        tests: end-to-end workflow
  
  phase_3_validation:
    - CODE_REVIEWER_AGENT:
        role: Code quality review
        checks: [functions <50 lines, error handling, tests]
        
    - SECURITY_REVIEWER_AGENT:
        role: Security review
        checks: [injection, auth, encryption]
        
    - PERFORMANCE_AGENT:
        role: Performance analysis
        runs: benchmarks
        
    - SYNTHESIS_AGENT:
        role: Merge all feedback
        creates: final_report.md
        decides: PASS/FAIL/REVISE
```

**Benefits**:
- High quality output
- Reduced defect rate (0.34 → 0.20)
- Architectural correctness
- Security validation

## Coordination Patterns

### 1. Hub-and-Spoke (Most Common)

```
        ORCHESTRATOR
       /  /  |  \  \
      /  /   |   \  \
   A1  A2   A3   A4  A5  (Workers)
      \  \   |   /  /
       \  \  |  /  /
      SYNTHESIS_AGENT
```

**Communication**: Orchestrator → Worker (1-way), Worker → Synthesis (1-way)
**Cross-verification**: By synthesis agent only
**Best for**: 5-15 independent tasks

### 2. Hierarchical (Complex Tasks)

```
Level 0: ORCHESTRATOR
            |
    +-------+-------+
    |       |       |
Level 1: PLANNER_1  PLANNER_2  PLANNER_3
            |       |       |
    +-------+   +---+---+   +-------+
    |   |       |   |   |       |   |
Level 2: C1 C2  C3  C4  C5      C6  C7  (Coders)
            |       |       |
    +-------+   +---+---+   +-------+
    |   |       |   |   |       |   |
Level 3: R1 R2  R3  R4  R5      R6  R7  (Reviewers)
```

**Communication**: Parent → Child (delegation), Child → Parent (results)
**Cross-verification**: Each level validates children
**Best for**: 20-50 tasks with complex dependencies

### 3. Peer Review (Quality-Critical)

```
CODER_AGENT → Output
                ↓
    +----------+----------+
    |          |          |
REVIEWER_1  REVIEWER_2  REVIEWER_3  (Parallel reviews)
    |          |          |
    +----------+----------+
                ↓
        CONSENSUS_AGENT
            (Majority voting)
```

**Communication**: All reviewers see same output
**Cross-verification**: 3+ independent reviews
**Best for**: Critical code (security, algorithms, architecture)

## Detailed Coordination Example

### Task: "Implement JWT Authentication"

```yaml
coordination_mode: HYBRID (parallel + cross-verification)

# Phase 1: Planning (Sequential, Cross-Verified)
planning:
  step_1:
    agent: ARCHITECT_AGENT
    output: auth_design.md
    
  step_2:
    agent: SECURITY_REVIEWER
    input: auth_design.md
    validates:
      - JWT algorithm (HS256 vs RS256)
      - Token expiry strategy
      - Refresh token flow
      - Secret management
    feedback: "Use RS256, rotate secrets, 15-min expiry"
    
  step_3:
    agent: ARCHITECT_AGENT
    input: SECURITY_REVIEWER.feedback
    output: auth_design_v2.md (revised)
    status: APPROVED

# Phase 2: Implementation (Parallel, Independent Files)
implementation:
  wave_1:  # No cross-verification needed (disjoint files)
    - agent: BACKEND_CODER_1
      task: JWT service (auth/jwt.go)
      
    - agent: BACKEND_CODER_2
      task: Middleware (auth/middleware.go)
      
    - agent: BACKEND_CODER_3
      task: User repository (user/repo.go)
      
    - agent: FRONTEND_CODER_1
      task: Auth service (auth.service.ts)

# Phase 3: Integration Testing (Cross-Verification)
integration:
  step_1:
    agent: TEST_AGENT_1
    tests: Unit tests for all components
    coverage: 87%
    
  step_2:
    agent: INTEGRATION_TEST_AGENT
    validates: End-to-end flow
    tests:
      - Login with valid credentials → Success
      - Login with invalid → 401
      - Expired token → 401 + refresh
      - Missing token → 401
    cross_verifies:
      - BACKEND_CODER_1.output
      - BACKEND_CODER_2.output
      - BACKEND_CODER_3.output
      - FRONTEND_CODER_1.output
    
  step_3:
    agent: SECURITY_TEST_AGENT
    validates: Security properties
    tests:
      - SQL injection attempts
      - XSS in login form
      - CSRF protection
      - Token replay attacks
    cross_verifies: ALL previous outputs

# Phase 4: Code Review (Peer Review Pattern)
code_review:
  reviewers:  # Parallel, independent reviews
    - REVIEWER_1:
        focus: Go backend code quality
        checks: [functions <50 lines, error handling, tests]
        score: 94/100
        
    - REVIEWER_2:
        focus: TypeScript frontend quality
        checks: [type safety, component structure]
        score: 91/100
        
    - REVIEWER_3:
        focus: Security
        checks: [secrets, validation, encryption]
        score: 96/100
  
  synthesis:
    agent: CONSENSUS_AGENT
    aggregates: [REVIEWER_1, REVIEWER_2, REVIEWER_3]
    average_score: 93.7/100
    threshold: 95/100
    decision: MINOR_REVISIONS_REQUIRED
    
    feedback:
      - "Add rate limiting to login endpoint"
      - "Increase JWT expiry to 30 min (UX feedback)"
    
    assigned_to: BACKEND_CODER_2

# Phase 5: Final Validation
final_validation:
  agent: SYNTHESIS_AGENT
  inputs:
    - All agent outputs
    - All test results
    - All review feedback
    - Revised code
  
  generates:
    - final_report.md (detailed analysis)
    - integration_test_results.html
    - security_audit_report.pdf
    - performance_benchmarks.json
  
  decision: APPROVED_FOR_MERGE
```

## When to Use Cross-Verification

### ✅ USE Cross-Verification For:
- Security-critical code
- Complex algorithms
- Shared state management
- API contracts between modules
- Database schema changes
- Architecture decisions

### ❌ SKIP Cross-Verification For:
- Independent UI components
- Disjoint file modifications
- Simple CRUD operations
- Documentation updates
- Test file generation

## Reference
- `.ai-config/rules/SPECIALIZED-SYSTEMS.md` (Section 1: Multi-Agent Orchestration)
- `.ai-config/commands/multi-agent-orchestrate.md`
