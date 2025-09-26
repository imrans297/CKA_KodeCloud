# Node Affinity

## 🎯 Node Affinity Concepts

### What is Node Affinity?
- Advanced node selection mechanism
- More expressive than nodeSelector
- Supports required and preferred rules
- Uses matchExpressions for complex logic

### Types of Node Affinity
- **requiredDuringSchedulingIgnoredDuringExecution** - Hard requirement
- **preferredDuringSchedulingIgnoredDuringExecution** - Soft preference

## 🔧 Required Node Affinity

### Basic Required Affinity
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: required-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/arch
            operator: In
            values:
            - amd64
  containers:
  - name: nginx
    image: nginx
```

### Multiple Required Conditions
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
          - key: zone
            operator: In
            values:
            - us-west-1a
            - us-west-1b
```

## 🎯 Preferred Node Affinity

### Basic Preferred Affinity
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: preferred-affinity-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-west-1a
  containers:
  - name: nginx
    image: nginx
```

### Multiple Preferences with Weights
```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      - weight: 20
        preference:
          matchExpressions:
          - key: instance-type
            operator: In
            values:
            - m5.large
```

## 🔄 Combining Required and Preferred

### Mixed Affinity Rules
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/arch
            operator: In
            values:
            - amd64
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-west-1a
      - weight: 50
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

## 📊 Operators in Node Affinity

### Available Operators
- **In** - Label value is in the list
- **NotIn** - Label value is not in the list
- **Exists** - Label key exists
- **DoesNotExist** - Label key does not exist
- **Gt** - Label value is greater than
- **Lt** - Label value is less than

### Operator Examples
```yaml
matchExpressions:
# In operator
- key: zone
  operator: In
  values: [us-west-1a, us-west-1b]

# NotIn operator
- key: instance-type
  operator: NotIn
  values: [t2.micro, t2.small]

# Exists operator
- key: gpu
  operator: Exists

# DoesNotExist operator
- key: spot-instance
  operator: DoesNotExist

# Gt operator (numeric comparison)
- key: cpu-cores
  operator: Gt
  values: ["4"]
```

## 🏷️ Node Labels for Affinity

### Common Node Labels
```bash
# Built-in labels
kubernetes.io/arch=amd64
kubernetes.io/os=linux
kubernetes.io/hostname=worker-1
node.kubernetes.io/instance-type=m5.large

# Custom labels
kubectl label node worker-1 disktype=ssd
kubectl label node worker-1 zone=us-west-1a
kubectl label node worker-1 gpu=nvidia-v100
kubectl label node worker-1 workload=compute-intensive
```

### View Node Labels
```bash
# Show all node labels
kubectl get nodes --show-labels

# Show specific labels
kubectl get nodes -L disktype,zone,instance-type

# Describe node for all labels
kubectl describe node worker-1
```

## 🚀 Deployment with Node Affinity

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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: gpu
                operator: Exists
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: gpu-type
                operator: In
                values:
                - nvidia-v100
      containers:
      - name: gpu-app
        image: tensorflow/tensorflow:latest-gpu
```

## 🚨 Troubleshooting Node Affinity

### Common Issues
```bash
# Pod stuck in Pending
kubectl describe pod <pod-name>

# Check node labels
kubectl get nodes --show-labels

# Check if any nodes match affinity rules
kubectl get nodes -l disktype=ssd

# Check scheduler events
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Debug Commands
```bash
# Test affinity rules
kubectl get nodes -l 'zone in (us-west-1a,us-west-1b)'

# Check pod affinity configuration
kubectl get pod <pod-name> -o yaml | grep -A 20 affinity

# Verify node has required labels
kubectl describe node <node-name> | grep Labels -A 10
```

## 📋 Best Practices

### Design Guidelines
- Use required affinity for hard constraints
- Use preferred affinity for optimization
- Combine with resource requests/limits
- Test affinity rules before production

### Performance Considerations
- Avoid overly complex affinity rules
- Use weights effectively in preferred rules
- Consider cluster size and node diversity
- Monitor scheduling latency

### Common Patterns
```yaml
# High availability across zones
requiredDuringSchedulingIgnoredDuringExecution:
  nodeSelectorTerms:
  - matchExpressions:
    - key: zone
      operator: In
      values: [us-west-1a, us-west-1b, us-west-1c]

# Prefer SSD but allow HDD
preferredDuringSchedulingIgnoredDuringExecution:
- weight: 100
  preference:
    matchExpressions:
    - key: disktype
      operator: In
      values: [ssd]
```