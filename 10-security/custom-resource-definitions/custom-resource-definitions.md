# Kubernetes Custom Resource Definitions (CRDs)

## 🎯 Overview

Custom Resource Definitions (CRDs) extend Kubernetes API to create custom resources, enabling you to define and manage application-specific objects using kubectl and Kubernetes APIs.

## 🔧 Basic CRD Structure

### Simple CRD Example
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: ["mysql", "postgres", "mongodb"]
              version:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 10
          status:
            type: object
            properties:
              phase:
                type: string
              ready:
                type: boolean
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
    shortNames:
    - db
```

### Using the Custom Resource
```yaml
apiVersion: stable.example.com/v1
kind: Database
metadata:
  name: my-database
  namespace: production
spec:
  engine: postgres
  version: "13.4"
  replicas: 3
```

## 📋 CRD Components

### Metadata Fields
```yaml
metadata:
  name: databases.stable.example.com  # Must be <plural>.<group>
  annotations:
    api-approved.kubernetes.io: "https://github.com/kubernetes/kubernetes/pull/12345"
```

### Spec Fields
```yaml
spec:
  group: stable.example.com           # API group
  scope: Namespaced                   # Namespaced or Cluster
  names:
    plural: databases                 # Plural name for API
    singular: database               # Singular name
    kind: Database                   # Kind for YAML
    shortNames: [db, dbs]           # kubectl shortcuts
    listKind: DatabaseList          # List kind (auto-generated)
  versions:                         # API versions
  - name: v1
    served: true                    # Accept requests
    storage: true                   # Store in etcd
```

## 🔍 Schema Validation

### OpenAPI v3 Schema
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.apps.example.com
spec:
  group: apps.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        required: ["spec"]
        properties:
          spec:
            type: object
            required: ["image", "replicas"]
            properties:
              image:
                type: string
                pattern: '^[a-zA-Z0-9._/-]+:[a-zA-Z0-9._-]+$'
              replicas:
                type: integer
                minimum: 1
                maximum: 100
              resources:
                type: object
                properties:
                  cpu:
                    type: string
                    pattern: '^[0-9]+m?$'
                  memory:
                    type: string
                    pattern: '^[0-9]+[KMGT]i?$'
              env:
                type: array
                items:
                  type: object
                  required: ["name", "value"]
                  properties:
                    name:
                      type: string
                    value:
                      type: string
          status:
            type: object
            properties:
              phase:
                type: string
                enum: ["Pending", "Running", "Failed"]
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                      enum: ["True", "False", "Unknown"]
                    lastTransitionTime:
                      type: string
                      format: date-time
  scope: Namespaced
  names:
    plural: applications
    singular: application
    kind: Application
    shortNames: [app]
```

## 🎯 Advanced CRD Features

### Multiple Versions
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
              version:
                type: string
              config:
                type: object
  - name: v1beta1
    served: true
    storage: false  # Not stored, converted to v1
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              type:  # Old field name
                type: string
              version:
                type: string
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          name: database-conversion-webhook
          namespace: default
          path: /convert
```

### Subresources
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    subresources:
      status: {}  # Enable /status subresource
      scale:      # Enable /scale subresource
        specReplicasPath: .spec.replicas
        statusReplicasPath: .status.replicas
        labelSelectorPath: .status.labelSelector
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              replicas:
                type: integer
          status:
            type: object
            properties:
              replicas:
                type: integer
              labelSelector:
                type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

### Additional Printer Columns
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    additionalPrinterColumns:
    - name: Engine
      type: string
      description: Database engine
      jsonPath: .spec.engine
    - name: Version
      type: string
      jsonPath: .spec.version
    - name: Replicas
      type: integer
      jsonPath: .spec.replicas
    - name: Status
      type: string
      jsonPath: .status.phase
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
    schema:
      openAPIV3Schema:
        # ... schema definition
```

## 🛠️ CRD Management Commands

### Create and Manage CRDs
```bash
# Create CRD
kubectl apply -f database-crd.yaml

# List CRDs
kubectl get crd
kubectl get customresourcedefinitions

# Describe CRD
kubectl describe crd databases.stable.example.com

# Delete CRD (deletes all custom resources too!)
kubectl delete crd databases.stable.example.com
```

### Work with Custom Resources
```bash
# Create custom resource
kubectl apply -f my-database.yaml

# List custom resources
kubectl get databases
kubectl get db  # Using short name

# Describe custom resource
kubectl describe database my-database

# Edit custom resource
kubectl edit database my-database

# Delete custom resource
kubectl delete database my-database
```

## 🎯 Real-World CRD Examples

### Application CRD
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.platform.company.com
spec:
  group: platform.company.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        required: ["spec"]
        properties:
          spec:
            type: object
            required: ["image", "port"]
            properties:
              image:
                type: string
                description: "Container image"
              port:
                type: integer
                minimum: 1
                maximum: 65535
              replicas:
                type: integer
                default: 1
                minimum: 0
                maximum: 100
              resources:
                type: object
                properties:
                  requests:
                    type: object
                    properties:
                      cpu:
                        type: string
                      memory:
                        type: string
                  limits:
                    type: object
                    properties:
                      cpu:
                        type: string
                      memory:
                        type: string
              ingress:
                type: object
                properties:
                  enabled:
                    type: boolean
                    default: false
                  host:
                    type: string
                  tls:
                    type: boolean
                    default: false
          status:
            type: object
            properties:
              phase:
                type: string
                enum: ["Pending", "Running", "Failed", "Succeeded"]
              replicas:
                type: integer
              readyReplicas:
                type: integer
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    reason:
                      type: string
                    message:
                      type: string
                    lastTransitionTime:
                      type: string
                      format: date-time
    additionalPrinterColumns:
    - name: Image
      type: string
      jsonPath: .spec.image
    - name: Replicas
      type: integer
      jsonPath: .spec.replicas
    - name: Ready
      type: integer
      jsonPath: .status.readyReplicas
    - name: Status
      type: string
      jsonPath: .status.phase
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
  scope: Namespaced
  names:
    plural: applications
    singular: application
    kind: Application
    shortNames: [app]
```

### Database Backup CRD
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.database.example.com
spec:
  group: database.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: ["database", "schedule"]
            properties:
              database:
                type: string
                description: "Target database name"
              schedule:
                type: string
                pattern: '^(@(annually|yearly|monthly|weekly|daily|hourly|reboot))|(@every (\d+(ns|us|µs|ms|s|m|h))+)|((((\d+,)+\d+|(\d+(\/|-)\d+)|\d+|\*) ?){5,7})$'
              retention:
                type: string
                default: "30d"
              storage:
                type: object
                required: ["type"]
                properties:
                  type:
                    type: string
                    enum: ["s3", "gcs", "azure", "local"]
                  bucket:
                    type: string
                  path:
                    type: string
                  credentials:
                    type: string
          status:
            type: object
            properties:
              lastBackup:
                type: string
                format: date-time
              nextBackup:
                type: string
                format: date-time
              phase:
                type: string
                enum: ["Active", "Suspended", "Failed"]
    additionalPrinterColumns:
    - name: Database
      type: string
      jsonPath: .spec.database
    - name: Schedule
      type: string
      jsonPath: .spec.schedule
    - name: Last Backup
      type: string
      jsonPath: .status.lastBackup
    - name: Status
      type: string
      jsonPath: .status.phase
  scope: Namespaced
  names:
    plural: backups
    singular: backup
    kind: Backup
    shortNames: [bkp]
```

## 🔒 CRD Security Considerations

### RBAC for CRDs
```yaml
# Allow creating/managing CRDs (cluster admin only)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: crd-admin
rules:
- apiGroups: ["apiextensions.k8s.io"]
  resources: ["customresourcedefinitions"]
  verbs: ["*"]
---
# Allow using custom resources
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: database-operator
rules:
- apiGroups: ["stable.example.com"]
  resources: ["databases"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["stable.example.com"]
  resources: ["databases/status"]
  verbs: ["get", "update", "patch"]
```

### Admission Controllers for CRDs
```yaml
# ValidatingAdmissionWebhook for custom resources
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionWebhook
metadata:
  name: database-validator
webhooks:
- name: validate.database.example.com
  clientConfig:
    service:
      name: database-webhook
      namespace: default
      path: /validate
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["stable.example.com"]
    apiVersions: ["v1"]
    resources: ["databases"]
  admissionReviewVersions: ["v1", "v1beta1"]
```

## 🚀 CRD Controllers and Operators

### Simple Controller Pattern
```go
// Example controller structure (Go)
type DatabaseController struct {
    client     kubernetes.Interface
    crdClient  clientset.Interface
    informer   cache.SharedIndexInformer
}

func (c *DatabaseController) processDatabase(obj interface{}) error {
    db := obj.(*v1.Database)
    
    // Reconcile desired state
    deployment := c.createDeployment(db)
    service := c.createService(db)
    
    // Update status
    db.Status.Phase = "Running"
    db.Status.Replicas = db.Spec.Replicas
    
    return c.updateStatus(db)
}
```

### Operator Framework Example
```yaml
# Operator deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database-operator
  template:
    metadata:
      labels:
        app: database-operator
    spec:
      serviceAccountName: database-operator
      containers:
      - name: operator
        image: database-operator:v1.0
        env:
        - name: WATCH_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: OPERATOR_NAME
          value: "database-operator"
```

## 🔍 CRD Validation and Testing

### Validation Examples
```yaml
# Complex validation schema
schema:
  openAPIV3Schema:
    type: object
    properties:
      spec:
        type: object
        properties:
          config:
            type: object
            additionalProperties:
              type: string
          resources:
            type: object
            properties:
              limits:
                type: object
                properties:
                  cpu:
                    type: string
                    pattern: '^[0-9]+m?$'
                  memory:
                    type: string
                    pattern: '^[0-9]+[KMGT]i?$'
            required: ["limits"]
        anyOf:  # At least one of these must be present
        - required: ["image"]
        - required: ["imageRef"]
        not:    # Cannot have both
          allOf:
          - required: ["image"]
          - required: ["imageRef"]
```

### Testing CRDs
```bash
# Validate CRD syntax
kubectl apply --dry-run=client -f database-crd.yaml

# Test custom resource creation
kubectl apply --dry-run=server -f my-database.yaml

# Validate against schema
kubectl create -f invalid-database.yaml
# Should show validation errors

# Test with kubectl explain
kubectl explain database.spec
kubectl explain database.spec.engine
```

## 📊 CRD Monitoring and Observability

### Metrics Collection
```yaml
# ServiceMonitor for CRD metrics
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: database-operator-metrics
spec:
  selector:
    matchLabels:
      app: database-operator
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```

### Custom Resource Events
```bash
# View events for custom resources
kubectl get events --field-selector involvedObject.kind=Database

# Describe custom resource with events
kubectl describe database my-database
```

## 🛠️ CRD Best Practices

### Design Guidelines
1. **Naming**: Use clear, descriptive names
2. **Versioning**: Plan for API evolution
3. **Validation**: Comprehensive schema validation
4. **Documentation**: Clear field descriptions
5. **Defaults**: Sensible default values

### Schema Best Practices
```yaml
# Good schema design
properties:
  spec:
    type: object
    required: ["image"]  # Required fields
    properties:
      image:
        type: string
        description: "Container image to deploy"
        example: "nginx:1.20"
      replicas:
        type: integer
        default: 1        # Default value
        minimum: 0        # Validation
        maximum: 100
        description: "Number of replicas"
      labels:
        type: object
        additionalProperties:
          type: string    # Allow arbitrary string labels
```

### Operational Guidelines
```bash
# Always backup before CRD changes
kubectl get crd databases.stable.example.com -o yaml > backup.yaml

# Test in development first
kubectl apply -f new-crd.yaml --dry-run=server

# Monitor custom resource usage
kubectl get databases --all-namespaces
kubectl top nodes  # Check resource usage
```

This comprehensive CRD guide covers everything from basic concepts to advanced features, security considerations, and operational best practices for extending Kubernetes with custom resources.