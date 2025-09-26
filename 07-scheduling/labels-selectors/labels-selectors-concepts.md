# Labels and Selectors

## 🎯 Labels Concepts

### What are Labels?
- Key-value pairs attached to objects
- Used for organizing and selecting resources
- Not unique - multiple objects can have same labels
- Used by selectors to identify groups of objects

### Label Syntax
```yaml
metadata:
  labels:
    app: nginx
    version: v1.0
    environment: production
    tier: frontend
```

## 🔧 Working with Labels

### Adding Labels
```bash
# Add label to pod
kubectl label pod nginx app=web

# Add multiple labels
kubectl label pod nginx tier=frontend env=prod

# Add label to all pods
kubectl label pods --all version=v1.0

# Add label from file
kubectl label -f pod.yaml app=web
```

### Viewing Labels
```bash
# Show labels
kubectl get pods --show-labels

# Show specific label columns
kubectl get pods -L app,version

# Describe to see all labels
kubectl describe pod nginx
```

### Updating Labels
```bash
# Update existing label
kubectl label pod nginx app=api --overwrite

# Remove label
kubectl label pod nginx app-

# Update multiple objects
kubectl label pods nginx web app=frontend --overwrite
```

## 🔍 Selectors

### Equality-based Selectors
```bash
# Select by single label
kubectl get pods -l app=nginx

# Select by multiple labels (AND)
kubectl get pods -l app=nginx,version=v1.0

# Not equal
kubectl get pods -l app!=nginx
```

### Set-based Selectors
```bash
# In operator
kubectl get pods -l 'app in (nginx,apache)'

# Not in operator
kubectl get pods -l 'app notin (nginx,apache)'

# Exists operator
kubectl get pods -l app

# Does not exist
kubectl get pods -l '!app'
```

## 📊 Label Selectors in Resources

### ReplicaSet Selector
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - name: nginx
        image: nginx
```

### Service Selector
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
    tier: web
  ports:
  - port: 80
    targetPort: 8080
```

### Deployment Selector
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        version: v1.0
    spec:
      containers:
      - name: web
        image: nginx
```

## 🎯 Node Selectors

### Basic Node Selection
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-selector-pod
spec:
  nodeSelector:
    disktype: ssd
    zone: us-west-1a
  containers:
  - name: nginx
    image: nginx
```

### Label Nodes
```bash
# Add label to node
kubectl label node worker-1 disktype=ssd

# Add zone label
kubectl label node worker-1 zone=us-west-1a

# View node labels
kubectl get nodes --show-labels

# Remove node label
kubectl label node worker-1 disktype-
```

## 🔧 Advanced Selectors

### matchExpressions
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: advanced-deployment
spec:
  selector:
    matchExpressions:
    - key: app
      operator: In
      values: [web, api]
    - key: version
      operator: Exists
    - key: deprecated
      operator: DoesNotExist
  template:
    metadata:
      labels:
        app: web
        version: v1.0
    spec:
      containers:
      - name: app
        image: nginx
```

### Combining matchLabels and matchExpressions
```yaml
spec:
  selector:
    matchLabels:
      app: web
    matchExpressions:
    - key: version
      operator: In
      values: [v1.0, v1.1]
```

## 📋 Best Practices

### Label Naming
- Use consistent naming conventions
- Include organization/team prefixes
- Use meaningful, descriptive names
- Avoid system reserved prefixes

### Common Label Patterns
```yaml
metadata:
  labels:
    # Application identification
    app.kubernetes.io/name: nginx
    app.kubernetes.io/instance: nginx-prod
    app.kubernetes.io/version: "1.20"
    app.kubernetes.io/component: frontend
    app.kubernetes.io/part-of: ecommerce
    app.kubernetes.io/managed-by: helm
    
    # Environment and deployment
    environment: production
    tier: frontend
    release: stable
```

## 🚨 Troubleshooting

### Common Issues
```bash
# No pods selected by service
kubectl get endpoints service-name
kubectl get pods -l app=web --show-labels

# ReplicaSet not managing pods
kubectl describe rs replicaset-name
kubectl get pods -l app=web --show-labels

# Check selector syntax
kubectl get deployment deployment-name -o yaml | grep -A 10 selector
```

### Debugging Selectors
```bash
# Test selector queries
kubectl get pods -l 'app in (web,api)'
kubectl get pods -l app=web,version=v1.0

# Check if labels match selectors
kubectl describe service service-name
kubectl get pods --show-labels
```