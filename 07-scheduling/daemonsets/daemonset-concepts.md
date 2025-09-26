# DaemonSets

## 🎯 DaemonSet Concepts

### What is a DaemonSet?
- Ensures a copy of a pod runs on all (or some) nodes
- Automatically schedules pods on new nodes
- Removes pods when nodes are removed
- Used for node-level services and monitoring

### Common Use Cases
- Log collection (Fluentd, Filebeat)
- Monitoring agents (Node Exporter, Datadog)
- Network plugins (Calico, Flannel)
- Storage daemons (Ceph, GlusterFS)

## 🔧 DaemonSet Configuration

### Basic DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## 📊 DaemonSet Scheduling

### Node Selection
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      nodeSelector:
        monitoring: "enabled"
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
```

### Tolerations for System Nodes
```yaml
spec:
  template:
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      containers:
      - name: system-monitor
        image: monitoring-agent:latest
```

## 🔄 DaemonSet Updates

### Rolling Update Strategy
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: network-plugin
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      app: network-plugin
  template:
    metadata:
      labels:
        app: network-plugin
    spec:
      containers:
      - name: calico-node
        image: calico/node:v3.24.0
```

### OnDelete Update Strategy
```yaml
spec:
  updateStrategy:
    type: OnDelete
  # Pods updated only when manually deleted
```

## 🚀 DaemonSet Operations

### Create DaemonSet
```bash
# Create from YAML
kubectl apply -f daemonset.yaml

# Generate DaemonSet YAML
kubectl create daemonset log-collector --image=fluentd:v1.14 --dry-run=client -o yaml
```

### Manage DaemonSet
```bash
# Get DaemonSets
kubectl get daemonsets
kubectl get ds

# Describe DaemonSet
kubectl describe daemonset log-collector

# Check DaemonSet status
kubectl get ds log-collector -o wide
```

### Update DaemonSet
```bash
# Update image
kubectl set image daemonset/log-collector fluentd=fluentd:v1.15

# Edit DaemonSet
kubectl edit daemonset log-collector

# Check rollout status
kubectl rollout status daemonset/log-collector
```

## 📋 DaemonSet vs Other Controllers

### DaemonSet vs Deployment
- **DaemonSet**: One pod per node
- **Deployment**: Specified number of replicas

### DaemonSet vs StatefulSet
- **DaemonSet**: Node-based scheduling
- **StatefulSet**: Ordered, persistent identity

### When to Use DaemonSet
- Node-level services required
- System monitoring/logging
- Network or storage plugins
- Security agents

## 🔍 Monitoring DaemonSets

### Check Pod Distribution
```bash
# See pods across nodes
kubectl get pods -l app=log-collector -o wide

# Check which nodes have DaemonSet pods
kubectl get nodes
kubectl describe daemonset log-collector
```

### DaemonSet Status
```bash
# Check desired vs current pods
kubectl get daemonset log-collector

# Detailed status
kubectl describe daemonset log-collector | grep -A 10 "Pod Status"
```

## 🚨 Troubleshooting DaemonSets

### Common Issues
```bash
# Pods not scheduled on all nodes
kubectl describe daemonset <daemonset-name>

# Check node selectors and tolerations
kubectl get daemonset <daemonset-name> -o yaml

# Check node labels and taints
kubectl get nodes --show-labels
kubectl describe node <node-name> | grep Taints
```

### Debug Commands
```bash
# Check DaemonSet events
kubectl describe daemonset <daemonset-name>

# Check pod logs
kubectl logs -l app=<app-label>

# Check why pod not scheduled on specific node
kubectl describe pod <pod-name>
```

## 🔐 Security Considerations

### Privileged DaemonSets
```yaml
spec:
  template:
    spec:
      securityContext:
        runAsUser: 0
      containers:
      - name: privileged-agent
        image: security-agent:latest
        securityContext:
          privileged: true
        volumeMounts:
        - name: host-root
          mountPath: /host
          readOnly: true
      volumes:
      - name: host-root
        hostPath:
          path: /
```

### Service Account
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: daemonset-sa
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: secure-daemonset
spec:
  template:
    spec:
      serviceAccountName: daemonset-sa
      containers:
      - name: app
        image: app:latest
```

## 📊 Resource Management

### Resource Limits
```yaml
spec:
  template:
    spec:
      containers:
      - name: resource-limited
        image: monitoring-agent:latest
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
```

### Priority Class
```yaml
spec:
  template:
    spec:
      priorityClassName: system-node-critical
      containers:
      - name: critical-daemon
        image: system-daemon:latest
```

## 📋 Best Practices

### Design Guidelines
- Use appropriate resource limits
- Implement proper health checks
- Use tolerations for system nodes
- Consider security implications

### Operational Tips
- Monitor DaemonSet pod distribution
- Plan for node maintenance
- Test updates in staging
- Use rolling updates carefully

### Common Patterns
```yaml
# System DaemonSet pattern
tolerations:
- operator: Exists
  effect: NoSchedule
- operator: Exists
  effect: NoExecute

# Monitoring DaemonSet pattern
hostNetwork: true
hostPID: true
```