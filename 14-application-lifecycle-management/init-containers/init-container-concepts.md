# Init Containers

## 🎯 Init Container Concepts

### What are Init Containers?
- **Specialized containers** that run before app containers
- **Setup tasks** - Database initialization, file downloads
- **Sequential execution** - Run one after another
- **Must complete successfully** before app containers start

### Use Cases
- Wait for services to be available
- Download configuration files
- Database schema setup
- Security scanning
- Pre-populate shared volumes

## 🔧 Init Container Characteristics

### Execution Model
- Run to completion before app containers
- Execute in the order specified
- If any init container fails, pod restarts
- Share volumes with app containers
- Have separate resource limits

### Differences from App Containers
- No readiness/liveness probes
- No lifecycle hooks
- Cannot use ports
- Always restart on failure
- Run with same security context

## 📊 Init Container Examples

### Basic Init Container
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init-service
    image: busybox:1.35
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done']
  containers:
  - name: myapp
    image: nginx
```

### Multiple Init Containers
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-init-demo
spec:
  initContainers:
  - name: init-db
    image: busybox:1.35
    command: ['sh', '-c', 'echo "Initializing database..." && sleep 10']
  - name: init-cache
    image: busybox:1.35
    command: ['sh', '-c', 'echo "Setting up cache..." && sleep 5']
  containers:
  - name: app
    image: nginx
```

### Init Container with Volume
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-volume-demo
spec:
  initContainers:
  - name: init-setup
    image: busybox:1.35
    command: ['sh', '-c', 'echo "Setup complete" > /shared/status.txt']
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: shared-data
    emptyDir: {}
```

## 🚀 Common Patterns

### Service Dependency
```yaml
initContainers:
- name: wait-for-db
  image: busybox:1.35
  command: ['sh', '-c']
  args:
  - |
    until nc -z postgres 5432; do
      echo "Waiting for postgres..."
      sleep 1
    done
    echo "Postgres is ready!"
```

### Configuration Download
```yaml
initContainers:
- name: download-config
  image: curlimages/curl:latest
  command: ['sh', '-c']
  args:
  - |
    curl -o /config/app.conf https://config-server/app.conf
    echo "Configuration downloaded"
  volumeMounts:
  - name: config-volume
    mountPath: /config
```

### Database Migration
```yaml
initContainers:
- name: db-migration
  image: migrate/migrate:latest
  command: ['migrate']
  args:
  - -path=/migrations
  - -database=postgres://user:pass@postgres:5432/db?sslmode=disable
  - up
  volumeMounts:
  - name: migrations
    mountPath: /migrations
```

## 🔍 Monitoring Init Containers

### Check Init Container Status
```bash
# View pod with init containers
kubectl describe pod init-demo

# Check init container logs
kubectl logs init-demo -c init-service

# Watch pod initialization
kubectl get pod init-demo -w
```

### Init Container Events
```bash
# Check events for init container issues
kubectl get events --field-selector involvedObject.name=init-demo

# Describe pod for detailed status
kubectl describe pod init-demo | grep -A 10 "Init Containers"
```

## 🚨 Troubleshooting Init Containers

### Common Issues
```bash
# Init container stuck
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <init-container-name>

# Init container failing
kubectl get events --field-selector involvedObject.name=<pod-name>

# Resource constraints
kubectl describe pod <pod-name> | grep -A 5 "Requests\|Limits"
```

### Debug Commands
```bash
# Check init container logs
kubectl logs <pod-name> -c <init-container-name>

# Get previous init container logs
kubectl logs <pod-name> -c <init-container-name> --previous

# Execute into running init container (if possible)
kubectl exec -it <pod-name> -c <init-container-name> -- /bin/sh
```

## 📋 Best Practices

### Design Guidelines
- Keep init containers lightweight
- Use specific, minimal images
- Implement proper error handling
- Set appropriate timeouts
- Use meaningful names

### Resource Management
```yaml
initContainers:
- name: init-setup
  image: busybox:1.35
  resources:
    requests:
      cpu: 100m
      memory: 64Mi
    limits:
      cpu: 200m
      memory: 128Mi
```

### Security Considerations
```yaml
initContainers:
- name: secure-init
  image: busybox:1.35
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    readOnlyRootFilesystem: true
```

## 🔄 Init Containers in Deployments

### Deployment with Init Containers
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-with-init
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
      initContainers:
      - name: wait-for-db
        image: busybox:1.35
        command: ['sh', '-c', 'until nc -z database 5432; do sleep 1; done']
      containers:
      - name: webapp
        image: nginx
        ports:
        - containerPort: 80
```

## 📊 Advanced Use Cases

### Multi-Stage Setup
```yaml
initContainers:
- name: stage1-download
  image: curlimages/curl:latest
  command: ['sh', '-c', 'curl -o /data/config.json https://api.example.com/config']
  volumeMounts:
  - name: shared-data
    mountPath: /data
- name: stage2-process
  image: jq:latest
  command: ['sh', '-c', 'jq ".database" /data/config.json > /data/db-config.json']
  volumeMounts:
  - name: shared-data
    mountPath: /data
- name: stage3-validate
  image: busybox:1.35
  command: ['sh', '-c', 'test -f /data/db-config.json && echo "Validation passed"']
  volumeMounts:
  - name: shared-data
    mountPath: /data
```

### Conditional Init Containers
```yaml
initContainers:
- name: conditional-setup
  image: busybox:1.35
  command: ['sh', '-c']
  args:
  - |
    if [ "$ENVIRONMENT" = "production" ]; then
      echo "Running production setup..."
      # Production-specific initialization
    else
      echo "Running development setup..."
      # Development-specific initialization
    fi
  env:
  - name: ENVIRONMENT
    value: "production"
```