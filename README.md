# AI Dev Toolkit

AI agents, commands, and skills for software development.

---

## System Architecture: 9-Layer Coordinated Multi-Agent Orchestrator

```mermaid
graph TB
    subgraph Layer1["🎯 LAYER 1: INPUT"]
        U[User Request:<br/>Implement Feature X]
        CMD[Commands:<br/>Operational Workflows]
    end

    subgraph Layer2["⚙️ LAYER 2: CONFIGURATION"]
        CFG[System Config<br/>configs/]
        ENV[Environment Vars<br/>env.mcp.template]
        MCP[MCP Settings<br/>Agent Configs]
    end

    subgraph Layer3["🎭 LAYER 3: ORCHESTRATION"]
        ORCH[Orchestrator Agent<br/>CEO Model]
        DECOMP[Task Decomposition]
        ASSIGN[Task Assignment<br/>One Owner per Task]
        DAG[DAG Construction<br/>Dependencies + Waves]

        ORCH --> DECOMP
        DECOMP --> ASSIGN
        ASSIGN --> DAG
    end

    subgraph Layer4["🤖 LAYER 4: EXECUTION (Coordinated)"]
        subgraph Wave1["Wave 1: Research & Design"]
            T1[Task 1: Research<br/>👤 AI_RESEARCH]
            T2[Task 2: API Design<br/>👤 ARCHITECT]

            T1 -.communicates.-> T2
            T2 -.validates with.-> T1
        end

        subgraph Wave2["Wave 2: Implementation"]
            T3[Task 3: Backend<br/>👤 BACKEND_DEV]
            T4[Task 4: Frontend<br/>👤 FRONTEND_DEV]
            T5[Task 5: Database<br/>👤 DB_SPECIALIST]

            T3 -.shares context.-> T4
            T4 -.validates API.-> T3
            T3 -.requests schema.-> T5
            T5 -.confirms model.-> T3
        end

        subgraph Wave3["Wave 3: Testing"]
            T6[Task 6: Unit Tests<br/>👤 TEST_AUTO]
            T7[Task 7: Integration<br/>👤 QA_AUTO]
            T8[Task 8: Security<br/>👤 SECURITY]

            T6 -.shares coverage.-> T7
            T7 -.validates integration.-> T6
            T8 -.reports vulns.-> T6
        end

        COMM[Inter-Agent Communication<br/>Share + Validate + Request]

        T1 --> COMM
        T2 --> COMM
        T3 --> COMM
        T4 --> COMM
        T5 --> COMM
        T6 --> COMM
        T7 --> COMM
        T8 --> COMM
    end

    subgraph Layer5["📊 LAYER 5: EVIDENCE COLLECTION"]
        EV1[Agent 1 Output<br/>Code + Proof]
        EV2[Agent 2 Output<br/>Design Docs]
        EV3[Agent 3 Output<br/>Implementation + Tests]
        EV4[Agent 4 Output<br/>Test Reports + Coverage]
        EV5[Agent 5 Output<br/>Security Report]

        EV1 -.evidence.-> EV_POOL[Evidence Pool<br/>All Agent Outputs]
        EV2 -.evidence.-> EV_POOL
        EV3 -.evidence.-> EV_POOL
        EV4 -.evidence.-> EV_POOL
        EV5 -.evidence.-> EV_POOL
    end

    subgraph Layer6["🔍 LAYER 6: SYNTHESIS (7-Step Process)"]
        SYN1[Step 1:<br/>Consolidate All Outputs]
        SYN2[Step 2:<br/>Cross-Verify Evidence]
        SYN3[Step 3:<br/>Detect Conflicts]
        SYN4[Step 4:<br/>Resolve Conflicts]
        SYN5[Step 5:<br/>Validate Root Causes]
        SYN6[Step 6:<br/>Verify Dependencies]
        SYN7[Step 7:<br/>Check Quality Gates]

        SYN1 --> SYN2
        SYN2 --> SYN3
        SYN3 --> SYN4
        SYN4 --> SYN5
        SYN5 --> SYN6
        SYN6 --> SYN7
    end

    subgraph Layer7["✅ LAYER 7: QUALITY ASSURANCE"]
        RULES[Engineering Rules<br/>CORE-ENGINEERING-RULES.md]
        REVIEW[Code Review<br/>👤 CODE_REVIEWER]
        TEST[Testing Validation<br/>👤 TEST_AUTO]

        RULES -.enforces.-> REVIEW
        RULES -.enforces.-> TEST
        REVIEW -.validates.-> TEST
        TEST -.confirms.-> REVIEW
    end

    subgraph Layer8["🛡️ LAYER 8: FINAL VALIDATION"]
        VAL1[Rules Compliance<br/>Check CORE + SPECIALIZED]
        VAL2[Evidence-Based Review<br/>All Claims Verified]
        VAL3[End-to-End Verification<br/>Complete Flow Check]

        VAL1 --> VAL2
        VAL2 --> VAL3
    end

    subgraph Layer9["📤 LAYER 9: OUTPUT"]
        REPORT[Comprehensive Report<br/>NOT Simple Aggregation<br/><br/>✅ Conflicts Resolved<br/>✅ Root Causes Validated<br/>✅ Evidence-Based<br/>✅ Cross-Verified<br/>✅ Dependencies Verified<br/>✅ Rules Compliant]
    end

    U --> CMD
    CMD --> ORCH
    CFG --> ORCH
    ENV --> MCP
    MCP --> DAG

    DAG --> T1
    DAG --> T2
    DAG --> T3
    DAG --> T4
    DAG --> T5
    DAG --> T6
    DAG --> T7
    DAG --> T8

    T1 --> EV1
    T2 --> EV2
    T3 --> EV3
    T4 --> EV3
    T5 --> EV3
    T6 --> EV4
    T7 --> EV4
    T8 --> EV5

    EV_POOL --> SYN1

    SYN7 --> RULES
    RULES --> VAL1

    VAL3 --> REPORT

    classDef input fill:#6f42c1,stroke:#5a32a3,stroke-width:3px,color:#fff
    classDef config fill:#fd7e14,stroke:#e8590c,stroke-width:2px,color:#fff
    classDef orchestrate fill:#dc3545,stroke:#bd2130,stroke-width:3px,color:#fff
    classDef execute fill:#007bff,stroke:#0056b3,stroke-width:2px,color:#fff
    classDef evidence fill:#17a2b8,stroke:#117a8b,stroke-width:2px,color:#fff
    classDef synthesis fill:#28a745,stroke:#1e7e34,stroke-width:3px,color:#fff
    classDef quality fill:#ffc107,stroke:#d39e00,stroke-width:2px,color:#000
    classDef validation fill:#6f42c1,stroke:#5a32a3,stroke-width:2px,color:#fff
    classDef output fill:#fd7e14,stroke:#e8590c,stroke-width:3px,color:#fff

    class U,CMD input
    class CFG,ENV,MCP config
    class ORCH,DECOMP,ASSIGN,DAG orchestrate
    class T1,T2,T3,T4,T5,T6,T7,T8,COMM execute
    class EV1,EV2,EV3,EV4,EV5,EV_POOL evidence
    class SYN1,SYN2,SYN3,SYN4,SYN5,SYN6,SYN7 synthesis
    class RULES,REVIEW,TEST quality
    class VAL1,VAL2,VAL3 validation
    class REPORT output
```

---

## Architecture Breakdown

**9-Layer Flow:**

1. **INPUT** → User request + commands
2. **CONFIGURATION** → System config (MCP, env, settings)
3. **ORCHESTRATION** → Task decomposition → assignment (one owner per task) → DAG construction
4. **EXECUTION** → 8 tasks in 3 waves, agents communicate (not independent)
5. **EVIDENCE** → All agents provide proof (code, tests, logs, metrics)
6. **SYNTHESIS** → 7-step process (consolidate → cross-verify → detect conflicts → resolve → validate root causes → verify dependencies → quality gates)
7. **QUALITY** → Rules enforcement + code review + testing validation
8. **VALIDATION** → 3-step final check (compliance → evidence review → end-to-end)
9. **OUTPUT** → Comprehensive report (NOT aggregation, includes conflict resolution)

**Coordination Patterns:**
- Task ownership: Each task = 1 dedicated agent (e.g., T1 → AI_RESEARCH)
- Inter-agent communication: Dotted lines show context sharing, validation, requests
- Evidence required: All agents must provide proof/validation
- Synthesis: NOT simple aggregation, 7-step comprehensive analysis
- Validation: Multi-layer checks before final output

**Scales by complexity:**
- Simple (1-5 min): 1 agent, direct path
- Medium (5-15 min): 5-10 agents, 2 waves, synthesis
- Complex (15-60 min): 20-30 agents, 3+ waves, full 9-layer flow

---

## Repository Structure

```
ai-dev-toolkit/
├── agents/     49 specialized agents
├── commands/   28 operational commands
├── skills/     17 domain modules
├── configs/    MCP, env, settings
└── rules/      Engineering standards
```

---

## Agents (49 total)

**Development (8):** code-assistant, code-reviewer, test-automation, go-linter, docker-builder, backend-dev, frontend-dev, refactoring

**Infrastructure (10):** k8s-troubleshooter, storage-debugger, devops, iac, cloud-architect, network, sre, monitoring, capacity-planner, cost-optimizer

**Security (6):** security-auditor, security-tester, secrets-manager, compliance-auditor, compliance-validator, chaos-engineer

**Docs/Research (5):** doc-generator, doc-writer, ai-research, research-assistant, code-explainer

**Specialized (20):** linkedin, prompt-engineer, git-identity, ml-trainer, database, api-designer, microservices, openshift, azure, filesystem, snapshot, manifest-validator, integration-tester, load-tester, qa-automation, rag-specialist, unified-doc, incident-responder, release-manager, performance-optimizer

---

## Commands (28 total)

**Core:** fix, review, test, deploy-driver, build-docker-images

**Dev:** code-review, generate-code-ai, format-go-code, run-all-tests, validate-k8s-manifests

**Docs:** generate-docs, document-fix, live-doc, explain-code

**Infra:** remote-server-ops, k8s-integration, benchmark-performance, system-diag

**Orchestration:** multi-agent-orchestrate, agent-coordinate, dag-visualize, create-agent

**Research/AI:** ai-research, read-cv, mcp-connect

---

## Skills (17 total)

**Storage:** storage-driver-development, storage-troubleshooting, distributed-filesystem, filesystem-performance-tuning

**Kubernetes:** kubernetes-storage, kubernetes-operator-patterns, openshift-deployment, container-security

**Development:** go-best-practices, grpc-development, docker-optimization, ci-cd-pipelines

**AI:** ai-assisted-development, llm-integration, prompt-engineering, research-methodology

**Other:** big-data-connector

---

## Configuration

```bash
# Setup
cp configs/env.mcp.template ~/.ai-config/env.mcp
# Edit with API keys
nano ~/.ai-config/env.mcp
# Source
echo 'source ~/.ai-config/env.mcp' >> ~/.zshrc
source ~/.ai-config/env.mcp
```

**MCP configs:** mcp-config-active.json, mcp-config-50-servers.json, mcp-config-enhanced.json
**Terminal:** starship.toml, tmux.conf, zshrc-enhanced, zshrc-remote

---

## Engineering Rules

**CORE-ENGINEERING-RULES.md:** Coding standards (50-line functions, 80%+ coverage), security (no hardcoded secrets), testing, Git workflow
**SPECIALIZED-SYSTEMS.md:** Multi-agent orchestration (46+ agents), token optimization (65-75% reduction), system optimization

---

## Quick Start

```bash
# Setup
git clone <repo-url> ai-dev-toolkit
cd ai-dev-toolkit
cp configs/env.mcp.template ~/.ai-config/env.mcp
# Edit with API keys

# Usage
/review path/to/code
/generate-docs path/to/project
/deploy-driver --namespace storage-driver
/multi-agent-orchestrate --task "Implement feature X"
```

**Placeholders to replace:** `<AUTHOR_NAME>`, `<PERSONAL_EMAIL>`, `<WORK_EMAIL>`, `<GITHUB_HANDLE>`, `<USER_HOME>`

---

## Security

**Credentials:** All use environment variables. Zero hardcoded secrets. Templates use placeholders only.
**Audit:** 14-point security scan passed. Zero credentials found. Production-ready.

---

## Statistics

49 agents, 28 commands, 17 skills, 2 rule files, 100+ total files

---

## Contributing

Follow `rules/CORE-ENGINEERING-RULES.md`: functions <50 lines, @author tags, 80%+ coverage, zero lint errors, no hardcoded secrets

---

## License

Configuration templates and documentation. Customize for your use.

---

## Support

Check `/agents/`, `/commands/`, `/skills/`, or `/rules/` for guidance

---

**Status:** Production-ready | **Updated:** 2026-07-11
