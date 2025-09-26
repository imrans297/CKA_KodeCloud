# Resource Quotas

## 🎯 Resource Quota Concepts

### What are Resource Quotas?
- Limits resource consumption per namespace
- Prevents resource exhaustion
- Enforces fair resource sharing
- Controls cluster resource allocation

### Types of Quotas
- **Compute Resources** - CPU, memory, storage
- **Object Count** - pods, services, secrets, configmaps
- **Extended Resources** - GPUs, custom resources

## 📊 Resource Quota Types

### Compute Resource Quotas
```yaml
spec:
  hard:
    requests.cpu: "4"        # Total CPU requests
    requests.memory: 8Gi     # Total memory requests
    limits.cpu: "8"          # Total CPU limits
    limits.memory: 16Gi      # Total memory limits
    requests.storage: 100Gi  # Total storage requests
```

### Object Count Quotas
```yaml
spec:
  hard:
    pods: "10"                    # Max number of pods
    services: "5"                 # Max number of services
    secrets: "10"                 # Max number of secrets
    configmaps: "10"              # Max number of configmaps
    persistentvolumeclaims: "4"   # Max number of PVCs
    services.loadbalancers: "2"   # Max LoadBalancer services
    services.nodeports: "5"       # Max NodePort services
```

## 🔧 Creating Resource Quotas

### Basic Resource Quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
```

### Imperative Creation
```bash
# Create namespace with quota
kubectl create namespace dev-team
kubectl create quota dev-quota --hard=cpu=4,memory=8Gi,pods=10 -n dev-team

# Create quota from command line
kubectl create quota storage-quota --hard=requests.storage=100Gi,persistentvolumeclaims=10 -n dev-team
```

## 📋 Resource Quota Management

### Check Quota Usage
```bash
# List resource quotas
kubectl get resourcequota -n development
kubectl get quota -n development

# Detailed quota information
kubectl describe quota compute-quota -n development

# Check quota across all namespaces
kubectl get quota --all-namespaces
```

### Monitor Quota Usage
```bash
# Watch quota usage
kubectl get quota compute-quota -n development -w

# Check resource usage vs limits
kubectl top pods -n development
kubectl top nodes
```

## 🔍 Quota Scopes

### Quality of Service Scopes
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: qos-quota
spec:
  scopes: ["BestEffort"]  # Only applies to BestEffort pods
  hard:
    pods: "5"
```

### Priority Class Scopes
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: priority-quota
spec:
  scopeSelector:
    matchExpressions:
    - operator: In
      scopeName: PriorityClass
      values: ["high-priority"]
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
```

## 🚨 Quota Enforcement

### Admission Control
- ResourceQuota admission controller enforces limits
- Requests rejected if quota would be exceeded
- Must specify resource requests when quota exists

### Common Errors
```bash
# Error when quota exceeded
Error from server (Forbidden): pods "nginx" is forbidden: 
exceeded quota: compute-quota, requested: requests.cpu=500m, 
used: requests.cpu=3500m, limited: requests.cpu=4
```

## 📊 Quota Monitoring

### Check Quota Status
```bash
# Current usage vs limits
kubectl describe quota compute-quota -n development

# Resource usage by pods
kubectl top pods -n development --sort-by=cpu
kubectl top pods -n development --sort-by=memory
```

### Quota Metrics
```bash
# Get quota usage in JSON
kubectl get quota compute-quota -n development -o json

# Extract specific usage
kubectl get quota compute-quota -n development -o jsonpath='{.status.used}'
```

## 🔧 Advanced Quota Configurations

### Multiple Quotas per Namespace
```yaml
# Separate quotas for different resource types
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-quota
spec:
  hard:
    pods: "10"
    services: "5"
    secrets: "10"
```

### Quota with LimitRange
```yaml
# ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi

---
# LimitRange (enforces individual pod limits)
apiVersion: v1
kind: LimitRange
metadata:
  name: namespace-limits
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

## 🚨 Troubleshooting Quotas

### Common Issues
```bash
# Pod creation fails due to quota
kubectl describe pod <pod-name>

# Check current quota usage
kubectl describe quota -n <namespace>

# Check if pod has resource requests
kubectl get pod <pod-name> -o yaml | grep -A 10 resources
```

### Quota Debugging
```bash
# Check admission controller logs
kubectl logs -n kube-system kube-apiserver-master | grep quota

# Verify quota configuration
kubectl get quota <quota-name> -o yaml

# Check resource requests in deployment
kubectl describe deployment <deployment-name>
```

## 📋 Best Practices

### Planning Quotas
- Start with generous limits and adjust
- Monitor actual usage patterns
- Consider peak usage scenarios
- Plan for growth and scaling

### Implementation
- Use both ResourceQuota and LimitRange
- Set quotas per team/environment
- Monitor quota usage regularly
- Implement alerting for quota violations

### Maintenance
- Review and adjust quotas periodically
- Clean up unused resources
- Educate teams on resource management
- Implement resource governance policies