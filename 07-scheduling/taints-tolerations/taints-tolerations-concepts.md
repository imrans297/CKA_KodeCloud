# Taints and Tolerations

## 🎯 Taints and Tolerations Concepts

### What are Taints?
- Applied to nodes to repel pods
- Prevent pods from being scheduled unless they tolerate the taint
- Used for dedicated nodes, special hardware, or maintenance

### What are Tolerations?
- Applied to pods to tolerate specific taints
- Allow pods to be scheduled on tainted nodes
- Must match taint key, value, and effect

## 🔧 Taint Effects

### NoSchedule
- New pods won't be scheduled on the node
- Existing pods remain unaffected
```bash
kubectl taint nodes node1 key1=value1:NoSchedule
```

### PreferNoSchedule
- Scheduler tries to avoid placing pods on the node
- Not a hard requirement
```bash
kubectl taint nodes node1 key1=value1:PreferNoSchedule
```

### NoExecute
- New pods won't be scheduled
- Existing pods without toleration are evicted
```bash
kubectl taint nodes node1 key1=value1:NoExecute
```

## 🏷️ Working with Taints

### Adding Taints
```bash
# Basic taint
kubectl taint nodes worker-1 dedicated=gpu:NoSchedule

# Taint with value
kubectl taint nodes worker-1 workload=database:NoSchedule

# NoExecute taint
kubectl taint nodes worker-1 maintenance=true:NoExecute

# PreferNoSchedule taint
kubectl taint nodes worker-1 spot-instance=true:PreferNoSchedule
```

### Viewing Taints
```bash
# Show node taints
kubectl describe node worker-1 | grep Taints

# Get all nodes with taints
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

### Removing Taints
```bash
# Remove specific taint
kubectl taint nodes worker-1 dedicated=gpu:NoSchedule-

# Remove taint by key (all effects)
kubectl taint nodes worker-1 dedicated-

# Remove all taints from node
kubectl patch node worker-1 -p '{"spec":{"taints":[]}}'
```

## 🛡️ Pod Tolerations

### Basic Toleration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
  containers:
  - name: app
    image: nginx
```

### Toleration Operators

#### Equal Operator
```yaml
tolerations:
- key: "workload"
  operator: "Equal"
  value: "database"
  effect: "NoSchedule"
```

#### Exists Operator
```yaml
tolerations:
- key: "dedicated"
  operator: "Exists"
  effect: "NoSchedule"
```

### Tolerate All Taints
```yaml
tolerations:
- operator: "Exists"
```

## ⏰ Toleration Seconds

### NoExecute with Timeout
```yaml
tolerations:
- key: "maintenance"
  operator: "Equal"
  value: "true"
  effect: "NoExecute"
  tolerationSeconds: 3600  # Stay for 1 hour
```

### Use Cases
- Graceful pod eviction during maintenance
- Temporary toleration of node issues
- Controlled workload migration

## 🎯 Common Taint Scenarios

### Dedicated Nodes
```bash
# Taint GPU nodes
kubectl taint nodes gpu-node-1 dedicated=gpu:NoSchedule

# GPU pod toleration
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
```

### Master Node Taints
```bash
# Default master taint
kubectl describe node master | grep Taints
# Taints: node-role.kubernetes.io/master:NoSchedule

# Tolerate master taint
tolerations:
- key: "node-role.kubernetes.io/master"
  operator: "Exists"
  effect: "NoSchedule"
```

### Spot Instance Nodes
```bash
# Taint spot instances
kubectl taint nodes spot-node-1 spot-instance=true:PreferNoSchedule

# Spot-tolerant workload
tolerations:
- key: "spot-instance"
  operator: "Equal"
  value: "true"
  effect: "PreferNoSchedule"
```

## 🚀 Deployment with Tolerations

### Deployment Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gpu-app
  template:
    metadata:
      labels:
        app: gpu-app
    spec:
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "gpu"
        effect: "NoSchedule"
      - key: "nvidia.com/gpu"
        operator: "Exists"
        effect: "NoSchedule"
      containers:
      - name: gpu-app
        image: tensorflow/tensorflow:latest-gpu
        resources:
          limits:
            nvidia.com/gpu: 1
```

## 🔄 Combining with Node Affinity

### Dedicated GPU Nodes
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "dedicated"
            operator: In
            values: ["gpu"]
  containers:
  - name: gpu-app
    image: tensorflow/tensorflow:latest-gpu
```

## 🚨 Troubleshooting

### Pod Scheduling Issues
```bash
# Check why pod is not scheduled
kubectl describe pod <pod-name>

# Check node taints
kubectl describe node <node-name> | grep Taints

# Check pod tolerations
kubectl get pod <pod-name> -o yaml | grep -A 10 tolerations
```

### Common Problems
```bash
# Pod pending due to taints
kubectl get pods
kubectl describe pod <pending-pod>

# Check if toleration matches taint exactly
kubectl describe node <node-name>
kubectl get pod <pod-name> -o yaml
```

## 📋 Best Practices

### Taint Management
- Use descriptive taint keys and values
- Document taint purposes
- Plan taint removal procedures
- Monitor tainted node utilization

### Toleration Design
- Match tolerations exactly to taints
- Use appropriate toleration seconds
- Combine with node affinity when needed
- Test toleration configurations

### Common Patterns
```yaml
# System pods toleration
tolerations:
- operator: "Exists"
  effect: "NoExecute"
- operator: "Exists"
  effect: "NoSchedule"

# Maintenance toleration
tolerations:
- key: "maintenance"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 300
```