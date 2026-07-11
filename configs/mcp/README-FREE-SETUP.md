# MCP Free-Only Configuration
> @author <AUTHOR_NAME>
> Last updated: July 2026

## Active MCP Servers (100% FREE - No API Keys Required)

### Core Productivity (Always Running)
✅ **Ollama** - Local LLMs (Llama 3.1, CodeLlama, Mistral)
✅ **ChromaDB** - Vector database for RAG
✅ **Memory** - Long-term persistent graph memory
✅ **Sequential Thinking** - Structured reasoning

### Infrastructure & Kubernetes (Critical for Storage Interface Work)
✅ **Kubernetes** - Cluster management via local kubeconfig
  - Get pods, deployments, services
  - View logs, exec into containers
  - Works with any K8s cluster you have kubeconfig access to

### Development Tools
✅ **Docker** - Container management (ps, images, build, run, logs, exec)
✅ **Git** - Repository management (status, diff, log, commit, branch)
✅ **Filesystem** - Multi-repository access
  - `<LOCAL_PROJECT_PATH>`
  - `~/.external-repos/storage-interface-host-path`
  - `~/.external-repos/csi-lib-utils`
  - `~/.external-repos/csi-spec`
✅ **Playwright** - Browser automation (navigate, screenshot, PDF)
✅ **Puppeteer** - Headless browser control

### Databases (Local/Optional)
✅ **SQLite** - Local database (always available)
⚪ **PostgreSQL** - If running locally (autoStart: false)
⚪ **MongoDB** - If running locally (autoStart: false)
⚪ **Redis** - If running locally (autoStart: false)

### Search & Utilities
✅ **Brave Search** - Free web search (2000 queries/month, no API key)
✅ **Everything** - macOS Spotlight search (local files/apps)

## Disabled Servers (Require API Keys)
❌ **GitHub** - Requires GITHUB_TOKEN
❌ **OpenAI** - Requires OPENAI_API_KEY

## Known Issues
⚠️ **AI Context Engine** - DISABLED due to 130GB+ memory leak on ARM64
- Workaround: Use shell aliases (ag-csi) or `.ai-config/scripts/auggie-safe.sh`

## Environment Setup

### Step 1: Copy environment template
```bash
cp .ai-config/configs/env.mcp.template ~/.ai-config/env.mcp
```

### Step 2: Configure free services
Edit `~/.ai-config/env.mcp` and set these (no API keys needed):

```bash
# Ollama (Local LLMs)
export OLLAMA_HOST="http://localhost:11434"

# ChromaDB (Local)
export CHROMA_HOST="localhost"
export CHROMA_PORT="8000"

# SQLite
export SQLITE_DB_PATH="./.data/local.db"

# Kubernetes (uses your existing kubeconfig)
export KUBECONFIG="~/.kube/config"
```

### Step 3: Load environment
```bash
source ~/.ai-config/env.mcp
```

### Step 4: Verify MCP servers
The active configuration is in `.ai-config/configs/mcp/mcp-config-active.json`

## Prerequisites

### Required (Free)
- **Node.js/npm** - For npx-based servers
- **Python/uv** - For Git server (`uvx`)
- **Docker** - For Docker MCP
- **Ollama** - Download from https://ollama.ai

### Optional (Free, if you want these features)
- **PostgreSQL** - `brew install postgresql` (macOS)
- **MongoDB** - `brew install mongodb-community` (macOS)
- **Redis** - `brew install redis` (macOS)
- **ChromaDB** - `pip install chromadb && chroma run` (runs on port 8000)

## Quick Start

### 1. Start Ollama
```bash
ollama serve
# In another terminal:
ollama pull llama3.1
ollama pull codellama
```

### 2. Start ChromaDB (optional, for RAG)
```bash
pip install chromadb
chroma run --host localhost --port 8000
```

### 3. Verify Kubernetes access
```bash
kubectl cluster-info
kubectl get nodes
```

### 4. Test MCP servers
All servers with `autoStart: true` will start automatically when AI IDE loads.

## Usage Tips

### Kubernetes Management
- "Show me all pods in the storage-system namespace"
- "Get logs for csi-controller pod"
- "Check deployment status"

### Local Development
- "Search my codebase for CreateVolume implementation"
- "Show git status and recent commits"
- "List all Docker containers"

### Research & Memory
- "Search web for Kubernetes Storage Interface best practices" (via Brave)
- "Remember that we use dtar for snapshots in production"
- "What did we discuss about PCC optimization?"

## Cost: $0/month
All configured servers are 100% free with no usage limits on local operations.

Brave Search has a free tier of 2000 queries/month (no credit card required).
