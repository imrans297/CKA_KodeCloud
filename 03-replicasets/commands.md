# ReplicaSet Commands & Operations

## Create ReplicaSets

```bash
# From YAML file
kubectl apply -f replicaset-examples.yml
kubectl create -f replicaset-examples.yml

# Generate ReplicaSet YAML (not directly supported, use deployment)
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

## Get ReplicaSet Information

```bash
# List ReplicaSets
kubectl get replicasets
kubectl get rs
kubectl get rs -o wide

# Detailed information
kubectl describe rs nginx-replicaset
kubectl get rs nginx-replicaset -o yaml
```

## Scale ReplicaSets

```bash
# Scale using kubectl
kubectl scale rs nginx-replicaset --replicas=5
kubectl scale --replicas=3 -f replicaset-examples.yml

# Edit ReplicaSet
kubectl edit rs nginx-replicaset

# Patch ReplicaSet
kubectl patch rs nginx-replicaset -p '{"spec":{"replicas":4}}'
```

## Update ReplicaSets

```bash
# Edit ReplicaSet
kubectl edit rs nginx-replicaset

# Replace ReplicaSet
kubectl replace -f updated-replicaset.yml

# Update image (manual pod deletion required)
kubectl set image rs/nginx-replicaset nginx=nginx:1.21
kubectl delete pods -l app=nginx  # Force pod recreation
```

## Delete ReplicaSets

```bash
# Delete ReplicaSet (keeps pods)
kubectl delete rs nginx-replicaset --cascade=orphan

# Delete ReplicaSet and pods
kubectl delete rs nginx-replicaset

# Delete from file
kubectl delete -f replicaset-examples.yml
```

## ReplicaSet Troubleshooting

```bash
# Check ReplicaSet status
kubectl get rs nginx-replicaset
kubectl describe rs nginx-replicaset

# Check pods managed by ReplicaSet
kubectl get pods -l app=nginx

# Check events
kubectl get events --field-selector involvedObject.kind=ReplicaSet

# Debug scaling issues
kubectl describe rs nginx-replicaset | grep -A 10 Events
```

## Key Concepts

- **Selector**: Must match pod template labels
- **Replicas**: Desired number of pod copies
- **Template**: Pod specification for creating new pods
- **Self-healing**: Automatically replaces failed pods