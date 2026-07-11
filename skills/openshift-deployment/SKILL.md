---
mode: auto
name: openshift-deployment
description: OpenShift-specific deployment patterns and configurations for storage interfaces
tags: [openshift, kubernetes, csi, security, deployment]
version: 1.0.0
---

# OpenShift Deployment Patterns

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for deploying storage interfaces on OpenShift with proper security contexts, SCCs, and OpenShift-specific configurations.

## Technology Stack

- **OpenShift**: 4.12+
- **Kubernetes**: 1.25+
- **Storage Interface Spec**: v1.8.0
- **Security**: SELinux, SCCs

## Security Context Constraints (SCCs)

### Create Custom SCC for Storage Interface
```yaml
# @author <AUTHOR_NAME>
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: storage-system-scc
allowHostDirVolumePlugin: true
allowHostNetwork: true
allowHostPID: false
allowHostPorts: false
allowPrivilegedContainer: true  # Required for mount operations
allowedCapabilities:
  - SYS_ADMIN  # Required for mount
defaultAddCapabilities: []
fsGroup:
  type: RunAsAny
readOnlyRootFilesystem: false
runAsUser:
  type: RunAsAny
seLinuxContext:
  type: RunAsAny
users:
  - system:serviceaccount:storage-system-storage-interface:storage-interface-node
volumes:
  - configMap
  - downwardAPI
  - emptyDir
  - hostPath
  - persistentVolumeClaim
  - projected
  - secret
```

### Apply SCC to Service Account
```bash
# @author <AUTHOR_NAME>
# Create SCC
oc create -f storage-system-scc.yaml

# Add SCC to service account
oc adm policy add-scc-to-user storage-system-scc \
  system:serviceaccount:storage-system-storage-interface:storage-interface-node

# Verify SCC assignment
oc describe scc storage-system-scc
```

## OpenShift-Specific Deployment

### Project Setup
```bash
# @author <AUTHOR_NAME>
# Create project (namespace)
oc new-project storage-system-storage-interface

# Label project
oc label namespace storage-system-storage-interface \
  openshift.io/cluster-monitoring=true

# Set resource quotas
oc create quota csi-quota \
  --hard=pods=100,requests.storage=10Ti \
  -n storage-system-storage-interface
```

### Controller Deployment with OpenShift Annotations
```yaml
# @author <AUTHOR_NAME>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: storage-system-controller
  namespace: storage-system-storage-interface
  annotations:
    # OpenShift-specific annotations
    openshift.io/scc: restricted-v2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: storage-system-controller
  template:
    metadata:
      labels:
        app: storage-system-controller
      annotations:
        # Trigger deployment on config changes
        config.openshift.io/inject-trusted-cabundle: "true"
    spec:
      serviceAccountName: storage-interface-controller
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: storage-interface
        image: registry.example.com/storage-interface:v1.2.3
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
          runAsNonRoot: true
```

### Node DaemonSet with Privileged SCC
```yaml
# @author <AUTHOR_NAME>
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: storage-system-node
  namespace: storage-system-storage-interface
spec:
  selector:
    matchLabels:
      app: storage-system-node
  template:
    metadata:
      labels:
        app: storage-system-node
    spec:
      serviceAccountName: storage-interface-node
      hostNetwork: true
      # Required for mount operations
      securityContext:
        seLinuxOptions:
          type: spc_t
      containers:
      - name: storage-interface
        image: registry.example.com/storage-interface:v1.2.3
        securityContext:
          privileged: true  # Required for mount
          seLinuxOptions:
            type: spc_t
        volumeMounts:
        - name: kubelet-dir
          mountPath: /var/lib/kubelet
          mountPropagation: Bidirectional
      volumes:
      - name: kubelet-dir
        hostPath:
          path: /var/lib/kubelet
          type: Directory
```

## Image Registry Configuration

### Internal Registry
```bash
# @author <AUTHOR_NAME>
# Get internal registry URL
REGISTRY=$(oc get route default-route -n openshift-image-registry \
  --template='{{ .spec.host }}')

# Tag image for internal registry
docker tag storage-system-storage-interface:v1.2.3 \
  ${REGISTRY}/storage-interface-driver/storage-interface:v1.2.3

# Login to registry
docker login -u $(oc whoami) -p $(oc whoami -t) ${REGISTRY}

# Push image
docker push ${REGISTRY}/storage-interface-driver/storage-interface:v1.2.3
```

### ImageStream
```yaml
# @author <AUTHOR_NAME>
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: storage-interface
  namespace: storage-system-storage-interface
spec:
  lookupPolicy:
    local: true
  tags:
  - name: v1.2.3
    from:
      kind: DockerImage
      name: registry.example.com/storage-interface:v1.2.3
    importPolicy:
      scheduled: true
    referencePolicy:
      type: Local
```

## Monitoring and Alerts

### ServiceMonitor for Prometheus
```yaml
# @author <AUTHOR_NAME>
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: storage-system-metrics
  namespace: storage-system-storage-interface
  labels:
    app: storage-system
spec:
  selector:
    matchLabels:
      app: storage-system-controller
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### PrometheusRule for Alerts
```yaml
# @author <AUTHOR_NAME>
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: storage-system-alerts
  namespace: storage-system-storage-interface
spec:
  groups:
  - name: storage-interface
    interval: 30s
    rules:
    - alert: CSIDriverDown
      expr: up{job="storage-system-controller"} == 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "storage interface controller is down"
```

## Route Configuration

### Expose Metrics
```yaml
# @author <AUTHOR_NAME>
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: csi-metrics
  namespace: storage-system-storage-interface
spec:
  to:
    kind: Service
    name: csi-controller-metrics
  port:
    targetPort: metrics
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

## Operator Lifecycle Manager (OLM)

### ClusterServiceVersion
```yaml
# @author <AUTHOR_NAME>
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: storage-system-operator.v1.2.3
spec:
  displayName: Storage System Storage Interface Driver
  description: storage interface for storage provider Storage System
  version: 1.2.3
  install:
    strategy: deployment
    spec:
      deployments:
      - name: storage-system-controller
        spec: # deployment spec
```

## Network Policies

### Restrict Traffic
```yaml
# @author <AUTHOR_NAME>
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: storage-interface-netpol
  namespace: storage-system-storage-interface
spec:
  podSelector:
    matchLabels:
      app: storage-system-controller
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: TCP
      port: 9898
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
```

## Best Practices

### 1. Use Restricted SCC When Possible
```yaml
# Controller: Use restricted-v2 SCC
securityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault

# Node: Use custom privileged SCC only
securityContext:
  privileged: true
```

### 2. Leverage OpenShift Features
```bash
# Use BuildConfig for automated builds
# Use ImageStreams for image management
# Use Routes for external access
# Use ServiceMonitors for monitoring
```

### 3. Resource Limits
```yaml
# Set appropriate resource limits
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

## Reference Files

- **Deploy Command**: `.ai-config/commands/deploy-driver.md`
- **K8s Storage**: `.ai-config/skills/kubernetes-storage/SKILL.md`
- **Container Security**: `.ai-config/skills/container-security/SKILL.md`
