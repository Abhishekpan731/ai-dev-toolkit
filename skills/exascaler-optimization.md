---
name: storage-system-optimization
description: >
  Automated optimization workflow for storage provider Storage System storage interface.
  Analyzes workload patterns, configures performance features (Hot Nodes, compression,
  multi-rail), and generates production-ready StorageClass/deployment manifests.
globs: ["storage-interface-driver/**/*", "*.yaml"]
---

# Storage System Storage Interface Driver Optimization Skill

## Purpose

Automate the configuration of storage provider Storage System storage interface for production AI/HPC workloads with optimal performance settings.

## Workflow

### 1. Workload Analysis
**Identify workload type**:
- AI/ML Training (repeated reads, large datasets)
- Checkpoint/Restore (snapshots, versioning)
- Scratch/Temp (high I/O, transient data)
- General Purpose (mixed workload)

### 2. Feature Recommendations

#### For AI/ML Training
**Primary**: Hot Nodes (PCC caching)
**Secondary**: Multi-rail networking, compression
```yaml
# Recommended StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-ai-training
provisioner: exa.csi.ddn.com
allowVolumeExpansion: true
parameters:
  hotNodes: "true"
  pccMode: "auto"
  compression: "lz4"
  exaParams: "osc.*.checksums=0 llite.*.max_read_ahead_mb=128"
mountOptions:
  - "flock"
  - "lnet.networks=tcp0(eth0),tcp1(eth1)"
volumeBindingMode: WaitForFirstConsumer
```

#### For Checkpoint/Restore
**Primary**: Fast snapshots (dtar)
**Secondary**: Topology awareness
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-checkpoint
provisioner: exa.csi.ddn.com
allowVolumeExpansion: true
parameters:
  snapshotUtility: "dtar"
  exaParams: "osc.*.checksums=0 ldlm.namespaces.*.lru_max_age=10000000"
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
  - matchLabelExpressions:
    - key: topology.kubernetes.io/zone
      values: ["zone-1"]
```

#### For Scratch/Temp
**Primary**: Compression, performance tuning
**Secondary**: No snapshots needed
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-scratch
provisioner: exa.csi.ddn.com
allowVolumeExpansion: true
parameters:
  compression: "lz4"
  exaParams: "osc.*.checksums=0 llite.*.max_cached_mb=1024"
mountOptions:
  - "noatime"
volumeBindingMode: Immediate
```

### 3. Driver Configuration

#### Memory Sizing
**Based on snapshot requirements**:
```yaml
# Small snapshots (<100GB)
resources:
  limits:
    memory: "2Gi"

# Medium snapshots (100GB-1TB)
resources:
  limits:
    memory: "4Gi"

# Large snapshots (>1TB)
resources:
  limits:
    memory: "8Gi"
```

#### Multi-Filesystem Support
```yaml
# Config map structure
storage-system_map:
  training-fs:
    exaFS: 10.1.1.1@tcp:10.1.1.2@tcp:/aitraining
    mountPoint: /mnt/training
    zone: zone-1
  
  checkpoint-fs:
    exaFS: 10.2.1.1@tcp:10.2.1.2@tcp:/checkpoint
    mountPoint: /mnt/checkpoint
    zone: zone-2
```

### 4. Monitoring Setup

#### Metrics Exporter Deployment
```yaml
# Enable metrics collection
kubectl apply -f deploy/kubernetes/metrics/daemonset.yaml

# Verify metrics endpoint
kubectl port-forward -n kube-system ds/exa-csi-metrics-exporter 9090:9090
curl http://localhost:9090/metrics | grep exa_csi_pvc
```

#### Prometheus Rules
```yaml
# Alert on low available space
- alert: Storage SystemLowSpace
  expr: (exa_csi_pvc_available_bytes / exa_csi_pvc_capacity_bytes) < 0.1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "PVC {{ $labels.pvc }} has <10% free space"

# Alert on high pod count per PVC
- alert: Storage SystemHighPodCount
  expr: exa_csi_pvc_pod_count > 100
  for: 10m
  labels:
    severity: info
  annotations:
    summary: "PVC {{ $labels.pvc }} has {{ $value }} pods"
```

### 5. Security Configuration

#### Encryption Setup (Production)
```bash
# 1. Create KMIP secret
kubectl create secret generic kmip-config \
  --from-literal=kmipServerAddress="kmip.example.com:5696" \
  --from-literal=kmipUsername="storage-interface" \
  --from-literal=kmipPassword="secure-password" \
  --from-file=kmipCertificate=./kmip-cert.pem

# 2. Create passphrase secret
kubectl create secret generic exa-encryption \
  --from-literal=passphrase="your-secure-passphrase"

# 3. Deploy encrypted StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-encrypted
provisioner: exa.csi.ddn.com
parameters:
  fscrypt: "true"
  passphrase: "exa-encryption"
  kmipSecretName: "kmip-config"
```

### 6. Performance Tuning

#### Network Optimization
```bash
# Check LNet configuration
lctl network show

# Verify multi-rail setup
lctl list_nids
# Should show: 10.1.1.100@tcp, 10.1.2.100@tcp

# Test bandwidth
lfs_mirror_verify --bandwidth
```

#### Client Tuning
```bash
# Check current settings
lctl get_param osc.*.checksums
lctl get_param llite.*.max_read_ahead_mb

# Apply via StorageClass exaParams
parameters:
  exaParams: "osc.*.checksums=0 llite.*.max_read_ahead_mb=256 llite.*.max_cached_mb=2048"
```

## Verification Steps

### 1. Driver Health
```bash
# Check driver pods
kubectl get pods -n kube-system -l app=storage-interface-driver

# Check logs
kubectl logs -n kube-system -l app=storage-interface-driver --tail=100
```

### 2. StorageClass Validation
```bash
# List storage classes
kubectl get sc | grep storage-system

# Test dynamic provisioning
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes: [ReadWriteMany]
  resources:
    requests:
      storage: 10Gi
  storageClassName: storage-system-ai-training
EOF

# Verify PVC
kubectl get pvc test-pvc
```

### 3. Snapshot Testing
```bash
# Create snapshot
kubectl apply -f - <<EOF
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: test-snapshot
spec:
  volumeSnapshotClassName: storage-system-snapclass
  source:
    persistentVolumeClaimName: test-pvc
EOF

# Verify snapshot
kubectl get volumesnapshot test-snapshot
```

## Troubleshooting

### Common Issues

**Issue**: OOM during snapshot creation
**Solution**: Increase driver memory limits, use dtar mode

**Issue**: Encryption not working with multiple filesystems
**Solution**: Dedicate nodes per filesystem OR disable encryption

**Issue**: Poor performance with checksums enabled
**Solution**: Add `osc.*.checksums=0` to exaParams

**Issue**: Hot Nodes not caching
**Solution**: Verify NVMe capacity, check lpcc status on nodes

**Issue**: Cross-zone traffic
**Solution**: Enable topology + WaitForFirstConsumer binding mode

## Production Deployment Checklist

- [ ] Deploy with appropriate memory limits based on snapshot size
- [ ] Configure StorageClass for each workload type
- [ ] Enable metrics exporter and Prometheus scraping
- [ ] Set up Grafana dashboards for capacity monitoring
- [ ] Configure topology for zone-aware scheduling
- [ ] Test volume expansion workflow
- [ ] Test snapshot create/restore procedures
- [ ] Verify encryption with KMIP (if required)
- [ ] Tune filesystem client parameters per workload
- [ ] Configure multi-rail networking for high throughput
- [ ] Set up alerting rules in Prometheus
- [ ] Document disaster recovery procedures

---

**Performance Targets**:
- Throughput: 10+ GB/s per node with multi-rail
- Latency: <1ms for cached reads (Hot Nodes)
- Scalability: 1000+ nodes, 10,000+ volumes
- Snapshot speed: TB/min with dtar mode
