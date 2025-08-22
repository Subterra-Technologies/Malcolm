# Malcolm OpenSearch Component Reference

## Overview
OpenSearch serves as Malcolm's primary data storage and search engine, providing distributed search capabilities, data indexing, and serving as the backend for visualization platforms. It stores and indexes all processed network traffic data, security events, and system logs.

## Container Configuration
- **Base Image**: OpenSearch 2.13.0
- **User**: opensearch (UID/GID from environment)
- **Process Manager**: Direct OpenSearch daemon
- **Memory**: Requires significant RAM allocation

## Key Directories and Files

### Configuration
- `/usr/share/opensearch/config/opensearch.yml` - Main configuration
- `/usr/share/opensearch/config/opensearch-security/` - Security plugin configuration
- `/usr/share/opensearch/config/log4j2.properties` - Logging configuration

### Data Storage
- `/usr/share/opensearch/data/` - Primary data directory
- `/opt/opensearch/backup/` - Backup snapshot repository

### Security and Certificates
- `/usr/share/opensearch/config/kirk.yml` - Security initialization
- `/usr/share/opensearch/config/certs/` - SSL/TLS certificates

## Core Features

### Distributed Search and Storage
- **Sharded Indices**: Distributed data storage across nodes
- **Replica Management**: Configurable data replication
- **Cluster Management**: Multi-node cluster coordination
- **Index Lifecycle Management**: Automated data retention policies

### Security Plugin
- **Authentication**: Multiple authentication backends
- **Authorization**: Role-based access control (RBAC)
- **TLS/SSL**: Encrypted communications
- **Audit Logging**: Security event tracking

### Advanced Analytics
- **Aggregations**: Complex data analytics capabilities
- **Machine Learning**: Anomaly detection and analysis
- **Alerting**: Configurable alerting and notifications
- **Cross-cluster Replication**: Multi-cluster data synchronization

## Index Structure

### Primary Indices

**Arkime Sessions** (`arkime_sessions3-*`)
- Network session data from Arkime
- Time-based indices with daily/weekly rotation
- Session metadata, connection details, protocol information

**Malcolm Beats** (`malcolm_beats_*`)
- System logs, diagnostics, and third-party data
- Structured log data from various sources
- Application and infrastructure monitoring data

**Arkime History** (`arkime_history_v*`)
- Historical statistics and metadata
- Long-term trend analysis data
- Aggregated network statistics

### Special Indices

**Arkime Fields** (`arkime_fields`)
- Field definitions and metadata
- Search field configurations
- Data type mappings

**Lookups** (`malcolm_*_lookup`)
- GeoIP and enrichment data
- Asset and network information
- Threat intelligence mappings

## Key Environment Variables

### Cluster Configuration
- `OPENSEARCH_PRIMARY` - Primary/secondary role designation
- `OPENSEARCH_URL` - Cluster connection URL
- `OPENSEARCH_SECONDARY` - Secondary cluster configuration

### Performance Settings
- `OPENSEARCH_JAVA_OPTS` - JVM memory and performance options
- `OPENSEARCH_HEAP_SIZE` - Heap size allocation
- `OPENSEARCH_REPLICAS` - Default replica count

### Security Configuration
- `OPENSEARCH_SSL_CERTIFICATE_VERIFICATION` - SSL verification mode
- `OPENSEARCH_CREDS_CONFIG_FILE` - Authentication configuration
- `OPENSEARCH_PRIMARY_REMOTE_CERT_VERIFICATION` - Remote cluster SSL settings

## Performance Optimization

### Memory Management
- **Heap Sizing**: Typically 50% of available RAM, max 32GB
- **JVM Options**: Garbage collection and performance tuning
- **Field Data**: Configurable field data circuit breakers
- **Query Cache**: Search result caching optimization

### Storage Optimization
- **Sharding Strategy**: Optimal shard size and distribution
- **Compression**: Index compression settings
- **Hot/Warm Architecture**: Tiered storage for data lifecycle
- **Force Merge**: Index optimization and segment merging

### Query Performance
- **Index Templates**: Optimized mapping configurations
- **Search Threading**: Concurrent search execution
- **Aggregation Optimization**: Efficient aggregation strategies
- **Caching**: Multi-level caching for frequent queries

## Security Configuration

### Authentication Methods
- **Internal Users**: Native user database
- **LDAP/Active Directory**: External directory integration
- **SAML/OIDC**: Single sign-on integration
- **Client Certificates**: Certificate-based authentication

### Role-Based Access Control
- **Index-level Permissions**: Fine-grained access control
- **Field-level Security**: Sensitive field protection
- **Document-level Security**: Row-level access control
- **API Access Control**: Method and endpoint restrictions

### TLS/SSL Configuration
- **Node-to-Node Encryption**: Cluster communication security
- **Client-to-Node Encryption**: API communication security
- **Certificate Management**: CA and certificate rotation
- **Cipher Suite Configuration**: Cryptographic controls

## Index Templates and Mappings

### Malcolm Template (`malcolm_template`)
```json
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "codec": "best_compression"
    },
    "mappings": {
      "properties": {
        "@timestamp": {"type": "date"},
        "network": {"type": "object"},
        "source": {"type": "object"},
        "destination": {"type": "object"}
      }
    }
  }
}
```

### Arkime Template (`arkime_template`)
- Optimized for session data
- Specialized field mappings for network analysis
- Performance-tuned for high ingestion rates

### Field Optimization
- **Keyword Fields**: Exact match and aggregation fields
- **Text Fields**: Full-text search capabilities  
- **Numeric Fields**: Range queries and mathematical operations
- **Date Fields**: Time-based queries and histograms

## Cluster Management

### Single Node Deployment
- **Development**: Simplified setup for testing
- **Small Networks**: Single-node production deployment
- **Resource Constraints**: Minimal resource requirements

### Multi-Node Cluster
- **High Availability**: Master node redundancy
- **Horizontal Scaling**: Data distribution across nodes
- **Load Distribution**: Query and indexing load balancing
- **Fault Tolerance**: Node failure recovery

### Cluster Roles
- **Master Nodes**: Cluster state management
- **Data Nodes**: Data storage and search execution
- **Ingest Nodes**: Data preprocessing and enrichment
- **Coordinating Nodes**: Request routing and aggregation

## Backup and Recovery

### Snapshot Configuration
- **Repository Setup**: Backup storage configuration
- **Automated Snapshots**: Scheduled backup creation
- **Incremental Backups**: Efficient differential backups
- **Cross-cluster Snapshots**: Remote backup capabilities

### Disaster Recovery
- **Cluster Restoration**: Complete cluster recovery procedures
- **Index Restoration**: Selective index recovery
- **Point-in-time Recovery**: Temporal data restoration
- **Configuration Backup**: Settings and mapping preservation

## Monitoring and Health

### Cluster Health Monitoring
- **Health API**: Real-time cluster status
- **Node Statistics**: Performance and resource metrics
- **Index Statistics**: Storage and performance data
- **Shard Allocation**: Distribution and balance monitoring

### Performance Metrics
- **Indexing Rate**: Documents per second
- **Search Latency**: Query response times
- **Resource Utilization**: CPU, memory, and disk usage
- **Cache Performance**: Hit rates and efficiency

### Alerting and Notifications
- **Threshold Alerts**: Performance and capacity alerts
- **Anomaly Detection**: Machine learning-based alerting
- **Integration**: External notification systems
- **Escalation**: Alert prioritization and routing

## Integration with Malcolm Components

### Logstash Integration
- **Bulk Indexing**: High-performance data ingestion
- **Index Templates**: Automatic mapping application
- **Pipeline Processing**: Real-time data transformation
- **Error Handling**: Failed document management

### Dashboard Integration
- **Visualization**: OpenSearch Dashboards backend
- **Query Execution**: Dashboard query processing
- **Aggregation**: Statistical analysis for visualizations
- **Real-time Updates**: Live dashboard data feeds

### API Access
- **Malcolm API**: Programmatic data access
- **Search API**: Complex query capabilities
- **Aggregation API**: Statistical analysis endpoints
- **Admin API**: Cluster management functions

## Common Configuration Examples

### Memory Configuration
```yaml
# opensearch.yml
bootstrap.memory_lock: true
indices.memory.index_buffer_size: 30%
indices.fielddata.cache.size: 20%
```

### Performance Tuning
```yaml
# High ingestion rate optimization
index.refresh_interval: 30s
index.translog.flush_threshold_size: 1gb
index.merge.policy.max_merged_segment: 5gb
```

### Security Configuration
```yaml
# Basic security setup
plugins.security.ssl.http.enabled: true
plugins.security.ssl.transport.enabled: true
plugins.security.authcz.admin_dn:
  - "CN=admin,O=Malcolm,L=security,C=US"
```

### Index Lifecycle Management
```json
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "10gb",
            "max_age": "1d"
          }
        }
      },
      "delete": {
        "min_age": "30d"
      }
    }
  }
}
```

## Troubleshooting

### Common Issues

**Memory Problems**
- OutOfMemoryError conditions
- GC performance issues
- Heap sizing problems
- Memory leak detection

**Performance Issues**
- Slow query performance
- High indexing latency
- Shard allocation problems
- Resource contention

**Cluster Issues**
- Split-brain scenarios
- Node discovery problems
- Network connectivity issues
- Configuration inconsistencies

### Diagnostic Commands

```bash
# Cluster health check
curl -X GET "localhost:9200/_cluster/health?pretty"

# Node statistics
curl -X GET "localhost:9200/_nodes/stats?pretty"

# Index statistics
curl -X GET "localhost:9200/_cat/indices?v"

# Shard allocation
curl -X GET "localhost:9200/_cat/shards?v"

# Performance analysis
curl -X GET "localhost:9200/_nodes/hot_threads"
```

### Log Analysis
- **Cluster logs**: Node startup and communication
- **Indexing logs**: Document processing issues
- **Query logs**: Search performance analysis
- **Security logs**: Authentication and authorization events

## Best Practices

### Deployment Recommendations
- **Hardware Sizing**: CPU, memory, and storage planning
- **Network Configuration**: Bandwidth and latency optimization
- **Security Hardening**: Access control and encryption
- **Monitoring Setup**: Comprehensive health monitoring

### Index Management
- **Template Design**: Efficient mapping configuration
- **Lifecycle Policies**: Automated data management
- **Shard Strategy**: Optimal shard sizing and distribution
- **Alias Usage**: Flexible index naming and rotation

### Performance Optimization
- **Query Optimization**: Efficient search patterns
- **Aggregation Tuning**: Statistical analysis optimization
- **Cache Configuration**: Memory usage optimization
- **Bulk Operations**: Batch processing for efficiency

### Security Best Practices
- **Access Control**: Least privilege principle
- **Network Security**: Firewall and network isolation
- **Certificate Management**: Regular rotation and renewal
- **Audit Compliance**: Comprehensive logging and monitoring

This reference provides comprehensive information for understanding, configuring, and maintaining Malcolm's OpenSearch component for effective data storage, search, and analytics capabilities.