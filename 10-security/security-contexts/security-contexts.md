# Kubernetes Security Contexts

## 🎯 Overview

Security contexts define privilege and access control settings for pods and containers, controlling user IDs, group IDs, capabilities, and security policies.

## 🔐 Security Context Levels

### Pod-Level Security Context
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-security-context
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    runAsNonRoot: true
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
    seLinuxOptions:
      level: "s0:c123,c456"
  containers:
  - name: app
    image: nginx
```

### Container-Level Security Context
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-security-context
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsUser: 2000
      runAsGroup: 3000
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN"]
        drop: ["ALL"]
      seccompProfile:
        type: RuntimeDefault
```

## 👤 User and Group Settings

### Run as Specific User
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: run-as-user
spec:
  securityContext:
    runAsUser: 1001    # UID
    runAsGroup: 2001   # GID
  containers:
  - name: app
    image: busybox
    command: ["sleep", "3600"]
```

### Run as Non-Root
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsNonRoot: true  # Prevents running as root
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsUser: 1000
```

### File System Group
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fs-group-pod
spec:
  securityContext:
    fsGroup: 2000  # All volumes owned by this group
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

## 🛡️ Capabilities Management

### Drop All Capabilities
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: drop-all-caps
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        drop:
        - ALL  # Drop all Linux capabilities
```

### Add Specific Capabilities
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: add-capabilities
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        add:
        - NET_ADMIN    # Network administration
        - SYS_TIME     # System time modification
        drop:
        - ALL
```

### Common Capabilities
```yaml
# Network capabilities
capabilities:
  add: ["NET_ADMIN", "NET_RAW", "NET_BIND_SERVICE"]

# File system capabilities  
capabilities:
  add: ["DAC_OVERRIDE", "CHOWN", "FOWNER"]

# Process capabilities
capabilities:
  add: ["KILL", "SETUID", "SETGID"]

# System capabilities
capabilities:
  add: ["SYS_ADMIN", "SYS_TIME", "SYS_RESOURCE"]
```

## 🔒 Advanced Security Settings

### Read-Only Root Filesystem
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-root
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
    - name: var-run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: var-cache
    emptyDir: {}
  - name: var-run
    emptyDir: {}
```

### Prevent Privilege Escalation
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-privilege-escalation
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false  # Prevents setuid binaries
      runAsNonRoot: true
      capabilities:
        drop:
        - ALL
```

### Privileged Containers (Avoid in Production)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      privileged: true  # Full host access - DANGEROUS
```

## 🔐 SELinux and AppArmor

### SELinux Options
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selinux-pod
spec:
  securityContext:
    seLinuxOptions:
      level: "s0:c123,c456"
      role: "system_r"
      type: "container_t"
      user: "system_u"
  containers:
  - name: app
    image: nginx
```

### AppArmor Profile
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/app: runtime/default
spec:
  containers:
  - name: app
    image: nginx
```

## 🛠️ Seccomp Profiles

### Default Seccomp Profile
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-default
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault  # Use container runtime default
  containers:
  - name: app
    image: nginx
```

### Custom Seccomp Profile
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-custom
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json
  containers:
  - name: app
    image: nginx
```

### Seccomp Profile Example
```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["read", "write", "open", "close", "stat"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

## 📋 Pod Security Standards

### Privileged Profile
```yaml
# No restrictions - allows privileged containers
apiVersion: v1
kind: Namespace
metadata:
  name: privileged-ns
  labels:
    pod-security.kubernetes.io/enforce: privileged
```

### Baseline Profile
```yaml
# Minimal restrictions - prevents most privilege escalations
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
# Heavily restricted - follows pod hardening best practices
apiVersion: v1
kind: Namespace
metadata:
  name: restricted-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

## 🎯 Real-World Examples

### Web Application Security Context
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: webapp
        image: nginx:1.20
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: cache
          mountPath: /var/cache/nginx
      volumes:
      - name: tmp
        emptyDir: {}
      - name: cache
        emptyDir: {}
```

### Database Security Context
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database
spec:
  securityContext:
    runAsUser: 999
    runAsGroup: 999
    fsGroup: 999
    runAsNonRoot: true
  containers:
  - name: postgres
    image: postgres:13
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    env:
    - name: POSTGRES_DB
      value: mydb
    - name: POSTGRES_USER
      value: dbuser
    - name: POSTGRES_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: data
      mountPath: /var/lib/postgresql/data
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: postgres-pvc
  - name: tmp
    emptyDir: {}
```

### Init Container Security
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-security
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  initContainers:
  - name: init-setup
    image: busybox
    securityContext:
      runAsUser: 0  # Init container may need root
      allowPrivilegeEscalation: false
      capabilities:
        add:
        - CHOWN
        drop:
        - ALL
    command: ['sh', '-c', 'chown -R 1000:2000 /shared-data']
    volumeMounts:
    - name: shared-data
      mountPath: /shared-data
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

## 🔍 Security Context Validation

### Check Running Processes
```bash
# Check user/group of running processes
kubectl exec -it pod-name -- ps aux

# Check file permissions
kubectl exec -it pod-name -- ls -la /

# Check capabilities
kubectl exec -it pod-name -- cat /proc/1/status | grep Cap
```

### Verify Security Settings
```bash
# Check effective user ID
kubectl exec -it pod-name -- id

# Check mounted filesystems
kubectl exec -it pod-name -- mount | grep -E "(ro|rw)"

# Check seccomp status
kubectl exec -it pod-name -- cat /proc/1/status | grep Seccomp
```

## 🚨 Security Context Best Practices

### Secure Defaults
```yaml
# Template for secure pod
apiVersion: v1
kind: Pod
metadata:
  name: secure-template
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

### Security Checklist
- ✅ Run as non-root user
- ✅ Drop all capabilities
- ✅ Read-only root filesystem
- ✅ Prevent privilege escalation
- ✅ Use seccomp profiles
- ✅ Set resource limits
- ✅ Use specific user/group IDs
- ❌ Avoid privileged containers
- ❌ Don't run as UID 0

## 🛠️ Troubleshooting Security Contexts

### Common Issues
```bash
# Permission denied errors
kubectl logs pod-name
# Solution: Check runAsUser, fsGroup settings

# Container won't start
kubectl describe pod pod-name
# Solution: Verify image supports non-root user

# File system errors
kubectl exec -it pod-name -- ls -la /path
# Solution: Check fsGroup, volume permissions
```

### Debug Commands
```bash
# Check security context applied
kubectl get pod pod-name -o jsonpath='{.spec.securityContext}'

# Check container security context
kubectl get pod pod-name -o jsonpath='{.spec.containers[0].securityContext}'

# Verify effective settings
kubectl exec -it pod-name -- cat /proc/1/status
```