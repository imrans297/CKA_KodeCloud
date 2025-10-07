# Container Storage Interface (CSI)

## 🎯 Overview

Container Storage Interface (CSI) is a standard for exposing arbitrary block and file storage systems to containerized workloads on Container Orchestration Systems (COS) like Kubernetes.

## 🏗️ CSI Architecture

### CSI Components
```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                       │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   kubelet       │    │ Controller      │                │
│  │                 │    │ Manager         │                │
│  │ ┌─────────────┐ │    │                 │                │
│  │ │Volume Plugin│ │    │ ┌─────────────┐ │                │
│  │ │Manager      │ │    │ │External     │ │                │
│  │ └─────────────┘ │    │ │Provisioner  │ │                │
│  └─────────────────┘    │ └─────────────┘ │                │
│           │              └─────────────────┘                │
│           │                       │                        │
└───────────┼───────────────────────┼────────────────────────┘
            │                       │
            ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Node Plugin   │    │Controller Plugin│    │  Storage System │
│                 │    │                 │    │                 │
│ - NodeStage     │    │ - CreateVolume  │    │ - AWS EBS       │
│ - NodePublish   │    │ - DeleteVolume  │    │ - GCE PD        │
│ - NodeUnpublish │    │ - Attach/Detach │    │ - Azure Disk    │
│ - NodeUnstage   │    │ - Snapshot      │    │ - NetApp        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### CSI Specification
- **Identity Service** - Plugin identification and capabilities
- **Controller Service** - Volume lifecycle management
- **Node Service** - Volume mounting and staging

## 🔧 CSI Driver Components

### 1. Identity Service
```protobuf
service Identity {
  rpc GetPluginInfo(GetPluginInfoRequest)
    returns (GetPluginInfoResponse) {}
  
  rpc GetPluginCapabilities(GetPluginCapabilitiesRequest)
    returns (GetPluginCapabilitiesResponse) {}
  
  rpc Probe(ProbeRequest)
    returns (ProbeResponse) {}
}
```

### 2. Controller Service
```protobuf
service Controller {
  rpc CreateVolume(CreateVolumeRequest)
    returns (CreateVolumeResponse) {}
  
  rpc DeleteVolume(DeleteVolumeRequest)
    returns (DeleteVolumeResponse) {}
  
  rpc ControllerPublishVolume(ControllerPublishVolumeRequest)
    returns (ControllerPublishVolumeResponse) {}
  
  rpc ControllerUnpublishVolume(ControllerUnpublishVolumeRequest)
    returns (ControllerUnpublishVolumeResponse) {}
  
  rpc CreateSnapshot(CreateSnapshotRequest)
    returns (CreateSnapshotResponse) {}
}
```

### 3. Node Service
```protobuf
service Node {
  rpc NodeStageVolume(NodeStageVolumeRequest)
    returns (NodeStageVolumeResponse) {}
  
  rpc NodeUnstageVolume(NodeUnstageVolumeRequest)
    returns (NodeUnstageVolumeResponse) {}
  
  rpc NodePublishVolume(NodePublishVolumeRequest)
    returns (NodePublishVolumeResponse) {}
  
  rpc NodeUnpublishVolume(NodeUnpublishVolumeRequest)
    returns (NodeUnpublishVolumeResponse) {}
  
  rpc NodeGetCapabilities(NodeGetCapabilitiesRequest)
    returns (NodeGetCapabilitiesResponse) {}
}
```

## 📋 CSI Volume Lifecycle

### Volume Creation Flow
```
1. CreateVolume (Controller)
   ├── Validate parameters
   ├── Create volume in storage system
   └── Return volume info

2. ControllerPublishVolume (Controller)
   ├── Attach volume to node
   └── Return publish context

3. NodeStageVolume (Node)
   ├── Format volume (if needed)
   ├── Mount to staging path
   └── Prepare for use

4. NodePublishVolume (Node)
   ├── Bind mount from staging
   └── Make available to pod
```

### Volume Deletion Flow
```
1. NodeUnpublishVolume (Node)
   └── Unmount from pod path

2. NodeUnstageVolume (Node)
   └── Unmount from staging path

3. ControllerUnpublishVolume (Controller)
   └── Detach from node

4. DeleteVolume (Controller)
   └── Delete from storage system
```

## 🛠️ CSI Driver Deployment

### CSI Driver DaemonSet (Node Plugin)
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: csi-driver-node
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: csi-driver-node
  template:
    metadata:
      labels:
        app: csi-driver-node
    spec:
      serviceAccountName: csi-driver-node-sa
      hostNetwork: true
      containers:
      - name: csi-driver
        image: my-csi-driver:v1.0
        args:
        - --endpoint=$(CSI_ENDPOINT)
        - --node-id=$(KUBE_NODE_NAME)
        - --v=5
        env:
        - name: CSI_ENDPOINT
          value: unix:///csi/csi.sock
        - name: KUBE_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        volumeMounts:
        - name: socket-dir
          mountPath: /csi
        - name: pods-mount-dir
          mountPath: /var/lib/kubelet/pods
          mountPropagation: Bidirectional
        - name: device-dir
          mountPath: /dev
        securityContext:
          privileged: true
      
      - name: node-driver-registrar
        image: k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.5.0
        args:
        - --csi-address=$(ADDRESS)
        - --kubelet-registration-path=$(DRIVER_REG_SOCK_PATH)
        - --v=5
        env:
        - name: ADDRESS
          value: /csi/csi.sock
        - name: DRIVER_REG_SOCK_PATH
          value: /var/lib/kubelet/plugins/my-csi-driver/csi.sock
        volumeMounts:
        - name: socket-dir
          mountPath: /csi
        - name: registration-dir
          mountPath: /registration
      
      volumes:
      - name: socket-dir
        hostPath:
          path: /var/lib/kubelet/plugins/my-csi-driver
          type: DirectoryOrCreate
      - name: registration-dir
        hostPath:
          path: /var/lib/kubelet/plugins_registry
          type: Directory
      - name: pods-mount-dir
        hostPath:
          path: /var/lib/kubelet/pods
          type: Directory
      - name: device-dir
        hostPath:
          path: /dev
          type: Directory
```

### CSI Controller Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: csi-driver-controller
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: csi-driver-controller
  template:
    metadata:
      labels:
        app: csi-driver-controller
    spec:
      serviceAccountName: csi-driver-controller-sa
      containers:
      - name: csi-driver
        image: my-csi-driver:v1.0
        args:
        - --endpoint=$(CSI_ENDPOINT)
        - --controller=true
        - --v=5
        env:
        - name: CSI_ENDPOINT
          value: unix:///csi/csi.sock
        volumeMounts:
        - name: socket-dir
          mountPath: /csi
      
      - name: csi-provisioner
        image: k8s.gcr.io/sig-storage/csi-provisioner:v3.2.0
        args:
        - --csi-address=$(ADDRESS)
        - --v=5
        - --feature-gates=Topology=true
        - --enable-leader-election
        - --leader-election-type=leases
        env:
        - name: ADDRESS
          value: /csi/csi.sock
        volumeMounts:
        - name: socket-dir
          mountPath: /csi
      
      - name: csi-attacher
        image: k8s.gcr.io/sig-storage/csi-attacher:v3.5.0
        args:
        - --csi-address=$(ADDRESS)
        - --v=5
        - --leader-election
        env:
        - name: ADDRESS
          value: /csi/csi.sock
        volumeMounts:
        - name: socket-dir
          mountPath: /csi
      
      - name: csi-snapshotter
        image: k8s.gcr.io/sig-storage/csi-snapshotter:v6.0.0
        args:
        - --csi-address=$(ADDRESS)
        - --v=5
        - --leader-election
        env:
        - name: ADDRESS
          value: /csi/csi.sock
        volumeMounts:
        - name: socket-dir
          mountPath: /csi
      
      volumes:
      - name: socket-dir
        emptyDir: {}
```

### CSI Driver Registration
```yaml
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: my-csi-driver.example.com
spec:
  attachRequired: true
  podInfoOnMount: true
  volumeLifecycleModes:
  - Persistent
  - Ephemeral
  fsGroupPolicy: ReadWriteOnceWithFSType
  tokenRequests:
  - audience: "my-csi-driver"
    expirationSeconds: 3600
```

## 🎯 Real-World CSI Drivers

### AWS EBS CSI Driver
```yaml
# StorageClass for EBS CSI
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
  kmsKeyId: "arn:aws:kms:us-west-2:123456789012:key/12345678-1234-1234-1234-123456789012"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
```

### Azure Disk CSI Driver
```yaml
# StorageClass for Azure Disk CSI
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-disk-premium
provisioner: disk.csi.azure.com
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
  cachingmode: ReadOnly
  diskIopsReadWrite: "500"
  diskMbpsReadWrite: "100"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
```

### Google Cloud Persistent Disk CSI
```yaml
# StorageClass for GCE PD CSI
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gce-pd-ssd
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
  replication-type: regional-pd
  zones: us-central1-a,us-central1-b
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
```

### NetApp Trident CSI Driver
```yaml
# StorageClass for NetApp Trident
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: netapp-nas
provisioner: csi.trident.netapp.io
parameters:
  backendType: "ontap-nas"
  storagePools: "ontapnas_pool"
  fsType: "nfs"
  snapshots: "true"
volumeBindingMode: Immediate
allowVolumeExpansion: true
reclaimPolicy: Delete
```

## 🔧 CSI Sidecar Containers

### External Provisioner
```yaml
- name: csi-provisioner
  image: k8s.gcr.io/sig-storage/csi-provisioner:v3.2.0
  args:
  - --csi-address=$(ADDRESS)
  - --v=5
  - --feature-gates=Topology=true
  - --enable-leader-election
  - --default-fstype=ext4
  - --extra-create-metadata
  env:
  - name: ADDRESS
    value: /csi/csi.sock
```

### External Attacher
```yaml
- name: csi-attacher
  image: k8s.gcr.io/sig-storage/csi-attacher:v3.5.0
  args:
  - --csi-address=$(ADDRESS)
  - --v=5
  - --leader-election
  - --timeout=60s
  env:
  - name: ADDRESS
    value: /csi/csi.sock
```

### External Snapshotter
```yaml
- name: csi-snapshotter
  image: k8s.gcr.io/sig-storage/csi-snapshotter:v6.0.0
  args:
  - --csi-address=$(ADDRESS)
  - --v=5
  - --leader-election
  - --snapshot-name-prefix=snap
  env:
  - name: ADDRESS
    value: /csi/csi.sock
```

### External Resizer
```yaml
- name: csi-resizer
  image: k8s.gcr.io/sig-storage/csi-resizer:v1.5.0
  args:
  - --csi-address=$(ADDRESS)
  - --v=5
  - --leader-election
  env:
  - name: ADDRESS
    value: /csi/csi.sock
```

## 📊 CSI Features and Capabilities

### Volume Capabilities
```yaml
# Raw block volumes
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: raw-block-pvc
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Block  # Raw block device
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast-ssd
```

### Volume Snapshots
```yaml
# VolumeSnapshotClass
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-snapclass
driver: ebs.csi.aws.com
deletionPolicy: Delete
parameters:
  encrypted: "true"
---
# VolumeSnapshot
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  volumeSnapshotClassName: csi-snapclass
  source:
    persistentVolumeClaimName: my-pvc
```

### Volume Cloning
```yaml
# Clone from existing PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloned-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  dataSource:
    kind: PersistentVolumeClaim
    name: source-pvc
  storageClassName: fast-ssd
```

### Ephemeral Volumes
```yaml
# CSI Ephemeral Volume
apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-volume-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: ephemeral-storage
      mountPath: /data
  volumes:
  - name: ephemeral-storage
    csi:
      driver: my-csi-driver.example.com
      volumeAttributes:
        size: "1Gi"
        type: "ephemeral"
```

## 🔍 CSI Monitoring and Observability

### CSI Metrics
```yaml
# ServiceMonitor for CSI metrics
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: csi-driver-metrics
spec:
  selector:
    matchLabels:
      app: csi-driver
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```

### CSI Events and Logs
```bash
# Check CSI driver logs
kubectl logs -n kube-system -l app=csi-driver-controller
kubectl logs -n kube-system -l app=csi-driver-node

# Check CSI events
kubectl get events --field-selector reason=VolumeMount
kubectl get events --field-selector reason=VolumeAttach

# Check volume attachment status
kubectl get volumeattachments
kubectl describe volumeattachment <attachment-name>
```

## 🚨 CSI Troubleshooting

### Common Issues
```bash
# Driver not registered
kubectl get csidriver
kubectl describe csidriver my-csi-driver.example.com

# Volume attachment issues
kubectl get volumeattachments
kubectl describe volumeattachment <name>

# Mount issues
kubectl describe pod <pod-name>
kubectl logs -n kube-system <csi-node-pod>

# Permission issues
kubectl get csistoragecapacities
kubectl describe csistoragecapacity <name>
```

### Debug Commands
```bash
# Check CSI driver pods
kubectl get pods -n kube-system | grep csi

# Verify driver registration
ls /var/lib/kubelet/plugins_registry/

# Check socket files
ls /var/lib/kubelet/plugins/*/

# Test CSI operations
kubectl apply -f test-pvc.yaml
kubectl get pvc,pv
```

## 📋 CSI Best Practices

### Development Guidelines
1. **Implement all required RPCs** correctly
2. **Handle errors gracefully** with proper codes
3. **Use idempotent operations** for reliability
4. **Implement proper logging** and metrics
5. **Follow CSI specification** strictly

### Deployment Best Practices
```yaml
# Use proper resource limits
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 10m
    memory: 64Mi

# Set appropriate security context
securityContext:
  privileged: true  # Only for node plugin
  capabilities:
    add: ["SYS_ADMIN"]

# Use proper volume mounts
volumeMounts:
- name: pods-mount-dir
  mountPath: /var/lib/kubelet/pods
  mountPropagation: Bidirectional
```

### Security Considerations
```yaml
# Use service accounts with minimal permissions
apiVersion: v1
kind: ServiceAccount
metadata:
  name: csi-driver-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: csi-driver-role
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list", "watch", "create", "delete"]
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "update"]
```

This comprehensive CSI guide covers the complete Container Storage Interface specification, implementation patterns, and real-world examples for building and deploying CSI drivers in Kubernetes environments.