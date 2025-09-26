# ETCD - Distributed Key-Value Store

## 🎯 What is ETCD?
- Distributed, reliable key-value store
- Stores all cluster data
- Source of truth for Kubernetes
- Uses Raft consensus algorithm

## 📊 ETCD in Kubernetes

### What ETCD Stores:
- Cluster state information
- Node information
- Pod specifications
- Service definitions
- ConfigMaps and Secrets
- Network policies
- RBAC policies

### ETCD Deployment Types:
1. **Stacked ETCD** - ETCD runs on master nodes
2. **External ETCD** - ETCD runs on separate nodes

## 🔧 ETCD Operations

### Check ETCD Status
```bash
# From master node
kubectl get pods -n kube-system | grep etcd

# ETCD health check
kubectl exec etcd-master -n kube-system -- etcdctl endpoint health

# ETCD member list
kubectl exec etcd-master -n kube-system -- etcdctl member list
```

### ETCD Data Structure
```bash
# View all keys
kubectl exec etcd-master -n kube-system -- sh -c \
"ETCDCTL_API=3 etcdctl get / --prefix --keys-only \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key"

# View specific resource
kubectl exec etcd-master -n kube-system -- sh -c \
"ETCDCTL_API=3 etcdctl get /registry/pods/default/nginx \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key"
```

## 💾 ETCD Backup & Restore

### Backup ETCD
```bash
# Create snapshot
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db
```

### Restore ETCD
```bash
# Stop API server
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# Restore from snapshot
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir=/var/lib/etcd-restore

# Update ETCD manifest
# Change data-dir to /var/lib/etcd-restore in /etc/kubernetes/manifests/etcd.yaml

# Start API server
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

## 🔐 ETCD Security

### Certificate Files:
- **CA Certificate**: `/etc/kubernetes/pki/etcd/ca.crt`
- **Server Certificate**: `/etc/kubernetes/pki/etcd/server.crt`
- **Server Key**: `/etc/kubernetes/pki/etcd/server.key`

### ETCD Configuration:
```yaml
# /etc/kubernetes/manifests/etcd.yaml
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.1.10:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.1.10:2380
    - --initial-cluster=master=https://192.168.1.10:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.1.10:2379
    - --listen-peer-urls=https://192.168.1.10:2380
    - --name=master
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

## 🚨 ETCD Troubleshooting

### Common Issues:
- Certificate problems
- Network connectivity
- Disk space issues
- Corruption

### Debug Commands:
```bash
# Check ETCD logs
kubectl logs etcd-master -n kube-system

# Check ETCD process
ps aux | grep etcd

# Check ETCD endpoints
kubectl exec etcd-master -n kube-system -- etcdctl endpoint status

# Check cluster health
kubectl exec etcd-master -n kube-system -- etcdctl cluster-health
```