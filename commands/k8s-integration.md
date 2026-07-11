---
name: k8s-integration
description: Analyze Kubernetes integration patterns for storage interface
---

# Kubernetes Storage Interface Integration Analysis

Analyze how the Storage System storage interface integrates with Kubernetes, using patterns from `k8s-core` repository.

## Analysis Areas

### 1. Storage Provisioning
- Dynamic provisioning via StorageClass
- Static provisioning via PV/PVC binding
- Volume cloning and snapshots

### 2. Volume Lifecycle
```
PVC Created → Provisioner → CreateVolume → PV Bound → 
NodeStageVolume → NodePublishVolume → Pod Running →
Pod Deleted → NodeUnpublishVolume → NodeUnstageVolume →
DeleteVolume → PV Deleted
```

### 3. Sidecar Containers
Check for proper use of:
- external-provisioner
- external-attacher
- external-resizer
- node-driver-registrar
- livenessprobe

### 4. RBAC and Security
- ServiceAccount permissions
- ClusterRole bindings
- SecurityContext settings
- SELinux/AppArmor labels

### 5. Topology and Scheduling
- CSINode object registration
- Topology keys (zone, region)
- Pod affinity rules
- Volume binding mode (Immediate vs WaitForFirstConsumer)

## Reference Kubernetes Patterns

Look for patterns in:
- `k8s-core/staging/src/k8s.io/csi-api/`
- `k8s-core/staging/src/k8s.io/mount-utils/`
- `k8s-core/pkg/volume/csi/`

## Output

Provide:
1. **Integration Maturity**: Alpha/Beta/GA
2. **Missing Components**: What K8s features aren't supported
3. **Optimization Opportunities**: Performance improvements
4. **Security Gaps**: RBAC or security issues
5. **Code Examples**: Show how to implement missing features using K8s patterns
