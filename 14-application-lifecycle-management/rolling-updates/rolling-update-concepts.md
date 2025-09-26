# Rolling Updates

## 🎯 Rolling Update Concepts

### What are Rolling Updates?
- **Gradual replacement** of old pods with new ones
- **Zero-downtime deployment** strategy
- **Default strategy** in Kubernetes deployments
- **Maintains service availability** during updates

### Rolling Update Process
1. Create new ReplicaSet with updated spec
2. Scale up new ReplicaSet gradually
3. Scale down old ReplicaSet gradually
4. Continue until all pods are updated
5. Keep old ReplicaSet for rollback

## 🔄 Update Strategies

### RollingUpdate Strategy
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1      # Max pods that can be unavailable
      maxSurge: 1           # Max pods above desired count
```

### Recreate Strategy
```yaml
spec:
  strategy:
    type: Recreate  # Stop all pods, then create new ones
```

## 🚀 Performing Rolling Updates

### Update Container Image
```bash
# Update deployment image
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Update with record for rollback history
kubectl set image deployment/nginx-deployment nginx=nginx:1.21 --record

# Update multiple containers
kubectl set image deployment/webapp web=nginx:1.21 sidecar=busybox:1.36
```

### Update Environment Variables
```bash
# Update environment variable
kubectl set env deployment/webapp ENV_VAR=new-value

# Update multiple env vars
kubectl set env deployment/webapp VAR1=value1 VAR2=value2

# Remove environment variable
kubectl set env deployment/webapp ENV_VAR-
```

### Update Resource Limits
```bash
# Update resource limits
kubectl patch deployment webapp -p '{"spec":{"template":{"spec":{"containers":[{"name":"webapp","resources":{"limits":{"cpu":"500m","memory":"1Gi"}}}]}}}}'
```

## 📊 Monitoring Rolling Updates

### Check Rollout Status
```bash
# Watch rollout progress
kubectl rollout status deployment/nginx-deployment

# Get rollout history
kubectl rollout history deployment/nginx-deployment

# Check specific revision
kubectl rollout history deployment/nginx-deployment --revision=2
```

### Monitor Pod Changes
```bash
# Watch pods being replaced
kubectl get pods -l app=nginx -w

# Check ReplicaSets
kubectl get rs -l app=nginx

# Monitor deployment events
kubectl describe deployment nginx-deployment | grep -A 10 Events
```

## ⚙️ Rolling Update Parameters

### maxUnavailable
- **Absolute number**: `maxUnavailable: 2`
- **Percentage**: `maxUnavailable: 25%`
- **Default**: 25%

### maxSurge
- **Absolute number**: `maxSurge: 2`
- **Percentage**: `maxSurge: 25%`
- **Default**: 25%

### Configuration Examples
```yaml
# Conservative (slow, safe)
rollingUpdate:
  maxUnavailable: 0
  maxSurge: 1

# Aggressive (fast, less safe)
rollingUpdate:
  maxUnavailable: 50%
  maxSurge: 50%

# Balanced (default)
rollingUpdate:
  maxUnavailable: 25%
  maxSurge: 25%
```

## 🛑 Controlling Rolling Updates

### Pause and Resume
```bash
# Pause ongoing rollout
kubectl rollout pause deployment/nginx-deployment

# Resume paused rollout
kubectl rollout resume deployment/nginx-deployment

# Check pause status
kubectl get deployment nginx-deployment -o yaml | grep -A 5 "paused"
```

### Manual Control
```bash
# Edit deployment during rollout
kubectl edit deployment nginx-deployment

# Scale during rollout
kubectl scale deployment nginx-deployment --replicas=10
```

## 🚨 Rolling Update Troubleshooting

### Failed Updates
```bash
# Check rollout status
kubectl rollout status deployment/nginx-deployment

# Check deployment conditions
kubectl describe deployment nginx-deployment | grep -A 10 Conditions

# Check pod issues
kubectl get pods -l app=nginx
kubectl describe pod <failing-pod>
```

### Common Issues
- Image pull failures
- Resource constraints
- Health check failures
- Configuration errors
- Network connectivity issues

### Debug Commands
```bash
# Check deployment events
kubectl describe deployment nginx-deployment

# Check ReplicaSet status
kubectl get rs -l app=nginx
kubectl describe rs <replicaset-name>

# Check pod logs
kubectl logs -l app=nginx --tail=50
```

## 📋 Best Practices

### Update Strategy
- Use health checks (readiness/liveness probes)
- Set appropriate resource requests/limits
- Test updates in staging environment
- Monitor application metrics during rollout
- Use meaningful update messages with --record

### Configuration Tips
- Conservative maxUnavailable for critical apps
- Use maxSurge=0 for resource-constrained clusters
- Implement proper monitoring and alerting
- Plan rollback procedures

### Health Checks
```yaml
spec:
  template:
    spec:
      containers:
      - name: app
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```