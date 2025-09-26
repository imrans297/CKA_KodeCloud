# Service Troubleshooting Guide

## 🚨 Common Service Issues

### Service Not Accessible
**Symptoms:** Cannot reach service from pods or external clients
**Causes:**
- Incorrect service selector
- No matching pods
- Wrong port configuration
- Network policy blocking traffic
- Endpoint issues

**Troubleshooting:**
```bash
# Check service configuration
kubectl describe service <service-name>

# Check service endpoints
kubectl get endpoints <service-name>

# Verify pod labels match selector
kubectl get pods --show-labels
kubectl get pods -l <selector-labels>
```

### No Endpoints
**Symptoms:** Service has no endpoints
**Causes:**
- No pods match the selector
- Pods not ready
- Readiness probe failing
- Wrong namespace

**Troubleshooting:**
```bash
# Check if pods exist with matching labels
kubectl get pods -l <service-selector>

# Check pod readiness
kubectl get pods -o wide

# Check readiness probes
kubectl describe pod <pod-name> | grep -A 5 Readiness
```

### External Access Issues
**Symptoms:** Cannot access NodePort or LoadBalancer service externally
**Causes:**
- Firewall blocking ports
- Cloud provider issues
- Wrong node IP
- Security group restrictions

**Troubleshooting:**
```bash
# Check service type and ports
kubectl get service <service-name> -o wide

# Test from node
ssh <node-ip>
curl localhost:<nodeport>

# Check firewall rules
sudo iptables -L | grep <port>
```

## 🔍 Service Diagnostic Commands

### Basic Service Information
```bash
# List services
kubectl get services
kubectl get svc -o wide
kubectl get svc --all-namespaces

# Detailed service information
kubectl describe service <service-name>

# Service YAML configuration
kubectl get service <service-name> -o yaml
```

### Endpoint Analysis
```bash
# Check service endpoints
kubectl get endpoints <service-name>
kubectl describe endpoints <service-name>

# Check endpoint slices (K8s 1.17+)
kubectl get endpointslices
kubectl describe endpointslice <service-name>
```

### Service Testing
```bash
# Test service from pod
kubectl exec -it <pod-name> -- curl <service-name>:<port>

# Test service with full FQDN
kubectl exec -it <pod-name> -- curl <service-name>.<namespace>.svc.cluster.local

# Test specific endpoint
kubectl exec -it <pod-name> -- curl <pod-ip>:<port>
```

## 🌐 DNS and Service Discovery

### DNS Resolution Issues
```bash
# Test DNS resolution
kubectl exec -it <pod-name> -- nslookup <service-name>
kubectl exec -it <pod-name> -- dig <service-name>

# Check DNS configuration
kubectl exec -it <pod-name> -- cat /etc/resolv.conf

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### Service Discovery Problems
```bash
# Check service environment variables
kubectl exec -it <pod-name> -- env | grep SERVICE

# Test different service formats
kubectl exec -it <pod-name> -- curl <service-name>
kubectl exec -it <pod-name> -- curl <service-name>.<namespace>
kubectl exec -it <pod-name> -- curl <service-name>.<namespace>.svc.cluster.local
```

## 🔧 Network Troubleshooting

### Connectivity Testing
```bash
# Test pod-to-service connectivity
kubectl exec -it <source-pod> -- telnet <service-name> <port>

# Test direct pod connectivity
kubectl exec -it <source-pod> -- ping <target-pod-ip>

# Check network policies
kubectl get networkpolicy
kubectl describe networkpolicy <policy-name>
```

### kube-proxy Issues
```bash
# Check kube-proxy status
kubectl get pods -n kube-system -l k8s-app=kube-proxy

# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy

# Check iptables rules (on nodes)
sudo iptables -t nat -L | grep <service-name>

# Check IPVS rules (if using IPVS)
sudo ipvsadm -L -n
```

## 📊 Service Types Troubleshooting

### ClusterIP Services
```bash
# Test internal connectivity
kubectl run debug --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>

# Check cluster IP allocation
kubectl get service <service-name> -o jsonpath='{.spec.clusterIP}'

# Verify service is in cluster CIDR
kubectl cluster-info dump | grep service-cluster-ip-range
```

### NodePort Services
```bash
# Check NodePort allocation
kubectl get service <service-name> -o jsonpath='{.spec.ports[0].nodePort}'

# Test from external client
curl <node-ip>:<nodeport>

# Check if port is open on nodes
sudo netstat -tulpn | grep <nodeport>

# Test from node itself
ssh <node-ip>
curl localhost:<nodeport>
```

### LoadBalancer Services
```bash
# Check LoadBalancer status
kubectl get service <service-name> -o wide

# Check cloud provider logs
kubectl logs -n kube-system -l component=cloud-controller-manager

# Verify external IP assignment
kubectl describe service <service-name> | grep "LoadBalancer Ingress"

# Test external connectivity
curl <external-ip>:<port>
```

## 🔐 Security and Access Control

### Network Policy Impact
```bash
# Check if network policies affect service
kubectl get networkpolicy --all-namespaces

# Test connectivity with policy disabled
kubectl delete networkpolicy <policy-name>
# Test connectivity
# Recreate policy

# Check policy selectors
kubectl describe networkpolicy <policy-name>
```

### Service Account Issues
```bash
# Check service account permissions
kubectl auth can-i get services --as=system:serviceaccount:<namespace>:<sa-name>

# Check if pod can access service
kubectl exec -it <pod-name> -- curl <service-name>
```

## 🛠️ Debug Tools and Utilities

### Network Debug Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: network-debug
spec:
  containers:
  - name: netshoot
    image: nicolaka/netshoot
    command: ['sleep', '3600']
```

### Service Testing Commands
```bash
# Comprehensive service test
kubectl exec -it network-debug -- nslookup <service-name>
kubectl exec -it network-debug -- telnet <service-name> <port>
kubectl exec -it network-debug -- curl -v <service-name>:<port>

# Network connectivity matrix
for pod in $(kubectl get pods -o name); do
  echo "Testing from $pod"
  kubectl exec $pod -- curl -m 5 <service-name>:<port>
done
```

## 📋 Service Performance Issues

### Latency Problems
```bash
# Test service response time
kubectl exec -it <pod-name> -- time curl <service-name>

# Check service load balancing
for i in {1..10}; do
  kubectl exec -it <pod-name> -- curl <service-name>/hostname
done

# Monitor service metrics
kubectl top pods -l <service-selector>
```

### Load Balancing Issues
```bash
# Check service session affinity
kubectl get service <service-name> -o yaml | grep sessionAffinity

# Test load distribution
kubectl exec -it <pod-name> -- ab -n 100 -c 10 http://<service-name>/
```

## 🔄 Service Configuration Issues

### Port Mapping Problems
```bash
# Check service port configuration
kubectl get service <service-name> -o yaml | grep -A 10 ports

# Verify target port matches container port
kubectl describe pod <pod-name> | grep -A 5 "Port\|Containers"

# Test direct pod access
kubectl exec -it <test-pod> -- curl <pod-ip>:<container-port>
```

### Selector Mismatches
```bash
# Compare service selector with pod labels
kubectl get service <service-name> -o yaml | grep -A 5 selector
kubectl get pods --show-labels

# Check if any pods match selector
kubectl get pods -l <service-selector-labels>
```

## 📊 Monitoring and Alerting

### Service Health Monitoring
```bash
# Check service availability
kubectl get endpoints <service-name> -w

# Monitor service events
kubectl get events --field-selector involvedObject.name=<service-name> -w

# Check service metrics (if available)
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/<namespace>/services/<service-name>
```

## 📋 Troubleshooting Checklist

### Service Configuration
- [ ] Service selector matches pod labels
- [ ] Port and targetPort are correct
- [ ] Service type is appropriate
- [ ] Namespace is correct

### Pod Readiness
- [ ] Pods are running and ready
- [ ] Readiness probes are passing
- [ ] Pods have correct labels
- [ ] Container ports are exposed

### Network Connectivity
- [ ] DNS resolution works
- [ ] Network policies allow traffic
- [ ] kube-proxy is functioning
- [ ] Firewall rules permit access

### External Access (NodePort/LoadBalancer)
- [ ] External IP is assigned
- [ ] Firewall/security groups allow traffic
- [ ] Cloud provider configuration is correct
- [ ] DNS points to correct IP