---
name: remote-server-ops
description: Execute commands on ES-1048 remote server (root@10.204.86.230)
shortcut: /remote
---

# Remote Server Operations

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-07  
**Server**: es1048-server (10.204.86.230)

Execute commands on the ES-1048 KMM validation environment.

## Server Configuration

### Connection Details
- **Hostname**: es1048-server
- **IP**: 10.204.86.230
- **User**: root
- **SSH**: `ssh root@10.204.86.230`

### Key Paths
```bash
# Remote server paths
REMOTE_BASE="/root/abhishek"
REMOTE_REPO="/root/abhishek/storage-interface-driver"
REMOTE_PROJECTS="/root/abhishek/PROJECTS"
FILESYSTEM_SOURCE="/root/abhishek/filesystem-client.tar.gz"

# Local paths (mapped)
LOCAL_BASE="<LOCAL_PROJECT_PATH>"
LOCAL_REPO="$LOCAL_BASE/storage-interface-driver"
```

## Common Operations

### Build Storage Interface Driver on Remote Server
```bash
# SSH into server and build
ssh root@10.204.86.230 << 'EOF'
cd /root/abhishek/storage-interface-driver
make build
make test
EOF
```

### Deploy KMM Manifests
```bash
# Deploy Distributed Filesystem module configuration
ssh root@10.204.86.230 << 'EOF'
cd /root/abhishek/storage-interface-driver
kubectl apply -f deploy/kubernetes/filesystem-module/filesystem-ubuntu24-dockerfile-configmap.yaml -n filesystem-kmm
kubectl apply -f deploy/kubernetes/filesystem-module/lnet-mod.yaml -n filesystem-kmm
EOF
```

### Check KMM Build Status
```bash
# Monitor build job
ssh root@10.204.86.230 'kubectl get pods -n filesystem-kmm -w'

# Get build logs
ssh root@10.204.86.230 'kubectl logs -n filesystem-kmm lnet-build-* -f'

# Check module status
ssh root@10.204.86.230 'kubectl get module lnet -n filesystem-kmm -o yaml'
```

### Manage Distributed Filesystem HTTP Server
```bash
# Start HTTP server for Distributed Filesystem source
ssh root@10.204.86.230 << 'EOF'
mkdir -p /tmp/filesystem-mock
cp /root/abhishek/filesystem-client.tar.gz /tmp/filesystem-mock/filesystem-client.tar.gz
cd /tmp/filesystem-mock
nohup python3 -m http.server 8080 --bind 10.204.86.230 > /tmp/http-server.log 2>&1 &
EOF

# Verify server
ssh root@10.204.86.230 'curl -I http://10.204.86.230:8080/filesystem-client.tar.gz'

# Stop server
ssh root@10.204.86.230 'pkill -f "python3 -m http.server 8080"'
```

### Docker Registry Operations
```bash
# Check registry
ssh root@10.204.86.230 'curl -s http://localhost:5000/v2/_catalog | jq'

# List images in registry
ssh root@10.204.86.230 'curl -s http://localhost:5000/v2/filesystem-client-ubuntu24/tags/list | jq'

# Push local image to remote registry
docker tag myimage:latest 10.204.86.230:5000/myimage:latest
docker push 10.204.86.230:5000/myimage:latest
```

### Kubernetes Operations
```bash
# Get cluster status
ssh root@10.204.86.230 'kubectl get nodes -o wide'

# Check KMM operator
ssh root@10.204.86.230 'kubectl get pods -n kmm-operator-system'

# List all resources in filesystem-kmm namespace
ssh root@10.204.86.230 'kubectl get all -n filesystem-kmm -o wide'

# Cleanup namespace
ssh root@10.204.86.230 'kubectl delete pods --all -n filesystem-kmm'
ssh root@10.204.86.230 'kubectl delete modules --all -n filesystem-kmm'
```

### Distributed Filesystem Filesystem Operations
```bash
# Check Distributed Filesystem mount
ssh root@10.204.86.230 'mount | grep filesystem'
ssh root@10.204.86.230 'df -h /mnt/testfs2'

# List Distributed Filesystem modules
ssh root@10.204.86.230 'lsmod | grep -E "filesystem|lnet|ksocklnd"'

# Distributed Filesystem utilities
ssh root@10.204.86.230 'lfs df -h /mnt/testfs2'
ssh root@10.204.86.230 'lctl get_param -n version'
```

## File Synchronization

### Sync Local to Remote
```bash
# Sync entire repository (exclude .git, bin, vendor)
rsync -avz --exclude='.git' --exclude='bin' --exclude='vendor' \
  <LOCAL_PROJECT_PATH>/storage-interface-driver/ \
  root@10.204.86.230:/root/abhishek/storage-interface-driver/

# Sync specific directory
rsync -avz ./pkg/driver/ root@10.204.86.230:/root/abhishek/storage-interface-driver/pkg/driver/
```

### Sync Remote to Local
```bash
# Download build artifacts
scp root@10.204.86.230:/root/abhishek/storage-interface-driver/bin/* ./bin/

# Download logs
scp root@10.204.86.230:/tmp/http-server.log ./logs/
```

## Validation Workflow

### Complete Deployment Test
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

# 1. Start Distributed Filesystem HTTP server
ssh root@10.204.86.230 << 'EOF'
pkill -f "python3 -m http.server 8080"
cp /root/abhishek/filesystem-client.tar.gz /tmp/filesystem-mock/filesystem-client.tar.gz
cd /tmp/filesystem-mock && nohup python3 -m http.server 8080 --bind 10.204.86.230 &
EOF

# 2. Deploy KMM manifests
ssh root@10.204.86.230 << 'EOF'
cd /root/abhishek/storage-interface-driver
kubectl apply -f deploy/kubernetes/filesystem-module/ -n filesystem-kmm
EOF

# 3. Monitor build
ssh root@10.204.86.230 'kubectl get pods -n filesystem-kmm -w'
```

## Reference
- Environment Setup: `ES-1048-ENVIRONMENT-SETUP.txt`
- Remote Settings: `.ai-config/settings.json` (remote section)
- Validation Docs: `/root/abhishek/PROJECTS/ES-1048/NEW-KMM-OPERATOR-APPROACH/demo-observation/`
