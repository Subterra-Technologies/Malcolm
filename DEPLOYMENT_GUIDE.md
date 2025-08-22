# Malcolm Deployment Guide

## Overview
This guide provides comprehensive deployment instructions for Malcolm across different environments and use cases, from single-host development setups to enterprise-scale distributed deployments.

## Prerequisites

### System Requirements

#### Minimum Requirements
- **CPU**: 4 cores
- **RAM**: 16 GB
- **Storage**: 200 GB available space
- **OS**: Linux (Ubuntu 20.04+, RHEL 8+, etc.), macOS 10.15+, Windows 10/11

#### Recommended Requirements
- **CPU**: 8+ cores
- **RAM**: 32+ GB
- **Storage**: 1+ TB SSD storage
- **Network**: Gigabit Ethernet
- **OS**: Linux distribution with container support

#### Enterprise Requirements
- **CPU**: 16+ cores per node
- **RAM**: 64+ GB per node
- **Storage**: 10+ TB with high IOPS
- **Network**: 10+ Gigabit Ethernet
- **Cluster**: 3+ nodes for high availability

### Software Dependencies

#### Required Software
- **Container Runtime**: Docker 20.10+ or Podman 4.0+
- **Container Compose**: Docker Compose 2.0+ or podman-compose
- **Git**: For cloning the repository
- **Curl**: For configuration and health checks

#### Optional Software
- **Kubernetes**: 1.24+ for distributed deployments
- **Helm**: 3.0+ for Kubernetes package management
- **Vagrant**: For development environments
- **VirtualBox**: For virtualized testing

## Deployment Methods

### 1. Docker Compose (Recommended for most users)

#### Quick Start Deployment

```bash
# Clone the repository
git clone https://github.com/cisagov/Malcolm.git
cd Malcolm

# Run the configuration script
./scripts/configure

# Start Malcolm
./scripts/start

# Access the web interface
# https://localhost (or your configured hostname)
```

#### Detailed Docker Compose Setup

**1. Repository Setup**
```bash
git clone https://github.com/cisagov/Malcolm.git
cd Malcolm
git checkout main  # or specific version tag
```

**2. Configuration**
```bash
# Interactive configuration
./scripts/configure

# Non-interactive configuration
./scripts/configure --defaults

# Advanced configuration
./scripts/configure --config-only
```

**3. Authentication Setup**
```bash
# Configure authentication
./scripts/auth_setup

# Set up local users
./scripts/auth_setup --local

# Configure LDAP
./scripts/auth_setup --ldap

# Set up Keycloak SSO
./scripts/auth_setup --keycloak
```

**4. Environment Customization**
```bash
# Edit environment files
nano ./config/process.env
nano ./config/opensearch.env
nano ./config/auth.env
```

**5. Start Services**
```bash
# Start all services
./scripts/start

# Start specific profile
docker-compose --profile malcolm up -d

# Start with custom compose file
docker-compose -f docker-compose-dev.yml up -d
```

### 2. Kubernetes Deployment

#### Prerequisites
- Kubernetes cluster (1.24+)
- kubectl configured
- Persistent volume support
- Load balancer (optional)

#### Basic Kubernetes Deployment

**1. Namespace Creation**
```bash
kubectl create namespace malcolm
kubectl config set-context --current --namespace=malcolm
```

**2. Persistent Volume Setup**
```yaml
# pv-local.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: malcolm-opensearch-pv
spec:
  capacity:
    storage: 500Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /opt/malcolm/opensearch
```

**3. Deploy Core Services**
```bash
# Apply configurations in order
kubectl apply -f kubernetes/01-volumes.yml
kubectl apply -f kubernetes/03-opensearch.yml
kubectl apply -f kubernetes/04-dashboards.yml
kubectl apply -f kubernetes/05-upload.yml
```

**4. Deploy Processing Services**
```bash
kubectl apply -f kubernetes/06-pcap-monitor.yml
kubectl apply -f kubernetes/07-arkime.yml
kubectl apply -f kubernetes/10-zeek.yml
kubectl apply -f kubernetes/11-suricata.yml
kubectl apply -f kubernetes/14-logstash.yml
```

**5. Deploy Support Services**
```bash
kubectl apply -f kubernetes/15-redis.yml
kubectl apply -f kubernetes/17-postgres.yml
kubectl apply -f kubernetes/18-netbox.yml
kubectl apply -f kubernetes/98-nginx-proxy.yml
```

#### Advanced Kubernetes Configuration

**Ingress Setup**
```yaml
# ingress.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: malcolm-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  tls:
  - hosts:
    - malcolm.example.com
    secretName: malcolm-tls
  rules:
  - host: malcolm.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-proxy
            port:
              number: 443
```

**Resource Limits**
```yaml
# Example resource configuration
resources:
  requests:
    memory: "4Gi"
    cpu: "1000m"
  limits:
    memory: "8Gi"
    cpu: "2000m"
```

### 3. Cloud Deployment

#### AWS Deployment

**ECS Deployment**
```bash
# Configure AWS CLI
aws configure

# Create ECS cluster
aws ecs create-cluster --cluster-name malcolm-cluster

# Deploy services using AWS CLI or CloudFormation
```

**EKS Deployment**
```bash
# Create EKS cluster
eksctl create cluster --name malcolm --region us-west-2

# Configure kubectl
aws eks --region us-west-2 update-kubeconfig --name malcolm

# Deploy using Kubernetes manifests
kubectl apply -f kubernetes/
```

#### Azure Deployment

**AKS Deployment**
```bash
# Create resource group
az group create --name malcolm-rg --location eastus

# Create AKS cluster
az aks create --resource-group malcolm-rg --name malcolm-aks

# Get credentials
az aks get-credentials --resource-group malcolm-rg --name malcolm-aks

# Deploy Malcolm
kubectl apply -f kubernetes/
```

#### Google Cloud Deployment

**GKE Deployment**
```bash
# Create GKE cluster
gcloud container clusters create malcolm-cluster --zone us-central1-a

# Get credentials
gcloud container clusters get-credentials malcolm-cluster --zone us-central1-a

# Deploy Malcolm
kubectl apply -f kubernetes/
```

## Configuration Management

### Environment Variables

#### Core Configuration Files
- `./config/process.env` - Global process settings
- `./config/opensearch.env` - OpenSearch configuration
- `./config/auth.env` - Authentication settings
- `./config/ssl.env` - TLS/SSL configuration

#### Key Settings

**Process Configuration**
```bash
# process.env
MALCOLM_PROFILE=malcolm
OPENSEARCH_PRIMARY=opensearch-local
COMPOSE_PROJECT_NAME=malcolm
```

**Resource Allocation**
```bash
# Memory settings (in bytes)
OPENSEARCH_JAVA_OPTS=-Xms4g -Xmx4g
LOGSTASH_JAVA_OPTS=-Xms2g -Xmx2g
```

**Network Configuration**
```bash
# Interface settings
PCAP_IFACE=eth0
PCAP_CAPTURE_HOST=localhost
PCAP_TCPDUMP_FILENAME_PATTERN=%Y%m%d%H%M%S.pcap
```

### Storage Configuration

#### Persistent Volume Management
```bash
# Create data directories
sudo mkdir -p /opt/malcolm/{opensearch,opensearch-backup,zeek-logs,pcap}
sudo chown -R 1000:1000 /opt/malcolm/

# Configure volume mounts in docker-compose.yml
volumes:
  - /opt/malcolm/opensearch:/usr/share/opensearch/data
  - /opt/malcolm/pcap:/pcap
```

#### Storage Sizing Guidelines
- **OpenSearch**: 10x raw data size for indices
- **PCAP Storage**: 1:1 ratio for full packet capture
- **Logs**: 20% of total data volume
- **Backups**: 50% of primary storage

### Network Configuration

#### Port Assignments
- **443/80**: Web interface (HTTPS/HTTP)
- **9200**: OpenSearch API
- **5601**: OpenSearch Dashboards
- **8005**: Arkime viewer
- **5044**: Logstash input
- **8080**: NetBox
- **8443**: Keycloak (if enabled)

#### Firewall Configuration
```bash
# UFW example
sudo ufw allow 443/tcp
sudo ufw allow 80/tcp
sudo ufw allow ssh
sudo ufw enable

# iptables example
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

## Security Configuration

### SSL/TLS Setup

#### Certificate Generation
```bash
# Generate self-signed certificates
./scripts/auth_setup --tls-self-signed

# Use existing certificates
cp /path/to/cert.pem ./nginx/certs/
cp /path/to/key.pem ./nginx/certs/
```

#### Certificate Configuration
```bash
# SSL environment settings
SSL_CERTIFICATE_PATH=/etc/nginx/certs/cert.pem
SSL_PRIVATE_KEY_PATH=/etc/nginx/certs/key.pem
SSL_CERTIFICATE_VERIFICATION=false
```

### Authentication Setup

#### Local Authentication
```bash
# Create local users
./scripts/auth_setup --local
htpasswd -c auth/htpasswd admin
```

#### LDAP Integration
```bash
# Configure LDAP
LDAP_URI="ldap://ldap.example.com:389"
LDAP_BIND_DN="cn=bind,dc=example,dc=com"
LDAP_SEARCH_BASE="ou=people,dc=example,dc=com"
```

#### Keycloak SSO
```bash
# Enable Keycloak
KEYCLOAK_ENABLE=true
KEYCLOAK_REALM=malcolm
KEYCLOAK_CLIENT_ID=malcolm-client
```

### Network Security

#### Network Isolation
```yaml
# Docker network configuration
networks:
  default:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

#### Access Control
```bash
# Restrict access by IP
NGINX_ALLOW_IP="192.168.1.0/24"
NGINX_DENY_ALL=true
```

## Performance Tuning

### Resource Optimization

#### Memory Configuration
```bash
# OpenSearch heap sizing (50% of available RAM, max 32GB)
OPENSEARCH_JAVA_OPTS="-Xms16g -Xmx16g -XX:+UseG1GC"

# Logstash configuration
LOGSTASH_JAVA_OPTS="-Xms4g -Xmx4g"
PIPELINE_WORKERS=8
PIPELINE_BATCH_SIZE=500
```

#### CPU Optimization
```bash
# Zeek processing threads
ZEEK_AUTO_ANALYZE_NODE_COUNT=8
PCAP_PROCESSOR_THREADS=4

# Arkime capture threads
ARKIME_PACKET_THREADS=4
```

#### Storage Optimization
```bash
# OpenSearch storage settings
OPENSEARCH_REPLICAS=0  # Single node deployment
OPENSEARCH_INDEX_REFRESH_INTERVAL=30s
OPENSEARCH_COMPRESSION_TYPE=best_compression
```

### Network Performance

#### High-Speed Capture
```bash
# Enable high-performance capture
ARKIME_LIVE_CAPTURE=true
ARKIME_PACKET_THREADS=8
ARKIME_PCAP_READ_METHOD=tpacketv3

# Zeek live capture optimization
ZEEK_LIVE_CAPTURE=true
ZEEK_AF_PACKET_THREADS=4
```

#### Buffer Tuning
```bash
# Increase network buffers
echo 'net.core.rmem_max = 268435456' >> /etc/sysctl.conf
echo 'net.core.netdev_max_backlog = 5000' >> /etc/sysctl.conf
sysctl -p
```

## Monitoring and Maintenance

### Health Monitoring

#### Service Health Checks
```bash
# Check service status
./scripts/status

# Detailed container status
docker-compose ps

# Service logs
docker-compose logs -f opensearch
```

#### Performance Monitoring
```bash
# OpenSearch cluster health
curl -X GET "localhost:9200/_cluster/health?pretty"

# Logstash performance
curl -X GET "localhost:9600/_node/stats"

# System resource usage
docker stats
```

### Log Management

#### Log Rotation
```bash
# Configure log rotation
cat > /etc/logrotate.d/malcolm << EOF
/opt/malcolm/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    copytruncate
}
EOF
```

#### Log Aggregation
```bash
# External log forwarding
BEATS_SSL=true
BEATS_CLIENT_CERTIFICATE_VERIFICATION=false
LOGSTASH_EXTERNAL_OUTPUT_ENABLED=true
```

### Backup and Recovery

#### Data Backup
```bash
# Create backup script
#!/bin/bash
# backup-malcolm.sh
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf /backup/malcolm_${DATE}.tar.gz \
  /opt/malcolm/opensearch \
  /opt/malcolm/zeek-logs \
  ./config/
```

#### Snapshot Management
```bash
# OpenSearch snapshots
curl -X PUT "localhost:9200/_snapshot/backup" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/opt/opensearch/backup"
  }
}'
```

## Troubleshooting

### Common Issues

#### Container Startup Problems
```bash
# Check container logs
docker-compose logs --tail=50 opensearch

# Check resource usage
docker system df
docker system prune

# Verify port availability
netstat -tulpn | grep :9200
```

#### Performance Issues
```bash
# Check system resources
top
free -h
df -h

# Monitor network usage
iftop
netstat -i
```

#### Storage Issues
```bash
# Check disk space
df -h /opt/malcolm/

# Clean up old data
./scripts/wipe

# Optimize indices
curl -X POST "localhost:9200/_forcemerge"
```

### Diagnostic Commands

#### Container Diagnostics
```bash
# Container health
docker-compose exec opensearch curl -X GET "localhost:9200/_cluster/health"

# Container shell access
docker-compose exec opensearch bash

# Configuration validation
docker-compose config
```

#### Network Diagnostics
```bash
# Check connectivity between containers
docker-compose exec logstash ping opensearch

# Verify SSL certificates
openssl s_client -connect localhost:443 -verify_return_error
```

## Best Practices

### Deployment Recommendations

1. **Start Small**: Begin with minimum requirements and scale up
2. **Use SSD Storage**: For optimal performance, especially for OpenSearch
3. **Monitor Resources**: Implement comprehensive monitoring from day one
4. **Regular Backups**: Automate backup procedures
5. **Security First**: Implement proper authentication and network security
6. **Documentation**: Maintain deployment documentation and runbooks

### Production Considerations

1. **High Availability**: Deploy across multiple nodes/zones
2. **Load Balancing**: Implement proper load balancing for web interfaces
3. **Disaster Recovery**: Plan and test recovery procedures
4. **Compliance**: Ensure deployment meets regulatory requirements
5. **Monitoring**: Implement comprehensive monitoring and alerting
6. **Updates**: Plan for regular updates and maintenance windows

This deployment guide provides a comprehensive foundation for deploying Malcolm across various environments. Adjust configurations based on your specific requirements, network topology, and security policies.