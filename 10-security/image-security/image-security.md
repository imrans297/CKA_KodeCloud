# Kubernetes Image Security

## 🎯 Overview

Image security ensures container images are trusted, scanned for vulnerabilities, and sourced from secure registries.

## 🔐 Private Registry Authentication

### Docker Registry Secret
```bash
# Create docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.io \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com

# View secret
kubectl get secret regcred -o yaml
```

### Using Registry Secret in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-image-pod
spec:
  containers:
  - name: app
    image: myregistry.io/myapp:v1.0
  imagePullSecrets:
  - name: regcred
```

### Service Account with Image Pull Secret
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
imagePullSecrets:
- name: regcred
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-sa
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: app
    image: myregistry.io/myapp:v1.0
```

## 🛡️ Image Pull Policies

### Pull Policy Options
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: image-policy-demo
spec:
  containers:
  - name: app
    image: nginx:1.20
    imagePullPolicy: Always    # Always, IfNotPresent, Never
```

### Policy Behavior
- **Always** - Pull image every time
- **IfNotPresent** - Pull only if not cached locally
- **Never** - Never pull, use local image only

## 🔍 Image Vulnerability Scanning

### Admission Controllers for Image Security
```yaml
# OPA Gatekeeper policy for allowed registries
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: allowedregistries
spec:
  crd:
    spec:
      names:
        kind: AllowedRegistries
      validation:
        properties:
          registries:
            type: array
            items:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package allowedregistries
        
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not starts_with(container.image, input.parameters.registries[_])
          msg := sprintf("Image '%v' not from allowed registry", [container.image])
        }
```

### Image Scanning Tools Integration
```yaml
# Falco rule for suspicious image pulls
- rule: Suspicious Image Pull
  desc: Detect pulls from untrusted registries
  condition: >
    k8s_audit and
    ka.verb=create and
    ka.target.resource=pods and
    not ka.target.image startswith "myregistry.io/"
  output: >
    Untrusted image pulled (user=%ka.user.name verb=%ka.verb 
    image=%ka.target.image)
  priority: WARNING
```

## 🏷️ Image Signing and Verification

### Cosign Image Signing
```bash
# Sign image with cosign
cosign sign --key cosign.key myregistry.io/myapp:v1.0

# Verify signed image
cosign verify --key cosign.pub myregistry.io/myapp:v1.0
```

### Admission Controller for Signed Images
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cosign-policy
data:
  policy.yaml: |
    apiVersion: v1
    kind: Policy
    metadata:
      name: signed-images-only
    spec:
      rules:
      - name: require-signature
        match:
        - apiGroups: [""]
          apiVersions: ["v1"]
          resources: ["pods"]
        validate:
          message: "Image must be signed"
          pattern:
            spec:
              containers:
              - image: "myregistry.io/*"
```

## 🔒 Secure Image Practices

### Multi-stage Dockerfile
```dockerfile
# Build stage
FROM golang:1.19 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Runtime stage - minimal base image
FROM gcr.io/distroless/base-debian11
COPY --from=builder /app/myapp /
USER 65534:65534
ENTRYPOINT ["/myapp"]
```

### Distroless Images
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: distroless-pod
spec:
  containers:
  - name: app
    image: gcr.io/distroless/java:11
    # No shell, minimal attack surface
```

### Image Security Scanning in CI/CD
```yaml
# GitHub Actions example
name: Image Security Scan
on: [push]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Build image
      run: docker build -t myapp:${{ github.sha }} .
    - name: Scan with Trivy
      run: |
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
          aquasec/trivy image --exit-code 1 myapp:${{ github.sha }}
```

## 🛠️ Image Security Tools

### Trivy Scanning
```bash
# Scan local image
trivy image nginx:1.20

# Scan with specific severity
trivy image --severity HIGH,CRITICAL nginx:1.20

# Output to JSON
trivy image --format json nginx:1.20
```

### Clair Integration
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: clair-config
data:
  config.yaml: |
    clair:
      database:
        type: pgsql
        options:
          source: postgresql://user:pass@host:5432/clair
      api:
        healthport: 6061
        port: 6062
      updater:
        interval: 2h
```

## 📋 Image Security Policies

### Pod Security Standards
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Network Policy for Registry Access
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-registry-access
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: registry-namespace
    ports:
    - protocol: TCP
      port: 5000
  - to: []
    ports:
    - protocol: TCP
      port: 443  # HTTPS for external registries
```

## 🚨 Image Security Monitoring

### Falco Rules for Image Security
```yaml
# /etc/falco/rules.d/image_security.yaml
- rule: Untrusted Image Repository
  desc: Detect container from untrusted repository
  condition: >
    container and
    not image.repository in (trusted_images)
  output: >
    Container from untrusted repository (user=%user.name 
    command=%proc.cmdline image=%container.image.repository)
  priority: WARNING

- list: trusted_images
  items: [
    "myregistry.io",
    "gcr.io/my-project",
    "docker.io/library"
  ]
```

### Prometheus Metrics for Image Security
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: image-security-metrics
data:
  rules.yml: |
    groups:
    - name: image_security
      rules:
      - alert: UntrustedImagePulled
        expr: increase(container_image_pulls_total{image!~"myregistry.io/.*"}[5m]) > 0
        labels:
          severity: warning
        annotations:
          summary: "Untrusted image pulled"
          description: "Image {{ $labels.image }} pulled from untrusted registry"
```