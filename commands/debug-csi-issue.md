---
name: debug-csi-issue
description: Debug storage interface issues with systematic troubleshooting
shortcut: /debug-csi
---

# Storage Interface Driver Debugging Protocol

Systematically debug storage interface issues following this protocol:

## Phase 1: Reproduce (5-10 min)
- Document exact steps to reproduce
- Identify affected Kubernetes resources (PVC, PV, Pod)
- Determine minimum conditions for issue

## Phase 2: Gather Evidence (10-15 min)
Collect the following:

```bash
# Controller logs
kubectl logs -n kube-system deployment/exa-csi-controller -c storage-interface-driver --tail=200

# Node driver logs
kubectl logs -n kube-system daemonset/exa-csi-node -c storage-interface-driver --tail=200

# PVC status
kubectl get pvc <pvc-name> -o yaml

# PV status
kubectl get pv <pv-name> -o yaml

# storage interface events
kubectl get events --sort-by='.lastTimestamp' | grep -i exa

# Metrics (if enabled)
curl http://localhost:32666/metrics | grep exa_csi
```

## Phase 3: Analyze (5-10 min)
Investigate:
- GRPC error codes in logs
- Volume lifecycle state transitions
- Mount/unmount operations on node
- Storage System filesystem connectivity

## Phase 4: Hypothesize (5 min)
List 3-5 possible root causes ranked by likelihood:
1. [Most likely cause]
2. [Second possibility]
3. [Less likely but possible]

## Phase 5: Test & Fix (15-30 min)
- Test hypotheses one at a time
- Document each test result
- Implement fix with proper error handling
- Add regression test

## Phase 6: Validate (10 min)
- Verify fix resolves original issue
- Check no side effects on other PVCs
- Review controller/node logs for errors
- Update documentation if needed

Provide detailed analysis with specific log excerpts and actionable recommendations.
