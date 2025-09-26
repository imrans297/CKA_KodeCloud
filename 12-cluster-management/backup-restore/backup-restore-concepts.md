# Cluster Backup and Restore

## 🎯 Backup and Restore Concepts

### What to Backup
- **ETCD data** - Cluster state and configuration
- **Certificates** - PKI certificates and keys
- **Configuration files** - kubeadm, kubelet configs
- **Application data** - Persistent volumes
- **Cluster resources** - YAML manifests

### Backup Types
- **Full backup** - Complete cluster state
- **Incremental backup** - Changes since last backup
- **Application backup** - Persistent data only
- **Configuration backup** - Cluster configuration

## 💾 ETCD Backup

### ETCD Snapshot Creation
```bash
# Set ETCD API version
export ETCDCTL_API=3

# Create ETCD snapshot
etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
etcdctl snapshot status /backup/etcd-snapshot.db
```

### ETCD Snapshot from Pod
```bash
# From etcd pod
kubectl exec etcd-master -n kube-system -- sh -c \
"ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key"

# Copy snapshot from pod
kubectl cp kube-system/etcd-master:/tmp/etcd-backup.db ./etcd-backup.db
```

## 🔄 ETCD Restore

### Restore Process
```bash
# 1. Stop API server
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# 2. Restore ETCD snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore \
  --initial-cluster=master=https://192.168.1.10:2380 \
  --initial-advertise-peer-urls=https://192.168.1.10:2380

# 3. Update ETCD configuration
sudo vim /etc/kubernetes/manifests/etcd.yaml
# Change --data-dir=/var/lib/etcd to --data-dir=/var/lib/etcd-restore

# 4. Start API server
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

# 5. Restart kubelet
sudo systemctl restart kubelet
```

### Restore Verification
```bash
# Check cluster status
kubectl cluster-info
kubectl get nodes
kubectl get pods --all-namespaces

# Verify restored resources
kubectl get deployments --all-namespaces
kubectl get services --all-namespaces
```

## 📋 Resource Backup

### Backup All Resources
```bash
# Backup all namespaced resources
kubectl get all --all-namespaces -o yaml > all-resources-backup.yaml

# Backup specific resource types
kubectl get deployments --all-namespaces -o yaml > deployments-backup.yaml
kubectl get services --all-namespaces -o yaml > services-backup.yaml
kubectl get configmaps --all-namespaces -o yaml > configmaps-backup.yaml
kubectl get secrets --all-namespaces -o yaml > secrets-backup.yaml
```

### Backup Cluster-scoped Resources
```bash
# Backup cluster roles and bindings
kubectl get clusterroles -o yaml > clusterroles-backup.yaml
kubectl get clusterrolebindings -o yaml > clusterrolebindings-backup.yaml

# Backup nodes and persistent volumes
kubectl get nodes -o yaml > nodes-backup.yaml
kubectl get pv -o yaml > persistentvolumes-backup.yaml

# Backup storage classes
kubectl get storageclass -o yaml > storageclasses-backup.yaml
```

### Selective Resource Backup
```bash
# Backup specific namespace
kubectl get all -n production -o yaml > production-namespace-backup.yaml

# Backup with labels
kubectl get all -l app=critical -o yaml > critical-apps-backup.yaml

# Backup custom resources
kubectl get crd -o yaml > custom-resources-backup.yaml
```

## 🔧 Certificate Backup

### Backup PKI Certificates
```bash
# Backup entire PKI directory
sudo tar -czf kubernetes-pki-backup.tar.gz /etc/kubernetes/pki/

# Backup specific certificates
sudo cp -r /etc/kubernetes/pki/ /backup/kubernetes-pki/

# Backup ETCD certificates
sudo cp -r /etc/kubernetes/pki/etcd/ /backup/etcd-pki/
```

### Backup kubeconfig Files
```bash
# Backup admin kubeconfig
sudo cp /etc/kubernetes/admin.conf /backup/

# Backup component kubeconfigs
sudo cp /etc/kubernetes/controller-manager.conf /backup/
sudo cp /etc/kubernetes/scheduler.conf /backup/
sudo cp /etc/kubernetes/kubelet.conf /backup/
```

## 📊 Automated Backup

### Backup Script
```bash
#!/bin/bash
# kubernetes-backup.sh

BACKUP_DIR="/backup/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# ETCD backup
ETCDCTL_API=3 etcdctl snapshot save $BACKUP_DIR/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Resource backup
kubectl get all --all-namespaces -o yaml > $BACKUP_DIR/all-resources.yaml
kubectl get pv -o yaml > $BACKUP_DIR/persistent-volumes.yaml

# Certificate backup
sudo cp -r /etc/kubernetes/pki/ $BACKUP_DIR/
sudo cp /etc/kubernetes/admin.conf $BACKUP_DIR/

echo "Backup completed: $BACKUP_DIR"
```

### Cron Job for Automated Backup
```bash
# Add to crontab
0 2 * * * /usr/local/bin/kubernetes-backup.sh

# Kubernetes CronJob for resource backup
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cluster-backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: bitnami/kubectl:latest
            command:
            - /bin/sh
            - -c
            - kubectl get all --all-namespaces -o yaml > /backup/resources-$(date +%Y%m%d).yaml
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

## 🔄 Disaster Recovery

### Complete Cluster Restore
```bash
# 1. Install Kubernetes on new nodes
kubeadm init --config=cluster-config.yaml

# 2. Restore ETCD data
sudo systemctl stop kubelet
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd

# 3. Restore certificates
sudo cp -r /backup/kubernetes-pki/* /etc/kubernetes/pki/

# 4. Start cluster
sudo systemctl start kubelet

# 5. Verify cluster
kubectl get nodes
kubectl get pods --all-namespaces
```

### Application Data Restore
```bash
# Restore persistent volume data
kubectl apply -f persistent-volumes-backup.yaml

# Restore application resources
kubectl apply -f all-resources-backup.yaml

# Verify applications
kubectl get deployments --all-namespaces
kubectl get services --all-namespaces
```

## 📋 Best Practices

### Backup Strategy
- Regular automated backups
- Multiple backup locations
- Test restore procedures
- Document recovery processes
- Monitor backup success

### Security
- Encrypt backup data
- Secure backup storage
- Limit backup access
- Audit backup activities
- Rotate backup encryption keys

### Testing
- Regular restore testing
- Disaster recovery drills
- Backup validation
- Recovery time objectives
- Recovery point objectives

## 🚨 Troubleshooting Backup/Restore

### Common Issues
```bash
# ETCD backup fails
# Check certificate paths and permissions
ls -la /etc/kubernetes/pki/etcd/

# Restore fails
# Verify ETCD data directory permissions
sudo chown -R etcd:etcd /var/lib/etcd-restore

# Resource restore conflicts
# Check for existing resources
kubectl get all --all-namespaces
```

### Verification Commands
```bash
# Verify ETCD snapshot
etcdctl snapshot status /backup/etcd-snapshot.db

# Check backup file integrity
tar -tzf kubernetes-pki-backup.tar.gz

# Validate YAML backups
kubectl apply --dry-run=client -f all-resources-backup.yaml
```