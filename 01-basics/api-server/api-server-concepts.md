# Kubernetes API Server

## 🎯 What is API Server?
- Central management component of Kubernetes
- REST API gateway for all cluster operations
- Only component that directly communicates with ETCD
- Handles authentication, authorization, and validation

## 🔧 API Server Functions

### Core Responsibilities:
1. **API Gateway** - Exposes Kubernetes API
2. **Authentication** - Verifies user identity
3. **Authorization** - Checks user permissions
4. **Admission Control** - Validates and mutates requests
5. **ETCD Interface** - Stores and retrieves cluster state

### API Server Workflow:
```
Client Request → Authentication → Authorization → Admission Controllers → Validation → ETCD
```

## 🌐 API Groups and Versions

### Core API Group (v1):
- Pods, Services, Nodes, Namespaces
- Path: `/api/v1/`

### Named API Groups:
- **apps/v1** - Deployments, ReplicaSets, DaemonSets
- **networking.k8s.io/v1** - NetworkPolicies, Ingress
- **rbac.authorization.k8s.io/v1** - Roles, RoleBindings
- **storage.k8s.io/v1** - StorageClasses, VolumeAttachments

### API Discovery:
```bash
# List API versions
kubectl api-versions

# List API resources
kubectl api-resources

# Get specific API group info
kubectl explain pod
kubectl explain deployment.apps
```

## 🔐 Authentication Methods

### Certificate-based Authentication:
```bash
# Client certificate authentication
kubectl config set-credentials user1 \
  --client-certificate=user1.crt \
  --client-key=user1.key

# Service account tokens
kubectl create serviceaccount my-sa
kubectl get serviceaccount my-sa -o yaml
```

### Token-based Authentication:
```bash
# Bearer token
curl -k -H "Authorization: Bearer <token>" \
  https://kubernetes-api-server:6443/api/v1/pods
```

## 🛡️ Authorization Modes

### RBAC (Role-Based Access Control):
```yaml
# Role definition
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: jane
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Check Permissions:
```bash
# Check if user can perform action
kubectl auth can-i create pods
kubectl auth can-i create pods --as=jane
kubectl auth can-i create pods --as=system:serviceaccount:default:my-sa
```

## 🔧 API Server Configuration

### Common Flags:
```bash
# /etc/kubernetes/manifests/kube-apiserver.yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.1.10
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```

## 📡 API Server Endpoints

### Health Endpoints:
```bash
# Health check
curl -k https://kubernetes-api-server:6443/healthz

# Ready check
curl -k https://kubernetes-api-server:6443/readyz

# Live check
curl -k https://kubernetes-api-server:6443/livez
```

### Resource Endpoints:
```bash
# List pods
curl -k -H "Authorization: Bearer <token>" \
  https://kubernetes-api-server:6443/api/v1/namespaces/default/pods

# Get specific pod
curl -k -H "Authorization: Bearer <token>" \
  https://kubernetes-api-server:6443/api/v1/namespaces/default/pods/nginx
```

## 🚨 API Server Troubleshooting

### Check API Server Status:
```bash
# Check if API server is running
kubectl get pods -n kube-system | grep apiserver

# Check API server logs
kubectl logs kube-apiserver-master -n kube-system

# Check API server process
ps aux | grep kube-apiserver
```

### Common Issues:
- Certificate problems
- ETCD connectivity
- Authorization failures
- Admission controller issues

### Debug Commands:
```bash
# Test API server connectivity
kubectl cluster-info

# Check API server health
kubectl get --raw /healthz

# Verbose kubectl output
kubectl get pods -v=8
```