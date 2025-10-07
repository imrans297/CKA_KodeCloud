# Docker Storage Deep Dive

## 🎯 Overview

Docker storage manages how containers store and persist data through various storage drivers, volume types, and the Container Storage Interface (CSI).

## 💾 Docker Storage Architecture

### Storage Layers
```
┌─────────────────────────────────────┐
│           Container Layer           │  ← Read/Write Layer
├─────────────────────────────────────┤
│          Image Layer N              │  ← Read-Only
├─────────────────────────────────────┤
│          Image Layer 2              │  ← Read-Only  
├─────────────────────────────────────┤
│          Image Layer 1              │  ← Read-Only
└─────────────────────────────────────┘
```

### Storage Driver Types
- **overlay2** - Default, most efficient
- **aufs** - Legacy Ubuntu
- **devicemapper** - Red Hat/CentOS
- **btrfs** - BTRFS filesystem
- **zfs** - ZFS filesystem
- **vfs** - No copy-on-write (testing only)

## 🔧 Docker Storage Drivers

### Check Current Storage Driver
```bash
# View storage driver info
docker info | grep "Storage Driver"
docker system info | grep -A 10 "Storage Driver"

# Detailed storage info
docker system df
docker system df -v
```

### Configure Storage Driver
```bash
# /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}

# Restart Docker
sudo systemctl restart docker
```

### Overlay2 Driver (Recommended)
```bash
# Overlay2 structure
/var/lib/docker/overlay2/
├── <container-id>/
│   ├── diff/          # Container changes
│   ├── lower          # Lower layer reference
│   ├── merged/        # Merged view
│   ├── upper/         # Upper layer
│   └── work/          # Work directory
```

### DeviceMapper Driver
```bash
# DeviceMapper configuration
{
  "storage-driver": "devicemapper",
  "storage-opts": [
    "dm.thinpooldev=/dev/mapper/docker-thinpool",
    "dm.use_deferred_removal=true",
    "dm.use_deferred_deletion=true"
  ]
}

# Check devicemapper info
docker info | grep -A 20 "Storage Driver"
```

## 📁 Docker Volume Types

### 1. Bind Mounts
```bash
# Bind mount host directory
docker run -v /host/path:/container/path nginx

# Read-only bind mount
docker run -v /host/path:/container/path:ro nginx

# Bind mount with specific options
docker run --mount type=bind,source=/host/path,target=/container/path,readonly nginx
```

### 2. Named Volumes
```bash
# Create named volume
docker volume create myvolume

# Use named volume
docker run -v myvolume:/data nginx

# Mount with options
docker run --mount source=myvolume,target=/data nginx
```

### 3. Anonymous Volumes
```bash
# Create anonymous volume
docker run -v /data nginx

# Anonymous volume in Dockerfile
FROM nginx
VOLUME ["/data"]
```

### 4. tmpfs Mounts (Memory)
```bash
# tmpfs mount
docker run --tmpfs /tmp nginx

# tmpfs with options
docker run --mount type=tmpfs,destination=/tmp,tmpfs-size=100m nginx
```

## 🛠️ Volume Management Commands

### Volume Operations
```bash
# List volumes
docker volume ls

# Create volume with driver
docker volume create --driver local myvolume

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune

# Remove all volumes
docker volume rm $(docker volume ls -q)
```

### Volume with Options
```bash
# Create volume with specific options
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  nfs-volume

# Use the NFS volume
docker run -v nfs-volume:/data nginx
```

## 🔌 Volume Driver Plugins

### Local Driver (Default)
```bash
# Local driver stores data on Docker host
docker volume create --driver local mylocal

# Local driver with bind mount
docker volume create --driver local \
  --opt type=none \
  --opt o=bind \
  --opt device=/host/path \
  mylocal
```

### NFS Driver Plugin
```bash
# Install NFS plugin
docker plugin install store/sumologic/docker-sumologic-driver:1.0.0

# Create NFS volume
docker volume create --driver nfs \
  --opt share=192.168.1.100:/nfs/share \
  nfs-volume

# Use NFS volume
docker run -v nfs-volume:/data nginx
```

### AWS EBS Plugin
```bash
# Install AWS EBS plugin
docker plugin install rexray/ebs

# Create EBS volume
docker volume create --driver rexray/ebs \
  --opt size=10 \
  --opt volumetype=gp2 \
  ebs-volume

# Use EBS volume
docker run -v ebs-volume:/data nginx
```

### Flocker Plugin
```bash
# Install Flocker plugin
docker plugin install clusterhq/flocker

# Create Flocker volume
docker volume create --driver flocker \
  --opt size=20GB \
  flocker-volume
```

## 🎯 Container Storage Interface (CSI)

### CSI Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Kubernetes    │    │   CSI Driver    │    │  Storage System │
│                 │◄──►│                 │◄──►│                 │
│  - kubelet      │    │  - Node Plugin  │    │  - AWS EBS      │
│  - Controller   │    │  - Controller   │    │  - GCE PD       │
│    Manager      │    │    Plugin       │    │  - Azure Disk   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### CSI Driver Components
1. **Controller Plugin** - Volume lifecycle management
2. **Node Plugin** - Volume mounting/unmounting
3. **Identity Plugin** - Driver identification

### CSI Volume Lifecycle
```bash
# 1. CreateVolume (Controller)
# 2. ControllerPublishVolume (Controller)
# 3. NodeStageVolume (Node)
# 4. NodePublishVolume (Node)
# 5. NodeUnpublishVolume (Node)
# 6. NodeUnstageVolume (Node)
# 7. ControllerUnpublishVolume (Controller)
# 8. DeleteVolume (Controller)
```

## 🔧 CSI Driver Examples

### AWS EBS CSI Driver
```yaml
# StorageClass for EBS CSI
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

### Azure Disk CSI Driver
```yaml
# StorageClass for Azure Disk CSI
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-disk-sc
provisioner: disk.csi.azure.com
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
  cachingmode: ReadOnly
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

### Google Cloud Persistent Disk CSI
```yaml
# StorageClass for GCE PD CSI
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gce-pd-sc
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

## 🛡️ Storage Security and Performance

### Security Best Practices
```bash
# Use read-only mounts when possible
docker run -v /host/data:/data:ro nginx

# Avoid mounting sensitive directories
# DON'T: docker run -v /:/host nginx
# DON'T: docker run -v /var/run/docker.sock:/var/run/docker.sock nginx

# Use specific paths
docker run -v /host/app-data:/data nginx
```

### Performance Optimization
```bash
# Use overlay2 driver
{
  "storage-driver": "overlay2"
}

# Configure storage options
{
  "storage-opts": [
    "overlay2.override_kernel_check=true",
    "overlay2.size=20G"
  ]
}

# Use tmpfs for temporary data
docker run --tmpfs /tmp:rw,noexec,nosuid,size=100m nginx
```

## 📊 Storage Monitoring

### Docker Storage Usage
```bash
# Check storage usage
docker system df

# Detailed usage
docker system df -v

# Check specific container
docker exec container-name df -h

# Monitor I/O stats
docker stats --format "table {{.Container}}\t{{.BlockIO}}"
```

### Volume Usage Monitoring
```bash
# Check volume usage
docker volume ls --format "table {{.Name}}\t{{.Driver}}\t{{.Mountpoint}}"

# Inspect volume details
docker volume inspect volume-name

# Check mount points
docker inspect container-name | grep -A 10 "Mounts"
```

## 🔧 Advanced Storage Configurations

### Multi-Stage Builds with Volumes
```dockerfile
# Multi-stage build with volume
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
VOLUME ["/usr/share/nginx/html"]
EXPOSE 80
```

### Docker Compose with Volumes
```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    image: nginx
    volumes:
      - web-data:/usr/share/nginx/html
      - ./config:/etc/nginx/conf.d:ro
    ports:
      - "80:80"
  
  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password

volumes:
  web-data:
    driver: local
  db-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /host/db-data
```

### Custom Volume Plugin
```bash
# Install custom plugin
docker plugin install store/plugin-name:tag

# Configure plugin
docker plugin set plugin-name key=value

# Create volume with custom plugin
docker volume create --driver plugin-name \
  --opt option1=value1 \
  --opt option2=value2 \
  custom-volume
```

## 🚨 Troubleshooting Storage Issues

### Common Storage Problems
```bash
# Check storage driver issues
docker info | grep -i error

# Check disk space
df -h /var/lib/docker

# Check inode usage
df -i /var/lib/docker

# Clean up storage
docker system prune -a
docker volume prune
```

### Volume Mount Issues
```bash
# Check mount permissions
ls -la /host/path

# Verify SELinux context (if applicable)
ls -Z /host/path

# Check container mounts
docker inspect container-name | jq '.[0].Mounts'

# Debug mount issues
docker run --rm -v /host/path:/test alpine ls -la /test
```

### Performance Issues
```bash
# Check I/O stats
iostat -x 1

# Monitor container I/O
docker stats --format "table {{.Container}}\t{{.BlockIO}}"

# Check storage driver performance
docker info | grep -A 10 "Storage Driver"

# Test volume performance
docker run --rm -v myvolume:/test alpine dd if=/dev/zero of=/test/testfile bs=1M count=100
```

## 📋 Storage Best Practices

### Volume Management
1. **Use named volumes** for persistent data
2. **Bind mounts** for configuration files
3. **tmpfs** for temporary/sensitive data
4. **Regular cleanup** of unused volumes
5. **Monitor storage usage** regularly

### Security Guidelines
```bash
# Use least privilege mounts
docker run -v /app/data:/data:ro nginx

# Avoid mounting system directories
# Bad: -v /:/host
# Good: -v /app/data:/data

# Use specific volume drivers
docker volume create --driver local myvolume
```

### Performance Tips
```bash
# Use overlay2 storage driver
# Configure appropriate storage options
# Use SSD storage for better performance
# Monitor and clean up regularly
# Use multi-stage builds to reduce image size
```

This comprehensive guide covers Docker storage from basic concepts to advanced CSI implementations, providing everything needed for understanding container storage in Kubernetes environments.