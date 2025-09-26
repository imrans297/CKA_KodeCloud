# Kubernetes Basics - Start Here!

## 🎯 Learning Objectives
- Understand Kubernetes architecture
- Learn about cluster components
- Master kubectl basics
- Understand ETCD's role

## 📚 Architecture Overview

### Master Node Components
- **API Server** - Central management hub
- **ETCD** - Cluster data store  
- **Controller Manager** - Maintains desired state
- **Scheduler** - Assigns pods to nodes

### Worker Node Components
- **Kubelet** - Node agent
- **Kube Proxy** - Network proxy
- **Container Runtime** - Runs containers

## 🚀 Essential kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes
kubectl get all

# Help and documentation
kubectl --help
kubectl explain pod

# Context and namespace
kubectl config current-context
kubectl config get-contexts
kubectl config use-context <context-name>

# Quick resource creation
kubectl run nginx --image=nginx
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
```

## 📖 Study Materials
- Review your course PDFs in `docs/` folder
- Practice with `scripts/commands.md`

## ➡️ Next Steps
Once you understand the architecture, move to `02-pods/` to start with practical exercises.