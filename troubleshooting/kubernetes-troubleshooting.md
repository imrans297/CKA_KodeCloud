# Kubernetes Troubleshooting Guide

## 🎯 Overview

This comprehensive guide covers troubleshooting techniques for common Kubernetes issues across different components and layers.

## 🚨 Application Failure

### Common Application Issues

#### 1. Pod Stuck in Pending State
```bash
# Check pod status
kubectl get pods
kubectl describe pod <pod-name>

# Common causes and solutions:
```

**Insufficient Resources**
```bash
# Check node resources
kubectl top nodes
kubectl describe nodes

# Check resource requests/limits
kubectl describe pod <pod-name> | grep -A 5 "Requests\|Limits"

# Solution: Adjust resource requests or add more nodes
kubectl patch deployment <deployment-name> -p '{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","resources":{"requests":{"memory":"100Mi","cpu":"100m"}}}]}}}}'
```

**Node Selector Issues**
```bash
# Check node labels
kubectl get nodes --show-labels

# Check pod node selector
kubectl get pod <pod-name> -o yaml | grep -A 5 nodeSelector

# Solution: Update node labels or remove node selector
kubectl label nodes <node-name> <key>=<value>
```

**Taints and Tolerations**
```bash
# Check node taints
kubectl describe node <node-name> | grep Taints

# Check pod tolerations
kubectl get pod <pod-name> -o yaml | grep -A 10 tolerations

# Solution: Add toleration to pod
kubectl patch deployment <deployment-name> -p '{"spec":{"template":{"spec":{"tolerations":[{"key":"<taint-key>","operator":"Equal","value":"<taint-value>","effect":"NoSchedule"}]}}}}'
```

#### 2. Pod Stuck in ContainerCreating
```bash
# Check pod events
kubectl describe pod <pod-name>

# Check kubelet logs
sudo journalctl -u kubelet -f
```

**Image Pull Issues**
```bash
# Check image pull policy
kubectl get pod <pod-name> -o yaml | grep imagePullPolicy

# Check image name and tag
kubectl get pod <pod-name> -o yaml | grep image:

# Test image pull manually
docker pull <image-name>

# Solution: Fix image name or add image pull secrets
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password>

kubectl patch deployment <deployment-name> -p '{"spec":{"template":{"spec":{"imagePullSecrets":[{"name":"regcred"}]}}}}'
```

**Volume Mount Issues**
```bash
# Check volume mounts
kubectl describe pod <pod-name> | grep -A 10 "Mounts\|Volumes"

# Check PVC status
kubectl get pvc
kubectl describe pvc <pvc-name>

# Check storage class
kubectl get storageclass
kubectl describe storageclass <storage-class-name>

# Solution: Fix PVC or storage class issues
kubectl get pv
kubectl describe pv <pv-name>
```

#### 3. Pod CrashLoopBackOff
```bash
# Check pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Check restart count
kubectl get pods
kubectl describe pod <pod-name>
```

**Application Configuration Issues**
```bash
# Check environment variables
kubectl get pod <pod-name> -o yaml | grep -A 20 env:

# Check config maps and secrets
kubectl get configmap
kubectl describe configmap <configmap-name>
kubectl get secret
kubectl describe secret <secret-name>

# Debug with interactive shell
kubectl exec -it <pod-name> -- /bin/sh
kubectl exec -it <pod-name> -- env
```

**Resource Limits**
```bash
# Check resource usage
kubectl top pod <pod-name>

# Check limits and requests
kubectl describe pod <pod-name> | grep -A 10 "Limits\|Requests"

# Solution: Adjust resource limits
kubectl patch deployment <deployment-name> -p '{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","resources":{"limits":{"memory":"512Mi","cpu":"500m"}}}]}}}}'
```

**Health Check Failures**
```bash
# Check liveness and readiness probes
kubectl describe pod <pod-name> | grep -A 5 "Liveness\|Readiness"

# Test probe endpoints manually
kubectl exec -it <pod-name> -- curl localhost:8080/health

# Solution: Fix probe configuration
kubectl patch deployment <deployment-name> -p '{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","livenessProbe":{"httpGet":{"path":"/health","port":8080},"initialDelaySeconds":30,"periodSeconds":10}}]}}}}'
```

#### 4. Service Connection Issues
```bash
# Check service endpoints
kubectl get endpoints <service-name>
kubectl describe service <service-name>

# Test service connectivity
kubectl run test-pod --image=busybox -it --rm -- wget -qO- <service-name>:<port>

# Check service selector
kubectl get service <service-name> -o yaml | grep -A 5 selector
kubectl get pods --show-labels
```

**DNS Resolution Issues**
```bash
# Test DNS resolution
kubectl run test-dns --image=busybox -it --rm -- nslookup <service-name>
kubectl run test-dns --image=busybox -it --rm -- nslookup <service-name>.<namespace>.svc.cluster.local

# Check CoreDNS
kubectl get pods -n kube-system | grep coredns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### Application Troubleshooting Commands
```bash
# Pod inspection
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name> -f
kubectl logs <pod-name> --previous
kubectl exec -it <pod-name> -- /bin/sh

# Resource debugging
kubectl top pods
kubectl top nodes
kubectl get events --sort-by=.metadata.creationTimestamp

# Configuration debugging
kubectl get configmap <name> -o yaml
kubectl get secret <name> -o yaml
kubectl describe deployment <name>
```

## 🎛️ Control Plane Failure

### API Server Issues

#### 1. API Server Not Responding
```bash
# Check API server status
kubectl cluster-info
kubectl get nodes

# Check API server pod
kubectl get pods -n kube-system | grep apiserver
kubectl describe pod -n kube-system <apiserver-pod>
```

**API Server Logs**
```bash
# Check API server logs
sudo journalctl -u kube-apiserver -f
kubectl logs -n kube-system <kube-apiserver-pod>

# Check API server configuration
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

**Certificate Issues**
```bash
# Check certificate expiration
sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep "Not After"

# Check certificate chain
sudo openssl verify -CAfile /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/apiserver.crt

# Regenerate certificates if needed
sudo kubeadm certs renew all
sudo systemctl restart kubelet
```

**Port and Connectivity Issues**
```bash
# Check if API server port is listening
sudo netstat -tlnp | grep 6443
sudo ss -tlnp | grep 6443

# Test API server connectivity
curl -k https://localhost:6443/version
telnet <master-ip> 6443
```

#### 2. etcd Issues
```bash
# Check etcd status
kubectl get pods -n kube-system | grep etcd
kubectl describe pod -n kube-system <etcd-pod>
```

**etcd Health Check**
```bash
# Check etcd health
sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health

# Check etcd members
sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  member list
```

**etcd Backup and Restore**
```bash
# Backup etcd
sudo ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Restore etcd
sudo systemctl stop kubelet
sudo systemctl stop etcd
sudo ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore
sudo mv /var/lib/etcd /var/lib/etcd-backup
sudo mv /var/lib/etcd-restore /var/lib/etcd
sudo systemctl start etcd
sudo systemctl start kubelet
```

#### 3. Controller Manager Issues
```bash
# Check controller manager
kubectl get pods -n kube-system | grep controller-manager
kubectl describe pod -n kube-system <controller-manager-pod>
kubectl logs -n kube-system <controller-manager-pod>

# Check controller manager logs
sudo journalctl -u kube-controller-manager -f
```

**Leader Election Issues**
```bash
# Check leader election
kubectl get endpoints -n kube-system kube-controller-manager
kubectl describe endpoints -n kube-system kube-controller-manager

# Check controller manager configuration
sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

#### 4. Scheduler Issues
```bash
# Check scheduler status
kubectl get pods -n kube-system | grep scheduler
kubectl describe pod -n kube-system <scheduler-pod>
kubectl logs -n kube-system <scheduler-pod>

# Check scheduler logs
sudo journalctl -u kube-scheduler -f
```

**Scheduling Problems**
```bash
# Check pending pods
kubectl get pods --field-selector=status.phase=Pending

# Check scheduler events
kubectl get events --field-selector reason=FailedScheduling

# Debug scheduling decisions
kubectl describe pod <pending-pod-name>
```

### Control Plane Troubleshooting Commands
```bash
# System component status
kubectl get componentstatuses
kubectl cluster-info
kubectl cluster-info dump

# Control plane pods
kubectl get pods -n kube-system
kubectl describe pods -n kube-system

# System logs
sudo journalctl -u kubelet -f
sudo journalctl -u kube-apiserver -f
sudo journalctl -u kube-controller-manager -f
sudo journalctl -u kube-scheduler -f
sudo journalctl -u etcd -f
```

## 🖥️ Worker Node Failure

### Node Status Issues

#### 1. Node NotReady Status
```bash
# Check node status
kubectl get nodes
kubectl describe node <node-name>

# Check node conditions
kubectl get nodes -o wide
kubectl describe node <node-name> | grep -A 10 Conditions
```

**Kubelet Issues**
```bash
# Check kubelet status
sudo systemctl status kubelet
sudo journalctl -u kubelet -f

# Check kubelet configuration
sudo cat /var/lib/kubelet/config.yaml
sudo cat /etc/kubernetes/kubelet.conf

# Restart kubelet
sudo systemctl restart kubelet
sudo systemctl enable kubelet
```

**Container Runtime Issues**
```bash
# Check container runtime (Docker/containerd)
sudo systemctl status docker
sudo systemctl status containerd

# Check container runtime logs
sudo journalctl -u docker -f
sudo journalctl -u containerd -f

# Test container runtime
sudo docker ps
sudo crictl ps
```

**Network Plugin Issues**
```bash
# Check CNI plugins
ls /opt/cni/bin/
ls /etc/cni/net.d/

# Check network plugin pods
kubectl get pods -n kube-system | grep -E "(flannel|calico|weave)"
kubectl logs -n kube-system <network-plugin-pod>

# Check node network configuration
ip addr show
ip route show
```

#### 2. Resource Exhaustion
```bash
# Check node resources
kubectl top node <node-name>
kubectl describe node <node-name> | grep -A 10 "Allocated resources"

# Check disk usage
df -h
du -sh /var/lib/docker
du -sh /var/lib/kubelet

# Check memory usage
free -h
cat /proc/meminfo

# Check CPU usage
top
htop
```

**Disk Pressure**
```bash
# Clean up Docker images and containers
sudo docker system prune -a
sudo docker volume prune

# Clean up kubelet cache
sudo rm -rf /var/lib/kubelet/pods/*

# Check for large log files
sudo find /var/log -type f -size +100M
sudo journalctl --vacuum-time=7d
```

**Memory Pressure**
```bash
# Check memory usage by pods
kubectl top pods --all-namespaces --sort-by=memory

# Check system memory
cat /proc/meminfo
free -h

# Check for memory leaks
ps aux --sort=-%mem | head -10
```

#### 3. Certificate Issues
```bash
# Check kubelet certificates
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout | grep "Not After"

# Check node certificate
kubectl get csr
kubectl describe csr <csr-name>

# Approve pending CSRs
kubectl certificate approve <csr-name>

# Regenerate kubelet certificates
sudo rm /var/lib/kubelet/pki/kubelet-client*
sudo systemctl restart kubelet
```

### Pod Eviction Issues
```bash
# Check evicted pods
kubectl get pods --all-namespaces | grep Evicted

# Check eviction events
kubectl get events --field-selector reason=Evicted

# Clean up evicted pods
kubectl get pods --all-namespaces | grep Evicted | awk '{print $2 " -n " $1}' | xargs kubectl delete pod
```

### Worker Node Troubleshooting Commands
```bash
# Node inspection
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl top nodes

# System services
sudo systemctl status kubelet
sudo systemctl status docker
sudo systemctl status containerd

# Resource monitoring
df -h
free -h
top
kubectl top pods --all-namespaces

# Network debugging
ip addr show
ip route show
ping <other-node-ip>
```

## 🌐 Networking Troubleshooting

### Pod-to-Pod Communication

#### 1. Pod Network Connectivity
```bash
# Test pod-to-pod communication
kubectl exec -it <source-pod> -- ping <target-pod-ip>
kubectl exec -it <source-pod> -- nc -zv <target-pod-ip> <port>

# Check pod IPs and network
kubectl get pods -o wide
kubectl describe pod <pod-name> | grep IP
```

**CNI Plugin Issues**
```bash
# Check CNI configuration
ls /etc/cni/net.d/
cat /etc/cni/net.d/*.conf

# Check CNI plugin pods
kubectl get pods -n kube-system | grep -E "(flannel|calico|weave|cilium)"
kubectl logs -n kube-system <cni-pod> -c <container>

# Check network interfaces
ip addr show
ip link show
```

**Bridge and Routing Issues**
```bash
# Check bridge configuration
brctl show
ip link show type bridge

# Check routing tables
ip route show
route -n

# Check iptables rules
sudo iptables -L -n -v
sudo iptables -t nat -L -n -v
```

#### 2. Service Discovery Issues
```bash
# Test service resolution
kubectl exec -it <pod> -- nslookup <service-name>
kubectl exec -it <pod> -- nslookup <service-name>.<namespace>.svc.cluster.local

# Check service endpoints
kubectl get endpoints <service-name>
kubectl describe service <service-name>

# Test service connectivity
kubectl exec -it <pod> -- curl <service-name>:<port>
kubectl exec -it <pod> -- telnet <service-name> <port>
```

**DNS Issues**
```bash
# Check CoreDNS
kubectl get pods -n kube-system | grep coredns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check DNS configuration in pods
kubectl exec -it <pod> -- cat /etc/resolv.conf

# Test DNS resolution
kubectl exec -it <pod> -- dig <service-name>
kubectl exec -it <pod> -- nslookup kubernetes.default
```

#### 3. Ingress and Load Balancer Issues
```bash
# Check ingress controller
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <ingress-controller-pod>

# Check ingress resources
kubectl get ingress
kubectl describe ingress <ingress-name>

# Test ingress connectivity
curl -H "Host: <hostname>" http://<ingress-ip>/<path>
```

**Load Balancer Issues**
```bash
# Check load balancer service
kubectl get service <lb-service> -o wide
kubectl describe service <lb-service>

# Check cloud provider integration
kubectl logs -n kube-system <cloud-controller-manager>

# Test load balancer connectivity
curl http://<external-ip>:<port>
telnet <external-ip> <port>
```

### Network Policy Troubleshooting
```bash
# Check network policies
kubectl get networkpolicy
kubectl describe networkpolicy <policy-name>

# Check if CNI supports network policies
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.containerRuntimeVersion}'

# Test connectivity with network policies
kubectl exec -it <source-pod> -- nc -zv <target-pod-ip> <port>

# Debug network policy rules
kubectl logs -n kube-system <cni-pod> | grep -i "policy\|denied"
```

### Networking Troubleshooting Commands
```bash
# Network connectivity
ping <target-ip>
traceroute <target-ip>
mtr <target-ip>
nc -zv <target-ip> <port>

# DNS debugging
nslookup <hostname>
dig <hostname>
host <hostname>

# Network interface debugging
ip addr show
ip route show
ip neigh show
ss -tuln

# Packet capture
tcpdump -i <interface> -n
kubectl exec -it <pod> -- tcpdump -i eth0 -n

# Service mesh debugging (if applicable)
istioctl proxy-status
istioctl proxy-config cluster <pod-name>
```

## 🔧 General Troubleshooting Tools

### Essential Commands
```bash
# Cluster information
kubectl cluster-info
kubectl cluster-info dump
kubectl version

# Resource inspection
kubectl get all --all-namespaces
kubectl describe <resource-type> <resource-name>
kubectl logs <pod-name> -f --previous

# Events and debugging
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector type=Warning
kubectl top nodes
kubectl top pods --all-namespaces

# Configuration debugging
kubectl config view
kubectl config current-context
kubectl auth can-i <verb> <resource>
```

### Debugging Tools
```bash
# Network debugging pod
kubectl run netshoot --rm -i --tty --image nicolaka/netshoot -- /bin/bash

# Debug pod with tools
kubectl run debug --rm -i --tty --image busybox -- /bin/sh

# Temporary debug container (K8s 1.18+)
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# Port forwarding for debugging
kubectl port-forward <pod-name> <local-port>:<pod-port>
kubectl port-forward service/<service-name> <local-port>:<service-port>
```

### Log Analysis
```bash
# System logs
sudo journalctl -u kubelet -f
sudo journalctl -u docker -f
sudo journalctl -u containerd -f

# Application logs
kubectl logs <pod-name> -c <container-name>
kubectl logs -l app=<label-value>
kubectl logs --tail=100 <pod-name>

# Log aggregation
kubectl logs -f deployment/<deployment-name>
kubectl logs --since=1h <pod-name>
```

```bash
Network Plugin in Kubernetes

--------------------

There are several plugins available and these are some.


1. Weave Net:


To install,


kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml


You can find details about the network plugins in the following documentation :

https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy


2. Flannel :


 To install,

 

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml

   

Note: As of now flannel does not support kubernetes network policies.


3. Calico :

   

   To install,

   curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O

  Apply the manifest using the following command.

      kubectl apply -f calico.yaml

   Calico is said to have most advanced cni network plugin.


In CKA and CKAD exam, you won't be asked to install the CNI plugin. But if asked you will be provided with the exact URL to install it.

Note: If there are multiple CNI configuration files in the directory, the kubelet uses the configuration file that comes first by name in lexicographic order.



DNS in Kubernetes
-----------------

Kubernetes uses CoreDNS. CoreDNS is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS.


Memory and Pods

In large scale Kubernetes clusters, CoreDNS's memory usage is predominantly affected by the number of Pods and Services in the cluster. Other factors include the size of the filled DNS answer cache, and the rate of queries received (QPS) per CoreDNS instance.


Kubernetes resources for coreDNS are:   

    a service account named coredns,

    cluster-roles named coredns and kube-dns

    clusterrolebindings named coredns and kube-dns, 

    a deployment named coredns,

    a configmap named coredns and a

    service named kube-dns.


While analyzing the coreDNS deployment you can see that the the Corefile plugin consists of important configuration which is defined as a configmap.


Port 53 is used for for DNS resolution.


        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }


This is the backend to k8s for cluster.local and reverse domains.


proxy . /etc/resolv.conf


Forward out of cluster domains directly to right authoritative DNS server.



Troubleshooting issues related to coreDNS

1. If you find CoreDNS pods in pending state first check network plugin is installed.

2. coredns pods have CrashLoopBackOff or Error state

If you have nodes that are running SELinux with an older version of Docker you might experience a scenario where the coredns pods are not starting. To solve that you can try one of the following options:

a)Upgrade to a newer version of Docker.

b)Disable SELinux.

c)Modify the coredns deployment to set allowPrivilegeEscalation to true:


    kubectl -n kube-system get deployment coredns -o yaml | \
      sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
      kubectl apply -f -

d)Another cause for CoreDNS to have CrashLoopBackOff is when a CoreDNS Pod deployed in Kubernetes detects a loop.


  There are many ways to work around this issue, some are listed here:


    Add the following to your kubelet config yaml: resolvConf: <path-to-your-real-resolv-conf-file> This flag tells kubelet to pass an alternate resolv.conf to Pods. For systems using systemd-resolved, /run/systemd/resolve/resolv.conf is typically the location of the "real" resolv.conf, although this can be different depending on your distribution.

    Disable the local DNS cache on host nodes, and restore /etc/resolv.conf to the original.

    A quick fix is to edit your Corefile, replacing forward . /etc/resolv.conf with the IP address of your upstream DNS, for example forward . 8.8.8.8. But this only fixes the issue for CoreDNS, kubelet will continue to forward the invalid resolv.conf to all default dnsPolicy Pods, leaving them unable to resolve DNS.


3. If CoreDNS pods and the kube-dns service is working fine, check the kube-dns service has valid endpoints.

              kubectl -n kube-system get ep kube-dns

If there are no endpoints for the service, inspect the service and make sure it uses the correct selectors and ports.



Kube-Proxy
---------

kube-proxy is a network proxy that runs on each node in the cluster. kube-proxy maintains network rules on nodes. These network rules allow network communication to the Pods from network sessions inside or outside of the cluster.


In a cluster configured with kubeadm, you can find kube-proxy as a daemonset.


kubeproxy is responsible for watching services and endpoint associated with each service. When the client is going to connect to the service using the virtual IP the kubeproxy is responsible for sending traffic to actual pods.


If you run a kubectl describe ds kube-proxy -n kube-system you can see that the kube-proxy binary runs with following command inside the kube-proxy container.


        Command:
          /usr/local/bin/kube-proxy
          --config=/var/lib/kube-proxy/config.conf
          --hostname-override=$(NODE_NAME)

 

    So it fetches the configuration from a configuration file ie, /var/lib/kube-proxy/config.conf and we can override the hostname with the node name of at which the pod is running.

 

  In the config file we define the clusterCIDR, kubeproxy mode, ipvs, iptables, bindaddress, kube-config etc.

 
Troubleshooting issues related to kube-proxy

1. Check kube-proxy pod in the kube-system namespace is running.

2. Check kube-proxy logs.

3. Check configmap is correctly defined and the config file for running kube-proxy binary is correct.

4. kube-config is defined in the config map.

5. check kube-proxy is running inside the container

    # netstat -plan | grep kube-proxy
    tcp        0      0 0.0.0.0:30081           0.0.0.0:*               LISTEN      1/kube-proxy
    tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      1/kube-proxy
    tcp        0      0 172.17.0.12:33706       172.17.0.12:6443        ESTABLISHED 1/kube-proxy
    tcp6       0      0 :::10256                :::*                    LISTEN      1/kube-proxy



References:

Debug Service issues:

                     https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

DNS Troubleshooting:

                     https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

```

This comprehensive troubleshooting guide provides systematic approaches to diagnose and resolve common Kubernetes issues across all major components and failure scenarios.