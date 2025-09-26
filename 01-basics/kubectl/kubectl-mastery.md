# kubectl Mastery Guide

## 🎯 kubectl Basics

### Configuration:
```bash
# View current config
kubectl config view

# Get current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Set namespace
kubectl config set-context --current --namespace=<namespace>
```

## 🚀 Essential Commands

### Resource Management:
```bash
# Get resources
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl get pods -l app=nginx

# Describe resources
kubectl describe pod nginx
kubectl describe node worker-1

# Create resources
kubectl create -f pod.yaml
kubectl apply -f pod.yaml

# Delete resources
kubectl delete pod nginx
kubectl delete -f pod.yaml
```

### Output Formats:
```bash
# YAML output
kubectl get pod nginx -o yaml

# JSON output
kubectl get pod nginx -o json

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# JSONPath
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

## 🔧 Advanced kubectl

### Imperative Commands:
```bash
# Create pod
kubectl run nginx --image=nginx

# Create deployment
kubectl create deployment nginx --image=nginx --replicas=3

# Expose service
kubectl expose deployment nginx --port=80 --type=NodePort

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Set image
kubectl set image deployment/nginx nginx=nginx:1.21
```

### Dry Run and Generate:
```bash
# Generate YAML without creating
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Generate deployment YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Generate service YAML
kubectl expose deployment nginx --port=80 --dry-run=client -o yaml > service.yaml
```

## 📊 Monitoring and Debugging

### Logs and Events:
```bash
# Pod logs
kubectl logs nginx
kubectl logs nginx -f
kubectl logs nginx --previous

# Events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp

# Describe for debugging
kubectl describe pod nginx
kubectl describe node worker-1
```

### Execute and Port Forward:
```bash
# Execute commands in pod
kubectl exec nginx -- ls /
kubectl exec -it nginx -- /bin/bash

# Port forwarding
kubectl port-forward pod/nginx 8080:80
kubectl port-forward service/nginx 8080:80
```

## 🔍 Resource Discovery

### API Resources:
```bash
# List all resource types
kubectl api-resources

# List API versions
kubectl api-versions

# Explain resource structure
kubectl explain pod
kubectl explain pod.spec
kubectl explain deployment.spec.template
```

### Field Selectors:
```bash
# Select by field
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.namespace=default

# Multiple selectors
kubectl get pods --field-selector status.phase=Running,spec.nodeName=worker-1
```

## 🏷️ Labels and Selectors

### Label Operations:
```bash
# Add label
kubectl label pod nginx app=web

# Remove label
kubectl label pod nginx app-

# Update label
kubectl label pod nginx app=frontend --overwrite

# Select by labels
kubectl get pods -l app=web
kubectl get pods -l 'app in (web,api)'
kubectl get pods -l app!=web
```

## 📝 kubectl Shortcuts

### Resource Abbreviations:
```bash
po = pods
svc = services
deploy = deployments
rs = replicasets
ns = namespaces
pv = persistentvolumes
pvc = persistentvolumeclaims
cm = configmaps
```

### Useful Aliases:
```bash
# Quick commands
kubectl get po
kubectl get svc
kubectl get deploy
kubectl get ns

# With output
kubectl get po -o wide
kubectl get svc -o yaml
```

## 🔧 kubectl Configuration

### Kubeconfig Structure:
```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: <ca-data>
    server: https://kubernetes-api:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
users:
- name: kubernetes-admin
  user:
    client-certificate-data: <cert-data>
    client-key-data: <key-data>
```

### Multiple Clusters:
```bash
# Add cluster
kubectl config set-cluster prod-cluster --server=https://prod-api:6443

# Add user
kubectl config set-credentials prod-user --client-certificate=prod.crt --client-key=prod.key

# Add context
kubectl config set-context prod --cluster=prod-cluster --user=prod-user

# Use context
kubectl config use-context prod
```

## 🚨 Troubleshooting with kubectl

### Debug Commands:
```bash
# Verbose output
kubectl get pods -v=8

# Raw API calls
kubectl get --raw /api/v1/nodes

# Cluster info
kubectl cluster-info
kubectl cluster-info dump

# Component status
kubectl get componentstatuses
```

### Common Issues:
- Connection refused → Check API server
- Forbidden → Check RBAC permissions
- Not found → Check resource name/namespace
- Timeout → Check network connectivity