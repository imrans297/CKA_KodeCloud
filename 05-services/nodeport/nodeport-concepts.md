# NodePort Services

## 🎯 NodePort Concepts

### What is NodePort?
- Exposes service on each node's IP at a static port
- External traffic can access service via NodeIP:NodePort
- Port range: 30000-32767 (default)
- Built on top of ClusterIP service

## 🔧 NodePort Configuration

### Basic NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80          # Service port
    targetPort: 8080   # Pod port
    nodePort: 30080    # External port (optional)
```

### Auto-assigned NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: auto-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    # nodePort will be auto-assigned
```

## 🚀 Creating NodePort Services

### Imperative Commands
```bash
# Expose deployment as NodePort
kubectl expose deployment nginx --type=NodePort --port=80

# Expose with specific NodePort
kubectl expose deployment nginx --type=NodePort --port=80 --node-port=30080

# Create from command line
kubectl create service nodeport nginx --tcp=80:8080 --node-port=30080
```

### Declarative Method
```bash
# Apply YAML file
kubectl apply -f nodeport-service.yaml

# Generate YAML template
kubectl expose deployment nginx --type=NodePort --port=80 --dry-run=client -o yaml > nodeport.yaml
```

## 🔍 NodePort Operations

### Check NodePort Services
```bash
# List services
kubectl get services
kubectl get svc -o wide

# Show NodePort details
kubectl describe service nodeport-service

# Get service endpoints
kubectl get endpoints nodeport-service
```

### Access NodePort Service
```bash
# Get node IPs
kubectl get nodes -o wide

# Access via any node IP
curl <node-ip>:30080

# From within cluster
curl nodeport-service:80
```

## 📊 NodePort Traffic Flow

### External Access Flow
```
External Client → Node IP:NodePort → kube-proxy → Pod
```

### Internal Access Flow
```
Pod → Service ClusterIP:Port → kube-proxy → Pod
```

## 🔧 NodePort Configuration Options

### Multiple Ports
```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - name: http
    port: 80
    targetPort: 8080
    nodePort: 30080
  - name: https
    port: 443
    targetPort: 8443
    nodePort: 30443
```

### Session Affinity
```yaml
apiVersion: v1
kind: Service
metadata:
  name: sticky-nodeport
spec:
  type: NodePort
  sessionAffinity: ClientIP
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

## 🚨 NodePort Troubleshooting

### Common Issues
```bash
# Service not accessible externally
kubectl get svc nodeport-service
kubectl describe svc nodeport-service

# Check if pods are running
kubectl get pods -l app=web

# Check endpoints
kubectl get endpoints nodeport-service

# Test from node
ssh <node-ip>
curl localhost:30080
```

### Firewall Issues
```bash
# Check if NodePort is open
sudo netstat -tulpn | grep :30080

# Check iptables rules
sudo iptables -t nat -L | grep 30080

# Check cloud provider security groups
# (AWS, GCP, Azure firewall rules)
```

### Network Debugging
```bash
# Test connectivity from external client
telnet <node-ip> 30080

# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy

# Check service endpoints
kubectl describe endpoints nodeport-service
```

## 📋 NodePort Best Practices

### Security Considerations
- Limit NodePort range if needed
- Use security groups/firewalls
- Consider using LoadBalancer instead
- Implement proper authentication

### Performance Tips
- Use session affinity when needed
- Monitor node resource usage
- Consider pod anti-affinity
- Use health checks

### Production Usage
- Prefer LoadBalancer for production
- Use Ingress for HTTP/HTTPS traffic
- Document NodePort assignments
- Monitor service availability