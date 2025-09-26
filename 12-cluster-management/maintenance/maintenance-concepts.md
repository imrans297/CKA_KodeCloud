# Cluster Maintenance

## 🎯 Maintenance Concepts

### Types of Maintenance
- **Planned maintenance** - Scheduled updates and patches
- **Emergency maintenance** - Critical security fixes
- **Preventive maintenance** - Regular health checks
- **Corrective maintenance** - Fix identified issues

### Maintenance Windows
- **Rolling maintenance** - Node-by-node updates
- **Scheduled downtime** - Planned service interruption
- **Blue-green maintenance** - Parallel environment approach
- **Canary maintenance** - Gradual rollout

## 🔧 Node Maintenance

### Drain Node for Maintenance
```bash
# Cordon node (prevent new pods)
kubectl cordon <node-name>

# Drain node (evict pods gracefully)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Force drain if needed
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data --force --grace-period=0

# Check node status
kubectl get nodes
```

### Node Maintenance Tasks
```bash
# System updates
sudo apt update && sudo apt upgrade -y
sudo yum update -y

# Kernel updates (requires reboot)
sudo apt install linux-image-generic
sudo reboot

# Docker maintenance
sudo systemctl stop kubelet
sudo systemctl stop docker
sudo docker system prune -a -f
sudo systemctl start docker
sudo systemctl start kubelet

# Disk cleanup
sudo journalctl --vacuum-time=7d
sudo find /var/log -name "*.log" -mtime +30 -delete
```

### Return Node to Service
```bash
# Uncordon node
kubectl uncordon <node-name>

# Verify node is ready
kubectl get nodes
kubectl describe node <node-name>

# Check pod distribution
kubectl get pods -o wide --all-namespaces | grep <node-name>
```

## 🔄 Rolling Maintenance

### Rolling Update Strategy
```bash
#!/bin/bash
# rolling-maintenance.sh

NODES=$(kubectl get nodes --no-headers -o custom-columns=NAME:.metadata.name)

for NODE in $NODES; do
    echo "Maintaining node: $NODE"
    
    # Drain node
    kubectl drain $NODE --ignore-daemonsets --delete-emptydir-data --timeout=300s
    
    # Perform maintenance on node
    ssh $NODE "sudo apt update && sudo apt upgrade -y"
    
    # Reboot if needed
    if ssh $NODE "test -f /var/run/reboot-required"; then
        ssh $NODE "sudo reboot"
        
        # Wait for node to come back
        while ! ssh $NODE "echo 'Node is up'"; do
            echo "Waiting for $NODE to come back online..."
            sleep 30
        done
    fi
    
    # Uncordon node
    kubectl uncordon $NODE
    
    # Wait for node to be ready
    kubectl wait --for=condition=Ready node/$NODE --timeout=300s
    
    echo "Node $NODE maintenance completed"
    sleep 60  # Wait before next node
done
```

## 🛡️ Cluster Health Monitoring

### Health Check Commands
```bash
# Cluster overview
kubectl cluster-info
kubectl get nodes
kubectl get pods --all-namespaces

# Component status
kubectl get componentstatuses

# Resource usage
kubectl top nodes
kubectl top pods --all-namespaces

# Check for issues
kubectl get events --sort-by=.metadata.creationTimestamp | tail -20
kubectl get pods --all-namespaces --field-selector=status.phase!=Running
```

### Automated Health Checks
```bash
#!/bin/bash
# health-check.sh

echo "=== Kubernetes Cluster Health Check ==="
echo "Date: $(date)"
echo

# Check API server
if kubectl cluster-info > /dev/null 2>&1; then
    echo "✓ API Server: Healthy"
else
    echo "✗ API Server: Unhealthy"
    exit 1
fi

# Check nodes
NOT_READY_NODES=$(kubectl get nodes --no-headers | grep -v Ready | wc -l)
if [ $NOT_READY_NODES -eq 0 ]; then
    echo "✓ Nodes: All ready ($(kubectl get nodes --no-headers | wc -l) nodes)"
else
    echo "✗ Nodes: $NOT_READY_NODES not ready"
    kubectl get nodes | grep -v Ready
fi

# Check system pods
SYSTEM_PODS_ISSUES=$(kubectl get pods -n kube-system --no-headers | grep -v Running | grep -v Completed | wc -l)
if [ $SYSTEM_PODS_ISSUES -eq 0 ]; then
    echo "✓ System Pods: All healthy"
else
    echo "✗ System Pods: $SYSTEM_PODS_ISSUES issues"
    kubectl get pods -n kube-system | grep -v Running | grep -v Completed
fi

# Check ETCD health
if kubectl get --raw /healthz > /dev/null 2>&1; then
    echo "✓ ETCD: Healthy"
else
    echo "✗ ETCD: Issues detected"
fi

echo
echo "Health check completed"
```

## 📊 Resource Cleanup

### Clean Up Unused Resources
```bash
# Clean up completed jobs
kubectl get jobs --all-namespaces -o json | \
jq -r '.items[] | select(.status.conditions[]?.type == "Complete") | select(.metadata.creationTimestamp | fromdateiso8601 < (now - 86400)) | "\(.metadata.namespace) \(.metadata.name)"' | \
while read namespace name; do
    kubectl delete job $name -n $namespace
done

# Clean up failed pods
kubectl delete pods --all-namespaces --field-selector=status.phase=Failed

# Clean up evicted pods
kubectl get pods --all-namespaces -o json | \
jq -r '.items[] | select(.status.reason == "Evicted") | "\(.metadata.namespace) \(.metadata.name)"' | \
while read namespace name; do
    kubectl delete pod $name -n $namespace
done

# Clean up unused ConfigMaps and Secrets
kubectl get configmaps --all-namespaces --no-headers | \
while read namespace name; do
    if ! kubectl get pods -n $namespace -o yaml | grep -q "configMapKeyRef.*$name\|configMap.*$name"; then
        echo "Unused ConfigMap: $namespace/$name"
    fi
done
```

### Docker System Cleanup
```bash
# On each node
ssh <node> "sudo docker system prune -a -f"
ssh <node> "sudo docker volume prune -f"
ssh <node> "sudo docker network prune -f"

# Clean up kubelet cache
ssh <node> "sudo rm -rf /var/lib/kubelet/pods/*"
```

## 🔐 Security Maintenance

### Certificate Renewal
```bash
# Check certificate expiration
kubeadm certs check-expiration

# Renew all certificates
kubeadm certs renew all

# Restart control plane components
sudo systemctl restart kubelet

# Update kubeconfig
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
```

### Security Updates
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Update Kubernetes components
sudo apt update
sudo apt install -y kubeadm=1.28.1-00 kubelet=1.28.1-00 kubectl=1.28.1-00

# Update container runtime
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

## 📋 Maintenance Schedules

### Daily Tasks
- Monitor cluster health
- Check resource usage
- Review logs for errors
- Verify backup completion

### Weekly Tasks
- Clean up unused resources
- Update security patches
- Review capacity planning
- Test disaster recovery

### Monthly Tasks
- Certificate renewal check
- Full system updates
- Performance optimization
- Security audit

### Quarterly Tasks
- Kubernetes version upgrades
- Hardware maintenance
- Disaster recovery testing
- Documentation updates

## 🚨 Emergency Procedures

### Node Failure Response
```bash
# Identify failed node
kubectl get nodes

# Check node events
kubectl describe node <failed-node>

# Cordon failed node
kubectl cordon <failed-node>

# Drain workloads
kubectl drain <failed-node> --ignore-daemonsets --delete-emptydir-data --force

# Remove node from cluster
kubectl delete node <failed-node>

# Add replacement node
kubeadm join <control-plane-endpoint> --token <token> --discovery-token-ca-cert-hash <hash>
```

### ETCD Recovery
```bash
# Stop API server
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# Restore ETCD from backup
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore

# Update ETCD configuration
sudo vim /etc/kubernetes/manifests/etcd.yaml
# Change data-dir to /var/lib/etcd-restore

# Start API server
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

## 📊 Maintenance Monitoring

### Maintenance Metrics
- Node availability percentage
- Pod eviction success rate
- Maintenance window duration
- Service disruption time
- Recovery time objectives

### Monitoring Commands
```bash
# Track maintenance progress
kubectl get events --sort-by=.metadata.creationTimestamp -w

# Monitor pod redistribution
kubectl get pods -o wide --all-namespaces -w

# Check resource usage during maintenance
kubectl top nodes
kubectl top pods --all-namespaces
```

## 📋 Maintenance Checklist

### Pre-Maintenance
- [ ] Schedule maintenance window
- [ ] Notify stakeholders
- [ ] Create cluster backup
- [ ] Verify rollback procedures
- [ ] Prepare maintenance scripts

### During Maintenance
- [ ] Monitor cluster health
- [ ] Track maintenance progress
- [ ] Document any issues
- [ ] Verify each step completion
- [ ] Test critical applications

### Post-Maintenance
- [ ] Verify cluster functionality
- [ ] Check all nodes are ready
- [ ] Validate application health
- [ ] Update documentation
- [ ] Notify completion