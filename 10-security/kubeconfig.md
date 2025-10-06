# Kubernetes kubeconfig - Complete Guide

## 🎯 What is kubeconfig?

**kubeconfig** is a YAML configuration file that contains:
- **Cluster information** - API server endpoints, certificates
- **User credentials** - Authentication details (certificates, tokens, passwords)
- **Context definitions** - Which user connects to which cluster
- **Current context** - Active cluster-user combination

Think of it as your "connection profile" to Kubernetes clusters.

## 📁 Default Locations

```bash
# Default kubeconfig location
~/.kube/config

# Environment variable override
export KUBECONFIG=/path/to/custom/config

# Multiple configs (merged)
export KUBECONFIG=~/.kube/config:~/.kube/config-prod:~/.kube/config-dev
```

## 🏗️ kubeconfig Structure

### Complete Structure
```yaml
apiVersion: v1
kind: Config
current-context: dev-context

# Cluster definitions (where to connect)
clusters:
- name: development
  cluster:
    server: https://dev-k8s-api.example.com:6443
    certificate-authority-data: LS0tLS1CRUdJTi... # Base64 encoded CA cert
    # OR
    certificate-authority: /path/to/ca.crt
    insecure-skip-tls-verify: false

- name: production
  cluster:
    server: https://prod-k8s-api.example.com:6443
    certificate-authority-data: LS0tLS1CRUdJTi...

# User definitions (how to authenticate)
users:
- name: dev-user
  user:
    client-certificate-data: LS0tLS1CRUdJTi... # Base64 encoded client cert
    client-key-data: LS0tLS1CRUdJTi...         # Base64 encoded client key
    # OR
    client-certificate: /path/to/client.crt
    client-key: /path/to/client.key

- name: admin-user
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6...      # Bearer token

- name: service-account-user
  user:
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token

# Context definitions (user + cluster combinations)
contexts:
- name: dev-context
  context:
    cluster: development
    user: dev-user
    namespace: development  # Default namespace

- name: prod-context
  context:
    cluster: production
    user: admin-user
    namespace: production
```

## 🔑 Authentication Methods

### 1. Client Certificates (Most Common)
```yaml
users:
- name: cert-user
  user:
    client-certificate: /path/to/client.crt
    client-key: /path/to/client.key
    # OR embedded (base64 encoded)
    client-certificate-data: LS0tLS1CRUdJTi...
    client-key-data: LS0tLS1CRUdJTi...
```

### 2. Bearer Tokens
```yaml
users:
- name: token-user
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6...
```

### 3. Username/Password (Basic Auth)
```yaml
users:
- name: basic-user
  user:
    username: admin
    password: secret123
```

### 4. Service Account Tokens
```yaml
users:
- name: sa-user
  user:
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
```

### 5. OIDC (OpenID Connect)
```yaml
users:
- name: oidc-user
  user:
    auth-provider:
      name: oidc
      config:
        client-id: kubernetes
        client-secret: secret
        id-token: eyJhbGciOiJSUzI1NiIsImtpZCI6...
        idp-issuer-url: https://accounts.google.com
```

### 6. Exec Plugin (External Authentication)
```yaml
users:
- name: exec-user
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: aws
      args:
      - eks
      - get-token
      - --cluster-name
      - my-cluster
```

## 🛠️ kubectl Commands for kubeconfig

### View Configuration
```bash
# View current config
kubectl config view

# View raw config (with secrets)
kubectl config view --raw

# View specific config file
kubectl config view --kubeconfig=/path/to/config

# Show current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# List all clusters
kubectl config get-clusters

# List all users
kubectl config get-users
```

### Manage Contexts
```bash
# Switch context
kubectl config use-context prod-context

# Set default namespace for context
kubectl config set-context dev-context --namespace=development

# Create new context
kubectl config set-context new-context \
  --cluster=development \
  --user=dev-user \
  --namespace=testing
```

### Manage Clusters
```bash
# Add cluster
kubectl config set-cluster development \
  --server=https://dev-k8s-api.example.com:6443 \
  --certificate-authority=/path/to/ca.crt

# Add cluster with embedded cert
kubectl config set-cluster development \
  --server=https://dev-k8s-api.example.com:6443 \
  --certificate-authority-data=$(cat /path/to/ca.crt | base64 -w 0)

# Skip TLS verification (NOT recommended for production)
kubectl config set-cluster development \
  --server=https://dev-k8s-api.example.com:6443 \
  --insecure-skip-tls-verify=true
```

### Manage Users
```bash
# Add user with client certificate
kubectl config set-credentials dev-user \
  --client-certificate=/path/to/client.crt \
  --client-key=/path/to/client.key

# Add user with embedded certificate
kubectl config set-credentials dev-user \
  --client-certificate-data=$(cat /path/to/client.crt | base64 -w 0) \
  --client-key-data=$(cat /path/to/client.key | base64 -w 0)

# Add user with token
kubectl config set-credentials token-user \
  --token=eyJhbGciOiJSUzI1NiIsImtpZCI6...

# Add user with username/password
kubectl config set-credentials basic-user \
  --username=admin \
  --password=secret123
```

## 🔧 Creating kubeconfig from Scratch

### Step 1: Generate Client Certificate
```bash
# Create private key
openssl genrsa -out client.key 2048

# Create certificate signing request
openssl req -new -key client.key -out client.csr -subj "/CN=dev-user/O=developers"

# Sign certificate with cluster CA
openssl x509 -req -in client.csr -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out client.crt -days 365
```

### Step 2: Build kubeconfig
```bash
# Set cluster
kubectl config set-cluster my-cluster \
  --server=https://k8s-api.example.com:6443 \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --kubeconfig=new-config

# Set user credentials
kubectl config set-credentials dev-user \
  --client-certificate=client.crt \
  --client-key=client.key \
  --kubeconfig=new-config

# Create context
kubectl config set-context dev-context \
  --cluster=my-cluster \
  --user=dev-user \
  --namespace=development \
  --kubeconfig=new-config

# Set current context
kubectl config use-context dev-context --kubeconfig=new-config
```

## 🎯 Real-World Scenarios

### Scenario 1: Multi-Environment Setup
```yaml
# ~/.kube/config
apiVersion: v1
kind: Config
current-context: dev

clusters:
- name: dev-cluster
  cluster:
    server: https://dev-api.company.com:6443
    certificate-authority-data: LS0tLS1CRUdJTi...

- name: staging-cluster
  cluster:
    server: https://staging-api.company.com:6443
    certificate-authority-data: LS0tLS1CRUdJTi...

- name: prod-cluster
  cluster:
    server: https://prod-api.company.com:6443
    certificate-authority-data: LS0tLS1CRUdJTi...

users:
- name: developer
  user:
    client-certificate-data: LS0tLS1CRUdJTi...
    client-key-data: LS0tLS1CRUdJTi...

- name: admin
  user:
    client-certificate-data: LS0tLS1CRUdJTi...
    client-key-data: LS0tLS1CRUdJTi...

contexts:
- name: dev
  context:
    cluster: dev-cluster
    user: developer
    namespace: development

- name: staging
  context:
    cluster: staging-cluster
    user: developer
    namespace: staging

- name: prod
  context:
    cluster: prod-cluster
    user: admin
    namespace: production
```

### Scenario 2: Service Account Access
```yaml
# Service account kubeconfig
apiVersion: v1
kind: Config
current-context: sa-context

clusters:
- name: local-cluster
  cluster:
    server: https://kubernetes.default.svc
    certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

users:
- name: service-account
  user:
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token

contexts:
- name: sa-context
  context:
    cluster: local-cluster
    user: service-account
    namespace: default
```

### Scenario 3: AWS EKS Configuration
```yaml
apiVersion: v1
kind: Config
current-context: eks-context

clusters:
- name: eks-cluster
  cluster:
    server: https://ABC123.gr7.us-west-2.eks.amazonaws.com
    certificate-authority-data: LS0tLS1CRUdJTi...

users:
- name: eks-user
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: aws
      args:
      - eks
      - get-token
      - --cluster-name
      - my-eks-cluster
      - --region
      - us-west-2

contexts:
- name: eks-context
  context:
    cluster: eks-cluster
    user: eks-user
```

## 🔒 Security Best Practices

### File Permissions
```bash
# Secure kubeconfig file
chmod 600 ~/.kube/config
chown $USER:$USER ~/.kube/config

# Verify permissions
ls -la ~/.kube/config
# Should show: -rw------- 1 user user
```

### Certificate Management
```bash
# Check certificate expiration
openssl x509 -in client.crt -text -noout | grep "Not After"

# Rotate certificates regularly
# Generate new certificates before expiration
```

### Environment Separation
```bash
# Use separate kubeconfig files
export KUBECONFIG_DEV=~/.kube/config-dev
export KUBECONFIG_PROD=~/.kube/config-prod

# Never mix dev and prod in same file
```

## 🛠️ Troubleshooting kubeconfig

### Common Issues and Solutions

#### 1. Connection Refused
```bash
# Check cluster endpoint
kubectl cluster-info

# Verify network connectivity
curl -k https://k8s-api.example.com:6443/version

# Check if API server is running
systemctl status kube-apiserver
```

#### 2. Certificate Errors
```bash
# Verify certificate validity
openssl x509 -in client.crt -text -noout

# Check CA certificate
openssl x509 -in ca.crt -text -noout

# Verify certificate chain
openssl verify -CAfile ca.crt client.crt
```

#### 3. Authentication Failures
```bash
# Check user permissions
kubectl auth can-i get pods --as=dev-user

# Verify token validity
kubectl auth can-i get pods --token=<token>

# Check RBAC bindings
kubectl get clusterrolebindings,rolebindings --all-namespaces | grep dev-user
```

#### 4. Context Issues
```bash
# List available contexts
kubectl config get-contexts

# Check current context
kubectl config current-context

# Switch to correct context
kubectl config use-context correct-context
```

### Debug Commands
```bash
# Verbose kubectl output
kubectl get pods -v=8

# Check kubeconfig syntax
kubectl config view --validate

# Test connection
kubectl cluster-info dump

# Check authentication
kubectl auth whoami
```

## 📋 kubeconfig Management Scripts

### Context Switcher Script
```bash
#!/bin/bash
# kube-switch.sh

if [ -z "$1" ]; then
    echo "Available contexts:"
    kubectl config get-contexts
    exit 1
fi

kubectl config use-context $1
echo "Switched to context: $1"
kubectl config current-context
```

### Multi-Environment Setup Script
```bash
#!/bin/bash
# setup-kubeconfig.sh

ENVIRONMENTS=("dev" "staging" "prod")
BASE_SERVER="https://k8s-api"
DOMAIN=".company.com:6443"

for env in "${ENVIRONMENTS[@]}"; do
    echo "Setting up $env environment..."
    
    kubectl config set-cluster ${env}-cluster \
        --server=${BASE_SERVER}-${env}${DOMAIN} \
        --certificate-authority=/etc/kubernetes/pki/ca.crt
    
    kubectl config set-credentials ${env}-user \
        --client-certificate=/etc/kubernetes/pki/${env}-client.crt \
        --client-key=/etc/kubernetes/pki/${env}-client.key
    
    kubectl config set-context $env \
        --cluster=${env}-cluster \
        --user=${env}-user \
        --namespace=$env
done

echo "kubeconfig setup completed"
kubectl config get-contexts
```

### Backup and Restore Script
```bash
#!/bin/bash
# kubeconfig-backup.sh

BACKUP_DIR="$HOME/.kube/backups"
DATE=$(date +%Y%m%d-%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup current config
cp ~/.kube/config $BACKUP_DIR/config-$DATE

echo "kubeconfig backed up to: $BACKUP_DIR/config-$DATE"

# List recent backups
ls -la $BACKUP_DIR/ | tail -5
```

## 🎯 Advanced kubeconfig Features

### Merging Multiple Configs
```bash
# Merge configs
KUBECONFIG=~/.kube/config:~/.kube/config-prod kubectl config view --flatten > ~/.kube/merged-config

# Use merged config
export KUBECONFIG=~/.kube/merged-config
```

### Conditional Context Loading
```bash
# .bashrc or .zshrc
function kube-env() {
    case $1 in
        dev)
            export KUBECONFIG=~/.kube/config-dev
            ;;
        prod)
            export KUBECONFIG=~/.kube/config-prod
            ;;
        *)
            export KUBECONFIG=~/.kube/config
            ;;
    esac
    kubectl config current-context
}
```

### Auto-completion Setup
```bash
# Add to .bashrc or .zshrc
source <(kubectl completion bash)
# or for zsh
source <(kubectl completion zsh)

# Enable context switching completion
complete -F __start_kubectl k
```

## 📊 kubeconfig in CI/CD

### GitLab CI Example
```yaml
# .gitlab-ci.yml
deploy:
  stage: deploy
  script:
    - echo "$KUBE_CONFIG" | base64 -d > ~/.kube/config
    - kubectl config use-context production
    - kubectl apply -f deployment.yaml
  only:
    - main
```

### GitHub Actions Example
```yaml
# .github/workflows/deploy.yml
- name: Setup kubectl
  uses: azure/setup-kubectl@v1
  
- name: Set kubeconfig
  run: |
    mkdir -p ~/.kube
    echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > ~/.kube/config
    kubectl config use-context production
```

This comprehensive guide covers everything about kubeconfig from basic concepts to advanced usage patterns, security practices, and real-world scenarios.