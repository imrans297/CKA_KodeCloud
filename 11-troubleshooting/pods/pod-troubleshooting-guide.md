# Pod Troubleshooting Guide

## 🚨 Common Pod Issues

### Pod Stuck in Pending
**Symptoms:** Pod remains in Pending state
**Causes:**
- Insufficient cluster resources
- Node selector constraints not met
- Taints and tolerations mismatch
- Pod affinity/anti-affinity rules
- Volume mounting issues

**Troubleshooting:**
```bash
# Check pod status and events
kubectl describe pod <pod-name>

# Check node resources
kubectl describe nodes
kubectl top nodes

# Check scheduler logs
kubectl logs -n kube-system -l component=kube-scheduler
```

### ImagePullBackOff
**Symptoms:** Pod fails to pull container image
**Causes:**
- Image doesn't exist
- Wrong image name/tag
- Registry authentication issues
- Network connectivity problems

**Troubleshooting:**
```bash
# Check pod events
kubectl describe pod <pod-name>

# Verify image exists
docker pull <image-name>

# Check image pull secrets
kubectl get secrets
kubectl describe secret <image-pull-secret>

# Test registry connectivity
kubectl run test --image=busybox --rm -it --restart=Never -- wget -qO- <registry-url>
```

### CrashLoopBackOff
**Symptoms:** Pod continuously restarts
**Causes:**
- Application crashes on startup
- Missing dependencies
- Configuration errors
- Resource constraints
- Health check failures

**Troubleshooting:**
```bash
# Check current logs
kubectl logs <pod-name>

# Check previous container logs
kubectl logs <pod-name> --previous

# Check resource limits
kubectl describe pod <pod-name> | grep -A 5 Limits

# Execute into running container
kubectl exec -it <pod-name> -- /bin/sh
```

## 🔍 Diagnostic Commands

### Basic Pod Information
```bash
# Get pod status
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels

# Detailed pod information
kubectl describe pod <pod-name>

# Pod YAML configuration
kubectl get pod <pod-name> -o yaml
```

### Pod Logs and Events
```bash
# View pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -f
kubectl logs <pod-name> --tail=50

# Multi-container pod logs
kubectl logs <pod-name> -c <container-name>

# Previous container logs
kubectl logs <pod-name> --previous

# Cluster events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### Pod Execution and Debugging
```bash
# Execute commands in pod
kubectl exec <pod-name> -- <command>
kubectl exec -it <pod-name> -- /bin/bash

# Multi-container pod execution
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh

# Copy files to/from pod
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# Port forwarding
kubectl port-forward <pod-name> 8080:80
```

## 🔧 Resource Issues

### CPU and Memory Problems
```bash
# Check resource usage
kubectl top pod <pod-name>
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# Check resource requests and limits
kubectl describe pod <pod-name> | grep -A 10 "Requests\|Limits"

# Check node capacity
kubectl describe node <node-name> | grep -A 10 "Capacity\|Allocatable"
```

### Storage Issues
```bash
# Check volume mounts
kubectl describe pod <pod-name> | grep -A 10 "Mounts\|Volumes"

# Check persistent volumes
kubectl get pv
kubectl describe pv <pv-name>

# Check persistent volume claims
kubectl get pvc
kubectl describe pvc <pvc-name>

# Check storage class
kubectl get storageclass
kubectl describe storageclass <sc-name>
```

## 🌐 Network Troubleshooting

### Connectivity Issues
```bash
# Test pod-to-pod connectivity
kubectl exec -it <source-pod> -- ping <target-pod-ip>

# Test service connectivity
kubectl exec -it <pod-name> -- curl <service-name>:<port>

# Check DNS resolution
kubectl exec -it <pod-name> -- nslookup <service-name>
kubectl exec -it <pod-name> -- dig <service-name>

# Check network policies
kubectl get networkpolicy
kubectl describe networkpolicy <policy-name>
```

### Service Discovery Issues
```bash
# Check service endpoints
kubectl get endpoints <service-name>
kubectl describe endpoints <service-name>

# Check service configuration
kubectl describe service <service-name>

# Test service from debug pod
kubectl run debug --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>
```

## 🔐 Security and Permissions

### RBAC Issues
```bash
# Check service account
kubectl get pod <pod-name> -o yaml | grep serviceAccount

# Check permissions
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<sa-name>

# Check role bindings
kubectl get rolebindings
kubectl describe rolebinding <binding-name>
```

### Security Context Problems
```bash
# Check security context
kubectl get pod <pod-name> -o yaml | grep -A 10 securityContext

# Check user and group
kubectl exec <pod-name> -- id

# Check file permissions
kubectl exec <pod-name> -- ls -la /
```

## 🛠️ Debug Utilities

### Debug Pod Template
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
    # Add network tools
  - name: netshoot
    image: nicolaka/netshoot
    command: ['sleep', '3600']
```

### Useful Debug Commands
```bash
# Network debugging
kubectl exec -it debug-pod -- netstat -tulpn
kubectl exec -it debug-pod -- ss -tulpn
kubectl exec -it debug-pod -- iptables -L

# Process debugging
kubectl exec -it <pod-name> -- ps aux
kubectl exec -it <pod-name> -- top

# File system debugging
kubectl exec -it <pod-name> -- df -h
kubectl exec -it <pod-name> -- mount
kubectl exec -it <pod-name> -- lsof
```

## 📊 Monitoring and Metrics

### Resource Monitoring
```bash
# Pod resource usage
kubectl top pods
kubectl top pods --containers

# Node resource usage
kubectl top nodes

# Detailed metrics (if metrics-server available)
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```

### Health Checks
```bash
# Check readiness and liveness probes
kubectl describe pod <pod-name> | grep -A 5 "Liveness\|Readiness"

# Test health endpoints
kubectl exec -it <pod-name> -- curl localhost:8080/health
```

## 🔄 Pod Lifecycle Issues

### Init Container Problems
```bash
# Check init container status
kubectl describe pod <pod-name> | grep -A 10 "Init Containers"

# Check init container logs
kubectl logs <pod-name> -c <init-container-name>
```

### Termination Issues
```bash
# Check termination reason
kubectl describe pod <pod-name> | grep -A 5 "Last State"

# Check termination grace period
kubectl get pod <pod-name> -o yaml | grep terminationGracePeriodSeconds

# Force delete stuck pod
kubectl delete pod <pod-name> --force --grace-period=0
```

## 📋 Troubleshooting Checklist

### Pre-flight Checks
- [ ] Image exists and is accessible
- [ ] Node has sufficient resources
- [ ] Network connectivity is working
- [ ] Required secrets/configmaps exist
- [ ] RBAC permissions are correct

### Runtime Checks
- [ ] Pod is scheduled to a node
- [ ] Containers are starting successfully
- [ ] Health probes are passing
- [ ] Application is responding correctly
- [ ] Logs show no errors

### Post-mortem Analysis
- [ ] Check pod logs for errors
- [ ] Review resource usage patterns
- [ ] Analyze failure patterns
- [ ] Update configurations as needed