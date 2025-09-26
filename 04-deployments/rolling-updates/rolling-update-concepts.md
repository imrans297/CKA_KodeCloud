# Rolling Updates

## 🎯 Rolling Update Concepts

### What are Rolling Updates?
- Gradual replacement of old pods with new ones
- Zero-downtime deployment strategy
- Default deployment strategy in Kubernetes
- Maintains service availability during updates

## 🔄 Rolling Update Process

### Update Flow
```
1. Create new ReplicaSet with updated spec
2. Scale up new ReplicaSet gradually
3. Scale down old ReplicaSet gradually
4. Continue until all pods are updated
5. Delete old ReplicaSet (keep for rollback)
```

### Strategy Configuration
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1      # Max pods that can be unavailable
      maxSurge: 1           # Max pods above desired count
```

## 🚀 Performing Rolling Updates

### Update Image
```bash
# Update deployment image
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Update with record (for rollback history)
kubectl set image deployment/nginx-deployment nginx=nginx:1.21 --record

# Update from file
kubectl apply -f updated-deployment.yaml
```

### Update Environment Variables
```bash
# Update environment variable
kubectl set env deployment/nginx-deployment ENV_VAR=new-value

# Update multiple env vars
kubectl set env deployment/nginx-deployment VAR1=value1 VAR2=value2
```

### Update Resources
```bash
# Update resource limits
kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","resources":{"limits":{"cpu":"200m"}}}]}}}}'
```

## 📊 Monitoring Rolling Updates

### Check Rollout Status
```bash
# Watch rollout progress
kubectl rollout status deployment/nginx-deployment

# Get rollout history
kubectl rollout history deployment/nginx-deployment

# Detailed rollout info
kubectl describe deployment nginx-deployment
```

### Monitor Pods During Update
```bash
# Watch pods being replaced
kubectl get pods -l app=nginx -w

# Check ReplicaSets
kubectl get rs -l app=nginx

# Monitor events
kubectl get events --sort-by=.metadata.creationTimestamp
```

## ⚙️ Rolling Update Parameters

### maxUnavailable
```yaml
# Absolute number
maxUnavailable: 2

# Percentage
maxUnavailable: 25%

# Default: 25%
```

### maxSurge
```yaml
# Absolute number
maxSurge: 2

# Percentage  
maxSurge: 25%

# Default: 25%
```

### Example Configurations
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

### Pause Rollout
```bash
# Pause ongoing rollout
kubectl rollout pause deployment/nginx-deployment

# Resume paused rollout
kubectl rollout resume deployment/nginx-deployment
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

### Configuration Tips
- Conservative maxUnavailable for critical apps
- Use maxSurge=0 for resource-constrained clusters
- Record rollout reasons with --record flag
- Implement proper monitoring and alerting