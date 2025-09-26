# Security Contexts

## 🎯 Security Context Concepts

### What are Security Contexts?
- Define privilege and access control settings
- Applied at pod or container level
- Control user ID, group ID, capabilities
- Enforce security policies

### Security Context Levels
- **Pod Security Context**: Applies to all containers
- **Container Security Context**: Overrides pod settings
- **Pod Security Standards**: Cluster-wide policies

## 🔧 User and Group Settings

### Run as User
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - name: app
    image: nginx
```

### Run as Non-Root
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: app
    image: nginx
```

### File System Group
```yaml
spec:
  securityContext:
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    emptyDir: {}
```

## 🛡️ Capabilities

### Add Capabilities
```yaml
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

### Drop Capabilities
```yaml
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]
```

### Drop All Capabilities
```yaml
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        drop: ["ALL"]
```

## 🔒 Privileged Containers

### Privileged Mode
```yaml
spec:
  containers:
  - name: privileged-app
    image: nginx
    securityContext:
      privileged: true
```

### Allow Privilege Escalation
```yaml
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
```

## 📁 File System Security

### Read-Only Root Filesystem
```yaml
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: var-cache
      mountPath: /var/cache/nginx
  volumes:
  - name: tmp
    emptyDir: {}
  - name: var-cache
    emptyDir: {}
```

### SELinux Options
```yaml
spec:
  securityContext:
    seLinuxOptions:
      level: "s0:c123,c456"
```

## 🔧 Container Security Context

### Container-Level Settings
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-security-demo
spec:
  containers:
  - name: app1
    image: nginx
    securityContext:
      runAsUser: 1000
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]
  - name: app2
    image: busybox
    command: ['sleep', '3600']
    securityContext:
      runAsUser: 2000
      runAsNonRoot: true
```

## 🛡️ Pod Security Standards

### Privileged Profile
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: privileged-ns
  labels:
    pod-security.kubernetes.io/enforce: privileged
    pod-security.kubernetes.io/audit: privileged
    pod-security.kubernetes.io/warn: privileged
```

### Baseline Profile
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: baseline-ns
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/audit: baseline
    pod-security.kubernetes.io/warn: baseline
```

### Restricted Profile
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: restricted-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

## 🚀 Security Context Operations

### Check Security Context
```bash
# Describe pod security context
kubectl describe pod <pod-name>

# Get pod security context
kubectl get pod <pod-name> -o yaml | grep -A 10 securityContext

# Check running processes in container
kubectl exec <pod-name> -- ps aux
```

### Test Security Settings
```bash
# Check user ID
kubectl exec <pod-name> -- id

# Check capabilities
kubectl exec <pod-name> -- capsh --print

# Check file permissions
kubectl exec <pod-name> -- ls -la /
```

## 🚨 Troubleshooting Security Contexts

### Common Issues
```bash
# Permission denied errors
kubectl logs <pod-name>
kubectl describe pod <pod-name>

# Check security context settings
kubectl get pod <pod-name> -o yaml

# Verify user/group settings
kubectl exec <pod-name> -- id
```

### Debug Commands
```bash
# Check pod security standards
kubectl get ns <namespace> -o yaml

# Test file system permissions
kubectl exec <pod-name> -- touch /tmp/test

# Check process capabilities
kubectl exec <pod-name> -- grep Cap /proc/self/status
```

## 📋 Best Practices

### Security Guidelines
- Always run as non-root user
- Drop all capabilities by default
- Use read-only root filesystem
- Disable privilege escalation
- Set appropriate file system group

### Common Patterns
```yaml
# Secure container template
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

### Monitoring
- Audit security context violations
- Monitor privileged containers
- Regular security assessments
- Automated policy enforcement