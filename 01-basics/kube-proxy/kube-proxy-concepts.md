# Kube Proxy - Network Proxy

## 🎯 What is Kube Proxy?
- Network proxy on each node
- Implements Kubernetes service abstraction
- Maintains network rules for services
- Handles load balancing to pods

## 🔧 Kube Proxy Functions

### Service Implementation:
- Creates iptables/IPVS rules
- Routes traffic to service endpoints
- Load balances across pod replicas
- Handles service discovery

### Network Modes:
1. **iptables** (default) - Uses iptables rules
2. **IPVS** - Uses IP Virtual Server
3. **userspace** - Legacy mode

## 📊 Service Types and Proxy

### ClusterIP Service:
```bash
# Creates iptables rules for cluster-internal access
# Traffic: Client → kube-proxy → Pod
```

### NodePort Service:
```bash
# Creates rules for external access via node ports
# Traffic: External → Node:Port → kube-proxy → Pod
```

### LoadBalancer Service:
```bash
# Combines NodePort with external load balancer
# Traffic: External LB → Node:Port → kube-proxy → Pod
```

## 🔧 Kube Proxy Configuration

### ConfigMap Configuration:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-proxy
  namespace: kube-system
data:
  config.conf: |
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    kind: KubeProxyConfiguration
    bindAddress: 0.0.0.0
    clientConnection:
      kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
    clusterCIDR: 10.244.0.0/16
    mode: "iptables"
    iptables:
      masqueradeAll: false
      masqueradeBit: 14
      minSyncPeriod: 0s
      syncPeriod: 30s
```

### DaemonSet Deployment:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-proxy
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: kube-proxy
  template:
    metadata:
      labels:
        k8s-app: kube-proxy
    spec:
      containers:
      - name: kube-proxy
        image: k8s.gcr.io/kube-proxy:v1.21.0
        command:
        - /usr/local/bin/kube-proxy
        - --config=/var/lib/kube-proxy/config.conf
        - --hostname-override=$(NODE_NAME)
```

## 🔍 Kube Proxy Operations

### Check Kube Proxy:
```bash
# Check kube-proxy pods
kubectl get pods -n kube-system | grep kube-proxy

# Check kube-proxy logs
kubectl logs -n kube-system kube-proxy-<node-name>

# Check kube-proxy configuration
kubectl get configmap kube-proxy -n kube-system -o yaml
```

### Network Rules:
```bash
# View iptables rules (on nodes)
sudo iptables -t nat -L | grep <service-name>

# View IPVS rules (if using IPVS mode)
sudo ipvsadm -L -n

# Check service endpoints
kubectl get endpoints <service-name>
```

## 🚨 Troubleshooting Kube Proxy

### Service Connectivity Issues:
```bash
# Check service exists
kubectl get svc <service-name>

# Check endpoints
kubectl get endpoints <service-name>

# Test service from pod
kubectl exec -it <pod-name> -- curl <service-name>:<port>

# Check kube-proxy logs
kubectl logs -n kube-system kube-proxy-<node-name>
```

### Network Debugging:
```bash
# Check if service is accessible
kubectl run test --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>

# Check iptables rules
sudo iptables -t nat -L KUBE-SERVICES

# Check for kube-proxy errors
kubectl describe pod -n kube-system kube-proxy-<node-name>
```

### Common Issues:
- Service not accessible
- Load balancing not working
- iptables rules missing
- Network policy conflicts

## 📋 Service Mesh Integration

### With Istio:
- Kube-proxy handles east-west traffic
- Istio sidecar handles service mesh features
- Both work together for complete networking