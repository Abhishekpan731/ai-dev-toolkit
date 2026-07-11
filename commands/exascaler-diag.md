---
name: system-diag
description: Quick diagnostics for Storage System storage interface deployment and performance
aliases: [exa-diag, filesystem-diag]
---

# `/system-diag` - Storage System Storage Interface Diagnostics

## Usage

```bash
/system-diag                    # Full diagnostic report
/system-diag driver             # Driver status only
/system-diag performance        # Performance metrics
/system-diag config             # Configuration validation
/system-diag network            # Network/LNet status
/system-diag volumes            # Volume health check
```

## What It Does

Automated diagnostics for storage provider Storage System storage interface deployments with actionable recommendations.

## Diagnostic Checks

### 1. Driver Status
```bash
# Check all driver components
kubectl get pods -n kube-system -l app=storage-interface-driver
kubectl get daemonset -n kube-system storage-system-node-server
kubectl get deployment -n kube-system storage-system-controller-server

# Check storage interface registration
kubectl get csidriver exa.csi.ddn.com

# Check driver version
kubectl describe pod -n kube-system -l app=storage-interface-driver | grep Image:
```

**Expected**:
- All pods Running
- storage interface registered
- Version >= 2.7.0 for latest features

### 2. Configuration Validation
```bash
# Check config map
kubectl get configmap -n kube-system storage-interface-driver-config -o yaml

# Validate storage-system_map structure
# - exaFS format: IP@proto:/path
# - mountPoint: absolute path
# - zone: matches topology
```

**Common Issues**:
- Missing zone in config but topology enabled
- Invalid exaFS format
- Duplicate mountPoint across filesystems

### 3. Network/LNet Status
```bash
# On each node, check LNet
lctl network show
lctl list_nids

# Verify connectivity to Distributed Filesystem MGS/MDS/OSS
lctl ping <mgs-nid>

# Check multi-rail configuration
lctl list_nids  # Should show multiple NIDs if multi-rail enabled
```

**Expected Output**:
```
10.1.1.100@tcp
10.1.2.100@tcp  # Second rail
```

### 4. Mount Status
```bash
# Check mounted filesystems
mount | grep filesystem

# Check mount health
lfs check servers
lfs df -h

# Check client parameters
lctl get_param osc.*.checksums
lctl get_param llite.*.max_read_ahead_mb
```

### 5. Volume Health
```bash
# List all Storage System PVCs
kubectl get pvc --all-namespaces -o json | \
  jq '.items[] | select(.metadata.annotations."volume.beta.kubernetes.io/storage-provisioner" == "exa.csi.ddn.com")'

# Check volume usage
for pvc in $(kubectl get pvc -o name); do
  echo "=== $pvc ==="
  kubectl exec -it <pod-using-pvc> -- df -h /mount/path
done
```

### 6. Snapshot Status
```bash
# List snapshot classes
kubectl get volumesnapshotclass | grep storage-system

# List snapshots
kubectl get volumesnapshot --all-namespaces

# Check snapshot readiness
kubectl get volumesnapshot <snapshot-name> -o jsonpath='{.status.readyToUse}'
```

### 7. Metrics Check
```bash
# Check metrics exporter
kubectl get daemonset -n kube-system exa-csi-metrics-exporter

# Query metrics
kubectl port-forward -n kube-system ds/exa-csi-metrics-exporter 9090:9090 &
curl http://localhost:9090/metrics | grep exa_csi_pvc

# Sample metrics
# exa_csi_pvc_capacity_bytes{pvc="test-pvc",storage_class="storage-system"} 10737418240
# exa_csi_pvc_used_bytes{pvc="test-pvc",storage_class="storage-system"} 1073741824
```

### 8. Performance Analysis
```bash
# Check Hot Nodes (PCC) status
lctl get_param llite.*.pcc

# Check compression status
lctl get_param llite.*.compression

# Run basic I/O test
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: fio-test
spec:
  containers:
  - name: fio
    image: ljishen/fio
    command: ["fio"]
    args:
      - "--name=seqread"
      - "--rw=read"
      - "--bs=1M"
      - "--size=10G"
      - "--numjobs=4"
      - "--directory=/mnt/test"
    volumeMounts:
    - name: test-vol
      mountPath: /mnt/test
  volumes:
  - name: test-vol
    persistentVolumeClaim:
      claimName: test-pvc
EOF
```

## Automated Report

Output format:
```
=== STORAGE_SYSTEM Storage Interface DRIVER DIAGNOSTICS ===

Driver Status:      ✅ Healthy (v2.7.0)
Configuration:      ✅ Valid
LNet Status:        ✅ Multi-rail active (2 NIDs)
Mount Health:       ✅ All filesystems accessible
Volume Count:       42 PVCs (38 healthy, 4 warnings)
Snapshot Count:     12 (all ready)
Metrics:            ✅ Exporter running

⚠️  WARNINGS:
- PVC "large-dataset" at 95% capacity
- Node "worker-5" missing second NID (single-rail)
- StorageClass "old-class" using tar snapshots (slow)

📊 PERFORMANCE:
- Average throughput: 8.5 GB/s per node
- Hot Nodes hit rate: 87%
- Snapshot creation: 2.3 TB/min

💡 RECOMMENDATIONS:
1. Expand "large-dataset" PVC or add quota alerts
2. Configure second NIC on worker-5 for multi-rail
3. Migrate to dtar snapshot mode for 10x faster snapshots
4. Enable compression for scratch workloads (30% space savings)
```

## Remediation Actions

### Fix Missing Multi-Rail
```bash
# Add second NID to node
lctl network configure --net tcp1 --if eth1
systemctl restart lnet

# Verify
lctl list_nids
# Should show both tcp0 and tcp1
```

### Fix Low Capacity
```bash
# Expand PVC
kubectl patch pvc large-dataset -p '{"spec":{"resources":{"requests":{"storage":"200Gi"}}}}'

# Verify expansion
kubectl get pvc large-dataset -w
```

### Migrate to dtar Snapshots
```bash
# Update config map
kubectl edit configmap -n kube-system storage-interface-driver-config

# Change:
# snapshotUtility: tar
# to:
# snapshotUtility: dtar

# Restart driver
kubectl rollout restart daemonset -n kube-system storage-system-node-server
```

### Enable Compression
```yaml
# Update StorageClass
kubectl edit storageclass storage-system-scratch

# Add parameter:
parameters:
  compression: "lz4"
```

## Exit Codes

- **0**: All checks passed
- **1**: Warnings present (non-critical)
- **2**: Errors detected (action required)

## Pro Tips

1. **Run before production deployment** - Catch config issues early
2. **Schedule weekly** - Automated health checks via CronJob
3. **Integrate with CI/CD** - Verify changes don't break driver
4. **Export metrics** - Track trends over time in Grafana

---

**Run time**: ~30 seconds for full diagnostics
**Output**: Terminal + optional JSON export
