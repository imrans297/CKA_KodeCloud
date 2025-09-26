# Service Commands & Operations

## Create Services

```bash
# Expose deployment as ClusterIP
kubectl expose deployment nginx-deployment --port=80 --target-port=80

# Expose as NodePort
kubectl expose deployment nginx-deployment --type=NodePort --port=80

# Expose as LoadBalancer
kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80

# Generate service YAML
kubectl expose deployment nginx-deployment --port=80 --dry-run=client -o yaml > service.yml

# Create from YAML
kubectl apply -f service-examples.yml
```

## Get Service Information

```bash
# List services
kubectl get services
kubectl get svc
kubectl get svc -o wide

# Detailed information
kubectl describe service nginx-service
kubectl get service nginx-service -o yaml

# Get service endpoints
kubectl get endpoints nginx-service
```

## Service Types

### ClusterIP (Default)
```bash
# Internal cluster access only
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

### NodePort
```bash
# External access via node IP:port
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl expose deployment nginx --port=80 --type=NodePort --node-port=30080
```

### LoadBalancer
```bash
# External load balancer (cloud provider)
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

### ExternalName
```bash
# DNS CNAME record
kubectl create service externalname my-service --external-name=example.com
```

## Test Services

```bash
# Test ClusterIP service
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- nginx-service

# Test NodePort service
curl <node-ip>:30080

# Port forward for testing
kubectl port-forward service/nginx-service 8080:80
curl localhost:8080

# Check service connectivity
kubectl exec -it <pod-name> -- nslookup nginx-service
kubectl exec -it <pod-name> -- curl nginx-service
```

## Update Services

```bash
# Edit service
kubectl edit service nginx-service

# Patch service
kubectl patch service nginx-service -p '{"spec":{"ports":[{"port":8080,"targetPort":80}]}}'

# Replace service
kubectl replace -f updated-service.yml
```

## Delete Services

```bash
# Delete service
kubectl delete service nginx-service

# Delete from file
kubectl delete -f service-examples.yml

# Delete all services
kubectl delete services --all
```

## Service Troubleshooting

```bash
# Check service status
kubectl get service nginx-service
kubectl describe service nginx-service

# Check endpoints
kubectl get endpoints nginx-service
kubectl describe endpoints nginx-service

# Check if pods are selected
kubectl get pods -l app=nginx --show-labels

# Test service resolution
kubectl run debug --image=busybox --rm -it --restart=Never -- nslookup nginx-service

# Check service from inside cluster
kubectl exec -it <pod-name> -- curl nginx-service:80

# Check iptables rules (on nodes)
sudo iptables -t nat -L | grep nginx-service
```

## Service Discovery

### DNS Resolution
```bash
# Service FQDN format
<service-name>.<namespace>.svc.cluster.local

# Examples
nginx-service.default.svc.cluster.local
web-service.production.svc.cluster.local
```

### Environment Variables
```bash
# Kubernetes automatically creates env vars for services
kubectl exec <pod-name> -- env | grep SERVICE
```

## Key Concepts

- **Selector**: Matches pods to include in service
- **Port**: Service port (what clients connect to)
- **TargetPort**: Pod port (where traffic is forwarded)
- **NodePort**: External port on nodes (30000-32767)
- **Endpoints**: List of pod IPs that match selector