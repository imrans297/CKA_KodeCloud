# Kubernetes Logging

## 🎯 Logging Concepts

### What is Kubernetes Logging?
- **Container Logs** - Application output from containers
- **System Logs** - Kubernetes component logs
- **Audit Logs** - API server access logs
- **Event Logs** - Cluster events and state changes

### Logging Architecture
- **Node-level logging** - kubelet and container runtime
- **Cluster-level logging** - Centralized log aggregation
- **Application-level logging** - Custom application logs

## 📊 Log Types

### Container Logs
```bash
# View pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>

# Follow logs
kubectl logs -f <pod-name>

# Previous container logs
kubectl logs <pod-name> --previous

# Logs with timestamps
kubectl logs <pod-name> --timestamps
```

### System Component Logs
```bash
# API server logs
kubectl logs -n kube-system kube-apiserver-master

# Controller manager logs
kubectl logs -n kube-system kube-controller-manager-master

# Scheduler logs
kubectl logs -n kube-system kube-scheduler-master

# kubelet logs (on nodes)
journalctl -u kubelet -f
```

### Cluster Events
```bash
# View cluster events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp

# Events for specific resource
kubectl get events --field-selector involvedObject.name=<pod-name>

# Events in specific namespace
kubectl get events -n <namespace>
```

## 🔧 Log Collection

### Fluentd DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch.logging.svc.cluster.local"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### Filebeat DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      containers:
      - name: filebeat
        image: docker.elastic.co/beats/filebeat:7.15.0
        args:
        - -c
        - /etc/filebeat.yml
        - -e
        volumeMounts:
        - name: config
          mountPath: /etc/filebeat.yml
          subPath: filebeat.yml
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: filebeat-config
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## 📋 Log Management

### Log Rotation
```bash
# Configure log rotation for containers
# In Docker daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}

# For containerd
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
  
[plugins."io.containerd.grpc.v1.cri"]
  max_container_log_line_size = 16384
```

### Log Aggregation
```bash
# ELK Stack components
# Elasticsearch - Log storage
# Logstash - Log processing
# Kibana - Log visualization

# EFK Stack components  
# Elasticsearch - Log storage
# Fluentd - Log collection
# Kibana - Log visualization
```

## 🔍 Log Analysis

### Log Queries
```bash
# Search logs with grep
kubectl logs <pod-name> | grep ERROR

# Filter logs by time
kubectl logs <pod-name> --since=1h
kubectl logs <pod-name> --since-time=2023-01-01T10:00:00Z

# Tail logs
kubectl logs <pod-name> --tail=100
```

### Structured Logging
```json
{
  "timestamp": "2023-01-01T10:00:00Z",
  "level": "INFO",
  "message": "Request processed",
  "request_id": "abc123",
  "user_id": "user456",
  "duration_ms": 150
}
```

## 🚨 Log Troubleshooting

### Common Issues
```bash
# Logs not appearing
kubectl describe pod <pod-name>
kubectl get events --field-selector involvedObject.name=<pod-name>

# Log collection not working
kubectl logs -n kube-system -l app=fluentd

# High log volume
kubectl top pods --sort-by=memory
df -h /var/log
```

### Debug Commands
```bash
# Check log files on nodes
ssh <node-ip>
ls -la /var/log/containers/
ls -la /var/lib/docker/containers/

# Check log collection agents
kubectl get pods -n kube-system | grep -E "fluentd|filebeat|logstash"

# Test log forwarding
kubectl exec -it <log-collector-pod> -- curl elasticsearch:9200/_cluster/health
```

## 📊 Best Practices

### Application Logging
- Use structured logging (JSON)
- Include correlation IDs
- Log at appropriate levels
- Avoid logging sensitive data
- Use consistent timestamp formats

### Infrastructure Logging
- Centralize log collection
- Implement log retention policies
- Monitor log collection health
- Secure log transmission
- Plan for log storage capacity

### Performance
- Use efficient log collectors
- Implement log sampling for high-volume apps
- Compress logs during transmission
- Use appropriate log levels
- Monitor log collection overhead