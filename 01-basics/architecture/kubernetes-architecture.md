# Kubernetes Architecture

## 🏗️ Cluster Components

### Master Node (Control Plane)
- **API Server** - Central management entity
- **ETCD** - Distributed key-value store
- **Controller Manager** - Maintains desired state
- **Scheduler** - Assigns pods to nodes

### Worker Nodes
- **Kubelet** - Node agent
- **Kube Proxy** - Network proxy
- **Container Runtime** - Docker/containerd/CRI-O

## 📊 Architecture Diagram
```
┌─────────────────────────────────────────────────────────────┐
│                    MASTER NODE                              │
├─────────────┬─────────────┬─────────────┬─────────────────┤
│  API Server │    ETCD     │ Controller  │   Scheduler     │
│             │             │  Manager    │                 │
└─────────────┴─────────────┴─────────────┴─────────────────┘
                            │
                    ┌───────┴───────┐
                    │   kubectl     │
                    └───────────────┘
                            │
┌───────────────────────────┼───────────────────────────────┐
│                  WORKER NODES                             │
├─────────────┬─────────────┼─────────────┬─────────────────┤
│   Kubelet   │ Kube Proxy  │   Pods      │ Container       │
│             │             │             │ Runtime         │
└─────────────┴─────────────┴─────────────┴─────────────────┘
```

## 🔧 Component Functions

### API Server
- REST API for all cluster operations
- Authentication and authorization
- Validates and processes API requests
- Only component that talks to ETCD

### ETCD
- Stores cluster state and configuration
- Distributed and consistent
- Source of truth for cluster
- Backup target for disaster recovery

### Controller Manager
- Node Controller - monitors node health
- Replication Controller - maintains pod replicas
- Service Controller - manages service endpoints
- Deployment Controller - handles deployments

### Scheduler
- Watches for unscheduled pods
- Selects best node for pod placement
- Considers resource requirements and constraints
- Applies scheduling policies

### Kubelet
- Manages pods on worker nodes
- Communicates with API server
- Monitors pod health
- Manages container lifecycle

### Kube Proxy
- Maintains network rules
- Implements service abstraction
- Load balances traffic to pods
- Handles service discovery

## 🌐 Communication Flow
1. **kubectl** → API Server
2. **API Server** → ETCD (store state)
3. **Controller Manager** ← API Server (watch changes)
4. **Scheduler** ← API Server (watch unscheduled pods)
5. **Kubelet** ← API Server (get pod assignments)
6. **Kube Proxy** ← API Server (get service updates)