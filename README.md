# AI Dev Toolkit

AI agents, commands, and skills for software development.

---

## System Architecture: 9-Layer Coordinated Multi-Agent Orchestrator

```mermaid
%%{init: {'theme':'dark', 'themeVariables': { 'fontSize':'18px', 'fontFamily':'arial', 'darkMode':true, 'background':'transparent', 'primaryColor':'#1f2937', 'primaryTextColor':'#fff', 'primaryBorderColor':'#60a5fa', 'lineColor':'#60a5fa', 'secondaryColor':'#374151', 'tertiaryColor':'#4b5563'}}}%%
graph TB
    subgraph L1["<b>🎯 LAYER 1: INPUT</b>"]
        U["<b>User Request</b><br/><font size=5>Implement Feature X</font>"]
        CMD["<b>Commands</b><br/><font size=5>Operational Workflows</font>"]
    end

    subgraph L2["<b>⚙️ LAYER 2: CONFIGURATION</b>"]
        CFG["<b>System Config</b><br/><font size=5>configs/</font>"]
        ENV["<b>Environment Vars</b><br/><font size=5>env.mcp.template</font>"]
        MCP["<b>MCP Settings</b><br/><font size=5>Agent Configs</font>"]
    end

    subgraph L3["<b>🎭 LAYER 3: ORCHESTRATION</b>"]
        ORCH["<b>Orchestrator Agent</b><br/><font size=5>CEO Model</font>"]
        DECOMP["<b>Task Decomposition</b>"]
        ASSIGN["<b>Task Assignment</b><br/><font size=5>One Owner per Task</font>"]
        DAG["<b>DAG Construction</b><br/><font size=5>Dependencies + Waves</font>"]

        ORCH --> DECOMP
        DECOMP --> ASSIGN
        ASSIGN --> DAG
    end

    subgraph L4["<b>🤖 LAYER 4: EXECUTION (Coordinated)</b>"]
        T1["<b>Research</b><br/><font size=5>👤 AI_RESEARCH</font>"]
        T2["<b>API Design</b><br/><font size=5>👤 ARCHITECT</font>"]
        T3["<b>Backend</b><br/><font size=5>👤 BACKEND_DEV</font>"]
        T4["<b>Frontend</b><br/><font size=5>👤 FRONTEND_DEV</font>"]
        T5["<b>Database</b><br/><font size=5>👤 DB_SPECIALIST</font>"]
        T6["<b>Unit Tests</b><br/><font size=5>👤 TEST_AUTO</font>"]
        T7["<b>Integration</b><br/><font size=5>👤 QA_AUTO</font>"]
        T8["<b>Security</b><br/><font size=5>👤 SECURITY</font>"]

        T1 -.communicates.-> T2
        T2 -.validates.-> T1
        T3 -.shares.-> T4
        T4 -.validates.-> T3
        T3 -.requests.-> T5
        T5 -.confirms.-> T3
        T6 -.coverage.-> T7
        T7 -.validates.-> T6
        T8 -.reports.-> T6

        COMM["<b>Inter-Agent Communication</b><br/><font size=5>Share + Validate + Request</font>"]
        T1 --> COMM
        T2 --> COMM
        T3 --> COMM
        T4 --> COMM
        T5 --> COMM
        T6 --> COMM
        T7 --> COMM
        T8 --> COMM
    end

    subgraph L5["<b>📊 LAYER 5: EVIDENCE COLLECTION</b>"]
        EV1["<b>Agent 1</b><br/><font size=5>Code + Proof</font>"]
        EV2["<b>Agent 2</b><br/><font size=5>Design Docs</font>"]
        EV3["<b>Agent 3</b><br/><font size=5>Implementation</font>"]
        EV4["<b>Agent 4</b><br/><font size=5>Test Reports</font>"]
        EV5["<b>Agent 5</b><br/><font size=5>Security Report</font>"]

        EV_POOL["<b>Evidence Pool</b><br/><font size=5>All Agent Outputs</font>"]

        EV1 --> EV_POOL
        EV2 --> EV_POOL
        EV3 --> EV_POOL
        EV4 --> EV_POOL
        EV5 --> EV_POOL
    end

    subgraph L6["<b>🔍 LAYER 6: SYNTHESIS (7-Step)</b>"]
        SYN1["<b>1. Consolidate</b><br/><font size=5>All Outputs</font>"]
        SYN2["<b>2. Cross-Verify</b><br/><font size=5>Evidence</font>"]
        SYN3["<b>3. Detect</b><br/><font size=5>Conflicts</font>"]
        SYN4["<b>4. Resolve</b><br/><font size=5>Conflicts</font>"]
        SYN5["<b>5. Validate</b><br/><font size=5>Root Causes</font>"]
        SYN6["<b>6. Verify</b><br/><font size=5>Dependencies</font>"]
        SYN7["<b>7. Check</b><br/><font size=5>Quality Gates</font>"]

        SYN1 --> SYN2 --> SYN3 --> SYN4 --> SYN5 --> SYN6 --> SYN7
    end

    subgraph L7["<b>✅ LAYER 7: QUALITY ASSURANCE</b>"]
        RULES["<b>Engineering Rules</b><br/><font size=5>CORE-ENGINEERING</font>"]
        REVIEW["<b>Code Review</b><br/><font size=5>👤 CODE_REVIEWER</font>"]
        TEST["<b>Testing</b><br/><font size=5>👤 TEST_AUTO</font>"]

        RULES -.enforces.-> REVIEW
        RULES -.enforces.-> TEST
        REVIEW -.validates.-> TEST
        TEST -.confirms.-> REVIEW
    end

    subgraph L8["<b>🛡️ LAYER 8: FINAL VALIDATION</b>"]
        VAL1["<b>Rules Compliance</b><br/><font size=5>Check Standards</font>"]
        VAL2["<b>Evidence Review</b><br/><font size=5>Verify Claims</font>"]
        VAL3["<b>End-to-End</b><br/><font size=5>Complete Check</font>"]

        VAL1 --> VAL2 --> VAL3
    end

    subgraph L9["<b>📤 LAYER 9: OUTPUT</b>"]
        REPORT["<b>Comprehensive Report</b><br/><font size=5>NOT Simple Aggregation</font><br/><br/><font size=5>✅ Conflicts Resolved<br/>✅ Root Causes Validated<br/>✅ Evidence-Based<br/>✅ Cross-Verified<br/>✅ Rules Compliant</font>"]
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

    classDef input fill:#7c3aed,stroke:#a78bfa,stroke-width:4px,color:#fff,font-size:16px
    classDef config fill:#f59e0b,stroke:#fbbf24,stroke-width:3px,color:#fff,font-size:16px
    classDef orchestrate fill:#ef4444,stroke:#f87171,stroke-width:4px,color:#fff,font-size:16px
    classDef execute fill:#3b82f6,stroke:#60a5fa,stroke-width:3px,color:#fff,font-size:16px
    classDef evidence fill:#06b6d4,stroke:#22d3ee,stroke-width:3px,color:#fff,font-size:16px
    classDef synthesis fill:#10b981,stroke:#34d399,stroke-width:4px,color:#fff,font-size:16px
    classDef quality fill:#eab308,stroke:#facc15,stroke-width:3px,color:#000,font-size:16px
    classDef validation fill:#8b5cf6,stroke:#a78bfa,stroke-width:3px,color:#fff,font-size:16px
    classDef output fill:#f97316,stroke:#fb923c,stroke-width:4px,color:#fff,font-size:16px

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
