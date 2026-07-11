---
name: validate-k8s-manifests
description: Validate Kubernetes manifests for storage interface deployment
shortcut: /validate-manifests
---

# Validate Kubernetes Manifests Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Validate Kubernetes YAML manifests for the Storage System storage interface deployment.

## Usage

```bash
# Validate all manifests
/validate-manifests

# Validate specific directory
/validate-manifests deploy/kubernetes

# Validate with policy checks
/validate-manifests --policy

# Dry-run apply
/validate-manifests --dry-run
```

## Command Execution

### Complete Validation Script
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

set -e

MANIFEST_DIR=${1:-deploy/kubernetes}

echo "=== Kubernetes Manifest Validation ==="
echo "Directory: $MANIFEST_DIR"
echo ""

# 1. Syntax validation with kubeval
echo "Step 1: Validating YAML syntax..."
if command -v kubeval &> /dev/null; then
    kubeval --strict $MANIFEST_DIR/*.yaml
    echo "✅ YAML syntax valid"
else
    echo "⚠️  kubeval not found. Install: https://kubeval.com"
fi

# 2. Schema validation with kubectl
echo ""
echo "Step 2: Validating against Kubernetes API..."
for file in $MANIFEST_DIR/*.yaml; do
    kubectl apply --dry-run=client -f $file > /dev/null
    echo "✅ $file"
done

# 3. Policy validation with kyverno or OPA
echo ""
echo "Step 3: Validating policies..."
if command -v kyverno &> /dev/null; then
    kyverno apply policies/ -r $MANIFEST_DIR/*.yaml
    echo "✅ Policy validation passed"
else
    echo "⚠️  kyverno not found. Skipping policy validation"
fi

# 4. Security checks
echo ""
echo "Step 4: Running security checks..."
./validate-security.sh $MANIFEST_DIR

echo ""
echo "=== Validation Complete ==="
```

## Validation Tools

### kubeval (Schema Validation)
```bash
# Install kubeval
wget https://github.com/instrumenta/kubeval/releases/latest/download/kubeval-linux-amd64.tar.gz
tar xf kubeval-linux-amd64.tar.gz
sudo cp kubeval /usr/local/bin

# Validate manifests
kubeval deploy/kubernetes/*.yaml

# Validate with strict mode
kubeval --strict --kubernetes-version 1.24.0 deploy/kubernetes/*.yaml
```

### kubectl dry-run
```bash
# Client-side validation (no cluster needed)
kubectl apply --dry-run=client -f deploy/kubernetes/driver.yaml

# Server-side validation (requires cluster)
kubectl apply --dry-run=server -f deploy/kubernetes/driver.yaml
```

### kustomize validation
```bash
# Build and validate kustomize overlay
kustomize build deploy/overlays/production | kubeval --strict -

# Apply dry-run
kustomize build deploy/overlays/production | kubectl apply --dry-run=client -f -
```

## Security Validation

### Security Checks Script
```bash
#!/bin/bash
# validate-security.sh
# @author <AUTHOR_NAME>

MANIFEST_DIR=$1
ISSUES=0

echo "Running security checks..."

for file in $MANIFEST_DIR/*.yaml; do
    echo "Checking $file..."
    
    # Check for privileged containers
    if grep -q "privileged: true" $file; then
        echo "  ❌ Found privileged container"
        ((ISSUES++))
    fi
    
    # Check for hostPath volumes (should be Storage Interface only)
    if grep -q "hostPath:" $file; then
        echo "  ⚠️  Found hostPath volume"
    fi
    
    # Check for proper security context
    if ! grep -q "securityContext:" $file; then
        echo "  ⚠️  Missing securityContext"
    fi
    
    # Check for resource limits
    if ! grep -q "limits:" $file; then
        echo "  ⚠️  Missing resource limits"
    fi
done

if [ $ISSUES -gt 0 ]; then
    echo "❌ Found $ISSUES security issues"
    exit 1
else
    echo "✅ Security checks passed"
fi
```

## Storage Interface-Specific Validations

### Validate Storage Interface Driver Object
```yaml
# Check required fields in CSIDriver
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: storage-system.csi.example.com
spec:
  attachRequired: true
  podInfoOnMount: true
  volumeLifecycleModes:
    - Persistent
```

```bash
# Validate Storage Interface Driver
kubectl explain csidrivers.spec
kubectl apply --dry-run=client -f deploy/kubernetes/storage-interface.yaml
```

### Validate Storage Class
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-fs
provisioner: storage-system.csi.example.com
parameters:
  fsType: "filesystem"
allowVolumeExpansion: true
```

```bash
# Validate Storage Class
kubectl apply --dry-run=client -f deploy/kubernetes/storageclass.yaml
```

### Validate RBAC
```bash
#!/bin/bash
# Validate RBAC permissions

echo "Validating RBAC..."

# Check ClusterRole
kubectl auth can-i --list --as=system:serviceaccount:storage-system:storage-interface-controller

# Check required permissions
kubectl auth can-i create persistentvolumes --as=system:serviceaccount:storage-system:storage-interface-controller
kubectl auth can-i get nodes --as=system:serviceaccount:storage-system:storage-interface-node
```

## Policy Validation (Kyverno)

### Policy Example
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resource-limits
spec:
  validationFailureAction: enforce
  rules:
  - name: check-resource-limits
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Resource limits are required"
      pattern:
        spec:
          containers:
          - resources:
              limits:
                memory: "?*"
                cpu: "?*"
```

## Expected Output

```
=== Kubernetes Manifest Validation ===
Directory: deploy/kubernetes

Step 1: Validating YAML syntax...
PASS - deploy/kubernetes/storage-interface.yaml
PASS - deploy/kubernetes/controller.yaml
PASS - deploy/kubernetes/node.yaml
PASS - deploy/kubernetes/storageclass.yaml
✅ YAML syntax valid

Step 2: Validating against Kubernetes API...
✅ deploy/kubernetes/storage-interface.yaml
✅ deploy/kubernetes/controller.yaml
✅ deploy/kubernetes/node.yaml
✅ deploy/kubernetes/storageclass.yaml

Step 3: Validating policies...
✅ Policy validation passed

Step 4: Running security checks...
Checking deploy/kubernetes/controller.yaml...
✅ Security checks passed

=== Validation Complete ===
```

## Common Validation Errors

### Error: Unknown field
```
error: error validating "deploy/kubernetes/controller.yaml": 
error validating data: ValidationError(Deployment.spec.template.spec.containers[0]): 
unknown field "securityContext.allowPrivilegeEscalation"
```

**Fix**: Remove or correct the unknown field

### Error: Required field missing
```
error: error validating "deploy/kubernetes/storageclass.yaml": 
error validating data: ValidationError(StorageClass): missing required field "provisioner"
```

**Fix**: Add required field `provisioner: storage-system.csi.example.com`

## CI/CD Integration

### GitHub Actions
```yaml
- name: Validate Manifests
  run: |
    kubeval --strict deploy/kubernetes/*.yaml
    kubectl apply --dry-run=client -f deploy/kubernetes/
```

## Reference Files

- **Deploy Manifests**: `storage-interface-driver/deploy/kubernetes/`
- **Storage Interface Spec**: `.ai-config/skills/storage-driver-development/SKILL.md`
