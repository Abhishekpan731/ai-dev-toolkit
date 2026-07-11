---
name: deploy-driver
description: Deploy Storage System storage interface to Kubernetes/OpenShift
shortcut: /deploy
---

# Deploy Storage Interface Driver Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Deploy the Storage System storage interface to a Kubernetes or OpenShift cluster.

## Usage

```bash
# Deploy to Kubernetes
/deploy --platform=kubernetes

# Deploy to OpenShift
/deploy --platform=openshift

# Deploy with custom namespace
/deploy --namespace=storage-system

# Dry-run deployment
/deploy --dry-run
```

## Command Execution

### Complete Deployment Script
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

set -e

PLATFORM=${1:-kubernetes}
NAMESPACE=${2:-storage-system-storage-interface}
DRY_RUN=${3:-false}

echo "=== Storage System Storage Interface Driver Deployment ==="
echo "Platform: $PLATFORM"
echo "Namespace: $NAMESPACE"
echo ""

# 1. Create namespace
echo "Step 1: Creating namespace..."
if [ "$DRY_RUN" = "false" ]; then
    kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -
    echo "✅ Namespace created/verified"
fi

# 2. Deploy RBAC
echo ""
echo "Step 2: Deploying RBAC..."
kubectl apply -f deploy/$PLATFORM/rbac.yaml -n $NAMESPACE --dry-run=$DRY_RUN

# 3. Deploy Storage Interface Driver object
echo ""
echo "Step 3: Deploying Storage Interface Driver..."
kubectl apply -f deploy/$PLATFORM/storage-interface.yaml --dry-run=$DRY_RUN

# 4. Deploy Controller
echo ""
echo "Step 4: Deploying Controller..."
kubectl apply -f deploy/$PLATFORM/controller.yaml -n $NAMESPACE --dry-run=$DRY_RUN

# 5. Deploy Node DaemonSet
echo ""
echo "Step 5: Deploying Node DaemonSet..."
kubectl apply -f deploy/$PLATFORM/node.yaml -n $NAMESPACE --dry-run=$DRY_RUN

# 6. Deploy StorageClass
echo ""
echo "Step 6: Deploying StorageClass..."
kubectl apply -f deploy/$PLATFORM/storageclass.yaml --dry-run=$DRY_RUN

# 7. Wait for pods to be ready
if [ "$DRY_RUN" = "false" ]; then
    echo ""
    echo "Step 7: Waiting for pods to be ready..."
    kubectl wait --for=condition=ready pod -l app=storage-system-controller -n $NAMESPACE --timeout=300s
    kubectl wait --for=condition=ready pod -l app=storage-system-node -n $NAMESPACE --timeout=300s
    echo "✅ All pods ready"
fi

echo ""
echo "=== Deployment Complete ==="
```

## Kubernetes Deployment

### Prerequisites
```bash
# Verify cluster access
kubectl cluster-info

# Check Kubernetes version (1.24+)
kubectl version --short

# Verify Storage Interface support
kubectl api-versions | grep storage.k8s.io
```

### Deploy Steps
```bash
# 1. Clone repository
git clone https://github.com/storage-provider/storage-interface-driver.git
cd storage-interface-driver

# 2. Create namespace
kubectl create namespace storage-system-storage-interface

# 3. Deploy using make
make deploy

# Or deploy manually
kubectl apply -f deploy/kubernetes/
```

### Verify Deployment
```bash
# Check pods
kubectl get pods -n storage-system-storage-interface

# Check storage interface registration
kubectl get csidrivers

# Check Storage Interface nodes
kubectl get csinodes

# Check storage class
kubectl get storageclass storage-system-fs
```

## OpenShift Deployment

### Prerequisites
```bash
# Login to OpenShift
oc login

# Create project
oc new-project storage-system-storage-interface

# Add SCC (Security Context Constraints)
oc adm policy add-scc-to-user privileged system:serviceaccount:storage-system-storage-interface:storage-interface-node
```

### Deploy Steps
```bash
# Deploy using OpenShift templates
oc apply -f deploy/openshift/

# Or use Helm
helm install storage-system deploy/helm-chart \
  --namespace storage-system-storage-interface \
  --create-namespace
```

### OpenShift-Specific Configuration
```yaml
# Deploy with OpenShift image registry
apiVersion: apps/v1
kind: Deployment
metadata:
  name: storage-system-controller
spec:
  template:
    spec:
      containers:
      - name: storage-interface
        image: image-registry.openshift-image-registry.svc:5000/storage/interface:latest
```

## Helm Deployment

### Install with Helm
```bash
# Add Helm repository
helm repo add storage-system https://ddn.github.io/storage-system-charts
helm repo update

# Install chart
helm install storage-system storage-system/storage-interface-driver \
  --namespace storage-system-storage-interface \
  --create-namespace \
  --set image.tag=v1.2.3

# Upgrade existing installation
helm upgrade storage-system storage-system/storage-interface-driver \
  --namespace storage-system-storage-interface \
  --set image.tag=v1.3.0
```

### Custom Values
```yaml
# values.yaml
image:
  repository: registry.example.com/storage-interface
  tag: v1.2.3
  pullPolicy: IfNotPresent

controller:
  replicas: 2
  resources:
    limits:
      cpu: 200m
      memory: 256Mi

node:
  resources:
    limits:
      cpu: 100m
      memory: 128Mi

storageClass:
  name: storage-system-fs
  parameters:
    fsType: filesystem
    mountOptions: "flock,user_xattr"
```

## Post-Deployment Validation

### Test Volume Creation
```bash
# Create test PVC
kubectl create -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: storage-system-fs
EOF

# Check PVC status
kubectl get pvc test-pvc

# Create test pod
kubectl run test-pod --image=nginx --restart=Never \
  --overrides='{"spec":{"volumes":[{"name":"vol","persistentVolumeClaim":{"claimName":"test-pvc"}}],"containers":[{"name":"nginx","image":"nginx","volumeMounts":[{"name":"vol","mountPath":"/data"}]}]}}'

# Verify mount
kubectl exec test-pod -- df -h /data
```

## Expected Output

```
=== Storage System Storage Interface Driver Deployment ===
Platform: kubernetes
Namespace: storage-system-storage-interface

Step 1: Creating namespace...
✅ Namespace created/verified

Step 2: Deploying RBAC...
serviceaccount/storage-interface-controller created
clusterrole.rbac.authorization.k8s.io/storage-interface-controller created
clusterrolebinding.rbac.authorization.k8s.io/storage-interface-controller created

Step 3: Deploying Storage Interface Driver...
csidriver.storage.k8s.io/storage-system.csi.example.com created

Step 4: Deploying Controller...
deployment.apps/storage-system-controller created

Step 5: Deploying Node DaemonSet...
daemonset.apps/storage-system-node created

Step 6: Deploying StorageClass...
storageclass.storage.k8s.io/storage-system-fs created

Step 7: Waiting for pods to be ready...
pod/storage-system-controller-abc123 condition met
pod/storage-system-node-xyz789 condition met
✅ All pods ready

=== Deployment Complete ===
```

## Reference Files

- **Deploy Manifests**: `storage-interface-driver/deploy/`
- **Validation Command**: `.ai-config/commands/validate-k8s-manifests.md`
