# Kubernetes Scheduler

## 🎯 What is Scheduler?
- Assigns pods to nodes
- Watches for unscheduled pods
- Selects best node based on constraints
- Updates pod with nodeName

## 🔧 Scheduling Process

### 1. Filtering Phase:
- Remove nodes that don't meet requirements
- Check resource availability
- Evaluate node selectors
- Apply taints and tolerations

### 2. Scoring Phase:
- Rank remaining nodes
- Consider resource utilization
- Apply priority functions
- Select highest scoring node

## 📊 Scheduling Factors

### Resource Requirements:
```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 200m
        memory: 256Mi
```

### Node Selection:
```yaml
# Node Selector
spec:
  nodeSelector:
    disktype: ssd

# Node Affinity
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
```

### Taints and Tolerations:
```bash
# Add taint to node
kubectl taint nodes node1 key1=value1:NoSchedule

# Pod toleration
spec:
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```

## 🔧 Scheduler Configuration

```bash
# /etc/kubernetes/manifests/kube-scheduler.yaml
spec:
  containers:
  - command:
    - kube-scheduler
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
```

## 🚨 Troubleshooting

```bash
# Check scheduler
kubectl get pods -n kube-system | grep scheduler
kubectl logs kube-scheduler-master -n kube-system

# Check unscheduled pods
kubectl get pods --field-selector=status.phase=Pending

# Debug scheduling
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 📋 Manual Scheduling

```yaml
# Bypass scheduler with nodeName
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  nodeName: worker-node-1
  containers:
  - name: nginx
    image: nginx
```