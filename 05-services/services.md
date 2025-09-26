# Kubernetes Services

## 🎯 Service Concepts

### What is a Service?
A Service in Kubernetes is an abstraction that defines a logical set of pods and a policy to access them. Since pods are ephemeral (they can die or get replaced), Services provide a stable endpoint to communicate with pods.

### Purpose of Services
- **Stable Endpoint** - Provides consistent IP and DNS name
- **Load Balancing** - Distributes traffic across multiple pods
- **Service Discovery** - Enables pods to find and communicate with each other
- **Decoupling** - Separates clients from pod lifecycles
- **Abstraction** - Hides pod implementation details

## 📊 Service Types

### ClusterIP (Default)
- **Description**: Exposes service on cluster-internal IP
- **Access**: Only accessible within the cluster
- **Use Case**: Internal communication between services
- **Port Range**: Any available port

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
```

### NodePort
- **Description**: Exposes service on each node's IP at a static port
- **Access**: External access via NodeIP:NodePort
- **Use Case**: Development, testing, or simple external access
- **Port Range**: 30000-32767 (default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

### LoadBalancer
- **Description**: Creates external load balancer (cloud provider)
- **Access**: External access via cloud load balancer
- **Use Case**: Production external services
- **Requirements**: Cloud provider support

```yaml
apiVersion: v1
kind: Service
metadata:
  name: public-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 8080
```

### ExternalName
- **Description**: Maps service to external DNS name
- **Access**: DNS CNAME record
- **Use Case**: Access external services through cluster DNS
- **No Selectors**: Points to external resource

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  type: ExternalName
  externalName: api.external.com
```

## 🔧 Service Configuration

### Basic Service Structure
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: my-app
spec:
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  type: ClusterIP
```

### Multi-Port Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: webapp
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  - name: metrics
    port: 9090
    targetPort: 9090
```

### Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: database
  ports:
  - port: 5432
    targetPort: 5432
```

## 🌐 Service Discovery

### DNS-based Discovery
```bash
# Service FQDN format
<service-name>.<namespace>.svc.cluster.local

# Examples
web-service.default.svc.cluster.local
api-service.production.svc.cluster.local
```

### Environment Variables
```bash
# Kubernetes creates environment variables for services
<SERVICE_NAME>_SERVICE_HOST=10.96.0.1
<SERVICE_NAME>_SERVICE_PORT=80
```

## 📋 Service Operations

### Create Services
```bash
# Expose deployment as service
kubectl expose deployment nginx --port=80 --type=ClusterIP

# Create NodePort service
kubectl expose deployment nginx --port=80 --type=NodePort

# Create LoadBalancer service
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

### Manage Services
```bash
# List services
kubectl get services
kubectl get svc

# Describe service
kubectl describe service nginx

# Check service endpoints
kubectl get endpoints nginx

# Delete service
kubectl delete service nginx
```

### Test Services
```bash
# Test service from pod
kubectl exec -it <pod> -- curl <service-name>:<port>

# Test DNS resolution
kubectl exec -it <pod> -- nslookup <service-name>

# Port forward for testing
kubectl port-forward service/nginx 8080:80
```

## 🚨 Service Troubleshooting

### Common Issues
```bash
# Service not accessible
kubectl describe service <service-name>
kubectl get endpoints <service-name>

# No endpoints
kubectl get pods -l <selector-labels>
kubectl describe pod <pod-name>

# DNS issues
kubectl exec -it <pod> -- nslookup <service-name>
```

### Debug Commands
```bash
# Check service configuration
kubectl get service <service-name> -o yaml

# Test connectivity
kubectl run debug --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>

# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy
```

## 📊 Service Best Practices

### Design Guidelines
- Use meaningful service names
- Implement health checks in pods
- Configure appropriate timeouts
- Use labels consistently
- Plan for high availability

### Performance
- Use session affinity when needed
- Monitor service latency
- Optimize endpoint updates
- Consider headless services for direct access

### Security
- Implement network policies
- Use TLS for sensitive services
- Limit service exposure
- Regular security audits

## 🔗 Related Resources

For detailed examples and configurations, check:
- `clusterip/` - ClusterIP service examples
- `nodeport/` - NodePort service examples  
- `loadbalancer/` - LoadBalancer service examples
- `../09-networking/services/` - Advanced service networking concepts