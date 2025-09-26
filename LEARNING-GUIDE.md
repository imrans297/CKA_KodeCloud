# Complete CKA Learning Guide

## 📚 What's Included in Each Directory

### 02-pods/
- **pod-examples.yml** - Basic, multi-container, environment variable examples
- **commands.md** - Complete pod operations, debugging, troubleshooting
- **basics/** - Your original pod definition files

### 03-replicasets/
- **replicaset-examples.yml** - Basic ReplicaSet, resource limits
- **commands.md** - Scaling, updating, troubleshooting ReplicaSets
- **basics/** - Your original replicaset files

### 04-deployments/
- **deployment-examples.yml** - Rolling updates, recreate strategy, health checks
- **commands.md** - Rollout management, scaling, rollback procedures
- **basics/** - Your original deployment files

### 05-services/
- **service-examples.yml** - ClusterIP, NodePort, LoadBalancer, Headless services
- **commands.md** - Service discovery, testing, troubleshooting
- **clusterip/** - Your original service files

### 06-namespaces/
- **namespace-examples.yml** - Namespaces with quotas, limits, network policies
- Resource management and isolation

### 07-scheduling/
- **scheduling-examples.yml** - Manual scheduling, node affinity, taints/tolerations
- Pod placement strategies

### 08-storage/
- **storage-examples.yml** - All volume types, PV/PVC, ConfigMaps, Secrets
- Data persistence patterns

## 🎯 How to Use This Structure

### Daily Study Routine:
1. **Read the README.md** in each directory
2. **Study the examples** in YAML files
3. **Practice the commands** from commands.md
4. **Apply the manifests** to your cluster
5. **Experiment** with modifications

### Practice Workflow:
```bash
# Navigate to topic
cd 02-pods/

# Study examples
cat pod-examples.yml

# Apply examples
kubectl apply -f pod-examples.yml

# Practice commands
# (Copy commands from commands.md)

# Clean up
kubectl delete -f pod-examples.yml
```

## 📖 Key Files Added:

- **Manifest Examples**: Complete YAML examples for each concept
- **Command References**: Comprehensive kubectl commands
- **Troubleshooting Guides**: Debug procedures for each resource type
- **Best Practices**: Production-ready configurations

## 🚀 Exam Preparation:

Each directory now contains:
- ✅ **Theory** - Concept explanations
- ✅ **Practice** - YAML manifests
- ✅ **Commands** - kubectl operations
- ✅ **Troubleshooting** - Debug procedures

Start with `01-basics/` and progress through each numbered directory! 🎉