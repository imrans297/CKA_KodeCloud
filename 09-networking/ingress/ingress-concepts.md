# Ingress

## 🎯 Ingress Concepts

### What is Ingress?
- HTTP/HTTPS routing to services
- Single entry point for multiple services
- Layer 7 load balancing
- SSL termination and path-based routing

### Ingress vs LoadBalancer
- **Ingress**: HTTP/HTTPS routing, single IP
- **LoadBalancer**: Layer 4, multiple IPs per service

## 🔧 Ingress Components

### Ingress Controller
- Implements ingress rules
- Popular controllers: NGINX, Traefik, HAProxy, Istio
- Must be deployed separately

### Ingress Resource
- Defines routing rules
- Specifies hosts, paths, and backends
- Configured via YAML manifests

## 🚀 Basic Ingress

### Simple Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: basic-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Path-based Routing
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 8080
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 8080
```

## 🌐 Host-based Routing

### Multiple Hosts
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  rules:
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

## 🔐 TLS/SSL Configuration

### TLS Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - secure.example.com
    secretName: tls-secret
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: secure-service
            port:
              number: 443
```

### TLS Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

## 📊 Path Types

### Exact
```yaml
- path: /api/v1/users
  pathType: Exact
```

### Prefix
```yaml
- path: /api
  pathType: Prefix
```

### ImplementationSpecific
```yaml
- path: /api/*
  pathType: ImplementationSpecific
```

## 🔧 Ingress Annotations

### NGINX Ingress Annotations
```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
```

### Traefik Annotations
```yaml
metadata:
  annotations:
    traefik.ingress.kubernetes.io/router.middlewares: default-auth@kubernetescrd
    traefik.ingress.kubernetes.io/router.tls: "true"
```

## 🚀 Ingress Operations

### Deploy Ingress Controller
```bash
# NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Check ingress controller
kubectl get pods -n ingress-nginx
```

### Manage Ingress Resources
```bash
# Create ingress
kubectl apply -f ingress.yaml

# List ingress
kubectl get ingress
kubectl get ing

# Describe ingress
kubectl describe ingress myapp-ingress

# Check ingress status
kubectl get ingress myapp-ingress -o wide
```

## 🚨 Troubleshooting Ingress

### Common Issues
```bash
# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Check ingress resource
kubectl describe ingress <ingress-name>

# Check backend services
kubectl get svc
kubectl describe svc <service-name>

# Check endpoints
kubectl get endpoints <service-name>
```

### Debug Commands
```bash
# Test ingress connectivity
curl -H "Host: myapp.example.com" http://<ingress-ip>/

# Check DNS resolution
nslookup myapp.example.com

# Verify TLS certificate
openssl s_client -connect myapp.example.com:443 -servername myapp.example.com
```

## 📋 Best Practices

### Design Guidelines
- Use meaningful host names
- Implement proper TLS certificates
- Configure rate limiting
- Use path-based routing efficiently

### Security
- Enable TLS for all ingress
- Use proper authentication
- Implement rate limiting
- Configure CORS policies

### Performance
- Use CDN for static content
- Configure caching headers
- Monitor ingress metrics
- Load test ingress rules