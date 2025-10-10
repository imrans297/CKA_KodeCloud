# Helm - The Kubernetes Package Manager

## 🎯 What is Helm?

**Definition**: Helm is a package manager for Kubernetes that helps you manage Kubernetes applications. It uses a packaging format called charts, which are collections of files that describe a related set of Kubernetes resources.

### Key Concepts
- **Chart** - A Helm package containing Kubernetes resource definitions
- **Release** - An instance of a chart running in a Kubernetes cluster
- **Repository** - A collection of charts that can be shared
- **Values** - Configuration parameters for charts

## 🚀 Helm Installation

### Install Helm
```bash
# Download and install Helm (Linux)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install via package manager (Ubuntu/Debian)
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

# Install via Homebrew (macOS)
brew install helm

# Verify installation
helm version
```

### Initialize Helm
```bash
# Add official Helm stable repository
helm repo add stable https://charts.helm.sh/stable

# Add Bitnami repository (popular charts)
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repositories
helm repo update

# List repositories
helm repo list
```

## 📦 Working with Charts

### Search and Install Charts
```bash
# Search for charts
helm search repo nginx
helm search repo mysql
helm search hub wordpress

# Show chart information
helm show chart bitnami/nginx
helm show values bitnami/nginx
helm show readme bitnami/nginx

# Install a chart
helm install my-nginx bitnami/nginx
helm install my-mysql bitnami/mysql

# Install with custom values
helm install my-nginx bitnami/nginx --set service.type=NodePort
helm install my-nginx bitnami/nginx --values custom-values.yaml

# Install from local chart
helm install my-app ./my-chart/
```

### Manage Releases
```bash
# List releases
helm list
helm list --all-namespaces
helm list -n production

# Get release status
helm status my-nginx
helm get values my-nginx
helm get manifest my-nginx

# Upgrade release
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
helm upgrade my-nginx bitnami/nginx --values new-values.yaml

# Rollback release
helm rollback my-nginx 1
helm rollback my-nginx --revision=2

# Uninstall release
helm uninstall my-nginx
helm uninstall my-nginx --keep-history
```

## 🛠️ Creating Custom Charts

### Create New Chart
```bash
# Create new chart
helm create my-app

# Chart structure
my-app/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default values
├── templates/          # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # Template helpers
│   └── NOTES.txt       # Installation notes
└── charts/             # Chart dependencies
```

### Chart.yaml Example
```yaml
apiVersion: v2
name: my-app
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.0.0"
keywords:
  - web
  - application
home: https://example.com
sources:
  - https://github.com/example/my-app
maintainers:
  - name: John Doe
    email: john@example.com
dependencies:
  - name: mysql
    version: 9.4.0
    repository: https://charts.bitnami.com/bitnami
    condition: mysql.enabled
```

### values.yaml Example
```yaml
# Default values for my-app
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.20"

nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80

nodeSelector: {}
tolerations: []
affinity: {}

# MySQL dependency
mysql:
  enabled: true
  auth:
    rootPassword: "secretpassword"
    database: "myapp"
```

## 📄 Template Examples

### Deployment Template
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          env:
            - name: DATABASE_HOST
              value: {{ include "my-app.fullname" . }}-mysql
            - name: DATABASE_NAME
              value: {{ .Values.mysql.auth.database }}
            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "my-app.fullname" . }}-mysql
                  key: mysql-root-password
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### Service Template
```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
      name: http
      {{- if and (eq .Values.service.type "NodePort") .Values.service.nodePort }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
  selector:
    {{- include "my-app.selectorLabels" . | nindent 4 }}
```

### Ingress Template
```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "my-app.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

### ConfigMap Template
```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-app.fullname" . }}-config
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
data:
  app.properties: |
    server.port={{ .Values.service.targetPort }}
    database.host={{ include "my-app.fullname" . }}-mysql
    database.name={{ .Values.mysql.auth.database }}
    {{- if .Values.config }}
    {{- range $key, $value := .Values.config }}
    {{ $key }}={{ $value }}
    {{- end }}
    {{- end }}
```

### Helper Templates
```yaml
# templates/_helpers.tpl
{{/*
Expand the name of the chart.
*/}}
{{- define "my-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "my-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "my-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "my-app.labels" -}}
helm.sh/chart: {{ include "my-app.chart" . }}
{{ include "my-app.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "my-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

## 🔧 Advanced Helm Features

### Conditional Templates
```yaml
# Conditional rendering
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
# ... ingress configuration
{{- end }}

# Conditional values
{{- if eq .Values.service.type "NodePort" }}
nodePort: {{ .Values.service.nodePort }}
{{- end }}

# Range loops
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}
```

### Template Functions
```yaml
# String functions
name: {{ .Values.name | upper | quote }}
image: {{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}

# Math functions
replicas: {{ add .Values.replicaCount 1 }}
memory: {{ mul .Values.memory 1024 }}

# Date functions
timestamp: {{ now | date "2006-01-02T15:04:05Z" }}

# Encoding functions
data: {{ .Values.config | b64enc }}
```

### Hooks
```yaml
# Pre-install hook
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-app.fullname" . }}-pre-install
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: pre-install
        image: busybox
        command: ['sh', '-c', 'echo "Pre-install hook executed"']
```

## 📊 Testing and Validation

### Helm Testing
```bash
# Lint chart
helm lint ./my-app/

# Dry run installation
helm install my-app ./my-app/ --dry-run --debug

# Template rendering
helm template my-app ./my-app/
helm template my-app ./my-app/ --values custom-values.yaml

# Test release
helm test my-release

# Validate against Kubernetes
helm install my-app ./my-app/ --dry-run --validate
```

### Test Templates
```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-app.fullname" . }}-test-connection"
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  restartPolicy: Never
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "my-app.fullname" . }}:{{ .Values.service.port }}']
```

## 🔐 Security and Best Practices

### Secrets Management
```yaml
# Using external secrets
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "my-app.fullname" . }}-secret
type: Opaque
data:
  {{- if .Values.secrets }}
  {{- range $key, $value := .Values.secrets }}
  {{ $key }}: {{ $value | b64enc }}
  {{- end }}
  {{- end }}
```

### RBAC Template
```yaml
# templates/rbac.yaml
{{- if .Values.rbac.create }}
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
rules:
{{- with .Values.rbac.rules }}
  {{- toYaml . | nindent 2 }}
{{- end }}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: {{ include "my-app.fullname" . }}
subjects:
- kind: ServiceAccount
  name: {{ include "my-app.serviceAccountName" . }}
  namespace: {{ .Release.Namespace }}
{{- end }}
```

## 📦 Chart Repositories

### Create Chart Repository
```bash
# Package chart
helm package ./my-app/

# Create repository index
helm repo index . --url https://my-charts.example.com

# Upload to repository (example with GitHub Pages)
git add .
git commit -m "Add chart package"
git push origin gh-pages
```

### Private Repository
```bash
# Add private repository with authentication
helm repo add my-private-repo https://charts.example.com \
  --username myuser \
  --password mypassword

# Add repository with certificate
helm repo add secure-repo https://secure-charts.example.com \
  --ca-file ca.crt \
  --cert-file client.crt \
  --key-file client.key
```

## 🚀 Production Examples

### Multi-Environment Values
```yaml
# values-dev.yaml
replicaCount: 1
image:
  tag: "dev"
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"

# values-prod.yaml
replicaCount: 3
image:
  tag: "v1.0.0"
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
```

### CI/CD Integration
```yaml
# .github/workflows/helm-deploy.yml
name: Deploy with Helm
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Install Helm
      uses: azure/setup-helm@v1
    - name: Deploy to staging
      run: |
        helm upgrade --install my-app ./charts/my-app \
          --namespace staging \
          --values ./charts/my-app/values-staging.yaml \
          --set image.tag=${{ github.sha }}
```

## 🛠️ Troubleshooting

### Common Issues
```bash
# Debug template rendering
helm template my-app ./my-app/ --debug

# Check release history
helm history my-release

# Get release manifest
helm get manifest my-release

# Check values
helm get values my-release

# Rollback failed release
helm rollback my-release --wait

# Force delete stuck release
helm delete my-release --no-hooks
```

### Validation Commands
```bash
# Validate chart structure
helm lint ./my-app/

# Check dependencies
helm dependency list ./my-app/
helm dependency update ./my-app/

# Verify installation
kubectl get all -l app.kubernetes.io/instance=my-release
```

This comprehensive Helm guide covers everything from basic concepts to advanced production usage, providing practical examples and best practices for managing Kubernetes applications with Helm.