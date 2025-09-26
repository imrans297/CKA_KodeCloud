# Pod Lifecycle

## 🔄 Pod Phases

### Pending
- Pod accepted by cluster
- Containers not yet created
- Waiting for scheduling or image pull

### Running
- Pod bound to node
- At least one container running
- Containers starting or restarting

### Succeeded
- All containers terminated successfully
- Will not be restarted

### Failed
- All containers terminated
- At least one failed with non-zero exit

### Unknown
- Pod state cannot be determined
- Communication error with node

## 📊 Container States

### Waiting
```yaml
state:
  waiting:
    reason: "ImagePullBackOff"
    message: "Back-off pulling image"
```

### Running
```yaml
state:
  running:
    startedAt: "2023-01-01T10:00:00Z"
```

### Terminated
```yaml
state:
  terminated:
    exitCode: 0
    finishedAt: "2023-01-01T11:00:00Z"
    reason: "Completed"
```

## 🔧 Pod Conditions

```bash
# Check pod conditions
kubectl describe pod nginx

# Common conditions:
# - PodScheduled: Pod has been scheduled to node
# - Initialized: Init containers completed
# - ContainersReady: All containers ready
# - Ready: Pod ready to serve requests
```

## 🚀 Init Containers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init-service
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do sleep 2; done']
  containers:
  - name: main
    image: nginx
```

## 🔍 Restart Policies

### Always (Default)
```yaml
spec:
  restartPolicy: Always
  # Restarts containers on failure
```

### OnFailure
```yaml
spec:
  restartPolicy: OnFailure
  # Restarts only on failure (exit code != 0)
```

### Never
```yaml
spec:
  restartPolicy: Never
  # Never restarts containers
```

## 📋 Pod Lifecycle Hooks

### PostStart Hook
```yaml
spec:
  containers:
  - name: app
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo 'Container started' > /tmp/started"]
```

### PreStop Hook
```yaml
spec:
  containers:
  - name: app
    image: nginx
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "nginx -s quit"]
```

## 🏥 Health Checks

### Liveness Probe
```yaml
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
```

### Readiness Probe
```yaml
spec:
  containers:
  - name: app
    image: nginx
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Startup Probe
```yaml
spec:
  containers:
  - name: app
    image: nginx
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

## 📈 Monitoring Lifecycle

```bash
# Watch pod status changes
kubectl get pods -w

# Check pod events
kubectl describe pod nginx | grep Events -A 10

# Monitor pod phases
kubectl get pods -o custom-columns=NAME:.metadata.name,PHASE:.status.phase

# Check container restart count
kubectl get pods -o custom-columns=NAME:.metadata.name,RESTARTS:.status.containerStatuses[0].restartCount
```