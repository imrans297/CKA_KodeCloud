# Network Policies

## 🎯 Network Policy Concepts

### What are Network Policies?
- Kubernetes firewall rules
- Control traffic flow between pods
- Layer 3/4 traffic filtering
- Default deny or allow behavior

### Requirements
- Network plugin must support NetworkPolicy
- Supported: Calico, Cilium, Weave Net
- Not supported: Flannel (basic mode)

## 🔧 Network Policy Types

### Ingress Rules
- Control incoming traffic to pods
- Specify allowed sources
- Define allowed ports and protocols

### Egress Rules
- Control outgoing traffic from pods
- Specify allowed destinations
- Define allowed ports and protocols

## 📊 Policy Selectors

### Pod Selector
```yaml
spec:
  podSelector:
    matchLabels:
      app: web
```

### Namespace Selector
```yaml
spec:
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production
```

### IP Block
```yaml
spec:
  ingress:
  - from:
    - ipBlock:
        cidr: 10.0.0.0/8
        except:
        - 10.0.1.0/24
```

## 🚫 Default Deny Policies

### Deny All Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

### Deny All Egress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

### Deny All Traffic
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## ✅ Allow Policies

### Allow Specific Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Allow Specific Egress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-database
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

## 🌐 Namespace Isolation

### Namespace-based Policy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-isolation
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: allowed-namespace
```

### Cross-namespace Communication
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cross-namespace-policy
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
      podSelector:
        matchLabels:
          app: frontend
```

## 🔧 Advanced Policies

### Multiple Rules
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: multi-rule-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
      podSelector:
        matchLabels:
          app: prometheus
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

## 🚀 Network Policy Operations

### Apply Policies
```bash
# Apply network policy
kubectl apply -f network-policy.yaml

# List network policies
kubectl get networkpolicies
kubectl get netpol

# Describe policy
kubectl describe networkpolicy <policy-name>

# Delete policy
kubectl delete networkpolicy <policy-name>
```

### Test Policies
```bash
# Test connectivity between pods
kubectl exec -it <source-pod> -- curl <target-pod-ip>:8080

# Test from specific namespace
kubectl exec -it <pod-name> -n <namespace> -- curl <target-service>

# Check DNS resolution
kubectl exec -it <pod-name> -- nslookup <service-name>
```

## 🚨 Troubleshooting Network Policies

### Common Issues
```bash
# Check if network plugin supports policies
kubectl get nodes -o wide

# Verify policy is applied
kubectl describe networkpolicy <policy-name>

# Check pod labels
kubectl get pods --show-labels

# Test connectivity
kubectl exec -it <pod> -- telnet <target-ip> <port>
```

### Debug Commands
```bash
# Check network plugin logs
kubectl logs -n kube-system -l k8s-app=calico-node

# Verify policy enforcement
kubectl get networkpolicy -o yaml

# Check endpoints
kubectl get endpoints <service-name>
```

## 📋 Best Practices

### Security Guidelines
- Start with default deny policies
- Use least privilege principle
- Test policies in staging first
- Document policy purposes

### Design Patterns
- Namespace-level isolation
- Tier-based segmentation
- Service-to-service communication
- External access control

### Monitoring
- Log policy violations
- Monitor network traffic
- Regular policy audits
- Performance impact assessment