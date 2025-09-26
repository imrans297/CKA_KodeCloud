# Kubelet - Node Agent

## 🎯 What is Kubelet?
- Primary node agent
- Manages pods on worker nodes
- Communicates with API server
- Monitors pod and container health

## 🔧 Kubelet Functions

### Pod Management:
- Creates and manages containers
- Pulls container images
- Mounts volumes
- Reports pod status to API server

### Health Monitoring:
- Runs liveness probes
- Runs readiness probes
- Restarts failed containers
- Reports node status

### Resource Management:
- Enforces resource limits
- Manages cgroups
- Handles QoS classes

## 📊 Kubelet Configuration

### Configuration File:
```yaml
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
authorization:
  mode: Webhook
```

### Service Configuration:
```bash
# /etc/systemd/system/kubelet.service
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/

[Service]
ExecStart=/usr/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## 🔧 Kubelet Operations

### Check Kubelet Status:
```bash
# Service status
systemctl status kubelet

# Kubelet logs
journalctl -u kubelet -f

# Kubelet process
ps aux | grep kubelet
```

### Kubelet API:
```bash
# Node metrics
curl -k https://node-ip:10250/metrics

# Pod specs
curl -k https://node-ip:10250/pods

# Running pods
curl -k https://node-ip:10250/runningpods
```

## 📋 Static Pods

### Static Pod Configuration:
```bash
# Static pod directory
/etc/kubernetes/manifests/

# Create static pod
cat > /etc/kubernetes/manifests/static-web.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: static-web
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
EOF
```

### Static Pod Management:
```bash
# List static pods
kubectl get pods --all-namespaces | grep -E "master|node"

# Static pods are managed by kubelet directly
# Cannot be deleted via kubectl
# Must remove from manifests directory
```

## 🚨 Troubleshooting Kubelet

### Common Issues:
- Container runtime problems
- Network connectivity
- Certificate issues
- Resource constraints

### Debug Commands:
```bash
# Check kubelet logs
journalctl -u kubelet --no-pager

# Check node status
kubectl describe node <node-name>

# Check kubelet configuration
kubelet --help | grep config

# Test container runtime
crictl ps
docker ps (if using Docker)
```

### Node Problems:
```bash
# Node not ready
kubectl get nodes
kubectl describe node <node-name>

# Check kubelet service
systemctl status kubelet
systemctl restart kubelet

# Check certificates
ls -la /var/lib/kubelet/pki/
```