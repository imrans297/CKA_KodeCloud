# Managed Kubernetes Cluster Upgrades

## 🎯 Overview

This guide covers upgrade procedures for managed Kubernetes services across major cloud providers with zero-downtime strategies.

### Supported Platforms
- **AWS EKS** (Elastic Kubernetes Service)
- **Azure AKS** (Azure Kubernetes Service)  
- **Google GKE** (Google Kubernetes Engine)

## 🔄 Zero-Downtime Upgrade Strategy

### Core Principles
- **Rolling updates** - Nodes upgraded one by one
- **Pod disruption budgets** - Maintain minimum replicas
- **Health checks** - Validate before proceeding
- **Graceful draining** - Proper workload migration

### Pre-Upgrade Requirements
```yaml
# Pod Disruption Budget Example
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: critical-app
```

## ☁️ AWS EKS Upgrades

### Control Plane Upgrade

#### Check Current Version
```bash
# Get cluster version
aws eks describe-cluster --name my-cluster --query 'cluster.version' --output text

# List available versions
aws eks describe-addon-versions --kubernetes-version 1.27
```

#### Upgrade Control Plane
```bash
# Upgrade cluster control plane (takes 20-30 minutes)
aws eks update-cluster-version \
    --name my-cluster \
    --kubernetes-version 1.28

# Monitor upgrade progress
aws eks describe-update \
    --name my-cluster \
    --update-id <update-id>

# Wait for completion
aws eks wait cluster-active --name my-cluster
```

### Node Group Upgrades

#### Managed Node Groups
```bash
# List node groups
aws eks describe-nodegroup \
    --cluster-name my-cluster \
    --nodegroup-name my-nodegroup

# Update node group (rolling update)
aws eks update-nodegroup-version \
    --cluster-name my-cluster \
    --nodegroup-name my-nodegroup \
    --kubernetes-version 1.28

# Monitor node group update
aws eks describe-nodegroup \
    --cluster-name my-cluster \
    --nodegroup-name my-nodegroup \
    --query 'nodegroup.status'
```

#### Self-Managed Node Groups
```bash
# Update launch template
aws ec2 create-launch-template-version \
    --launch-template-id lt-12345 \
    --version-description "k8s-1.28" \
    --source-version 1 \
    --launch-template-data '{
        "ImageId": "ami-0abcdef1234567890",
        "UserData": "base64-encoded-user-data"
    }'

# Update Auto Scaling Group
aws autoscaling update-auto-scaling-group \
    --auto-scaling-group-name my-asg \
    --launch-template '{
        "LaunchTemplateId": "lt-12345",
        "Version": "$Latest"
    }'

# Perform rolling update
aws autoscaling start-instance-refresh \
    --auto-scaling-group-name my-asg \
    --preferences '{
        "InstanceWarmup": 300,
        "MinHealthyPercentage": 50
    }'
```

### Add-ons Upgrade
```bash
# List installed add-ons
aws eks list-addons --cluster-name my-cluster

# Update CoreDNS
aws eks update-addon \
    --cluster-name my-cluster \
    --addon-name coredns \
    --addon-version v1.10.1-eksbuild.2

# Update kube-proxy
aws eks update-addon \
    --cluster-name my-cluster \
    --addon-name kube-proxy \
    --addon-version v1.28.1-eksbuild.1

# Update VPC CNI
aws eks update-addon \
    --cluster-name my-cluster \
    --addon-name vpc-cni \
    --addon-version v1.13.4-eksbuild.1
```

### EKS Upgrade Script
```bash
#!/bin/bash
# eks-upgrade.sh

CLUSTER_NAME="my-cluster"
TARGET_VERSION="1.28"
NODEGROUP_NAME="my-nodegroup"

echo "Starting EKS cluster upgrade to $TARGET_VERSION"

# Backup cluster configuration
kubectl get all --all-namespaces -o yaml > backup-$(date +%Y%m%d).yaml

# Upgrade control plane
echo "Upgrading control plane..."
UPDATE_ID=$(aws eks update-cluster-version \
    --name $CLUSTER_NAME \
    --kubernetes-version $TARGET_VERSION \
    --query 'update.id' --output text)

echo "Update ID: $UPDATE_ID"
echo "Waiting for control plane upgrade..."
aws eks wait cluster-active --name $CLUSTER_NAME

# Upgrade node groups
echo "Upgrading node groups..."
aws eks update-nodegroup-version \
    --cluster-name $CLUSTER_NAME \
    --nodegroup-name $NODEGROUP_NAME \
    --kubernetes-version $TARGET_VERSION

# Wait for node group update
while true; do
    STATUS=$(aws eks describe-nodegroup \
        --cluster-name $CLUSTER_NAME \
        --nodegroup-name $NODEGROUP_NAME \
        --query 'nodegroup.status' --output text)
    
    if [ "$STATUS" = "ACTIVE" ]; then
        echo "Node group upgrade completed"
        break
    fi
    
    echo "Node group status: $STATUS. Waiting..."
    sleep 30
done

echo "EKS upgrade completed successfully"
```

## 🔵 Azure AKS Upgrades

### Control Plane Upgrade

#### Check Available Versions
```bash
# Get current version
az aks show --resource-group myResourceGroup --name myAKSCluster --query kubernetesVersion

# List available upgrades
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster --output table
```

#### Upgrade Control Plane
```bash
# Upgrade control plane only
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version 1.28.0 \
    --control-plane-only

# Monitor upgrade progress
az aks show --resource-group myResourceGroup --name myAKSCluster --query provisioningState
```

### Node Pool Upgrades

#### List Node Pools
```bash
# List all node pools
az aks nodepool list --resource-group myResourceGroup --cluster-name myAKSCluster --output table
```

#### Upgrade Node Pools
```bash
# Upgrade specific node pool
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name nodepool1 \
    --kubernetes-version 1.28.0

# Upgrade with surge settings for zero downtime
az aks nodepool update \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name nodepool1 \
    --max-surge 33%
```

#### Blue-Green Node Pool Strategy
```bash
# Create new node pool with updated version
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name nodepool-new \
    --kubernetes-version 1.28.0 \
    --node-count 3 \
    --max-pods 30

# Cordon old nodes
kubectl get nodes -l agentpool=nodepool1 -o name | xargs kubectl cordon

# Drain old nodes
kubectl get nodes -l agentpool=nodepool1 -o name | xargs -I {} kubectl drain {} --ignore-daemonsets --delete-emptydir-data

# Delete old node pool
az aks nodepool delete \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name nodepool1
```

### AKS Upgrade Script
```bash
#!/bin/bash
# aks-upgrade.sh

RESOURCE_GROUP="myResourceGroup"
CLUSTER_NAME="myAKSCluster"
TARGET_VERSION="1.28.0"

echo "Starting AKS cluster upgrade to $TARGET_VERSION"

# Check current version
CURRENT_VERSION=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query kubernetesVersion -o tsv)
echo "Current version: $CURRENT_VERSION"

# Backup cluster resources
kubectl get all --all-namespaces -o yaml > aks-backup-$(date +%Y%m%d).yaml

# Upgrade control plane
echo "Upgrading control plane..."
az aks upgrade \
    --resource-group $RESOURCE_GROUP \
    --name $CLUSTER_NAME \
    --kubernetes-version $TARGET_VERSION \
    --control-plane-only \
    --yes

# Wait for control plane upgrade
while true; do
    STATE=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query provisioningState -o tsv)
    if [ "$STATE" = "Succeeded" ]; then
        echo "Control plane upgrade completed"
        break
    fi
    echo "Control plane state: $STATE. Waiting..."
    sleep 30
done

# Upgrade node pools
NODE_POOLS=$(az aks nodepool list --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --query '[].name' -o tsv)

for pool in $NODE_POOLS; do
    echo "Upgrading node pool: $pool"
    az aks nodepool upgrade \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name $pool \
        --kubernetes-version $TARGET_VERSION \
        --yes
done

echo "AKS upgrade completed successfully"
```

## 🟢 Google GKE Upgrades

### Control Plane Upgrade

#### Check Available Versions
```bash
# Get current version
gcloud container clusters describe my-cluster --zone us-central1-a --format="value(currentMasterVersion)"

# List available versions
gcloud container get-server-config --zone us-central1-a
```

#### Upgrade Control Plane
```bash
# Upgrade control plane
gcloud container clusters upgrade my-cluster \
    --master \
    --cluster-version 1.28.0 \
    --zone us-central1-a

# For regional clusters
gcloud container clusters upgrade my-cluster \
    --master \
    --cluster-version 1.28.0 \
    --region us-central1
```

### Node Pool Upgrades

#### List Node Pools
```bash
# List node pools
gcloud container node-pools list --cluster my-cluster --zone us-central1-a
```

#### Upgrade Node Pools
```bash
# Upgrade specific node pool
gcloud container clusters upgrade my-cluster \
    --node-pool default-pool \
    --cluster-version 1.28.0 \
    --zone us-central1-a

# Upgrade with surge settings
gcloud container node-pools update default-pool \
    --cluster my-cluster \
    --zone us-central1-a \
    --max-surge-upgrade 1 \
    --max-unavailable-upgrade 0
```

#### Blue-Green Node Pool Strategy
```bash
# Create new node pool
gcloud container node-pools create new-pool \
    --cluster my-cluster \
    --zone us-central1-a \
    --node-version 1.28.0 \
    --num-nodes 3 \
    --machine-type n1-standard-2

# Cordon old nodes
kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o name | xargs kubectl cordon

# Drain old nodes
kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o name | \
    xargs -I {} kubectl drain {} --ignore-daemonsets --delete-emptydir-data --force

# Delete old node pool
gcloud container node-pools delete default-pool \
    --cluster my-cluster \
    --zone us-central1-a
```

### GKE Upgrade Script
```bash
#!/bin/bash
# gke-upgrade.sh

CLUSTER_NAME="my-cluster"
ZONE="us-central1-a"
TARGET_VERSION="1.28.0"

echo "Starting GKE cluster upgrade to $TARGET_VERSION"

# Get current version
CURRENT_VERSION=$(gcloud container clusters describe $CLUSTER_NAME --zone $ZONE --format="value(currentMasterVersion)")
echo "Current version: $CURRENT_VERSION"

# Backup cluster resources
kubectl get all --all-namespaces -o yaml > gke-backup-$(date +%Y%m%d).yaml

# Upgrade control plane
echo "Upgrading control plane..."
gcloud container clusters upgrade $CLUSTER_NAME \
    --master \
    --cluster-version $TARGET_VERSION \
    --zone $ZONE \
    --quiet

# Wait for control plane upgrade
while true; do
    STATUS=$(gcloud container clusters describe $CLUSTER_NAME --zone $ZONE --format="value(status)")
    if [ "$STATUS" = "RUNNING" ]; then
        echo "Control plane upgrade completed"
        break
    fi
    echo "Control plane status: $STATUS. Waiting..."
    sleep 30
done

# Upgrade node pools
NODE_POOLS=$(gcloud container node-pools list --cluster $CLUSTER_NAME --zone $ZONE --format="value(name)")

for pool in $NODE_POOLS; do
    echo "Upgrading node pool: $pool"
    gcloud container clusters upgrade $CLUSTER_NAME \
        --node-pool $pool \
        --cluster-version $TARGET_VERSION \
        --zone $ZONE \
        --quiet
done

echo "GKE upgrade completed successfully"
```

## 🎯 Zero-Downtime Scenarios

### Scenario 1: E-commerce Platform
```yaml
# High availability deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-app
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 2
  selector:
    matchLabels:
      app: ecommerce
  template:
    spec:
      containers:
      - name: app
        image: ecommerce:v1.0
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: ecommerce-pdb
spec:
  minAvailable: 4
  selector:
    matchLabels:
      app: ecommerce
```

### Scenario 2: Database Workload
```yaml
# StatefulSet with careful upgrade
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  replicas: 3
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0
  selector:
    matchLabels:
      app: database
  template:
    spec:
      containers:
      - name: db
        image: postgres:13
        readinessProbe:
          exec:
            command:
            - pg_isready
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: database-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: database
```

## 🔍 Upgrade Validation

### Pre-Upgrade Checks
```bash
# Check cluster health
kubectl get nodes
kubectl get pods --all-namespaces | grep -v Running
kubectl top nodes

# Check critical workloads
kubectl get deployments --all-namespaces
kubectl get statefulsets --all-namespaces
kubectl get daemonsets --all-namespaces

# Verify PDBs
kubectl get pdb --all-namespaces
```

### Post-Upgrade Validation
```bash
# Verify cluster version
kubectl version --short

# Check node versions
kubectl get nodes -o wide

# Test workload functionality
kubectl run test-pod --image=nginx --rm -it --restart=Never -- curl -I http://my-service

# Check system components
kubectl get pods -n kube-system
kubectl get events --sort-by=.metadata.creationTimestamp | tail -20
```

### Automated Health Checks
```bash
#!/bin/bash
# health-check.sh

echo "Running post-upgrade health checks..."

# Check all nodes are ready
NOT_READY=$(kubectl get nodes --no-headers | grep -v Ready | wc -l)
if [ $NOT_READY -gt 0 ]; then
    echo "ERROR: $NOT_READY nodes are not ready"
    exit 1
fi

# Check system pods
SYSTEM_PODS_NOT_RUNNING=$(kubectl get pods -n kube-system --no-headers | grep -v Running | wc -l)
if [ $SYSTEM_PODS_NOT_RUNNING -gt 0 ]; then
    echo "ERROR: $SYSTEM_PODS_NOT_RUNNING system pods are not running"
    exit 1
fi

# Check application pods
APP_PODS_NOT_RUNNING=$(kubectl get pods --all-namespaces --no-headers | grep -v -E "(Running|Completed)" | wc -l)
if [ $APP_PODS_NOT_RUNNING -gt 0 ]; then
    echo "WARNING: $APP_PODS_NOT_RUNNING application pods are not running"
fi

echo "Health checks completed successfully"
```

## 🚨 Rollback Procedures

### AWS EKS Rollback
```bash
# Rollback is not directly supported
# Create new cluster with previous version
aws eks create-cluster \
    --name my-cluster-rollback \
    --version 1.27 \
    --role-arn arn:aws:iam::account:role/eks-service-role

# Restore applications from backup
kubectl apply -f backup-$(date +%Y%m%d).yaml
```

### Azure AKS Rollback
```bash
# AKS doesn't support direct rollback
# Use backup and restore strategy
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster-rollback \
    --kubernetes-version 1.27.0

kubectl apply -f aks-backup-$(date +%Y%m%d).yaml
```

### Google GKE Rollback
```bash
# GKE doesn't support direct rollback
# Create new cluster or node pool with previous version
gcloud container node-pools create rollback-pool \
    --cluster my-cluster \
    --zone us-central1-a \
    --node-version 1.27.0

# Migrate workloads to rollback pool
```

## 📊 Monitoring During Upgrades

### Key Metrics
- Node readiness status
- Pod restart counts  
- API server latency
- Application response times
- Error rates

### Monitoring Commands
```bash
# Watch cluster status
watch kubectl get nodes

# Monitor pod status
kubectl get pods --all-namespaces -w

# Check events
kubectl get events -w --sort-by=.metadata.creationTimestamp

# Monitor resource usage
watch kubectl top nodes
```

## 🛡️ Best Practices

### Planning
- Test in staging environment first
- Schedule during maintenance windows
- Prepare rollback procedures
- Set up monitoring and alerting
- Communicate with stakeholders

### Execution
- Upgrade one minor version at a time
- Always backup before upgrading
- Use Pod Disruption Budgets
- Monitor throughout the process
- Validate each step

### Post-Upgrade
- Run comprehensive health checks
- Monitor applications for issues
- Update documentation
- Review lessons learned
- Plan next upgrade cycle