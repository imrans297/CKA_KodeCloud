# TLS Certificates in Kubernetes

## 🎯 TLS Certificate Concepts

### What are TLS Certificates?
- Secure communication between components
- Authentication and encryption
- Public Key Infrastructure (PKI)
- Certificate Authority (CA) based trust

### Kubernetes Certificate Usage
- API server authentication
- kubelet communication
- etcd encryption
- Service mesh security

## 🔧 Kubernetes PKI

### Certificate Authority (CA)
```bash
# Kubernetes CA certificate location
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/ca.key
```

### Component Certificates
```bash
# API Server certificates
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/apiserver.key

# ETCD certificates
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key

# Kubelet certificates
/var/lib/kubelet/pki/kubelet.crt
/var/lib/kubelet/pki/kubelet.key
```

## 🔐 Certificate Types

### Server Certificates
- API server certificate
- ETCD server certificate
- Kubelet server certificate

### Client Certificates
- Admin user certificate
- Kubelet client certificate
- Controller manager certificate
- Scheduler certificate

### Peer Certificates
- ETCD peer certificates
- Service mesh certificates

## 🚀 Certificate Operations

### View Certificates
```bash
# View certificate details
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

# Check certificate expiration
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates

# Verify certificate chain
openssl verify -CAfile /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/apiserver.crt
```

### Generate Certificates
```bash
# Generate private key
openssl genrsa -out server.key 2048

# Generate certificate signing request
openssl req -new -key server.key -out server.csr -subj "/CN=myserver"

# Generate certificate
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
```

## 📋 TLS Secrets

### Create TLS Secret
```bash
# Create TLS secret from files
kubectl create secret tls my-tls-secret \
  --cert=server.crt \
  --key=server.key
```

### TLS Secret YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-certificate>
  tls.key: <base64-encoded-private-key>
```

## 🌐 Ingress TLS

### TLS Ingress Configuration
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
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

## 🔧 Certificate Management

### Certificate Signing Requests (CSR)
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser-csr
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

### Approve CSR
```bash
# List CSRs
kubectl get csr

# Approve CSR
kubectl certificate approve myuser-csr

# Get approved certificate
kubectl get csr myuser-csr -o jsonpath='{.status.certificate}' | base64 -d > myuser.crt
```

## 🔍 Certificate Troubleshooting

### Common Issues
```bash
# Certificate expired
kubectl get pods -n kube-system

# Check API server logs
kubectl logs -n kube-system kube-apiserver-master

# Verify certificate dates
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates
```

### Debug Commands
```bash
# Test TLS connection
openssl s_client -connect kubernetes-api:6443 -servername kubernetes

# Check certificate chain
curl -k -v https://kubernetes-api:6443/

# Verify kubelet certificates
curl -k https://node-ip:10250/metrics
```

## 🔄 Certificate Rotation

### Automatic Rotation
```bash
# Enable kubelet certificate rotation
--rotate-certificates=true
--rotate-server-certificates=true
```

### Manual Rotation
```bash
# Backup existing certificates
cp -r /etc/kubernetes/pki /etc/kubernetes/pki.backup

# Generate new certificates
kubeadm certs renew all

# Restart components
systemctl restart kubelet
```

## 📊 Certificate Monitoring

### Check Certificate Expiration
```bash
# Check all certificates
kubeadm certs check-expiration

# Monitor certificate expiry
for cert in /etc/kubernetes/pki/*.crt; do
  echo "Certificate: $cert"
  openssl x509 -in "$cert" -noout -dates
  echo "---"
done
```

### Certificate Metrics
```bash
# Prometheus metrics for certificate expiry
curl -k https://kubernetes-api:6443/metrics | grep certificate
```

## 🛡️ Security Best Practices

### Certificate Security
- Use strong key lengths (2048+ bits)
- Regular certificate rotation
- Secure private key storage
- Monitor certificate expiration

### PKI Management
- Secure CA private keys
- Use intermediate CAs
- Implement certificate policies
- Audit certificate usage

### Automation
- Automated certificate renewal
- Certificate lifecycle management
- Integration with external CAs
- Monitoring and alerting