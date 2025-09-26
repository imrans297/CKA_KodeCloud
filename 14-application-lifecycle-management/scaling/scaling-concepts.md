# Application Scaling

## 🎯 Scaling Concepts

### What is Application Scaling?
- **Adjusting resources** to meet demand
- **Horizontal scaling** - Add/remove pod replicas
- **Vertical scaling** - Increase/decrease resource limits
- **Auto-scaling** - Automatic scaling based on metrics

### Types of Scaling
- **Manual Scaling** - Operator-controlled
- **Horizontal Pod Autoscaler (HPA)** - CPU/memory based
- **Vertical Pod Autoscaler (VPA)** - Resource optimization
- **Cluster Autoscaler** - Node-level scaling

## 📊 Horizontal Scaling

### Manual Horizontal Scaling
```bash
# Scale deployment replicas
kubectl scale deployment webapp --replicas=5

# Scale ReplicaSet
kubectl scale rs webapp-rs --replicas=3

# Scale from file
kubectl scale --replicas=10 -f deployment.yaml
```

### Horizontal Pod Autoscaler (HPA)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### HPA Commands
```bash
# Create HPA
kubectl autoscale deployment webapp --cpu-percent=70 --min=2 --max=10

# List HPAs
kubectl get hpa

# Describe HPA
kubectl describe hpa webapp-hpa

# Check HPA status
kubectl get hpa webapp-hpa -w
```

## 📈 Vertical Scaling

### Manual Resource Updates
```bash
# Update resource limits
kubectl patch deployment webapp -p '{"spec":{"template":{"spec":{"containers":[{"name":"webapp","resources":{"limits":{"cpu":"1000m","memory":"1Gi"}}}]}}}}'

# Edit deployment
kubectl edit deployment webapp
```

### Vertical Pod Autoscaler (VPA)
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: webapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: webapp
      maxAllowed:
        cpu: 2
        memory: 2Gi
      minAllowed:
        cpu: 100m
        memory: 128Mi
```

## 🔄 Auto-Scaling Strategies

### CPU-Based Scaling
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cpu-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

### Memory-Based Scaling
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: memory-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 8
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75
```

### Custom Metrics Scaling
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: custom-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
```

## 🎛️ Scaling Behavior

### Advanced HPA Configuration
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: advanced-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

## 📊 Monitoring Scaling

### Scaling Metrics
```bash
# Check HPA status
kubectl get hpa
kubectl describe hpa webapp-hpa

# Monitor pod scaling
kubectl get pods -l app=webapp -w

# Check resource usage
kubectl top pods -l app=webapp
kubectl top nodes
```

### Scaling Events
```bash
# Check scaling events
kubectl get events --field-selector involvedObject.name=webapp-hpa

# Monitor deployment events
kubectl describe deployment webapp | grep -A 10 Events
```

## 🚨 Scaling Troubleshooting

### Common Issues
```bash
# HPA not scaling
kubectl describe hpa webapp-hpa
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods

# Metrics server issues
kubectl get pods -n kube-system | grep metrics-server
kubectl logs -n kube-system deployment/metrics-server

# Resource constraints
kubectl describe nodes
kubectl get events --field-selector reason=FailedScheduling
```

### Debug Commands
```bash
# Check pod resource requests
kubectl describe pod <pod-name> | grep -A 5 "Requests\|Limits"

# Verify metrics availability
kubectl top pods
kubectl top nodes

# Check HPA conditions
kubectl get hpa webapp-hpa -o yaml | grep -A 10 conditions
```

## 📋 Scaling Best Practices

### Design Guidelines
- Set appropriate resource requests and limits
- Use meaningful scaling metrics
- Configure proper scaling policies
- Test scaling behavior under load
- Monitor scaling performance

### Resource Planning
```yaml
# Proper resource configuration
spec:
  containers:
  - name: webapp
    resources:
      requests:
        cpu: 200m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi
```

### Scaling Policies
- Conservative scale-down policies
- Aggressive scale-up for traffic spikes
- Appropriate stabilization windows
- Consider application startup time

## 🔄 Load Testing for Scaling

### Generate Load
```bash
# Simple load test
kubectl run load-generator --image=busybox --rm -it --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://webapp-service/; done"

# Multiple load generators
for i in {1..5}; do
  kubectl run load-generator-$i --image=busybox --rm --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://webapp-service/; sleep 0.01; done" &
done
```

### Monitor Scaling Response
```bash
# Watch HPA scaling
kubectl get hpa webapp-hpa -w

# Monitor pod count
watch kubectl get pods -l app=webapp

# Check resource usage
watch kubectl top pods -l app=webapp
```

## 🎯 Scaling Strategies

### Predictive Scaling
- Schedule scaling based on known patterns
- Use CronJobs for time-based scaling
- Implement custom controllers

### Reactive Scaling
- Scale based on current metrics
- Use HPA for automatic scaling
- Monitor and adjust thresholds

### Proactive Scaling
- Scale before demand increases
- Use external metrics and predictions
- Implement custom scaling logic