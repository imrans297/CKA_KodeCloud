# Kubernetes Cluster Upgrades

## 🎯 Upgrade Concepts

### Upgrade Strategy
- **Rolling upgrades** - Update nodes one by one
- **Blue-green deployment** - Parallel cluster approach
- **In-place upgrades** - Update existing cluster
- **Canary upgrades** - Gradual rollout

### Version Compatibility
- **API Server** - Highest version component
- **kubelet** - Can be 1-2 minor versions behind API server
- **kube-proxy** - Same version as kubelet
- **kubectl** - ±1 minor version from API server

## 📋 Pre-Upgrade Planning

### Version Planning
```bash
# Check current versions
kubectl version --short
kubeadm version
kubelet --version

# Check available versions
apt list -a kubeadm
yum list --showduplicates kubeadm

# Check upgrade path
kubeadm upgrade plan
```

### Backup Before Upgrade
```bash
# Backup ETCD
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-pre-upgrade.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Backup certificates
sudo cp -r /etc/kubernetes/pki /backup/pki-pre-upgrade

# Backup cluster resources
kubectl get all --all-namespaces -o yaml > /backup/resources-pre-upgrade.yaml
```

## 🚀 Control Plane Upgrade

### Step 1: Upgrade kubeadm
```bash
# Update package repository
sudo apt update
# or
sudo yum update

# Upgrade kubeadm (example: to v1.28.0)
sudo apt-mark unhold kubeadm
sudo apt install -y kubeadm=1.28.0-00
sudo apt-mark hold kubeadm

# Verify kubeadm version
kubeadm version
```

### Step 2: Plan and Apply Upgrade
```bash
# Check upgrade plan
sudo kubeadm upgrade plan

# Apply upgrade to first control plane node
sudo kubeadm upgrade apply v1.28.0

# For additional control plane nodes
sudo kubeadm upgrade node
```

### Step 3: Upgrade kubelet and kubectl
```bash
# Drain the node
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt install -y kubelet=1.28.0-00 kubectl=1.28.0-00
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Uncordon the node
kubectl uncordon <node-name>
```

## 🔧 Worker Node Upgrade

### Upgrade Process
```bash
# On control plane: drain worker node
kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data

# On worker node: upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt install -y kubeadm=1.28.0-00
sudo apt-mark hold kubeadm

# Upgrade node configuration
sudo kubeadm upgrade node

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt install -y kubelet=1.28.0-00 kubectl=1.28.0-00
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# On control plane: uncordon node
kubectl uncordon <worker-node>
```

## 📊 Upgrade Verification

### Post-Upgrade Checks
```bash
# Check cluster status
kubectl cluster-info
kubectl get nodes
kubectl get pods --all-namespaces

# Check component versions
kubectl version --short
kubectl get nodes -o wide

# Check system pods
kubectl get pods -n kube-system

# Verify cluster functionality
kubectl run test-pod --image=nginx --rm -it --restart=Never -- curl google.com
```

### Health Checks
```bash
# Check API server health
kubectl get --raw /healthz

# Check component status
kubectl get componentstatuses

# Check cluster events
kubectl get events --sort-by=.metadata.creationTimestamp

# Test workload deployment
kubectl create deployment test-upgrade --image=nginx
kubectl scale deployment test-upgrade --replicas=3
kubectl delete deployment test-upgrade
```

## 🔄 Rollback Procedures

### Rollback Control Plane
```bash
# If upgrade fails, restore from backup
sudo systemctl stop kubelet

# Restore ETCD data
sudo systemctl stop etcd
sudo rm -rf /var/lib/etcd
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-pre-upgrade.db \
  --data-dir=/var/lib/etcd

# Restore certificates if needed
sudo cp -r /backup/pki-pre-upgrade/* /etc/kubernetes/pki/

# Downgrade packages
sudo apt-mark unhold kubeadm kubelet kubectl
sudo apt install -y kubeadm=1.27.0-00 kubelet=1.27.0-00 kubectl=1.27.0-00
sudo apt-mark hold kubeadm kubelet kubectl

# Restart services
sudo systemctl start etcd
sudo systemctl start kubelet
```

## 🛠️ Automated Upgrade Scripts

### Upgrade Script Template
```bash
#!/bin/bash
# kubernetes-upgrade.sh

set -e

OLD_VERSION="1.27.0"
NEW_VERSION="1.28.0"
NODE_TYPE="control-plane"  # or "worker"

echo "Starting Kubernetes upgrade from $OLD_VERSION to $NEW_VERSION"

# Backup
echo "Creating backup..."
kubectl get all --all-namespaces -o yaml > /backup/pre-upgrade-$(date +%Y%m%d).yaml

# Upgrade kubeadm
echo "Upgrading kubeadm..."
sudo apt-mark unhold kubeadm
sudo apt install -y kubeadm=${NEW_VERSION}-00
sudo apt-mark hold kubeadm

if [ "$NODE_TYPE" = "control-plane" ]; then
    # Control plane upgrade
    echo "Upgrading control plane..."
    sudo kubeadm upgrade apply $NEW_VERSION -y
else
    # Worker node upgrade
    echo "Upgrading worker node..."
    sudo kubeadm upgrade node
fi

# Upgrade kubelet and kubectl
echo "Upgrading kubelet and kubectl..."
sudo apt-mark unhold kubelet kubectl
sudo apt install -y kubelet=${NEW_VERSION}-00 kubectl=${NEW_VERSION}-00
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

echo "Upgrade completed successfully"
```

## 📋 Upgrade Best Practices

### Planning
- Test upgrades in staging environment
- Review release notes and breaking changes
- Plan maintenance windows
- Prepare rollback procedures
- Notify stakeholders

### Execution
- Upgrade one minor version at a time
- Always backup before upgrading
- Upgrade control plane first
- Upgrade worker nodes one by one
- Monitor cluster health throughout

### Validation
- Verify all components are healthy
- Test critical applications
- Check resource quotas and limits
- Validate network policies
- Confirm monitoring and logging

## 🚨 Troubleshooting Upgrades

### Common Issues
```bash
# API server won't start
sudo journalctl -u kubelet -f
kubectl logs -n kube-system kube-apiserver-master

# Pods stuck in pending
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp

# Node not ready after upgrade
kubectl describe node <node-name>
sudo systemctl status kubelet
```

### Recovery Procedures
```bash
# Reset failed upgrade
sudo kubeadm reset
sudo systemctl stop kubelet
sudo systemctl stop docker

# Clean up
sudo rm -rf /var/lib/cni/
sudo rm -rf /var/lib/kubelet/*
sudo rm -rf /etc/cni/

# Restore from backup and retry
```

## 📊 Upgrade Monitoring

### Metrics to Monitor
- Node readiness status
- Pod restart counts
- API server response times
- ETCD performance
- Application availability

### Monitoring Commands
```bash
# Watch node status
kubectl get nodes -w

# Monitor pod status
kubectl get pods --all-namespaces -w

# Check resource usage
kubectl top nodes
kubectl top pods --all-namespaces

# Monitor events
kubectl get events -w
```