# Kustomize - Kubernetes Native Configuration Management

## 🎯 Problem Statement and Ideology

### The Problem
Managing Kubernetes configurations across multiple environments (dev, staging, production) leads to:
- **Configuration Drift** - Inconsistencies between environments
- **Duplication** - Copying YAML files with minor changes
- **Maintenance Overhead** - Managing multiple versions of similar configurations
- **Error-Prone** - Manual edits across multiple files

### Kustomize Ideology
**"Configuration as Code"** - Kustomize follows the principle of declarative configuration management:
- **Template-Free** - No templating language, pure YAML
- **Kubernetes Native** - Built into kubectl (kubectl apply -k)
- **Overlay Pattern** - Base configurations with environment-specific overlays
- **Composable** - Combine and reuse configurations

**Core Philosophy**: "Customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is."

## 🆚 Kustomize vs Helm

| Feature | Kustomize | Helm |
|---------|-----------|------|
| **Approach** | Overlay-based patching | Template-based |
| **Learning Curve** | Low (pure YAML) | Medium (Go templates) |
| **Kubernetes Integration** | Native (kubectl -k) | External tool |
| **Package Management** | No | Yes (Charts, Repositories) |
| **Templating** | No templating | Rich templating |
| **Configuration** | Patches and overlays | Values files |
| **Reusability** | Components and bases | Chart repositories |
| **Complexity** | Simple for basic use | Complex but powerful |

### When to Use What?
- **Use Kustomize** for: Simple configurations, GitOps workflows, template-free approach
- **Use Helm** for: Complex applications, package distribution, rich templating needs

## 🚀 Installation and Setup

### Install Kustomize
```bash
# Kustomize is built into kubectl (v1.14+)
kubectl version --client

# Install standalone kustomize
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
sudo mv kustomize /usr/local/bin/

# Install via package manager
# Ubuntu/Debian
sudo apt install kustomize

# macOS
brew install kustomize

# Verify installation
kustomize version
```

### Basic Usage
```bash
# Using kubectl (recommended)
kubectl apply -k ./path/to/kustomization/

# Using standalone kustomize
kustomize build ./path/to/kustomization/ | kubectl apply -f -

# Generate output without applying
kubectl kustomize ./path/to/kustomization/
kustomize build ./path/to/kustomization/
```

## 📄 The kustomization.yaml File

### Basic Structure
```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Metadata
metadata:
  name: my-app
  namespace: default

# Resources to include
resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml

# Common labels applied to all resources
commonLabels:
  app: my-app
  version: v1.0.0

# Common annotations
commonAnnotations:
  managed-by: kustomize
  
# Namespace for all resources
namespace: production

# Name prefix/suffix
namePrefix: prod-
nameSuffix: -v1

# Images to replace
images:
  - name: nginx
    newTag: 1.20.0
    newName: nginx-custom

# ConfigMap generators
configMapGenerator:
  - name: app-config
    files:
      - config.properties
    literals:
      - DATABASE_URL=postgres://localhost:5432/mydb

# Secret generators
secretGenerator:
  - name: app-secrets
    literals:
      - username=admin
      - password=secret123

# Patches
patches:
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 3
    target:
      kind: Deployment
      name: my-app
```

## 🔧 Kustomize Output

### Generate and View Output
```bash
# Generate YAML output
kubectl kustomize ./base/

# Apply directly
kubectl apply -k ./base/

# Save output to file
kubectl kustomize ./base/ > output.yaml

# Dry run
kubectl apply -k ./base/ --dry-run=client -o yaml
```

### Example Output
```yaml
# Original deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: nginx:1.19

# After kustomization
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prod-my-app-v1  # namePrefix + name + nameSuffix
  namespace: production  # namespace applied
  labels:
    app: my-app         # commonLabels applied
    version: v1.0.0
  annotations:
    managed-by: kustomize  # commonAnnotations applied
spec:
  replicas: 3           # patched value
  selector:
    matchLabels:
      app: my-app
      version: v1.0.0   # commonLabels in selector
  template:
    metadata:
      labels:
        app: my-app
        version: v1.0.0
    spec:
      containers:
      - name: app
        image: nginx:1.20.0  # image tag updated
```

## 📋 ApiVersion and Kind

### Supported API Versions
```yaml
# Current stable version
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Legacy versions (deprecated)
apiVersion: kustomize.config.k8s.io/v1alpha1  # Deprecated
apiVersion: kustomize.config.k8s.io/v1beta1   # Current
```

### Kind Types
```yaml
# Main kustomization file
kind: Kustomization

# Component (reusable piece)
kind: Component
```

## 📁 Managing Directories

### Directory Structure
```
my-app/
├── base/                    # Base configuration
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── overlays/               # Environment-specific overlays
│   ├── development/
│   │   ├── kustomization.yaml
│   │   └── dev-patch.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── staging-patch.yaml
│   └── production/
│       ├── kustomization.yaml
│       ├── prod-patch.yaml
│       └── hpa.yaml
└── components/             # Reusable components
    ├── monitoring/
    │   ├── kustomization.yaml
    │   └── servicemonitor.yaml
    └── security/
        ├── kustomization.yaml
        └── networkpolicy.yaml
```

### Base Configuration
```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml

commonLabels:
  app: my-app
```

### Overlay Configuration
```yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Reference base
bases:
  - ../../base

# Additional resources for production
resources:
  - hpa.yaml
  - ingress.yaml

# Production-specific labels
commonLabels:
  environment: production

# Production namespace
namespace: production

# Production image tag
images:
  - name: my-app
    newTag: v1.2.3

# Production replicas
patches:
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
    target:
      kind: Deployment
      name: my-app
```

## 🔄 Common Transformers

### Name Transformers
```yaml
# Add prefix to all resource names
namePrefix: prod-

# Add suffix to all resource names
nameSuffix: -v2

# Example result: prod-my-app-v2
```

### Label and Annotation Transformers
```yaml
# Apply to all resources
commonLabels:
  app: my-app
  version: v1.0.0
  environment: production
  team: backend

commonAnnotations:
  managed-by: kustomize
  deployment-date: "2023-12-01"
  contact: "team@example.com"
```

### Namespace Transformer
```yaml
# Set namespace for all resources
namespace: production

# Resources without namespace will get this namespace
# Resources with existing namespace will be overridden
```

## 🖼️ Image Transformers

### Basic Image Replacement
```yaml
images:
  # Replace image tag
  - name: nginx
    newTag: 1.20.0
  
  # Replace image name and tag
  - name: my-app
    newName: registry.example.com/my-app
    newTag: v1.2.3
  
  # Replace with digest
  - name: redis
    digest: sha256:abc123...
```

### Advanced Image Transformations
```yaml
images:
  # Multiple transformations
  - name: nginx
    newName: custom-nginx
    newTag: latest
  
  # Using environment variables (with envsubst)
  - name: my-app
    newTag: ${IMAGE_TAG}
  
  # Registry replacement
  - name: gcr.io/my-project/my-app
    newName: docker.io/my-company/my-app
    newTag: production
```

### Image Examples
```yaml
# Before transformation
containers:
- name: web
  image: nginx:1.19
- name: app  
  image: my-app:dev

# After transformation
containers:
- name: web
  image: nginx:1.20.0
- name: app
  image: registry.example.com/my-app:v1.2.3
```

## 🩹 Patches Introduction

### What are Patches?
Patches modify existing Kubernetes resources without changing the original files. Kustomize supports multiple patch formats:

1. **Strategic Merge Patches** - Merge YAML structures
2. **JSON Patches (RFC 6902)** - Precise JSON operations
3. **Merge Patches (RFC 7396)** - Simple merge operations

### Patch Targets
```yaml
patches:
  - patch: |
      # patch content
    target:
      group: apps
      version: v1
      kind: Deployment
      name: my-app
      namespace: default
```

## 🔧 Different Types of Patches

### 1. Strategic Merge Patch (Default)
```yaml
# Strategic merge - merges YAML structures intelligently
patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: my-app
      spec:
        replicas: 3
        template:
          spec:
            containers:
            - name: app
              resources:
                limits:
                  memory: "512Mi"
                  cpu: "500m"
    target:
      kind: Deployment
      name: my-app
```

### 2. JSON Patch (RFC 6902)
```yaml
# JSON patch - precise operations
patches:
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
      - op: add
        path: /spec/template/spec/containers/0/env
        value:
          - name: DEBUG
            value: "true"
      - op: remove
        path: /spec/template/spec/containers/0/ports/1
    target:
      kind: Deployment
      name: my-app
```

### 3. Merge Patch (RFC 7396)
```yaml
# Merge patch - simple merge
patches:
  - patch: |-
      spec:
        replicas: 3
        template:
          spec:
            containers:
            - name: app
              image: nginx:1.20
    target:
      kind: Deployment
      name: my-app
```

## 📚 Patches Dictionary

### Patch Operations (JSON Patch)
```yaml
patches:
  - patch: |-
      # ADD - Add new value
      - op: add
        path: /spec/template/spec/containers/0/env/-
        value:
          name: NEW_VAR
          value: "new-value"
      
      # REPLACE - Replace existing value
      - op: replace
        path: /spec/replicas
        value: 3
      
      # REMOVE - Remove existing value
      - op: remove
        path: /spec/template/spec/containers/0/ports/0
      
      # COPY - Copy value from one location to another
      - op: copy
        from: /spec/template/spec/containers/0/resources/requests
        path: /spec/template/spec/containers/0/resources/limits
      
      # MOVE - Move value from one location to another
      - op: move
        from: /spec/template/metadata/labels/old-label
        path: /spec/template/metadata/labels/new-label
      
      # TEST - Test if value exists (conditional)
      - op: test
        path: /spec/replicas
        value: 1
    target:
      kind: Deployment
      name: my-app
```

### Path Examples
```yaml
# Common JSON Patch paths
/metadata/name                           # Resource name
/metadata/labels/app                     # Specific label
/metadata/annotations                    # All annotations
/spec/replicas                          # Deployment replicas
/spec/template/spec/containers/0/image  # First container image
/spec/template/spec/containers/0/env/-  # Add to end of env array
/spec/template/spec/containers/0/env/0  # First env variable
/spec/ports/0/port                      # First port number
```

## 📝 Patches List

### Multiple Patches Example
```yaml
patches:
  # Patch 1: Update replicas
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 3
    target:
      kind: Deployment
      name: web-app
  
  # Patch 2: Add environment variables
  - patch: |-
      spec:
        template:
          spec:
            containers:
            - name: app
              env:
              - name: ENVIRONMENT
                value: production
              - name: LOG_LEVEL
                value: info
    target:
      kind: Deployment
      name: web-app
  
  # Patch 3: Update service type
  - patch: |-
      spec:
        type: LoadBalancer
    target:
      kind: Service
      name: web-service
  
  # Patch 4: Add ingress annotations
  - patch: |-
      metadata:
        annotations:
          nginx.ingress.kubernetes.io/rewrite-target: /
          cert-manager.io/cluster-issuer: letsencrypt-prod
    target:
      kind: Ingress
      name: web-ingress
```

### Conditional Patches
```yaml
patches:
  # Only apply if replicas is currently 1
  - patch: |-
      - op: test
        path: /spec/replicas
        value: 1
      - op: replace
        path: /spec/replicas
        value: 3
    target:
      kind: Deployment
      name: my-app
```

## 🏗️ Overlays and Components

### Overlays Structure
```
overlays/
├── development/
│   ├── kustomization.yaml
│   ├── dev-config.yaml
│   └── dev-patches.yaml
├── staging/
│   ├── kustomization.yaml
│   ├── staging-config.yaml
│   └── staging-patches.yaml
└── production/
    ├── kustomization.yaml
    ├── prod-config.yaml
    ├── prod-patches.yaml
    └── hpa.yaml
```

### Development Overlay
```yaml
# overlays/development/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
  - ../../base

namespace: development

commonLabels:
  environment: dev

images:
  - name: my-app
    newTag: dev-latest

resources:
  - dev-config.yaml

patches:
  - patch: |-
      spec:
        replicas: 1
        template:
          spec:
            containers:
            - name: app
              resources:
                requests:
                  memory: "128Mi"
                  cpu: "100m"
    target:
      kind: Deployment
      name: my-app
```

### Production Overlay
```yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
  - ../../base

namespace: production

commonLabels:
  environment: prod

images:
  - name: my-app
    newTag: v1.2.3

resources:
  - prod-config.yaml
  - hpa.yaml
  - ingress.yaml

patches:
  - patch: |-
      spec:
        replicas: 5
        template:
          spec:
            containers:
            - name: app
              resources:
                requests:
                  memory: "512Mi"
                  cpu: "500m"
                limits:
                  memory: "1Gi"
                  cpu: "1000m"
    target:
      kind: Deployment
      name: my-app
```

### Components
```yaml
# components/monitoring/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - servicemonitor.yaml
  - prometheusrule.yaml

commonLabels:
  monitoring: enabled
```

### Using Components
```yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
  - ../../base

components:
  - ../../components/monitoring
  - ../../components/security

# ... rest of configuration
```

### Component Example Files
```yaml
# components/monitoring/servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-metrics
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s

# components/security/networkpolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
```

## 🚀 Advanced Kustomize Features

### Generators
```yaml
# ConfigMap generator
configMapGenerator:
  - name: app-config
    files:
      - application.properties
      - log4j.properties
    literals:
      - DATABASE_URL=postgres://localhost:5432/mydb
      - CACHE_SIZE=100

# Secret generator
secretGenerator:
  - name: app-secrets
    files:
      - secret.txt
    literals:
      - username=admin
      - password=secret123
    type: Opaque

# Generic generator options
generatorOptions:
  disableNameSuffixHash: false  # Add hash suffix to generated names
  labels:
    generator: kustomize
  annotations:
    generated-by: kustomize
```

### Replacements
```yaml
# Advanced field replacement
replacements:
  - source:
      kind: ConfigMap
      name: app-config
      fieldPath: data.DATABASE_URL
    targets:
      - select:
          kind: Deployment
          name: my-app
        fieldPaths:
          - spec.template.spec.containers.[name=app].env.[name=DATABASE_URL].value
```

This comprehensive Kustomize guide covers all the essential concepts from basic usage to advanced features, providing practical examples for managing Kubernetes configurations in a template-free, overlay-based approach.