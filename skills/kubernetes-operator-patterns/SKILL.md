---
mode: auto
name: kubernetes-operator-patterns
description: Kubernetes operator and controller patterns for storage interfaces
tags: [kubernetes, operator, controller, csi]
version: 1.0.0
---

# Kubernetes Operator Patterns

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for building Kubernetes operators and controllers, specifically for storage interface management.

## Technology Stack

- **Language**: Go 1.21+
- **Framework**: controller-runtime, client-go
- **Platform**: Kubernetes 1.24+, OpenShift 4.12+
- **Patterns**: Reconciliation loop, watch-based controllers

## Operator Pattern Fundamentals

### Reconciliation Loop
```go
// @author <AUTHOR_NAME>
func (r *VolumeReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. Fetch the resource
    volume := &storagev1.Volume{}
    if err := r.Get(ctx, req.NamespacedName, volume); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // 2. Examine DeletionTimestamp (finalizer pattern)
    if !volume.ObjectMeta.DeletionTimestamp.IsZero() {
        return r.handleDeletion(ctx, volume)
    }
    
    // 3. Add finalizer if not present
    if !containsFinalizer(volume, volumeFinalizer) {
        volume.Finalizers = append(volume.Finalizers, volumeFinalizer)
        return ctrl.Result{}, r.Update(ctx, volume)
    }
    
    // 4. Reconcile to desired state
    return r.reconcileVolume(ctx, volume)
}
```

## Controller Patterns

### 1. Watch-Based Controller
```go
// @author <AUTHOR_NAME>
func (r *VolumeReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&storagev1.Volume{}).
        Owns(&corev1.PersistentVolume{}).
        Watches(
            &source.Kind{Type: &storagev1.VolumeSnapshot{}},
            &handler.EnqueueRequestForOwner{
                OwnerType:    &storagev1.Volume{},
                IsController: true,
            },
        ).
        Complete(r)
}
```

### 2. Finalizer Pattern
```go
// @author <AUTHOR_NAME>
func (r *Reconciler) handleDeletion(ctx context.Context, obj client.Object) (ctrl.Result, error) {
    if containsFinalizer(obj, myFinalizer) {
        // Perform cleanup
        if err := r.cleanupResources(ctx, obj); err != nil {
            return ctrl.Result{}, err
        }
        
        // Remove finalizer
        obj.SetFinalizers(removeFinalizer(obj.GetFinalizers(), myFinalizer))
        if err := r.Update(ctx, obj); err != nil {
            return ctrl.Result{}, err
        }
    }
    return ctrl.Result{}, nil
}
```

### 3. Status Subresource Pattern
```go
// @author <AUTHOR_NAME>
func (r *Reconciler) updateStatus(ctx context.Context, volume *storagev1.Volume, condition storagev1.VolumeCondition) error {
    volume.Status.Conditions = updateCondition(volume.Status.Conditions, condition)
    volume.Status.Phase = calculatePhase(volume.Status.Conditions)
    return r.Status().Update(ctx, volume)
}
```

## Storage Interface Driver Operator Patterns

### Driver Deployment
```go
// @author <AUTHOR_NAME>
func (r *CSIDriverReconciler) reconcileDeployment(ctx context.Context, driver *csiv1.CSIDriver) error {
    // Create controller deployment
    controllerDep := &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      driver.Name + "-controller",
            Namespace: driver.Namespace,
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: ptr.To(int32(1)),
            Selector: &metav1.LabelSelector{
                MatchLabels: map[string]string{
                    "app": driver.Name + "-controller",
                },
            },
            Template: r.buildControllerPodTemplate(driver),
        },
    }
    
    // Set owner reference
    if err := ctrl.SetControllerReference(driver, controllerDep, r.Scheme); err != nil {
        return fmt.Errorf("set controller reference: %w", err)
    }
    
    // Create or update
    return r.createOrUpdate(ctx, controllerDep)
}
```

### Volume Snapshot Controller
```go
// @author <AUTHOR_NAME>
func (r *SnapshotReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    snapshot := &snapshotv1.VolumeSnapshot{}
    if err := r.Get(ctx, req.NamespacedName, snapshot); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // Check if snapshot is ready
    if snapshot.Status != nil && snapshot.Status.ReadyToUse != nil && *snapshot.Status.ReadyToUse {
        return ctrl.Result{}, nil
    }
    
    // Create snapshot content
    content, err := r.createSnapshotContent(ctx, snapshot)
    if err != nil {
        return ctrl.Result{}, err
    }
    
    // Update snapshot status
    snapshot.Status = &snapshotv1.VolumeSnapshotStatus{
        BoundVolumeSnapshotContentName: &content.Name,
    }
    
    return ctrl.Result{}, r.Status().Update(ctx, snapshot)
}
```

## Best Practices

### 1. Idempotency
```go
// ✅ GOOD: Idempotent reconciliation
func (r *Reconciler) reconcile(ctx context.Context, obj client.Object) error {
    existing := &corev1.ConfigMap{}
    err := r.Get(ctx, client.ObjectKeyFromObject(obj), existing)
    
    if err != nil && !errors.IsNotFound(err) {
        return err
    }
    
    if errors.IsNotFound(err) {
        // Create new
        return r.Create(ctx, obj)
    }
    
    // Update existing
    return r.Update(ctx, obj)
}
```

### 2. Error Handling with Requeue
```go
// @author <AUTHOR_NAME>
func (r *Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // Transient error - requeue with backoff
    if err := r.reconcile(ctx, req); err != nil {
        if isTransientError(err) {
            return ctrl.Result{RequeueAfter: 30 * time.Second}, nil
        }
        return ctrl.Result{}, err
    }
    
    // Success - requeue for periodic reconciliation
    return ctrl.Result{RequeueAfter: 5 * time.Minute}, nil
}
```

### 3. Event Recording
```go
// @author <AUTHOR_NAME>
func (r *Reconciler) recordEvent(obj client.Object, eventType, reason, message string) {
    r.Recorder.Event(obj, eventType, reason, message)
}

// Usage
r.recordEvent(volume, corev1.EventTypeNormal, "VolumeCreated", "Volume created successfully")
r.recordEvent(volume, corev1.EventTypeWarning, "VolumeFailed", "Failed to create volume")
```

## Anti-Patterns

### ❌ Don't: Modify status in main reconciliation
```go
// BAD: Status updated in main body
func (r *Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    obj.Status.Phase = "Ready"
    return ctrl.Result{}, r.Update(ctx, obj)  // Wrong!
}
```

### ✅ Do: Use status subresource
```go
// GOOD: Status updated separately
func (r *Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    obj.Status.Phase = "Ready"
    return ctrl.Result{}, r.Status().Update(ctx, obj)  // Correct
}
```

## Testing Operators

### Unit Test with Fake Client
```go
// @author <AUTHOR_NAME>
func TestReconcile(t *testing.T) {
    scheme := runtime.NewScheme()
    _ = clientgoscheme.AddToScheme(scheme)
    
    client := fake.NewClientBuilder().
        WithScheme(scheme).
        WithObjects(&storagev1.Volume{}).
        Build()
    
    reconciler := &VolumeReconciler{Client: client}
    
    result, err := reconciler.Reconcile(context.Background(), ctrl.Request{})
    assert.NoError(t, err)
    assert.False(t, result.Requeue)
}
```

## Reference Resources

- **controller-runtime**: https://github.com/kubernetes-sigs/controller-runtime
- **kubebuilder**: https://book.kubebuilder.io/
- **operator-sdk**: https://sdk.operatorframework.io/
