# ReplicaSet Scaling

## 🎯 Scaling Concepts

### What is Scaling?
- Adjusting number of pod replicas
- Horizontal scaling (more/fewer pods)
- Maintains desired state automatically
- Handles pod failures and recovery

## 📈 Scaling Methods

### Imperative Scaling
```bash
# Scale using kubectl
kubectl scale rs nginx-replicaset --replicas=5

# Scale from file
kubectl scale --replicas=3 -f replicaset.yaml

# Scale multiple resources
kubectl scale rs nginx-rs web-rs --replicas=2
```

### Declarative Scaling
```bash
# Edit ReplicaSet
kubectl edit rs nginx-replicaset

# Patch ReplicaSet
kubectl patch rs nginx-replicaset -p '{"spec":{"replicas":4}}'

# Apply updated YAML
kubectl apply -f updated-replicaset.yaml
```

## 🔧 Scaling Commands

### Basic Scaling
```bash
# Current replica count
kubectl get rs nginx-replicaset

# Scale up
kubectl scale rs nginx-replicaset --replicas=10

# Scale down
kubectl scale rs nginx-replicaset --replicas=2

# Scale to zero (delete all pods)
kubectl scale rs nginx-replicaset --replicas=0
```

### Conditional Scaling
```bash
# Scale only if current replicas match condition
kubectl scale rs nginx-replicaset --current-replicas=3 --replicas=5

# This prevents accidental scaling if someone else changed it
```

## 📊 Monitoring Scaling

### Watch Scaling Process
```bash
# Watch ReplicaSet status
kubectl get rs nginx-replicaset -w

# Watch pods being created/deleted
kubectl get pods -l app=nginx -w

# Check scaling events
kubectl describe rs nginx-replicaset | grep Events -A 10
```

### Scaling Status
```bash
# Check desired vs current replicas
kubectl get rs nginx-replicaset -o wide

# Detailed status
kubectl describe rs nginx-replicaset

# Pod distribution across nodes
kubectl get pods -l app=nginx -o wide
```

## 🚨 Scaling Troubleshooting

### Common Issues
```bash
# Pods not scaling up
kubectl describe rs nginx-replicaset
kubectl get events --field-selector involvedObject.name=nginx-replicaset

# Check node resources
kubectl describe nodes
kubectl top nodes

# Check pod resource requests
kubectl describe rs nginx-replicaset | grep -A 10 "Pod Template"
```

### Resource Constraints
```bash
# Check if nodes have enough resources
kubectl describe node <node-name> | grep -A 10 "Allocated resources"

# Check resource quotas
kubectl describe quota --all-namespaces

# Check limit ranges
kubectl describe limitrange --all-namespaces
```

## 📋 Best Practices

### Scaling Guidelines
- Start with small replica counts
- Monitor resource usage during scaling
- Consider node capacity
- Use resource requests/limits
- Test scaling in non-production first

### Performance Considerations
- Scaling up: Gradual increase for large numbers
- Scaling down: Monitor application impact
- Resource planning: Ensure cluster capacity
- Network impact: Consider service mesh overhead