# LoadBalancer Services

## 🎯 LoadBalancer Concepts

### What is LoadBalancer?
- Exposes service externally using cloud provider's load balancer
- Automatically provisions external load balancer
- Built on top of NodePort service
- Provides single external IP for service access

## ☁️ Cloud Provider Integration

### Supported Providers
- **AWS** - Elastic Load Balancer (ELB/ALB/NLB)
- **GCP** - Google Cloud Load Balancer
- **Azure** - Azure Load Balancer
- **DigitalOcean** - DigitalOcean Load Balancer

### Requirements
- Kubernetes cluster on supported cloud provider
- Cloud controller manager configured
- Proper IAM permissions for load balancer creation

## 🔧 LoadBalancer Configuration

### Basic LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

### LoadBalancer with Annotations
```yaml
apiVersion: v1
kind: Service
metadata:
  name: aws-loadbalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

## 🚀 Creating LoadBalancer Services

### Imperative Commands
```bash
# Expose deployment as LoadBalancer
kubectl expose deployment nginx --type=LoadBalancer --port=80

# Create LoadBalancer service
kubectl create service loadbalancer nginx --tcp=80:8080
```

### Declarative Method
```bash
# Apply YAML file
kubectl apply -f loadbalancer-service.yaml

# Generate YAML template
kubectl expose deployment nginx --type=LoadBalancer --port=80 --dry-run=client -o yaml > lb.yaml
```

## 🔍 LoadBalancer Operations

### Check LoadBalancer Status
```bash
# List services (check EXTERNAL-IP)
kubectl get services
kubectl get svc -o wide

# Detailed service information
kubectl describe service loadbalancer-service

# Check service endpoints
kubectl get endpoints loadbalancer-service
```

### Access LoadBalancer Service
```bash
# Get external IP
EXTERNAL_IP=$(kubectl get svc loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# Access via external IP
curl http://$EXTERNAL_IP

# Or via hostname (some providers)
EXTERNAL_HOSTNAME=$(kubectl get svc loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$EXTERNAL_HOSTNAME
```

## 📊 LoadBalancer Traffic Flow

### External Access Flow
```
Internet → Cloud LB → NodePort → kube-proxy → Pod
```

### Load Balancer Provisioning
```
1. Service created with type: LoadBalancer
2. Cloud controller manager detects service
3. Provisions external load balancer
4. Configures load balancer to point to nodes
5. Updates service status with external IP
```

## ☁️ Cloud-Specific Configurations

### AWS LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: aws-nlb-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

### GCP LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: gcp-lb-service
  annotations:
    cloud.google.com/load-balancer-type: "External"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

### Azure LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-lb-service
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "false"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

## 🚨 LoadBalancer Troubleshooting

### Common Issues
```bash
# External IP stuck in <pending>
kubectl describe service loadbalancer-service
kubectl get events --field-selector involvedObject.name=loadbalancer-service

# Check cloud controller manager logs
kubectl logs -n kube-system -l component=cloud-controller-manager

# Verify cloud provider configuration
kubectl get nodes -o yaml | grep providerID
```

### Debugging Steps
```bash
# 1. Check service configuration
kubectl get svc loadbalancer-service -o yaml

# 2. Check if cloud controller is running
kubectl get pods -n kube-system | grep cloud-controller

# 3. Check IAM permissions (cloud-specific)
# AWS: Check if cluster has proper ELB permissions
# GCP: Check if service account has compute.loadBalancers.* permissions

# 4. Check node security groups/firewall rules
kubectl get nodes -o wide
```

## 💰 Cost Considerations

### LoadBalancer Costs
- Each LoadBalancer service creates a cloud load balancer
- Cloud providers charge for load balancer usage
- Consider using Ingress for multiple services
- Monitor and clean up unused LoadBalancer services

### Cost Optimization
```bash
# Use single LoadBalancer with Ingress
# Instead of multiple LoadBalancer services

# Check for unused LoadBalancer services
kubectl get svc --all-namespaces | grep LoadBalancer

# Delete unused services
kubectl delete svc unused-loadbalancer-service
```

## 📋 LoadBalancer Best Practices

### Design Considerations
- Use Ingress for HTTP/HTTPS traffic
- Reserve LoadBalancer for TCP/UDP services
- Consider internal load balancers for private services
- Plan for high availability across zones

### Security
- Use security groups/firewall rules
- Implement proper SSL/TLS termination
- Consider WAF integration
- Monitor access logs

### Monitoring
- Track load balancer health
- Monitor backend target health
- Set up alerts for service unavailability
- Monitor costs and usage