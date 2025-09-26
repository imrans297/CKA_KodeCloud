# Configuration Management

## 🎯 Configuration Management Concepts

### What is Configuration Management?
- **Externalize configuration** from application code
- **Environment-specific settings** without rebuilding images
- **Secure handling** of sensitive information
- **Dynamic configuration updates** without restarts

### Configuration Types
- **Application Settings** - Database URLs, API keys
- **Environment Variables** - Runtime configuration
- **Configuration Files** - Complex configurations
- **Secrets** - Passwords, certificates, tokens

## 📊 ConfigMaps vs Secrets

### ConfigMaps
- **Non-sensitive data** - configuration files, environment variables
- **Plain text storage** - visible in etcd
- **Easy to manage** - kubectl commands and YAML
- **Automatic updates** - pods can detect changes

### Secrets
- **Sensitive data** - passwords, tokens, certificates
- **Base64 encoded** - not encrypted by default
- **Secure handling** - mounted as tmpfs
- **Access control** - RBAC integration

## 🔧 ConfigMap Usage Patterns

### Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    envFrom:
    - configMapRef:
        name: app-config
```

### Volume Mounts
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
      - key: app.properties
        path: application.properties
```

### Command Line Arguments
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-args-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      echo "Database URL: $(DATABASE_URL)"
      echo "Debug Mode: $(DEBUG_MODE)"
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
```

## 🔐 Secret Usage Patterns

### Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: api-secret
          key: key
```

### Volume Mounts
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secrets
      defaultMode: 0400
```

### Image Pull Secrets
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-image-pod
spec:
  containers:
  - name: app
    image: private-registry.com/myapp:latest
  imagePullSecrets:
  - name: registry-credentials
```

## 🔄 Configuration Updates

### ConfigMap Updates
```bash
# Update ConfigMap
kubectl create configmap app-config --from-literal=key=new-value --dry-run=client -o yaml | kubectl apply -f -

# Update from file
kubectl create configmap app-config --from-file=config.properties --dry-run=client -o yaml | kubectl apply -f -

# Patch ConfigMap
kubectl patch configmap app-config -p '{"data":{"key":"new-value"}}'
```

### Secret Updates
```bash
# Update Secret
kubectl create secret generic app-secret --from-literal=password=newpassword --dry-run=client -o yaml | kubectl apply -f -

# Update TLS secret
kubectl create secret tls tls-secret --cert=new-cert.pem --key=new-key.pem --dry-run=client -o yaml | kubectl apply -f -
```

### Automatic Reload
```yaml
# Using reloader annotation
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  annotations:
    reloader.stakater.com/auto: "true"
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx
        envFrom:
        - configMapRef:
            name: app-config
```

## 📊 Configuration Validation

### Validation Strategies
```yaml
# Init container for validation
apiVersion: v1
kind: Pod
metadata:
  name: config-validation-pod
spec:
  initContainers:
  - name: validate-config
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      if [ -z "$DATABASE_URL" ]; then
        echo "ERROR: DATABASE_URL not set"
        exit 1
      fi
      if [ -z "$API_KEY" ]; then
        echo "ERROR: API_KEY not set"
        exit 1
      fi
      echo "Configuration validation passed"
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secrets
  containers:
  - name: app
    image: nginx
```

### Schema Validation
```yaml
# Using JSON Schema validation
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-schema
data:
  schema.json: |
    {
      "type": "object",
      "properties": {
        "database_url": {"type": "string"},
        "port": {"type": "integer", "minimum": 1, "maximum": 65535},
        "debug": {"type": "boolean"}
      },
      "required": ["database_url", "port"]
    }
```

## 🚨 Configuration Troubleshooting

### Common Issues
```bash
# ConfigMap not found
kubectl describe pod <pod-name>
kubectl get configmap <configmap-name>

# Secret not found
kubectl get secret <secret-name>
kubectl describe secret <secret-name>

# Permission issues
kubectl auth can-i get configmaps
kubectl auth can-i get secrets
```

### Debug Commands
```bash
# Check ConfigMap data
kubectl get configmap <name> -o yaml

# Check Secret data (base64 encoded)
kubectl get secret <name> -o yaml

# Decode Secret values
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 -d

# Check environment variables in pod
kubectl exec <pod-name> -- env

# Check mounted files
kubectl exec <pod-name> -- ls -la /etc/config
kubectl exec <pod-name> -- cat /etc/config/app.properties
```

## 📋 Best Practices

### ConfigMap Best Practices
- Use meaningful names and labels
- Keep configuration files small
- Version your configurations
- Use namespaces for isolation
- Document configuration keys
- Validate configuration format

### Secret Best Practices
- Never commit secrets to version control
- Use external secret management systems
- Rotate secrets regularly
- Limit secret access with RBAC
- Enable encryption at rest
- Use service accounts with minimal permissions

### Security Considerations
- Secrets are base64 encoded, not encrypted
- Use external secret managers (Vault, AWS Secrets Manager)
- Enable etcd encryption
- Audit secret access
- Use network policies to limit access

### Configuration Management
- Separate configuration by environment
- Use consistent naming conventions
- Implement configuration validation
- Monitor configuration changes
- Plan for configuration rollbacks