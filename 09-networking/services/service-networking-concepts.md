# Service Networking

## 🎯 Service Networking Concepts

### Kubernetes Networking Model
- Every pod gets unique IP address
- Pods can communicate without NAT
- Services provide stable endpoints
- kube-proxy implements service abstraction

### Service Types Review
- **ClusterIP**: Internal cluster communication
- **NodePort**: External access via node ports
- **LoadBalancer**: Cloud provider load balancer
- **ExternalName**: DNS CNAME records

## 🔧 Service Discovery

### DNS-based Discovery
```bash
# Service FQDN format
<service-name>.<namespace>.svc.cluster.local

# Examples
web-service.default.svc.cluster.local
api-service.production.svc.cluster.local
```

### Environment Variables
```bash
# Kubernetes creates env vars for services
<SERVICE_NAME>_SERVICE_HOST
<SERVICE_NAME>_SERVICE_PORT
```

## 📊 Service Endpoints

### Endpoint Objects
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
- addresses:
  - ip: 10.244.1.5
  - ip: 10.244.2.3
  ports:
  - port: 8080
```

### EndpointSlices (v1.19+)
```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-abc123
  labels:
    kubernetes.io/service-name: my-service
addressType: IPv4
endpoints:
- addresses:
  - "10.244.1.5"
  conditions:
    ready: true
ports:
- name: http
  port: 8080
  protocol: TCP
```

## 🌐 External Services

### ExternalName Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  type: ExternalName
  externalName: api.external.com
```

### Service without Selector
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
- addresses:
  - ip: 203.0.113.1
  ports:
  - port: 80
```

## 🔧 Service Mesh Integration

### Headless Services
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: database
  ports:
  - port: 5432
```

### Session Affinity
```yaml
apiVersion: v1
kind: Service
metadata:
  name: sticky-service
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  selector:
    app: web
  ports:
  - port: 80
```

## 📋 Service Annotations

### Cloud Provider Annotations
```yaml
# AWS Load Balancer
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"

# GCP Load Balancer
metadata:
  annotations:
    cloud.google.com/load-balancer-type: "External"

# Azure Load Balancer
metadata:
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "false"
```

## 🚀 Service Operations

### Service Management
```bash
# Create service
kubectl expose deployment nginx --port=80 --type=ClusterIP

# List services
kubectl get services
kubectl get svc -o wide

# Describe service
kubectl describe service nginx

# Check endpoints
kubectl get endpoints nginx
```

### Service Testing
```bash
# Test service from pod
kubectl exec -it <pod> -- curl <service-name>:<port>

# Test service DNS
kubectl exec -it <pod> -- nslookup <service-name>

# Port forward for testing
kubectl port-forward service/nginx 8080:80
```

## 🔍 Service Networking Debugging

### Common Issues
```bash
# Service not accessible
kubectl get svc <service-name>
kubectl describe svc <service-name>

# No endpoints
kubectl get endpoints <service-name>
kubectl get pods -l <selector>

# DNS resolution issues
kubectl exec -it <pod> -- nslookup <service-name>
```

### Network Troubleshooting
```bash
# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy

# Check iptables rules
sudo iptables -t nat -L | grep <service-name>

# Test connectivity
kubectl run debug --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>
```

## 📊 Service Performance

### Load Balancing
- Round-robin by default
- Session affinity options
- External load balancer integration
- Health check configuration

### Monitoring
```bash
# Service metrics
kubectl top pods -l app=<app-name>

# Connection tracking
netstat -an | grep <port>

# Service latency
curl -w "@curl-format.txt" -o /dev/null -s <service-url>
```

## 🔐 Service Security

### Network Policies
- Control service access
- Namespace isolation
- Pod-to-pod communication
- External traffic filtering

### TLS Termination
```yaml
apiVersion: v1
kind: Service
metadata:
  name: https-service
spec:
  ports:
  - port: 443
    targetPort: 8443
    name: https
  - port: 80
    targetPort: 8080
    name: http
  selector:
    app: web
```

## 📋 Best Practices

### Service Design
- Use meaningful service names
- Implement health checks
- Configure appropriate timeouts
- Plan for high availability

### Performance Optimization
- Use headless services for direct pod access
- Configure session affinity when needed
- Monitor service latency
- Optimize endpoint updates

### Security
- Implement network policies
- Use TLS for sensitive services
- Limit service exposure
- Regular security audits