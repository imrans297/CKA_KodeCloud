# Pods - The Foundation of Kubernetes

## 🎯 Learning Objectives
- Understand what pods are and why they exist
- Create pods using YAML and kubectl commands
- Manage pod lifecycle
- Troubleshoot common pod issues

## 📚 What You'll Learn

### 1. Pod Basics
- Smallest deployable unit in Kubernetes
- Can contain one or more containers
- Shared network and storage
- Ephemeral by nature

### 2. Your Practice Files
Located in `basics/` directory:
- `pod_defination1.yml` - Basic pod
- `pod_defination2.yml` - Pod with labels
- `pod_defination3.yml` - Multi-container pod
- `pod_defination4.yml` - Pod with resources
- `pod_defination5.yml` - Pod with volumes

## 🚀 Essential Commands

```bash
# Create pod from YAML
kubectl apply -f pod_defination1.yml

# Create pod imperatively
kubectl run nginx --image=nginx

# Get pods
kubectl get pods
kubectl get pods -o wide

# Describe pod
kubectl describe pod <pod-name>

# Get pod logs
kubectl logs <pod-name>

# Execute into pod
kubectl exec -it <pod-name> -- /bin/bash

# Delete pod
kubectl delete pod <pod-name>
```

## 🔧 Practice Exercises

1. **Create your first pod** using `pod_defination1.yml`
2. **Inspect the pod** with describe and logs
3. **Modify a pod** by editing the YAML
4. **Troubleshoot** a failing pod

## ➡️ Next Steps
Once comfortable with pods, move to `03-replicasets/` to learn about pod replication.