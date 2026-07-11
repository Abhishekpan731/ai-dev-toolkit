# `/mcp-connect` - MCP Server Management & Troubleshooting

**Author**: <AUTHOR_NAME>  
**Created**: 2026-07-07  
**Category**: Infrastructure  
**Tags**: `mcp`, `troubleshooting`, `infrastructure`, `automation`

## Overview

Comprehensive MCP (Model Context Protocol) server management command. Automatically detects issues, repairs configuration, starts required services, and verifies all MCP servers are operational.

## Usage

```bash
/mcp-connect                    # Full check and auto-repair
/mcp-connect --test             # Test all servers
/mcp-connect --start            # Start required services
/mcp-connect --repair           # Repair configuration
/mcp-connect --status           # Show status only
/mcp-connect --verbose          # Detailed diagnostics
```

## What It Does

### 1. **Service Health Check**
- ✅ Verifies Ollama is running (starts if needed)
- ✅ Checks kubectl access
- ✅ Validates Docker daemon
- ✅ Tests database connections (PostgreSQL, MongoDB, Redis)

### 2. **Configuration Validation**
- ✅ Verifies MCP config file exists and is valid JSON
- ✅ Checks environment variables are set
- ✅ Validates package availability (npm/npx/uvx)
- ✅ Ensures file permissions are correct

### 3. **MCP Server Testing**
- ✅ Tests all 15 FREE servers individually
- ✅ Verifies package availability
- ✅ Checks server responsiveness
- ✅ Reports which servers are failing

### 4. **Auto-Repair**
- ✅ Starts Ollama if stopped
- ✅ Recreates missing environment files
- ✅ Fixes file permissions
- ✅ Updates packages if needed
- ✅ Restarts failed services

### 5. **Status Report**
- ✅ Shows running/stopped services
- ✅ Lists available MCP servers
- ✅ Reports Ollama models
- ✅ Displays resource usage

## Examples

### Quick Health Check
```bash
/mcp-connect
```
**Output**: Runs full diagnostics, auto-repairs issues, shows status.

### Just Test Servers
```bash
/mcp-connect --test
```
**Output**: Tests all MCP servers without making changes.

### Start All Services
```bash
/mcp-connect --start
```
**Output**: Starts Ollama, checks databases, verifies all prerequisites.

### Repair Configuration
```bash
/mcp-connect --repair
```
**Output**: Fixes config files, permissions, environment variables.

### Detailed Diagnostics
```bash
/mcp-connect --verbose
```
**Output**: 
```
╔═══════════════════════════════════════════════╗
║  MCP DIAGNOSTIC REPORT - VERBOSE MODE        ║
╚═══════════════════════════════════════════════╝

[1/6] Checking Prerequisites...
  ✅ Node.js v26.0.0 (/opt/homebrew/bin/node)
  ✅ npm 11.12.1
  ✅ uvx 0.11.19
  ✅ kubectl v1.36.1
  ✅ Docker 24.0.2

[2/6] Checking Services...
  ✅ Ollama: Running (PID 52773)
     • Models: qwen3-coder:30b
     • Memory: 450MB
  ⚠️  ChromaDB: Not running
  ✅ Docker: Running

[3/6] Testing MCP Servers...
  ✅ Ollama MCP (ollama-mcp)
  ✅ Kubernetes MCP (kubernetes-mcp-server@latest)
  ✅ Memory MCP (@modelcontextprotocol/server-memory)
  ...

[4/6] Validating Configuration...
  ✅ Config: .ai-config/configs/mcp/mcp-config-active.json
  ✅ Environment: ~/.ai-config/env.mcp
  ✅ Permissions: OK

[5/6] Resource Usage...
  • Total MCP Memory: ~500MB
  • Ollama Memory: ~450MB
  • Available Disk: 120GB

[6/6] Summary...
  Status: 13/15 servers operational
  Issues: ChromaDB not running, Brave Search not configured
  Cost: $0/month
```

## Technical Details

### Script Location
`.ai-config/scripts/mcp-connect.sh`

### Configuration Files Checked
- `.ai-config/configs/mcp/mcp-config-active.json`
- `~/.ai-config/env.mcp`
- `.ai-config/scripts/test-mcp-servers.sh`

### Services Managed
1. **Ollama** - Local LLM server (port 11434)
2. **ChromaDB** - Vector database (port 8000)
3. **Docker** - Container runtime
4. **PostgreSQL** - Database (port 5432)
5. **MongoDB** - NoSQL database (port 27017)
6. **Redis** - Cache (port 6379)

### MCP Servers Tested (15)
- Ollama, Kubernetes, Memory, Sequential Thinking
- Git, Filesystem, SQLite, Docker
- Playwright, Puppeteer
- PostgreSQL, MongoDB, Redis (optional)
- Brave Search, Everything (optional)

## Troubleshooting

### Issue: "Ollama not running"
**Solution**: Command automatically runs `ollama serve &`

### Issue: "MCP config not found"
**Solution**: Command recreates from template

### Issue: "Server test failed"
**Solution**: Check verbose output, update package, or disable server

### Issue: "Environment not loaded"
**Solution**: Run `source ~/.ai-config/env.mcp`

## Integration

Works seamlessly with:
- `/k8s-integration` - Kubernetes operations
- `/test` - Testing workflows
- `/debug-csi-issue` - Storage Interface debugging
- All other commands requiring MCP servers

## Performance

- **Execution Time**: 15-30 seconds (full check)
- **Startup Time**: <5 seconds (Ollama already running)
- **Memory Overhead**: ~50MB (script + checks)
- **Network**: Local only (no external calls)

## Success Criteria

Command succeeds when:
- ✅ All required services are running
- ✅ All MCP servers respond to health checks
- ✅ Configuration files are valid
- ✅ Environment is properly loaded
- ✅ No permission errors

## Notes

- **Free Only**: Only manages FREE servers (no API keys)
- **Non-Destructive**: Never deletes data or stops running services
- **Idempotent**: Safe to run multiple times
- **Fast**: Cached checks, parallel execution

## See Also

- `/k8s-integration` - Kubernetes management
- `.ai-config/configs/mcp/README-FREE-SETUP.md` - Setup guide
- `.ai-config/scripts/test-mcp-servers.sh` - Test suite
