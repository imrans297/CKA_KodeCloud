# Pod Commands & Operations

## Create Pods

```bash
# Imperative - Quick creation
kubectl run nginx --image=nginx
kubectl run redis --image=redis --dry-run=client -o yaml > redis-pod.yml

# Declarative - From YAML
kubectl apply -f pod-examples.yml
kubectl create -f pod-examples.yml

# With specific options
kubectl run nginx --image=nginx --port=80 --labels="app=web,env=prod"
kubectl run busybox --image=busybox --rm -it --restart=Never -- /bin/sh
```

## Get Pod Information

```bash
# List pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl get pods -l app=web

# Detailed information
kubectl describe pod nginx
kubectl get pod nginx -o yaml
kubectl get pod nginx -o json
```

## Pod Logs & Debugging

```bash
# View logs
kubectl logs nginx
kubectl logs nginx -f                    # Follow logs
kubectl logs nginx --previous           # Previous container logs
kubectl logs multi-container-pod -c web # Specific container

# Execute commands
kubectl exec nginx -- ls /
kubectl exec -it nginx -- /bin/bash
kubectl exec nginx -c web -- env        # Multi-container
```

## Edit & Update Pods

```bash
# Edit running pod
kubectl edit pod nginx

# Update image (creates new pod)
kubectl set image pod/nginx nginx=nginx:1.21

# Replace pod
kubectl replace -f updated-pod.yml --force
```

## Delete Pods

```bash
# Delete specific pod
kubectl delete pod nginx

# Delete from file
kubectl delete -f pod-examples.yml

# Force delete
kubectl delete pod nginx --force --grace-period=0

# Delete all pods
kubectl delete pods --all
```

## Pod Troubleshooting

```bash
# Check pod status
kubectl get pod nginx -o wide
kubectl describe pod nginx

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Debug failing pod
kubectl logs nginx
kubectl exec nginx -- ps aux
kubectl exec nginx -- netstat -tulpn
```