# ConfigMaps and Secrets

## 🎯 ConfigMaps Concepts

### What are ConfigMaps?
- Store non-confidential configuration data
- Key-value pairs or configuration files
- Decouple configuration from container images
- Can be consumed as environment variables, command-line arguments, or files

### ConfigMap Use Cases
- Application configuration files
- Environment-specific settings
- Command-line arguments
- Configuration templates

## 🔐 Secrets Concepts

### What are Secrets?
- Store sensitive information (passwords, tokens, keys)
- Base64 encoded (not encrypted by default)
- More secure handling than ConfigMaps
- Automatically mounted as tmpfs in containers

### Secret Types
- **Opaque**: Arbitrary user data (default)
- **kubernetes.io/service-account-token**: Service account tokens
- **kubernetes.io/dockercfg**: Docker registry credentials
- **kubernetes.io/tls**: TLS certificates

## 🔧 Creating ConfigMaps

### From Literal Values
```bash
# Single key-value
kubectl create configmap app-config --from-literal=database_url=mysql://localhost:3306/mydb

# Multiple key-values
kubectl create configmap app-config \
  --from-literal=database_url=mysql://localhost:3306/mydb \
  --from-literal=debug_mode=true \
  --from-literal=log_level=info
```

### From Files
```bash
# From single file
kubectl create configmap nginx-config --from-file=nginx.conf

# From directory
kubectl create configmap app-configs --from-file=config/

# From file with custom key
kubectl create configmap app-config --from-file=config=app.properties
```

### Declarative ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "mysql://localhost:3306/mydb"
  debug_mode: "true"
  log_level: "info"
  app.properties: |
    database.host=localhost
    database.port=3306
    database.name=mydb
    cache.enabled=true
```

## 🔐 Creating Secrets

### From Literal Values
```bash
# Create secret from literals
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpassword
```

### From Files
```bash
# From files
kubectl create secret generic ssl-certs \
  --from-file=tls.crt=server.crt \
  --from-file=tls.key=server.key
```

### Docker Registry Secret
```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com
```

### Declarative Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded 'admin'
  password: c2VjcmV0cGFzc3dvcmQ=  # base64 encoded 'secretpassword'
```

## 📊 Using ConfigMaps in Pods

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
    - name: DEBUG_MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: debug_mode
```

### All Keys as Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-envfrom-pod
spec:
  containers:
  - name: app
    image: nginx
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
```

### Specific Keys as Files
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-selective-pod
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
      - key: log_level
        path: log-level.txt
```

## 🔐 Using Secrets in Pods

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
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
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
      secretName: db-secret
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
    image: private-registry.example.com/myapp:latest
  imagePullSecrets:
  - name: regcred
```

## 🔄 ConfigMap and Secret Operations

### Management Commands
```bash
# List ConfigMaps
kubectl get configmaps
kubectl get cm

# Describe ConfigMap
kubectl describe configmap app-config

# Edit ConfigMap
kubectl edit configmap app-config

# Delete ConfigMap
kubectl delete configmap app-config

# List Secrets
kubectl get secrets

# Describe Secret (values hidden)
kubectl describe secret db-secret

# View Secret values
kubectl get secret db-secret -o yaml
```

### Update and Reload
```bash
# Update ConfigMap
kubectl create configmap app-config --from-literal=new_key=new_value --dry-run=client -o yaml | kubectl apply -f -

# Restart deployment to pick up changes
kubectl rollout restart deployment/myapp
```

## 🚨 Troubleshooting

### Common Issues
```bash
# ConfigMap not found
kubectl describe pod <pod-name>

# Check if ConfigMap exists
kubectl get configmap <configmap-name>

# Check ConfigMap data
kubectl get configmap <configmap-name> -o yaml

# Secret decoding
kubectl get secret <secret-name> -o jsonpath='{.data.password}' | base64 -d
```

### Debug Commands
```bash
# Check environment variables in pod
kubectl exec <pod-name> -- env

# Check mounted files
kubectl exec <pod-name> -- ls -la /etc/config

# Check file contents
kubectl exec <pod-name> -- cat /etc/config/app.properties
```

## 📋 Best Practices

### ConfigMap Best Practices
- Use meaningful names and labels
- Keep configuration files small
- Version your configurations
- Use namespaces for isolation
- Document configuration keys

### Secret Best Practices
- Never commit secrets to version control
- Use external secret management systems
- Rotate secrets regularly
- Limit secret access with RBAC
- Enable encryption at rest

### Security Considerations
- Secrets are base64 encoded, not encrypted
- Use external secret managers (Vault, AWS Secrets Manager)
- Enable etcd encryption
- Use service accounts with minimal permissions
- Audit secret access

## 🔄 Advanced Patterns

### Immutable ConfigMaps/Secrets
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: immutable-config
data:
  config.yaml: |
    app:
      name: myapp
      version: v1.0
immutable: true
```

### ConfigMap with Binary Data
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: binary-config
binaryData:
  image.png: <base64-encoded-binary-data>
data:
  config.txt: "text configuration"
```