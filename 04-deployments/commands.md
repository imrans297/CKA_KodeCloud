# Deployment Commands & Operations

## Create Deployments

```bash
# Imperative creation
kubectl create deployment nginx --image=nginx
kubectl create deployment nginx --image=nginx --replicas=3

# Generate YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yml

# From YAML file
kubectl apply -f deployment-examples.yml
kubectl create -f deployment-examples.yml

# With specific options
kubectl create deployment web --image=nginx --port=80
```

## Get Deployment Information

```bash
# List deployments
kubectl get deployments
kubectl get deploy
kubectl get deploy -o wide

# Detailed information
kubectl describe deployment nginx-deployment
kubectl get deployment nginx-deployment -o yaml

# Check rollout status
kubectl rollout status deployment/nginx-deployment
```

## Scale Deployments

```bash
# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Auto-scale (requires metrics server)
kubectl autoscale deployment nginx-deployment --min=2 --max=10 --cpu-percent=80
```

## Update Deployments

```bash
# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Edit deployment
kubectl edit deployment nginx-deployment

# Patch deployment
kubectl patch deployment nginx-deployment -p '{"spec":{"replicas":4}}'

# Replace deployment
kubectl replace -f updated-deployment.yml
```

## Rollout Management

```bash
# Check rollout status
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Pause/Resume rollout
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment
```

## Delete Deployments

```bash
# Delete deployment
kubectl delete deployment nginx-deployment

# Delete from file
kubectl delete -f deployment-examples.yml

# Delete all deployments
kubectl delete deployments --all
```

## Deployment Troubleshooting

```bash
# Check deployment status
kubectl get deployment nginx-deployment
kubectl describe deployment nginx-deployment

# Check ReplicaSets created by deployment
kubectl get rs -l app=nginx

# Check pods
kubectl get pods -l app=nginx

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Debug failed rollout
kubectl rollout status deployment/nginx-deployment
kubectl describe deployment nginx-deployment | grep -A 10 Conditions
```

## Deployment Strategies

### Rolling Update (Default)
- Gradually replaces old pods with new ones
- Zero downtime deployment
- Configurable with `maxUnavailable` and `maxSurge`

### Recreate
- Terminates all old pods before creating new ones
- Causes downtime but ensures no mixed versions

## Key Concepts

- **Revision**: Each deployment update creates a new revision
- **Rollback**: Can revert to any previous revision
- **Strategy**: Controls how updates are performed
- **Selector**: Must match pod template labels