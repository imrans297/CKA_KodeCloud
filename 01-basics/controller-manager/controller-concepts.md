# Kubernetes Controller Manager

## 🎯 What is Controller Manager?
- Runs controller processes that regulate cluster state
- Watches API server for changes
- Takes corrective actions to maintain desired state
- Implements control loops

## 🔧 Built-in Controllers

### Node Controller:
- Monitors node health
- Marks nodes as Ready/NotReady
- Evicts pods from unhealthy nodes

### Replication Controller:
- Ensures desired number of pod replicas
- Creates/deletes pods as needed
- Handles pod failures

### Endpoints Controller:
- Populates service endpoints
- Watches services and pods
- Updates endpoint objects

### Service Account Controller:
- Creates default service accounts
- Manages service account tokens

### Deployment Controller:
- Manages ReplicaSets for deployments
- Handles rolling updates
- Manages deployment history

## 📊 Control Loop Pattern

```
1. Watch desired state (API server)
2. Observe current state (cluster)
3. Take action if difference exists
4. Repeat
```

## 🔧 Configuration

```bash
# /etc/kubernetes/manifests/kube-controller-manager.yaml
spec:
  containers:
  - command:
    - kube-controller-manager
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --service-cluster-ip-range=10.96.0.0/12
```

## 🚨 Troubleshooting

```bash
# Check controller manager
kubectl get pods -n kube-system | grep controller-manager
kubectl logs kube-controller-manager-master -n kube-system

# Debug ReplicaSet issues
kubectl describe rs <replicaset-name>

# Check service endpoints
kubectl get endpoints <service-name>
```