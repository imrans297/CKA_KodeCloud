# Multi-Container Pods

## 🎯 Multi-Container Concepts

### What are Multi-Container Pods?
- **Multiple containers** in a single pod
- **Shared resources** - network, storage, lifecycle
- **Tight coupling** - containers work together
- **Common patterns** - sidecar, ambassador, adapter

### Why Use Multi-Container Pods?
- Separation of concerns
- Modular architecture
- Shared storage and network
- Coordinated lifecycle management

## 📊 Container Patterns

### Sidecar Pattern
**Purpose**: Helper container alongside main container
**Use Cases**: Logging, monitoring, proxying, data synchronization

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  containers:
  - name: main-app
    image: nginx
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: log-shipper
    image: fluent/fluentd
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  volumes:
  - name: shared-logs
    emptyDir: {}
```

### Ambassador Pattern / Co-located Containers
**Purpose**: Proxy container for external services
**Use Cases**: Service discovery, load balancing, circuit breaking

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-example
spec:
  containers:
  - name: main-app
    image: myapp
    env:
    - name: DATABASE_HOST
      value: "localhost"
    - name: DATABASE_PORT
      value: "5432"
  - name: database-proxy
    image: haproxy
    ports:
    - containerPort: 5432
```

### Adapter Pattern
**Purpose**: Transform container output format
**Use Cases**: Data format conversion, protocol translation

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-example
spec:
  containers:
  - name: legacy-app
    image: legacy-monitoring-app
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: metrics-adapter
    image: prometheus-adapter
    volumeMounts:
    - name: shared-data
      mountPath: /input
  volumes:
  - name: shared-data
    emptyDir: {}
```

## 🔧 Container Communication

### Shared Network
- All containers share the same IP address
- Communicate via localhost
- Port conflicts must be avoided
- Network policies apply to the entire pod

```yaml
spec:
  containers:
  - name: web-server
    image: nginx
    ports:
    - containerPort: 80
  - name: metrics-exporter
    image: nginx-prometheus-exporter
    ports:
    - containerPort: 9113
    env:
    - name: NGINX_STATUS_URI
      value: "http://localhost/nginx_status"
```

### Shared Storage
- Volumes can be mounted by multiple containers
- Data sharing between containers
- Persistent storage across container restarts

```yaml
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'while true; do echo "$(date)" >> /shared/log.txt; sleep 10; done']
    volumeMounts:
    - name: shared-volume
      mountPath: /shared
  - name: reader
    image: busybox
    command: ['sh', '-c', 'tail -f /shared/log.txt']
    volumeMounts:
    - name: shared-volume
      mountPath: /shared
  volumes:
  - name: shared-volume
    emptyDir: {}
```

## 🚀 Common Use Cases

### Web Server with Log Processor
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logging
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nginx-logs
      mountPath: /var/log/nginx
  - name: log-processor
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        if [ -f /logs/access.log ]; then
          tail -f /logs/access.log | while read line; do
            echo "Processed: $line"
          done
        fi
        sleep 5
      done
    volumeMounts:
    - name: nginx-logs
      mountPath: /logs
  volumes:
  - name: nginx-logs
    emptyDir: {}
```

### Application with Monitoring
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-monitoring
spec:
  containers:
  - name: application
    image: myapp:latest
    ports:
    - containerPort: 8080
  - name: monitoring-agent
    image: prometheus/node-exporter
    ports:
    - containerPort: 9100
  - name: log-collector
    image: fluentd
    env:
    - name: FLUENTD_CONF
      value: "fluent.conf"
```

### Database with Backup Sidecar
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database-with-backup
spec:
  containers:
  - name: postgres
    image: postgres:13
    env:
    - name: POSTGRES_PASSWORD
      value: "password"
    volumeMounts:
    - name: postgres-data
      mountPath: /var/lib/postgresql/data
  - name: backup-agent
    image: postgres:13
    command: ['sh', '-c']
    args:
    - |
      while true; do
        sleep 3600  # Backup every hour
        pg_dump -h localhost -U postgres mydb > /backups/backup-$(date +%Y%m%d-%H%M%S).sql
      done
    volumeMounts:
    - name: backup-storage
      mountPath: /backups
  volumes:
  - name: postgres-data
    persistentVolumeClaim:
      claimName: postgres-pvc
  - name: backup-storage
    persistentVolumeClaim:
      claimName: backup-pvc
```

## 🔍 Container Lifecycle

### Startup Order
- All containers start simultaneously
- No guaranteed startup order
- Use init containers for sequential startup
- Implement readiness checks

### Health Checks
```yaml
spec:
  containers:
  - name: main-app
    image: myapp
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
  - name: sidecar
    image: sidecar-app
    readinessProbe:
      exec:
        command: ['sh', '-c', 'test -f /tmp/ready']
```

### Resource Management
```yaml
spec:
  containers:
  - name: main-app
    image: myapp
    resources:
      requests:
        cpu: 500m
        memory: 1Gi
      limits:
        cpu: 1000m
        memory: 2Gi
  - name: sidecar
    image: sidecar
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 200m
        memory: 256Mi
```

## 🚨 Troubleshooting Multi-Container Pods

### Debug Commands
```bash
# Check all containers in pod
kubectl describe pod <pod-name>

# Get logs from specific container
kubectl logs <pod-name> -c <container-name>

# Execute into specific container
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash

# Check container status
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].name}'
```

### Common Issues
```bash
# Container not starting
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <container-name>

# Port conflicts
kubectl describe pod <pod-name> | grep -A 5 "Port"

# Resource constraints
kubectl describe pod <pod-name> | grep -A 10 "Requests\|Limits"

# Volume mount issues
kubectl describe pod <pod-name> | grep -A 10 "Mounts\|Volumes"
```

## 📋 Best Practices

### Design Guidelines
- Keep containers focused on single responsibility
- Use shared volumes for data exchange
- Implement proper health checks
- Plan resource allocation carefully
- Use meaningful container names

### Communication Patterns
- Use localhost for inter-container communication
- Implement proper error handling
- Use shared volumes for file-based communication
- Consider container startup dependencies

### Resource Management
- Set appropriate resource requests and limits
- Monitor resource usage per container
- Consider container resource ratios
- Plan for peak load scenarios

### Security Considerations
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    fsGroup: 2000
  containers:
  - name: main-app
    securityContext:
      runAsUser: 1000
      readOnlyRootFilesystem: true
  - name: sidecar
    securityContext:
      runAsUser: 1001
      allowPrivilegeEscalation: false
```