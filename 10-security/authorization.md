# Kubernetes Authorization - Complete Guide

## 🎯 What is Authorization?

**Authorization** determines what actions an authenticated user can perform on Kubernetes resources. It answers the question: "What can this user do?"

### Authentication vs Authorization
- **Authentication** - "Who are you?" (Identity verification)
- **Authorization** - "What can you do?" (Permission verification)

```
User Request → Authentication → Authorization → Admission Control → API Server
```

## 🔐 Authorization Flow

### Request Attributes
Every API request has these attributes checked:
- **User** - Username or service account
- **Group** - User groups
- **Resource** - pods, services, deployments, etc.
- **Namespace** - Target namespace
- **Verb** - get, list, create, update, delete, etc.
- **API Group** - core, apps, extensions, etc.
- **Resource Name** - Specific resource instance

### Authorization Decision
```
Request → Authorization Module → ALLOW/DENY
```

## 🛡️ Authorization Modes

Kubernetes supports multiple authorization modes (processed in order):

### 1. Node Authorization
**Purpose**: Authorizes kubelet requests
**Scope**: Node-specific operations only

```yaml
# Enabled by default for kubelet
--authorization-mode=Node,RBAC
```

**What Node authorizer allows**:
- Read services, endpoints, nodes
- Write node status, events
- Read/write pods bound to the node

### 2. ABAC (Attribute-Based Access Control)
**Purpose**: Policy-based authorization using JSON files
**Status**: Legacy, not recommended

```json
# /etc/kubernetes/abac-policy.json
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec": {
    "user": "alice",
    "namespace": "development",
    "resource": "pods",
    "apiGroup": "",
    "verb": "get"
  }
}
```

```bash
# Enable ABAC
--authorization-mode=ABAC
--authorization-policy-file=/etc/kubernetes/abac-policy.json
```

### 3. RBAC (Role-Based Access Control) ⭐
**Purpose**: Most common and recommended authorization mode
**Scope**: Fine-grained permissions using roles and bindings

```bash
# Enable RBAC (default in most clusters)
--authorization-mode=RBAC
```

### 4. Webhook Authorization
**Purpose**: External authorization service
**Use Case**: Integration with external systems

```yaml
# Webhook config
apiVersion: v1
kind: Config
clusters:
- name: webhook
  cluster:
    server: https://authz.example.com/authorize
users:
- name: webhook
contexts:
- context:
    cluster: webhook
    user: webhook
  name: webhook
current-context: webhook
```

```bash
# Enable webhook
--authorization-mode=Webhook
--authorization-webhook-config-file=/etc/kubernetes/webhook-config.yaml
```

### 5. AlwaysAllow
**Purpose**: Allows all requests (DANGEROUS)
**Use Case**: Testing only

```bash
--authorization-mode=AlwaysAllow
```

### 6. AlwaysDeny
**Purpose**: Denies all requests
**Use Case**: Emergency lockdown

```bash
--authorization-mode=AlwaysDeny
```

## 🎭 RBAC Deep Dive

### RBAC Components

#### 1. Role (Namespace-scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
```

#### 2. ClusterRole (Cluster-wide)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
```

#### 3. RoleBinding (Namespace-scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: pod-reader
  namespace: development
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### 4. ClusterRoleBinding (Cluster-wide)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: User
  name: admin
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: system:masters
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## 🔧 RBAC Commands

### View RBAC Resources
```bash
# List roles
kubectl get roles --all-namespaces
kubectl get clusterroles

# List bindings
kubectl get rolebindings --all-namespaces
kubectl get clusterrolebindings

# Describe specific role
kubectl describe role pod-reader -n development
kubectl describe clusterrole cluster-admin
```

### Create RBAC Resources
```bash
# Create role
kubectl create role pod-reader \
  --verb=get,list,watch \
  --resource=pods \
  --namespace=development

# Create cluster role
kubectl create clusterrole node-reader \
  --verb=get,list,watch \
  --resource=nodes

# Create role binding
kubectl create rolebinding read-pods \
  --role=pod-reader \
  --user=jane \
  --namespace=development

# Create cluster role binding
kubectl create clusterrolebinding read-nodes \
  --clusterrole=node-reader \
  --user=admin
```

### Check Permissions
```bash
# Check if current user can perform action
kubectl auth can-i create pods
kubectl auth can-i get pods --namespace=development

# Check permissions for specific user
kubectl auth can-i create pods --as=jane
kubectl auth can-i get nodes --as=system:serviceaccount:default:my-sa

# Check permissions in specific namespace
kubectl auth can-i create deployments --namespace=production --as=developer

# List all permissions for user
kubectl auth can-i --list --as=jane
kubectl auth can-i --list --as=jane --namespace=development
```

## 🎯 Real-World RBAC Examples

### Example 1: Developer Role
```yaml
# Developer role - can manage apps but not cluster resources
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: development
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

### Example 2: Read-Only User
```yaml
# Read-only access to specific namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: readonly
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps", "extensions"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: readonly-binding
  namespace: production
subjects:
- kind: User
  name: auditor
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: readonly
  apiGroup: rbac.authorization.k8s.io
```

### Example 3: Service Account for CI/CD
```yaml
# Service account for deployment automation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-deployer
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cicd-deployer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cicd-deployer-binding
subjects:
- kind: ServiceAccount
  name: cicd-deployer
  namespace: default
roleRef:
  kind: ClusterRole
  name: cicd-deployer
  apiGroup: rbac.authorization.k8s.io
```

### Example 4: Namespace Admin
```yaml
# Full admin access to specific namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: team-alpha
  name: namespace-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: namespace-admin-binding
  namespace: team-alpha
subjects:
- kind: User
  name: team-lead
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: namespace-admin
  apiGroup: rbac.authorization.k8s.io
```

## 🔒 Advanced RBAC Patterns

### Resource Names Restriction
```yaml
# Allow access to specific resources only
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: specific-pod-access
rules:
- apiGroups: [""]
  resources: ["pods"]
  resourceNames: ["my-pod", "another-pod"]
  verbs: ["get", "update", "patch"]
```

### Subresources Access
```yaml
# Access to pod logs and exec
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-debugger
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods/log", "pods/exec", "pods/portforward"]
  verbs: ["get", "create"]
```

### API Group Specific Access
```yaml
# Access to specific API groups
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: metrics-reader
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
- apiGroups: ["custom.metrics.k8s.io"]
  resources: ["*"]
  verbs: ["get", "list"]
```

## 🛠️ Built-in ClusterRoles

### System Roles
```bash
# View built-in cluster roles
kubectl get clusterroles | grep system:

# Important system roles:
# system:admin - Full cluster access
# system:basic-user - Basic authenticated user access
# system:discovery - Discovery API access
# system:public-info-viewer - Public cluster info access
```

### Default Roles
```bash
# Common default roles
kubectl describe clusterrole cluster-admin
kubectl describe clusterrole admin
kubectl describe clusterrole edit
kubectl describe clusterrole view
```

#### cluster-admin (Super User)
```yaml
# Full cluster access
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
- nonResourceURLs: ["*"]
  verbs: ["*"]
```

#### admin (Namespace Admin)
```yaml
# Full namespace access + some cluster resources
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps", "extensions"]
  resources: ["*"]
  verbs: ["*"]
# ... (read access to some cluster resources)
```

#### edit (Editor)
```yaml
# Read/write access to most resources
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
# ... (no access to roles/rolebindings)
```

#### view (Read-Only)
```yaml
# Read-only access to most resources
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
# ... (no secrets access)
```

## 🔍 Authorization Troubleshooting

### Common Issues

#### 1. Permission Denied
```bash
# Check user permissions
kubectl auth can-i create pods --as=user

# Check role bindings
kubectl get rolebindings,clusterrolebindings --all-namespaces | grep user

# Describe role to see permissions
kubectl describe role role-name -n namespace
```

#### 2. Service Account Issues
```bash
# Check service account
kubectl get serviceaccount sa-name -n namespace

# Check service account token
kubectl describe secret $(kubectl get serviceaccount sa-name -n namespace -o jsonpath='{.secrets[0].name}') -n namespace

# Check service account permissions
kubectl auth can-i create pods --as=system:serviceaccount:namespace:sa-name
```

#### 3. RBAC Not Working
```bash
# Check if RBAC is enabled
kubectl api-versions | grep rbac

# Check API server configuration
ps aux | grep kube-apiserver | grep authorization-mode

# Check for conflicting policies
kubectl get clusterrolebindings | grep user
```

### Debug Commands
```bash
# Verbose kubectl output
kubectl create deployment test --image=nginx --dry-run=client -o yaml -v=8

# Check API server logs
journalctl -u kube-apiserver -f

# Test with different users
kubectl auth can-i create pods --as=system:serviceaccount:default:default
```

## 📋 RBAC Best Practices

### Security Principles
1. **Principle of Least Privilege** - Grant minimum required permissions
2. **Separation of Duties** - Different roles for different responsibilities
3. **Regular Audits** - Review and cleanup unused permissions
4. **Use Groups** - Manage users through groups, not individually

### Implementation Guidelines
```yaml
# Good: Specific permissions
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# Bad: Wildcard permissions
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

### Namespace Strategy
```bash
# Create namespace-specific roles
kubectl create role developer --verb=get,list,create,update,delete --resource=pods,services --namespace=development

# Avoid cluster-wide permissions when possible
# Use ClusterRole only when truly needed across namespaces
```

## 🚀 Automation Scripts

### RBAC Audit Script
```bash
#!/bin/bash
# rbac-audit.sh

echo "=== RBAC Audit Report ==="
echo "Date: $(date)"
echo

echo "=== Cluster Roles ==="
kubectl get clusterroles --no-headers | wc -l
echo "Total ClusterRoles: $(kubectl get clusterroles --no-headers | wc -l)"

echo
echo "=== Cluster Role Bindings ==="
kubectl get clusterrolebindings --no-headers | wc -l
echo "Total ClusterRoleBindings: $(kubectl get clusterrolebindings --no-headers | wc -l)"

echo
echo "=== Users with cluster-admin ==="
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .subjects[]? | select(.kind=="User") | .name'

echo
echo "=== Service Accounts with cluster-admin ==="
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .subjects[]? | select(.kind=="ServiceAccount") | "\(.namespace)/\(.name)"'

echo
echo "=== Roles per Namespace ==="
kubectl get roles --all-namespaces --no-headers | awk '{print $1}' | sort | uniq -c
```

### User Permission Check Script
```bash
#!/bin/bash
# check-user-permissions.sh

USER=$1
NAMESPACE=${2:-default}

if [ -z "$USER" ]; then
    echo "Usage: $0 <username> [namespace]"
    exit 1
fi

echo "Checking permissions for user: $USER in namespace: $NAMESPACE"
echo

# Common resources to check
RESOURCES=("pods" "services" "deployments" "configmaps" "secrets" "ingresses")
VERBS=("get" "list" "create" "update" "delete")

for resource in "${RESOURCES[@]}"; do
    echo "=== $resource ==="
    for verb in "${VERBS[@]}"; do
        if kubectl auth can-i $verb $resource --as=$USER --namespace=$NAMESPACE >/dev/null 2>&1; then
            echo "  ✓ $verb"
        else
            echo "  ✗ $verb"
        fi
    done
    echo
done
```

### Role Template Generator
```bash
#!/bin/bash
# generate-role.sh

ROLE_NAME=$1
NAMESPACE=$2
RESOURCES=$3
VERBS=$4

cat <<EOF > ${ROLE_NAME}-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: $NAMESPACE
  name: $ROLE_NAME
rules:
- apiGroups: [""]
  resources: [$(echo $RESOURCES | sed 's/,/", "/g' | sed 's/^/"/' | sed 's/$/"/')"]
  verbs: [$(echo $VERBS | sed 's/,/", "/g' | sed 's/^/"/' | sed 's/$/"/')"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ${ROLE_NAME}-binding
  namespace: $NAMESPACE
subjects:
- kind: User
  name: REPLACE_WITH_USERNAME
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: $ROLE_NAME
  apiGroup: rbac.authorization.k8s.io
EOF

echo "Generated ${ROLE_NAME}-role.yaml"
echo "Remember to replace REPLACE_WITH_USERNAME with actual username"
```

This comprehensive authorization guide covers all aspects of Kubernetes authorization mechanisms, with detailed examples and practical commands for the CKA exam and real-world scenarios.