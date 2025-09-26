# Persistent Volumes (PV) and Persistent Volume Claims (PVC)

## 🎯 Persistent Volume Concepts

### What are Persistent Volumes?
- Cluster-level storage resources
- Independent of pod lifecycle
- Provisioned by administrators or dynamically
- Abstraction layer for storage backends

### What are Persistent Volume Claims?
- User requests for storage
- Bind to available Persistent Volumes
- Specify size, access modes, and storage class
- Used by pods to mount storage

## 📊 PV Lifecycle

### Provisioning
- **Static**: Admin pre-creates PVs
- **Dynamic**: Created on-demand via StorageClass

### Binding
- PVC binds to suitable PV
- One-to-one relationship
- Considers size, access modes, storage class

### Using
- Pod mounts PVC as volume
- Storage available to containers

### Reclaiming
- **Retain**: Manual cleanup required
- **Delete**: PV deleted automatically
- **Recycle**: Data scrubbed (deprecated)

## 🔧 Access Modes

### ReadWriteOnce (RWO)
- Mounted read-write by single node
- Most common for block storage
```yaml
accessModes:
- ReadWriteOnce
```

### ReadOnlyMany (ROX)
- Mounted read-only by many nodes
- Useful for shared configuration
```yaml
accessModes:
- ReadOnlyMany
```

### ReadWriteMany (RWX)
- Mounted read-write by many nodes
- Requires shared filesystem (NFS, CephFS)
```yaml
accessModes:
- ReadWriteMany
```

## 📋 Persistent Volume Example

### Basic PV Definition
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

### NFS Persistent Volume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 50Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: nfs-server.example.com
    path: /exported/path
```

## 📝 Persistent Volume Claim Example

### Basic PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

### Dynamic PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: fast-ssd
```

## 🚀 Using PVC in Pods

### Pod with PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-example
```

### Deployment with PVC
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx
        volumeMounts:
        - name: web-storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: web-storage
        persistentVolumeClaim:
          claimName: web-pvc
```

## 🔍 PV/PVC Operations

### Create and Manage PVs
```bash
# Create PV
kubectl apply -f pv.yaml

# List PVs
kubectl get pv
kubectl get persistentvolumes

# Describe PV
kubectl describe pv pv-example

# Delete PV
kubectl delete pv pv-example
```

### Create and Manage PVCs
```bash
# Create PVC
kubectl apply -f pvc.yaml

# List PVCs
kubectl get pvc
kubectl get persistentvolumeclaims

# Describe PVC
kubectl describe pvc pvc-example

# Check PVC status
kubectl get pvc pvc-example -o wide
```

## 📊 PV/PVC Status

### PV Status
- **Available**: Ready for binding
- **Bound**: Bound to PVC
- **Released**: PVC deleted, awaiting reclaim
- **Failed**: Reclamation failed

### PVC Status
- **Pending**: Waiting for suitable PV
- **Bound**: Successfully bound to PV
- **Lost**: PV lost

## 🔧 Storage Classes

### Dynamic Provisioning
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
allowVolumeExpansion: true
reclaimPolicy: Delete
```

### Using StorageClass in PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: fast-ssd
```

## 🚨 Troubleshooting PV/PVC

### Common Issues
```bash
# PVC stuck in Pending
kubectl describe pvc <pvc-name>

# Check available PVs
kubectl get pv

# Check StorageClass
kubectl get storageclass

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Debug Commands
```bash
# Check PVC binding
kubectl get pvc <pvc-name> -o yaml

# Check PV details
kubectl describe pv <pv-name>

# Check pod volume mounts
kubectl describe pod <pod-name>
```

## 📋 Best Practices

### PV Management
- Use appropriate reclaim policies
- Monitor storage usage
- Plan for backup and recovery
- Use storage classes for dynamic provisioning

### PVC Design
- Request appropriate storage size
- Choose correct access modes
- Use meaningful names and labels
- Consider storage class requirements

### Security
- Use appropriate file permissions
- Implement access controls
- Encrypt sensitive data
- Regular security audits

## 🔄 Volume Expansion

### Expandable StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable-storage
provisioner: kubernetes.io/aws-ebs
allowVolumeExpansion: true
```

### Expand PVC
```bash
# Edit PVC to increase size
kubectl edit pvc <pvc-name>

# Check expansion status
kubectl describe pvc <pvc-name>
```