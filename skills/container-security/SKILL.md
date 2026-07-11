---
mode: auto
name: container-security
description: Container security best practices and hardening techniques
tags: [security, containers, docker, kubernetes, hardening]
version: 1.0.0
---

# Container Security

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for securing containers, implementing security best practices, and hardening storage interface deployments.

## Technology Stack

- **Container Runtime**: Docker, containerd, CRI-O
- **Kubernetes**: 1.24+
- **Security Tools**: Trivy, Falco, Kubesec
- **Compliance**: CIS Benchmarks, PCI-DSS

## Image Security

### Secure Base Images
```dockerfile
# @author <AUTHOR_NAME>
# ❌ BAD: Using latest or full OS
FROM ubuntu:latest

# ✅ GOOD: Specific version, minimal base
FROM alpine:3.18

# ✅ BETTER: Distroless for production
FROM gcr.io/distroless/static:nonroot
```

### Multi-Stage Builds
```dockerfile
# @author <AUTHOR_NAME>
# Build stage - can use full toolchain
FROM golang:1.21-alpine AS builder
WORKDIR /build
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-w -s" -o /app

# Production stage - minimal
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

### Image Scanning
```bash
# @author <AUTHOR_NAME>
# Scan with Trivy
trivy image storage-system-storage-interface:latest

# Fail CI on HIGH/CRITICAL
trivy image --severity HIGH,CRITICAL \
  --exit-code 1 \
  storage-system-storage-interface:latest

# Scan for misconfigurations
trivy config Dockerfile

# Generate SBOM
trivy image --format cyclonedx \
  -o sbom.json \
  storage-system-storage-interface:latest
```

## Runtime Security

### Security Contexts
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: app:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
      runAsNonRoot: true
      runAsUser: 1000
```

### Capabilities Management
```yaml
# @author <AUTHOR_NAME>
# Drop all capabilities, add only required
securityContext:
  capabilities:
    drop:
    - ALL
    add:
    - NET_BIND_SERVICE  # Only if needed
```

### Read-Only Root Filesystem
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    # Writable directories as tmpfs
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /var/cache
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

## Pod Security Standards

### Restricted Policy
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: Namespace
metadata:
  name: storage-system-storage-interface
  labels:
    # Enforce restricted pod security
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Pod Security Admission
```yaml
# @author <AUTHOR_NAME>
# PodSecurityConfiguration
apiVersion: pod-security.admission.config.k8s.io/v1
kind: PodSecurityConfiguration
defaults:
  enforce: restricted
  enforce-version: latest
  audit: restricted
  audit-version: latest
  warn: restricted
  warn-version: latest
exemptions:
  # Exempt system namespaces
  namespaces:
  - kube-system
  - storage-system-storage-interface  # Storage Interface needs privileged
```

## Network Security

### Network Policies
```yaml
# @author <AUTHOR_NAME>
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: storage-interface-netpol
spec:
  podSelector:
    matchLabels:
      app: storage-system-controller
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow only from kube-system
  - from:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: TCP
      port: 9898
  egress:
  # Allow DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
  # Allow API server
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
```

## Secret Management

### Kubernetes Secrets
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: Secret
metadata:
  name: csi-credentials
type: Opaque
stringData:
  username: admin
  password: ${PASSWORD}  # Use external secret manager
```

### External Secrets Operator
```yaml
# @author <AUTHOR_NAME>
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: csi-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault
    kind: SecretStore
  target:
    name: csi-credentials
  data:
  - secretKey: password
    remoteRef:
      key: csi/credentials
      property: password
```

### Sealed Secrets
```bash
# @author <AUTHOR_NAME>
# Install kubeseal
kubeseal --fetch-cert > pub-cert.pem

# Seal secret
kubectl create secret generic csi-creds \
  --from-literal=password=secret \
  --dry-run=client -o yaml | \
  kubeseal --cert pub-cert.pem -o yaml > sealed-secret.yaml

# Apply sealed secret
kubectl apply -f sealed-secret.yaml
```

## RBAC Hardening

### Least Privilege
```yaml
# @author <AUTHOR_NAME>
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: storage-interface-controller
rules:
# Only required permissions
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list", "watch", "create", "delete"]
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "update"]
# No wildcard permissions
```

### Service Account Token Security
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: ServiceAccount
metadata:
  name: storage-interface-controller
automountServiceAccountToken: false  # Explicitly mount only where needed
```

## Compliance and Auditing

### Audit Logging
```yaml
# @author <AUTHOR_NAME>
# audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
# Log all requests to Storage Interface resources
- level: RequestResponse
  resources:
  - group: storage.k8s.io
    resources: ["csidrivers", "csinodes"]
```

### CIS Benchmark Compliance
```bash
# @author <AUTHOR_NAME>
# Run kube-bench
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

# Check results
kubectl logs -n kube-system $(kubectl get pods -n kube-system -l app=kube-bench -o name)
```

## Runtime Protection

### Falco Rules
```yaml
# @author <AUTHOR_NAME>
- rule: Detect Storage Interface Driver Anomalies
  desc: Detect unusual behavior in storage interface
  condition: >
    container.name = "storage-interface" and
    (spawned_process or file_access)
  output: "Unusual activity in storage interface (user=%user.name command=%proc.cmdline)"
  priority: WARNING
```

### AppArmor Profile
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: Pod
metadata:
  name: csi-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/storage-interface: localhost/csi-profile
spec:
  containers:
  - name: storage-interface
    image: storage-system-storage-interface:latest
```

## Supply Chain Security

### Image Signing
```bash
# @author <AUTHOR_NAME>
# Sign image with cosign
cosign sign --key cosign.key storage-system-storage-interface:v1.2.3

# Verify signature
cosign verify --key cosign.pub storage-system-storage-interface:v1.2.3
```

### Admission Controllers
```yaml
# @author <AUTHOR_NAME>
# Validate image signatures with Kyverno
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: enforce
  rules:
  - name: verify-signature
    match:
      resources:
        kinds:
        - Pod
    verifyImages:
    - imageReferences:
      - "quay.io/storage-system/*"
      attestors:
      - count: 1
        entries:
        - keys:
            publicKeys: |-
              -----BEGIN PUBLIC KEY-----
              ...
              -----END PUBLIC KEY-----
```

## Security Checklist

- [ ] Use minimal base images (Alpine, Distroless)
- [ ] Run as non-root user
- [ ] Read-only root filesystem
- [ ] Drop all capabilities
- [ ] Security scanning in CI/CD
- [ ] Network policies configured
- [ ] RBAC least privilege
- [ ] Secrets encrypted at rest
- [ ] Audit logging enabled
- [ ] Image signatures verified

## Reference Files

- **Docker Optimization**: `.ai-config/skills/docker-optimization/SKILL.md`
- **OpenShift Deployment**: `.ai-config/skills/openshift-deployment/SKILL.md`
- **Security Rules**: `.ai-config/rules/security.md`
