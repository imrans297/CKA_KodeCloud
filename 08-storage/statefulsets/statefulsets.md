# Kubernetes StatefulSets - Complete Guide

## 🎯 Stateless vs Stateful Applications

### Stateless Applications
**Definition**: Applications that don't store any data or state between requests. Each request is independent and contains all necessary information.

**Characteristics**:
- No persistent data storage
- Any instance can handle any request
- Easy to scale horizontally
- Instances are interchangeable
- No dependency on previous requests

**Examples**:
```yaml
# Stateless Web Server
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-stateless
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

**Real-World Stateless Examples**:
- **Web Servers** (Nginx, Apache) - Serve static content
- **API Gateways** - Route requests without storing state
- **Load Balancers** - Distribute traffic
- **Microservices** - Process requests independently
- **CDN Edge Servers** - Cache and serve content

### Stateful Applications
**Definition**: Applications that maintain state/data between requests and require persistent storage and stable network identities.

**Characteristics**:
- Persistent data storage required
- Specific instance identity matters
- Complex scaling requirements
- Order of deployment/termination matters
- Network identity must be stable

**Examples**:
```yaml
# Stateful Database
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-stateful
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: mydb
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

**Real-World Stateful Examples**:
- **Databases** (PostgreSQL, MySQL, MongoDB) - Store persistent data
- **Message Queues** (Kafka, RabbitMQ) - Maintain message state
- **Distributed Storage** (Cassandra, Elasticsearch) - Data persistence
- **Caching Systems** (Redis) - In-memory state
- **Monitoring Systems** (Prometheus) - Time-series data

## 🏗️ What is StatefulSet?

**StatefulSet** is a Kubernetes workload API object used to manage stateful applications. It provides guarantees about the ordering and uniqueness of pods.

### Key Features
1. **Stable Network Identity** - Each pod gets a persistent hostname
2. **Stable Storage** - Each pod gets its own persistent volume
3. **Ordered Deployment** - Pods are created/updated in order
4. **Ordered Termination** - Pods are deleted in reverse order
5. **Ordered Scaling** - Scaling happens one pod at a time

### StatefulSet vs Deployment

| Feature | Deployment | StatefulSet |
|---------|------------|-------------|
| Pod Identity | Random names | Predictable names (pod-0, pod-1) |
| Network Identity | Service discovery | Stable hostnames |
| Storage | Shared volumes | Individual persistent volumes |
| Scaling | Parallel | Sequential (ordered) |
| Updates | Rolling updates | Ordered updates |
| Use Case | Stateless apps | Stateful apps |

## 🔧 StatefulSet Components

### Basic StatefulSet Structure
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-statefulset
spec:
  serviceName: "web-service"        # Headless service name
  replicas: 3                       # Number of pods
  selector:
    matchLabels:
      app: web
  template:                         # Pod template
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.20
        ports:
        - containerPort: 80
        volumeMounts:
        - name: web-storage
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:            # PVC template for each pod
  - metadata:
      name: web-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### Headless Service (Required)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  clusterIP: None                   # Headless service
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

## 🎯 StatefulSet Naming and Identity

### Pod Naming Convention
```bash
# StatefulSet pods are named predictably
<statefulset-name>-<ordinal-index>

# Example: web-statefulset with 3 replicas
web-statefulset-0
web-statefulset-1  
web-statefulset-2
```

### Network Identity
```bash
# Each pod gets a stable DNS name
<pod-name>.<service-name>.<namespace>.svc.cluster.local

# Examples:
web-statefulset-0.web-service.default.svc.cluster.local
web-statefulset-1.web-service.default.svc.cluster.local
web-statefulset-2.web-service.default.svc.cluster.local
```

### Storage Identity
```bash
# Each pod gets its own PVC
<pvc-template-name>-<pod-name>

# Examples:
web-storage-web-statefulset-0
web-storage-web-statefulset-1
web-storage-web-statefulset-2
```

## 📊 StatefulSet Lifecycle Management

### Deployment Order
```
1. web-statefulset-0 (created first, must be Running and Ready)
2. web-statefulset-1 (created after 0 is ready)
3. web-statefulset-2 (created after 1 is ready)
```

### Scaling Up
```bash
# Scale from 3 to 5 replicas
kubectl scale statefulset web-statefulset --replicas=5

# Order of creation:
# web-statefulset-3 (created first)
# web-statefulset-4 (created after 3 is ready)
```

### Scaling Down
```bash
# Scale from 5 to 3 replicas
kubectl scale statefulset web-statefulset --replicas=3

# Order of termination (reverse order):
# web-statefulset-4 (terminated first)
# web-statefulset-3 (terminated after 4 is gone)
```

### Update Strategy
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0    # Update pods with ordinal >= partition
```

## 🗄️ Real-World StatefulSet Examples

### 1. PostgreSQL Database Cluster
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-cluster
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: myapp
        - name: POSTGRES_USER
          value: postgres
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        - name: postgres-config
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgresql.conf
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: postgres-config
        configMap:
          name: postgres-config
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 20Gi
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-read
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

### 2. MongoDB Replica Set
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb-replica
spec:
  serviceName: mongodb
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:4.4
        command:
        - mongod
        - --replSet
        - rs0
        - --bind_ip_all
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: admin
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
        volumeMounts:
        - name: mongodb-storage
          mountPath: /data/db
        livenessProbe:
          exec:
            command:
            - mongo
            - --eval
            - "db.adminCommand('ping')"
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - mongo
            - --eval
            - "db.adminCommand('ping')"
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: mongodb-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
```

### 3. Elasticsearch Cluster
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      initContainers:
      - name: increase-vm-max-map
        image: busybox
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
        securityContext:
          privileged: true
      containers:
      - name: elasticsearch
        image: elasticsearch:7.14.0
        env:
        - name: cluster.name
          value: "elasticsearch-cluster"
        - name: node.name
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: discovery.seed_hosts
          value: "elasticsearch-0.elasticsearch,elasticsearch-1.elasticsearch,elasticsearch-2.elasticsearch"
        - name: cluster.initial_master_nodes
          value: "elasticsearch-0,elasticsearch-1,elasticsearch-2"
        - name: ES_JAVA_OPTS
          value: "-Xms512m -Xmx512m"
        ports:
        - containerPort: 9200
        - containerPort: 9300
        volumeMounts:
        - name: elasticsearch-storage
          mountPath: /usr/share/elasticsearch/data
        resources:
          requests:
            memory: 1Gi
            cpu: 500m
          limits:
            memory: 2Gi
            cpu: 1000m
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 5Gi
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
spec:
  clusterIP: None
  selector:
    app: elasticsearch
  ports:
  - port: 9200
    name: http
  - port: 9300
    name: transport
```

### 4. Redis Cluster
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis
  replicas: 6
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.2
        command:
        - redis-server
        - /etc/redis/redis.conf
        - --cluster-enabled
        - "yes"
        - --cluster-config-file
        - /data/nodes.conf
        - --cluster-node-timeout
        - "5000"
        - --appendonly
        - "yes"
        ports:
        - containerPort: 6379
        - containerPort: 16379
        volumeMounts:
        - name: redis-storage
          mountPath: /data
        - name: redis-config
          mountPath: /etc/redis
        resources:
          requests:
            memory: 256Mi
            cpu: 100m
          limits:
            memory: 512Mi
            cpu: 200m
      volumes:
      - name: redis-config
        configMap:
          name: redis-config
  volumeClaimTemplates:
  - metadata:
      name: redis-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
  - port: 16379
    targetPort: 16379
```

## 🛠️ StatefulSet Management Commands

### Basic Operations
```bash
# Create StatefulSet
kubectl apply -f statefulset.yaml

# List StatefulSets
kubectl get statefulsets
kubectl get sts  # Short form

# Describe StatefulSet
kubectl describe statefulset web-statefulset

# Get StatefulSet details
kubectl get statefulset web-statefulset -o yaml
```

### Scaling Operations
```bash
# Scale up
kubectl scale statefulset web-statefulset --replicas=5

# Scale down
kubectl scale statefulset web-statefulset --replicas=2

# Check scaling status
kubectl rollout status statefulset web-statefulset
```

### Update Operations
```bash
# Update image
kubectl set image statefulset/web-statefulset web=nginx:1.21

# Patch StatefulSet
kubectl patch statefulset web-statefulset -p '{"spec":{"template":{"spec":{"containers":[{"name":"web","image":"nginx:1.21"}]}}}}'

# Check rollout status
kubectl rollout status statefulset web-statefulset

# Rollback to previous version
kubectl rollout undo statefulset web-statefulset
```

### Pod Management
```bash
# List pods in order
kubectl get pods -l app=web --sort-by=.metadata.name

# Delete specific pod (will be recreated)
kubectl delete pod web-statefulset-1

# Force delete stuck pod
kubectl delete pod web-statefulset-1 --force --grace-period=0
```

## 🔧 Advanced StatefulSet Features

### Partition Updates
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2  # Only update pods with ordinal >= 2
```

```bash
# Update only pods 2 and above
kubectl patch statefulset web-statefulset -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":2}}}}'

# Gradually roll out updates
kubectl patch statefulset web-statefulset -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":1}}}}'
kubectl patch statefulset web-statefulset -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":0}}}}'
```

### Pod Management Policy
```yaml
spec:
  podManagementPolicy: Parallel  # Default: OrderedReady
```

### Persistent Volume Claim Retention
```yaml
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain    # Retain, Delete
    whenScaled: Delete     # Retain, Delete
```

## 📊 StatefulSet Monitoring

### Check StatefulSet Status
```bash
# Get StatefulSet status
kubectl get statefulset web-statefulset -o wide

# Check ready replicas
kubectl get statefulset web-statefulset -o jsonpath='{.status.readyReplicas}'

# Check current replicas
kubectl get statefulset web-statefulset -o jsonpath='{.status.replicas}'
```

### Monitor Pod Status
```bash
# Watch pod creation/deletion
kubectl get pods -l app=web -w

# Check pod readiness
kubectl get pods -l app=web -o wide

# Check events
kubectl get events --field-selector involvedObject.kind=StatefulSet
```

### Volume Status
```bash
# List PVCs for StatefulSet
kubectl get pvc -l app=web

# Check PVC status
kubectl describe pvc web-storage-web-statefulset-0

# Check PV binding
kubectl get pv
```

## 🚨 StatefulSet Troubleshooting

### Common Issues

#### 1. Pod Stuck in Pending
```bash
# Check PVC status
kubectl get pvc
kubectl describe pvc web-storage-web-statefulset-0

# Check storage class
kubectl get storageclass
kubectl describe storageclass fast-ssd

# Check node resources
kubectl describe node <node-name>
```

#### 2. Pod Not Ready
```bash
# Check pod logs
kubectl logs web-statefulset-0

# Check pod events
kubectl describe pod web-statefulset-0

# Check readiness probe
kubectl get pod web-statefulset-0 -o yaml | grep -A 10 readinessProbe
```

#### 3. Scaling Issues
```bash
# Check if previous pod is ready
kubectl get pods -l app=web

# Check StatefulSet events
kubectl describe statefulset web-statefulset

# Force delete stuck pod
kubectl delete pod web-statefulset-1 --force --grace-period=0
```

#### 4. Update Stuck
```bash
# Check rollout status
kubectl rollout status statefulset web-statefulset

# Check partition setting
kubectl get statefulset web-statefulset -o yaml | grep partition

# Check pod disruption budget
kubectl get pdb
```

### Debug Commands
```bash
# Get detailed StatefulSet info
kubectl get statefulset web-statefulset -o yaml

# Check controller logs
kubectl logs -n kube-system -l component=kube-controller-manager

# Check storage provisioner logs
kubectl logs -n kube-system -l app=storage-provisioner
```

## 📋 StatefulSet Best Practices

### Design Guidelines
1. **Use headless services** for stable network identity
2. **Implement proper health checks** (liveness/readiness probes)
3. **Set resource limits** to prevent resource starvation
4. **Use init containers** for setup tasks
5. **Plan for data persistence** and backup strategies

### Storage Best Practices
```yaml
# Use appropriate storage class
volumeClaimTemplates:
- metadata:
    name: data-storage
  spec:
    accessModes: ["ReadWriteOnce"]
    storageClassName: fast-ssd  # Use SSD for databases
    resources:
      requests:
        storage: 20Gi
```

### Security Best Practices
```yaml
# Use security contexts
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
      - name: app
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
```

### Operational Guidelines
- **Monitor resource usage** regularly
- **Plan capacity** for scaling
- **Test disaster recovery** procedures
- **Implement proper backup** strategies
- **Use rolling updates** carefully with partition strategy

This comprehensive StatefulSet guide covers everything from basic concepts to advanced operational patterns, providing complete understanding for managing stateful applications in Kubernetes.