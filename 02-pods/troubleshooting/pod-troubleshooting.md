# Pod Troubleshooting Guide

## 🚨 Common Pod Issues

### ImagePullBackOff
**Symptoms:** Pod stuck in ImagePullBackOff state
```bash
# Check pod status
kubectl describe pod <pod-name>

# Common causes:
# - Wrong image name/tag
# - Image doesn't exist
# - Registry authentication issues
# - Network connectivity problems
```

### CrashLoopBackOff
**Symptoms:** Pod continuously restarting
```bash
# Check pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Common causes:
# - Application crashes on startup
# - Missing dependencies
# - Configuration errors
# - Resource constraints
```

### Pending State
**Symptoms:** Pod stuck in Pending phase
```bash
# Check scheduling issues
kubectl describe pod <pod-name>

# Common causes:
# - Insufficient resources
# - Node selector constraints
# - Taints and tolerations
# - Pod affinity/anti-affinity
```

## 🔍 Debugging Commands

### Basic Diagnostics
```bash
# Get pod status
kubectl get pods
kubectl get pods -o wide

# Detailed pod information
kubectl describe pod <pod-name>

# Pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -f
kubectl logs <pod-name> --previous
kubectl logs <pod-name> -c <container-name>

# Execute into pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

### Advanced Debugging
```bash
# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=<pod-name>

# Check resource usage
kubectl top pod <pod-name>

# Port forwarding for testing
kubectl port-forward <pod-name> 8080:80

# Copy files from/to pod
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

## 🔧 Troubleshooting Scenarios

### Scenario 1: Pod Won't Start
```bash
# Step 1: Check pod status
kubectl get pod <pod-name> -o yaml

# Step 2: Check events
kubectl describe pod <pod-name>

# Step 3: Check node resources
kubectl describe node <node-name>

# Step 4: Check image availability
docker pull <image-name>
```

### Scenario 2: Pod Crashes
```bash
# Step 1: Check current logs
kubectl logs <pod-name>

# Step 2: Check previous container logs
kubectl logs <pod-name> --previous

# Step 3: Check resource limits
kubectl describe pod <pod-name> | grep -A 5 Limits

# Step 4: Check liveness probe
kubectl describe pod <pod-name> | grep -A 10 Liveness
```

### Scenario 3: Pod Not Ready
```bash
# Step 1: Check readiness probe
kubectl describe pod <pod-name> | grep -A 10 Readiness

# Step 2: Test application endpoint
kubectl exec -it <pod-name> -- curl localhost:8080/health

# Step 3: Check service endpoints
kubectl get endpoints <service-name>
```

## 🛠️ Debug Pod Examples

### Debug Pod for Network Testing
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  containers:
  - name: debug
    image: busybox:1.35
    command: ['sleep', '3600']
    # Tools available: wget, nslookup, ping, telnet
```

### Debug Pod with Network Tools
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: netshoot-debug
spec:
  containers:
  - name: netshoot
    image: nicolaka/netshoot
    command: ['sleep', '3600']
    # Advanced network debugging tools
```

## 📊 Resource Issues

### CPU/Memory Limits
```bash
# Check resource usage
kubectl top pod <pod-name>

# Check resource requests/limits
kubectl describe pod <pod-name> | grep -A 10 "Requests\|Limits"

# Check node capacity
kubectl describe node <node-name> | grep -A 10 "Capacity\|Allocatable"
```

### Storage Issues
```bash
# Check volume mounts
kubectl describe pod <pod-name> | grep -A 10 Mounts

# Check persistent volumes
kubectl get pv
kubectl describe pv <pv-name>

# Check persistent volume claims
kubectl get pvc
kubectl describe pvc <pvc-name>
```

## 🔐 Security Issues

### Service Account Problems
```bash
# Check service account
kubectl get pod <pod-name> -o yaml | grep serviceAccount

# Check RBAC permissions
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<sa-name>
```

### Image Pull Secrets
```bash
# Check image pull secrets
kubectl get pod <pod-name> -o yaml | grep imagePullSecrets

# Create image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<username> \
  --docker-password=<password>
```

## 📋 Troubleshooting Checklist

### Pre-flight Checks:
- [ ] Image exists and is accessible
- [ ] Node has sufficient resources
- [ ] Network connectivity is working
- [ ] Required secrets/configmaps exist

### Runtime Checks:
- [ ] Pod is scheduled to a node
- [ ] Containers are starting successfully
- [ ] Health probes are passing
- [ ] Application is responding correctly

### Post-mortem Analysis:
- [ ] Check pod logs for errors
- [ ] Review resource usage patterns
- [ ] Analyze failure patterns
- [ ] Update configurations as needed