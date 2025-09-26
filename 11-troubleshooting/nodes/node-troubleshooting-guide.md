# Node Troubleshooting Guide

## 🚨 Common Node Issues

### Node Not Ready
**Symptoms:** Node shows NotReady status
**Causes:**
- kubelet not running
- Network connectivity issues
- Resource exhaustion
- Container runtime problems
- Certificate issues

**Troubleshooting:**
```bash
# Check node status
kubectl get nodes
kubectl describe node <node-name>

# Check kubelet status
systemctl status kubelet
journalctl -u kubelet -f

# Check kubelet logs
journalctl -u kubelet --no-pager
```

### Node Resource Exhaustion
**Symptoms:** Pods evicted, scheduling failures
**Causes:**
- High CPU/memory usage
- Disk space full
- Too many pods
- Resource limits exceeded

**Troubleshooting:**
```bash
# Check node resources
kubectl top nodes
kubectl describe node <node-name> | grep -A 10 "Allocated resources"

# Check disk usage
df -h
du -sh /var/lib/docker
du -sh /var/lib/kubelet
```

### Network Issues
**Symptoms:** Pod-to-pod communication fails
**Causes:**
- CNI plugin issues
- Network configuration problems
- Firewall blocking traffic
- DNS resolution failures

**Troubleshooting:**
```bash
# Check network plugin pods
kubectl get pods -n kube-system | grep -E "calico|flannel|weave"

# Check network configuration
ip route show
ip addr show

# Test connectivity
ping <other-node-ip>
telnet <api-server-ip> 6443
```

## 🔍 Node Diagnostic Commands

### Basic Node Information
```bash
# List nodes
kubectl get nodes
kubectl get nodes -o wide
kubectl get nodes --show-labels

# Detailed node information
kubectl describe node <node-name>

# Node YAML configuration
kubectl get node <node-name> -o yaml
```

### Node Status and Conditions
```bash
# Check node conditions
kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[-1].type,REASON:.status.conditions[-1].reason

# Check specific conditions
kubectl describe node <node-name> | grep -A 10 Conditions

# Check node events
kubectl get events --field-selector involvedObject.name=<node-name>
```

### Resource Usage
```bash
# Node resource usage
kubectl top nodes
kubectl top node <node-name>

# Detailed resource allocation
kubectl describe node <node-name> | grep -A 20 "Allocated resources"

# Check system resources
free -h
df -h
iostat -x 1
```

## 🔧 kubelet Troubleshooting

### kubelet Service Issues
```bash
# Check kubelet service status
systemctl status kubelet
systemctl is-active kubelet
systemctl is-enabled kubelet

# Restart kubelet
sudo systemctl restart kubelet

# Check kubelet configuration
cat /var/lib/kubelet/config.yaml
ps aux | grep kubelet
```

### kubelet Logs
```bash
# View kubelet logs
journalctl -u kubelet -f
journalctl -u kubelet --no-pager
journalctl -u kubelet --since "1 hour ago"

# Check for specific errors
journalctl -u kubelet | grep -i error
journalctl -u kubelet | grep -i failed
```

### kubelet API
```bash
# Check kubelet API (from node)
curl -k https://localhost:10250/metrics
curl -k https://localhost:10250/pods

# Check kubelet health
curl -k https://localhost:10250/healthz
```

## 🐳 Container Runtime Issues

### Docker Runtime
```bash
# Check Docker status
systemctl status docker
docker version
docker info

# Check Docker logs
journalctl -u docker -f

# Clean up Docker resources
docker system prune -f
docker volume prune -f
```

### containerd Runtime
```bash
# Check containerd status
systemctl status containerd
ctr version

# List containers
ctr containers list
ctr images list

# Check containerd logs
journalctl -u containerd -f
```

### CRI-O Runtime
```bash
# Check CRI-O status
systemctl status crio
crictl version

# List containers and images
crictl ps
crictl images

# Check CRI-O logs
journalctl -u crio -f
```

## 🌐 Network Troubleshooting

### CNI Plugin Issues
```bash
# Check CNI configuration
ls -la /etc/cni/net.d/
cat /etc/cni/net.d/*

# Check CNI plugin pods
kubectl get pods -n kube-system | grep -E "calico|flannel|weave|cilium"

# Check CNI plugin logs
kubectl logs -n kube-system -l k8s-app=calico-node
kubectl logs -n kube-system -l app=flannel
```

### Network Connectivity
```bash
# Test basic connectivity
ping <api-server-ip>
ping <other-node-ip>
ping 8.8.8.8

# Check network interfaces
ip addr show
ip route show
ip link show

# Check iptables rules
sudo iptables -L
sudo iptables -t nat -L
```

### DNS Issues
```bash
# Test DNS resolution
nslookup kubernetes.default.svc.cluster.local
dig kubernetes.default.svc.cluster.local

# Check DNS configuration
cat /etc/resolv.conf

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

## 💾 Storage Issues

### Disk Space Problems
```bash
# Check disk usage
df -h
du -sh /var/lib/docker
du -sh /var/lib/kubelet
du -sh /var/log

# Clean up disk space
docker system prune -a -f
journalctl --vacuum-time=7d

# Check for large files
find /var -type f -size +100M -exec ls -lh {} \;
```

### Volume Mount Issues
```bash
# Check mounted volumes
mount | grep kubelet
lsblk

# Check volume plugin logs
kubectl logs -n kube-system -l app=csi-driver

# Check persistent volume status
kubectl get pv
kubectl describe pv <pv-name>
```

## 🔐 Security and Certificates

### Certificate Issues
```bash
# Check kubelet certificates
ls -la /var/lib/kubelet/pki/
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text -noout

# Check certificate expiration
kubeadm certs check-expiration

# Renew certificates
kubeadm certs renew all
```

### Node Authorization
```bash
# Check node authorization
kubectl auth can-i get nodes --as=system:node:<node-name>

# Check node registration
kubectl get csr
kubectl describe csr <csr-name>
```

## 📊 Performance Issues

### High CPU/Memory Usage
```bash
# Check system load
top
htop
iostat -x 1

# Check process usage
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10

# Check kubelet resource usage
systemctl status kubelet
cat /proc/$(pgrep kubelet)/status
```

### I/O Performance
```bash
# Check I/O statistics
iostat -x 1
iotop

# Check disk performance
hdparm -tT /dev/sda

# Check file system
fsck /dev/sda1
```

## 🔄 Node Maintenance

### Drain and Cordon
```bash
# Cordon node (prevent new pods)
kubectl cordon <node-name>

# Drain node (evict pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Uncordon node
kubectl uncordon <node-name>
```

### Node Replacement
```bash
# Remove node from cluster
kubectl delete node <node-name>

# Reset node (on the node)
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
```

## 🛠️ Debug Tools

### System Information
```bash
# System information
uname -a
lscpu
lsmem
lsblk
lspci

# Check system logs
dmesg | tail -50
journalctl --since "1 hour ago"
```

### Network Tools
```bash
# Network debugging
netstat -tulpn
ss -tulpn
tcpdump -i any port 6443
wireshark

# Connectivity testing
telnet <ip> <port>
nc -zv <ip> <port>
```

## 📋 Node Health Checklist

### System Health
- [ ] Node is powered on and accessible
- [ ] Operating system is running
- [ ] Sufficient CPU and memory available
- [ ] Disk space is adequate
- [ ] Network connectivity is working

### Kubernetes Components
- [ ] kubelet service is running
- [ ] Container runtime is functional
- [ ] CNI plugin is working
- [ ] kube-proxy is running
- [ ] Node is registered with API server

### Security
- [ ] Certificates are valid and not expired
- [ ] Node has proper authorization
- [ ] Firewall rules allow required traffic
- [ ] Security updates are applied

### Performance
- [ ] System load is acceptable
- [ ] I/O performance is adequate
- [ ] Network latency is low
- [ ] No resource exhaustion