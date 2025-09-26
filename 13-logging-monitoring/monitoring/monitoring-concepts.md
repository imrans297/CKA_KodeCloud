# Kubernetes Monitoring

## 🎯 Monitoring Concepts

### What is Kubernetes Monitoring?
- **Resource Monitoring** - CPU, memory, disk, network usage
- **Application Monitoring** - Application metrics and health
- **Cluster Monitoring** - Kubernetes components and API server
- **Infrastructure Monitoring** - Node and container health

### Monitoring Layers
- **Infrastructure Layer** - Nodes, containers, network
- **Platform Layer** - Kubernetes components and API
- **Application Layer** - Custom application metrics
- **User Experience Layer** - End-user performance

## 📊 Metrics Types

### Resource Metrics
```bash
# Node resource usage
kubectl top nodes

# Pod resource usage
kubectl top pods
kubectl top pods --all-namespaces

# Container resource usage
kubectl top pods --containers
```

### Custom Metrics
```yaml
# Application metrics endpoint
apiVersion: v1
kind: Service
metadata:
  name: app-metrics
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
spec:
  selector:
    app: myapp
  ports:
  - port: 8080
    name: metrics
```

### Cluster Metrics
```bash
# API server metrics
kubectl get --raw /metrics

# Component status
kubectl get componentstatuses

# Cluster events
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 🔧 Monitoring Tools

### Metrics Server
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1
        args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

### Prometheus
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
        - name: storage
          mountPath: /prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: storage
        persistentVolumeClaim:
          claimName: prometheus-pvc
```

## 📈 Monitoring Stack

### Prometheus + Grafana
```bash
# Prometheus for metrics collection
# Grafana for visualization
# AlertManager for alerting
# Node Exporter for node metrics
# kube-state-metrics for Kubernetes metrics
```

### Monitoring Architecture
```
Applications → Prometheus → Grafana
     ↓              ↓
Node Exporter → AlertManager → Notifications
     ↓
kube-state-metrics
```

## 🚨 Alerting

### Prometheus Alerts
```yaml
groups:
- name: kubernetes-alerts
  rules:
  - alert: PodCrashLooping
    expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Pod {{ $labels.pod }} is crash looping"
      
  - alert: NodeNotReady
    expr: kube_node_status_condition{condition="Ready",status="true"} == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Node {{ $labels.node }} is not ready"
```

### AlertManager Configuration
```yaml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@company.com'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
- name: 'web.hook'
  email_configs:
  - to: 'admin@company.com'
    subject: 'Kubernetes Alert: {{ .GroupLabels.alertname }}'
    body: |
      {{ range .Alerts }}
      Alert: {{ .Annotations.summary }}
      Description: {{ .Annotations.description }}
      {{ end }}
```

## 📊 Health Checks

### Liveness Probes
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

### Readiness Probes
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  successThreshold: 1
  failureThreshold: 3
```

### Startup Probes
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 30
```

## 🔍 Observability

### Three Pillars
- **Metrics** - Numerical data over time
- **Logs** - Event records
- **Traces** - Request flow tracking

### Distributed Tracing
```yaml
# Jaeger for distributed tracing
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:latest
        ports:
        - containerPort: 16686
        - containerPort: 14268
```

## 📋 Best Practices

### Monitoring Strategy
- Monitor what matters to users
- Set up proper alerting thresholds
- Use both proactive and reactive monitoring
- Implement SLIs and SLOs
- Regular monitoring review

### Performance
- Use efficient metric collection
- Implement proper retention policies
- Optimize query performance
- Use appropriate sampling rates
- Monitor monitoring overhead

### Security
- Secure monitoring endpoints
- Implement proper authentication
- Encrypt metrics transmission
- Audit monitoring access
- Protect sensitive metrics