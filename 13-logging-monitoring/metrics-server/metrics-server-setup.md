# Metrics Server Setup

## 🎯 Metrics Server Overview

### What is Metrics Server?
- **Resource Metrics API** - Provides CPU and memory usage metrics
- **Horizontal Pod Autoscaler** - Enables HPA functionality
- **kubectl top** - Powers resource usage commands
- **Lightweight** - Minimal resource overhead

### Architecture
```
kubelet → Metrics Server → API Server → kubectl top / HPA
```

## 🚀 Installation

### Method 1: Official Manifest
```bash
# Download and apply metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Method 2: Custom Installation
```bash
# Create metrics-server namespace
kubectl create namespace metrics-server

# Apply custom configuration
kubectl apply -f metrics-server-deployment.yaml
```

### Method 3: Helm Installation
```bash
# Add metrics-server helm repo
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

# Install metrics-server
helm upgrade --install metrics-server metrics-server/metrics-server \
  --namespace metrics-server \
  --create-namespace
```

## 🔧 Configuration

### Basic Configuration
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
        image: k8s.gcr.io/metrics-server/metrics-server:v0.6.2
        args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

### Development/Testing Configuration
```yaml
# For development clusters with self-signed certificates
args:
- --cert-dir=/tmp
- --secure-port=4443
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
- --kubelet-use-node-status-port
- --metric-resolution=15s
- --kubelet-insecure-tls  # For development only
```

## 📊 Usage

### kubectl top Commands
```bash
# Node resource usage
kubectl top nodes
kubectl top nodes --sort-by=cpu
kubectl top nodes --sort-by=memory

# Pod resource usage
kubectl top pods
kubectl top pods --all-namespaces
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# Container resource usage
kubectl top pods --containers
kubectl top pods --containers --all-namespaces
```

### Resource Metrics API
```bash
# Raw metrics API
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods

# Specific node metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes/<node-name>

# Specific pod metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/<namespace>/pods/<pod-name>
```

## 🔄 Horizontal Pod Autoscaler

### HPA Configuration
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### HPA Commands
```bash
# Create HPA
kubectl autoscale deployment webapp --cpu-percent=70 --min=2 --max=10

# List HPAs
kubectl get hpa
kubectl get hpa --all-namespaces

# Describe HPA
kubectl describe hpa webapp-hpa

# Check HPA status
kubectl get hpa webapp-hpa -w
```

## 🚨 Troubleshooting

### Common Issues

#### Metrics Server Not Ready
```bash
# Check metrics server pods
kubectl get pods -n kube-system | grep metrics-server

# Check metrics server logs
kubectl logs -n kube-system deployment/metrics-server

# Check metrics server service
kubectl get svc -n kube-system metrics-server
```

#### kubectl top Not Working
```bash
# Check if metrics server is running
kubectl get deployment metrics-server -n kube-system

# Check API availability
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml

# Test metrics API directly
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
```

#### HPA Not Scaling
```bash
# Check HPA status
kubectl describe hpa <hpa-name>

# Check pod resource requests
kubectl describe pod <pod-name> | grep -A 5 "Requests"

# Check metrics availability
kubectl top pods -n <namespace>
```

### Debug Commands
```bash
# Check kubelet metrics endpoint
curl -k https://<node-ip>:10250/metrics

# Check metrics server configuration
kubectl get deployment metrics-server -n kube-system -o yaml

# Check certificate issues
kubectl logs -n kube-system deployment/metrics-server | grep -i cert
```

## 📋 Best Practices

### Resource Configuration
- Set appropriate resource requests and limits
- Configure proper metric resolution
- Use secure TLS connections in production
- Monitor metrics server resource usage

### Security
- Use proper RBAC permissions
- Enable TLS verification in production
- Secure kubelet API access
- Regular security updates

### Performance
- Adjust metric resolution based on needs
- Monitor metrics server overhead
- Use efficient storage for metrics
- Implement proper retention policies

## 🔧 Advanced Configuration

### Custom Metrics
```yaml
# Custom metrics adapter for external metrics
apiVersion: v1
kind: ConfigMap
metadata:
  name: adapter-config
data:
  config.yaml: |
    rules:
    - seriesQuery: 'http_requests_per_second{namespace!="",pod!=""}'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          pod: {resource: "pod"}
      name:
        matches: "^(.*)_per_second"
        as: "${1}_rate"
      metricsQuery: 'sum(<<.Series>>{<<.LabelMatchers>>}) by (<<.GroupBy>>)'
```

### Monitoring Integration
```yaml
# ServiceMonitor for Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  endpoints:
  - port: https
    scheme: https
    tlsConfig:
      insecureSkipVerify: true
```