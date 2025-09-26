# Role-Based Access Control (RBAC)

## 🎯 RBAC Concepts

### What is RBAC?
- Authorization mechanism in Kubernetes
- Controls who can access what resources
- Based on roles and role bindings
- Fine-grained permission control

### RBAC Components
- **Subjects**: Users, groups, service accounts
- **Resources**: Pods, services, deployments, etc.
- **Verbs**: Actions like get, list, create, delete
- **Roles**: Define permissions
- **RoleBindings**: Bind roles to subjects

## 🔧 RBAC Resources

### Role (Namespace-scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### ClusterRole (Cluster-scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

### RoleBinding (Namespace-scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding (Cluster-scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: User
  name: admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## 👤 Subjects

### User
```yaml
subjects:
- kind: User
  name: jane@example.com
  apiGroup: rbac.authorization.k8s.io
```

### Group
```yaml
subjects:
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
```

### Service Account
```yaml
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
```

## 📊 Common Verbs

### Resource Verbs
- **get**: Read a specific resource
- **list**: List resources of a type
- **watch**: Watch for changes
- **create**: Create new resources
- **update**: Update existing resources
- **patch**: Partially update resources
- **delete**: Delete resources
- **deletecollection**: Delete multiple resources

### Special Verbs
- **use**: Use resources (PodSecurityPolicies)
- **bind**: Bind roles to subjects
- **escalate**: Grant more permissions than you have
- **impersonate**: Act as another user

## 🔧 Service Accounts

### Create Service Account
```bash
# Create service account
kubectl create serviceaccount my-sa

# Get service account
kubectl get serviceaccount my-sa

# Describe service account
kubectl describe serviceaccount my-sa
```

### Service Account in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: my-sa
  containers:
  - name: app
    image: nginx
```

## 🚀 RBAC Operations

### Check Permissions
```bash
# Check if you can perform action
kubectl auth can-i create pods

# Check for specific user
kubectl auth can-i create pods --as=jane

# Check for service account
kubectl auth can-i create pods --as=system:serviceaccount:default:my-sa

# Check in specific namespace
kubectl auth can-i create pods --namespace=production
```

### List Permissions
```bash
# List roles
kubectl get roles
kubectl get clusterroles

# List role bindings
kubectl get rolebindings
kubectl get clusterrolebindings

# Describe role
kubectl describe role pod-reader
```

## 📋 Common RBAC Patterns

### Read-Only Access
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: readonly
rules:
- apiGroups: ["", "apps", "extensions"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

### Namespace Admin
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: namespace-admin
rules:
- apiGroups: ["", "apps", "extensions"]
  resources: ["*"]
  verbs: ["*"]
```

### Pod Manager
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-manager
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec"]
  verbs: ["get", "list", "create", "delete", "watch"]
```

## 🔐 Advanced RBAC

### Resource Names
```yaml
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["my-config"]
  verbs: ["get", "update"]
```

### Subresources
```yaml
rules:
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "create"]
```

### Multiple API Groups
```yaml
rules:
- apiGroups: ["", "apps", "extensions"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

## 🚨 RBAC Troubleshooting

### Common Issues
```bash
# Permission denied errors
kubectl auth can-i <verb> <resource> --as=<user>

# Check role bindings
kubectl get rolebindings -o wide
kubectl describe rolebinding <binding-name>

# Check cluster role bindings
kubectl get clusterrolebindings -o wide
```

### Debug Commands
```bash
# Check effective permissions
kubectl auth can-i --list --as=<user>

# Check service account token
kubectl get secret <sa-token-secret> -o yaml

# Verify API groups
kubectl api-resources
```

## 📋 Best Practices

### Security Guidelines
- Use least privilege principle
- Prefer namespace-scoped roles
- Regular permission audits
- Avoid wildcard permissions

### Design Patterns
- Role per function/team
- Service account per application
- Separate read/write permissions
- Use groups for user management

### Monitoring
- Audit RBAC changes
- Monitor permission usage
- Regular access reviews
- Automated compliance checks