# Kubernetes Network Policies

## 🎯 Overview

Network Policies provide network-level security by controlling traffic flow between pods, namespaces, and external endpoints using label selectors and rules.

## 🔐 Network Policy Basics

### Default Behavior
- **Without Network Policies**: All pods can communicate with each other
- **With Network Policies**: Only explicitly allowed traffic is permitted
- **CNI Requirement**: Requires CNI plugin that supports Network Policies (Calico, Cilium, Weave)

## What is a Network Policy in Kubernetes?

A NetworkPolicy is a Kubernetes resource that controls how Pods communicate with each other and with other network endpoints.

It defines which traffic is allowed to/from a Pod (by default, all traffic is allowed until a policy says otherwise).

### 🧠 Think of it as:

A firewall for Pods — defined at the network layer (Layer 3/4).

⚙️ Key Concepts
Termi	Description
Pod Selector	Which Pods this policy applies to.
Ingress	Rules for incomng traffic to the selected Pods.
Egress	Rules for outgoing traffic from the selected Pods.
Namespace Selector	Allows you to filter traffic from Pods in specific namespaces.
IPBlock	Allow or deny traffic from specific CIDR IP ranges.


### Policy Types
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-policy
spec:
  podSelector: {}
  policyTypes:
  - Ingress    # Controls incoming traffic
  - Egress     # Controls outgoing traffic
```

## 🚪 Ingress Rules

### Allow Specific Pods
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: production
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

### Allow from Specific Namespace
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-namespace
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
          name: frontend-namespace
    ports:
    - protocol: TCP
      port: 3000
```

### Allow from IP Blocks
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external-ips
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
        except:
        - 192.168.1.5/32
    ports:
    - protocol: TCP
      port: 80
```

## 🚪 Egress Rules

### Allow to Specific Pods
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-to-database
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

### Allow DNS Resolution
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

### Allow External APIs
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external-api
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 10.0.0.0/8
        - 192.168.0.0/16
        - 172.16.0.0/12
    ports:
    - protocol: TCP
      port: 443
```

## 🎯 Real-World Scenarios

### Three-Tier Application
```yaml
# Frontend Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: webapp
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from: []  # Allow from anywhere
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 8080
  - to: []  # DNS
    ports:
    - protocol: UDP
      port: 53
---
# Backend Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: webapp
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []  # DNS
    ports:
    - protocol: UDP
      port: 53
---
# Database Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: webapp
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 5432
```

### Microservices Communication
```yaml
# Service A can only talk to Service B
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: service-a-policy
spec:
  podSelector:
    matchLabels:
      app: service-a
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: service-b
    ports:
    - protocol: TCP
      port: 8080
  - to: []
    ports:
    - protocol: UDP
      port: 53  # DNS
---
# Service B accepts from Service A only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: service-b-policy
spec:
  podSelector:
    matchLabels:
      app: service-b
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: service-a
    ports:
    - protocol: TCP
      port: 8080
```

### Multi-Tenant Isolation
```yaml
# Tenant A isolation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: tenant-a-isolation
  namespace: tenant-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-a
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-a
  - to: []  # Allow DNS and external
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 443
```

## 🔒 Security Patterns

### Default Deny All
```yaml
# Deny all ingress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
# Deny all egress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

### Allow Only Necessary Traffic
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-app-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 10.0.0.0/8  # Internal network only
    ports:
    - protocol: TCP
      port: 8080
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
    - protocol: UDP
      port: 53
```

### Namespace Isolation
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-isolation
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production
    - namespaceSelector:
        matchLabels:
          name: monitoring  # Allow monitoring
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: production
  - to: []  # External traffic
    ports:
    - protocol: TCP
      port: 443
    - protocol: UDP
      port: 53
```

## 🛠️ Testing Network Policies

### Test Connectivity
```bash
# Test from one pod to another
kubectl exec -it source-pod -- nc -zv target-pod-ip 8080

# Test HTTP connectivity
kubectl exec -it source-pod -- curl -I http://target-service:8080

# Test with timeout
kubectl exec -it source-pod -- timeout 5 nc -zv target-ip 8080
```

### Debug Network Policies
```bash
# Check if CNI supports Network Policies
kubectl get nodes -o wide

# List network policies
kubectl get networkpolicies --all-namespaces

# Describe network policy
kubectl describe networkpolicy policy-name -n namespace

# Check pod labels
kubectl get pods --show-labels
```

### Network Policy Testing Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: network-test
  labels:
    app: network-test
spec:
  containers:
  - name: test
    image: nicolaka/netshoot
    command: ["sleep", "3600"]
```

## 📊 Monitoring Network Policies

### Calico Network Policy Logs
```bash
# Enable Calico logging
kubectl patch felixconfiguration default --type merge --patch '{"spec":{"flowLogsEnabled":true}}'

# View Calico logs
kubectl logs -n calico-system -l k8s-app=calico-node
```

### Cilium Network Policy Monitoring
```bash
# Check Cilium policy status
kubectl exec -n cilium cilium-pod -- cilium policy get

# Monitor network flows
kubectl exec -n cilium cilium-pod -- cilium monitor --type policy-verdict
```

### Prometheus Metrics
```yaml
# Network policy metrics
apiVersion: v1
kind: ConfigMap
metadata:
  name: network-policy-metrics
data:
  rules.yml: |
    groups:
    - name: network_policies
      rules:
      - alert: NetworkPolicyDenied
        expr: increase(cilium_policy_verdict_total{verdict="DENIED"}[5m]) > 10
        labels:
          severity: warning
        annotations:
          summary: "High number of denied network connections"
```

## 🔧 Advanced Network Policies

### Named Ports
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: named-port-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: http  # Named port from pod spec
```

### Multiple Selectors
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: multiple-selectors
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: external-services
    ports:
    - protocol: TCP
      port: 8080
```

### Complex Egress Rules
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: complex-egress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  # Allow to database in same namespace
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  # Allow to external APIs
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 10.0.0.0/8
        - 192.168.0.0/16
    ports:
    - protocol: TCP
      port: 443
  # Allow DNS
  - to: []
    ports:
    - protocol: UDP
      port: 53
```

## 🚨 Network Policy Best Practices

### Security Guidelines
1. **Default Deny**: Start with deny-all policies
2. **Least Privilege**: Allow only necessary traffic
3. **Label Strategy**: Use consistent labeling
4. **Testing**: Test policies in staging first
5. **Monitoring**: Monitor denied connections

### Implementation Strategy
```yaml
# Step 1: Default deny all
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

# Step 2: Allow DNS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53

# Step 3: Add specific application rules
```

### Common Patterns
```bash
# Allow ingress from ingress controller
kubectl label namespace ingress-nginx name=ingress-nginx

# Allow monitoring namespace access
kubectl label namespace monitoring name=monitoring

# Label pods consistently
kubectl label pods -l app=web tier=frontend
```

## 🛠️ Troubleshooting Network Policies

### Common Issues
```bash
# Policy not working - check CNI support
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.containerRuntimeVersion}'

# Check pod labels match selectors
kubectl get pods --show-labels | grep app=web

# Verify namespace labels
kubectl get namespaces --show-labels

# Test connectivity
kubectl exec -it test-pod -- nc -zv target-service 8080
```

### Debug Commands
```bash
# List all network policies
kubectl get networkpolicies --all-namespaces -o wide

# Check policy details
kubectl describe networkpolicy policy-name

# View effective policies on pod
kubectl describe pod pod-name | grep -A 10 "Network Policy"

# Check CNI logs
kubectl logs -n kube-system -l k8s-app=calico-node
```


### Important Notes

NetworkPolicies are enforced only if your CNI plugin supports them.
Examples of CNIs that support it:

Calico ✅

Cilium ✅

Weave Net ✅

Flannel ❌ (does not support NetworkPolicies)

They are “default deny” once a policy selects a Pod.
That Pod can only communicate as per allowed rules.

Policies are additive — multiple policies can apply to the same Pod.
The union of all allowed traffic is permitted.