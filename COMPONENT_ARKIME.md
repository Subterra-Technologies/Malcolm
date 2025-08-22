# Malcolm Arkime Component Reference

## Overview
Arkime is Malcolm's packet capture analysis engine, providing full-packet visibility and session-based network traffic analysis. It captures, indexes, and provides search capabilities for network traffic data, complementing Zeek's protocol analysis with packet-level inspection.

## Container Configuration
- **Base Image**: Ubuntu 24.04
- **Arkime Version**: 5.4.0
- **User**: arkime (non-privileged)
- **Process Manager**: supervisord

## Key Components and Services

### Core Services (supervisord)
- **arkime-capture** - Packet capture and indexing
- **arkime-viewer** - Web interface and API
- **wise** - Web Intelligence Service Engine (threat intelligence)
- **cron** - Scheduled maintenance tasks

### Key Directories
- `/opt/arkime/` - Arkime installation directory
- `/opt/arkime/etc/config.ini` - Primary configuration file
- `/opt/arkime/wiseService/` - WISE service files
- `/opt/arkime/viewer/` - Web interface assets

## Configuration Files

### Primary Configuration (`/opt/arkime/etc/config.ini`)
Core Arkime configuration including:
- OpenSearch cluster settings
- Interface and capture configuration
- Performance tuning parameters
- Security and authentication settings

### Security Roles (`/opt/arkime/etc/roles/`)
- `arkime_hunt_access.json` - Hunt capabilities
- `arkime_pcap_access.json` - PCAP download permissions
- `arkime_read_access.json` - Read-only access
- `arkime_read_write_access.json` - Full access permissions

### Viewer Settings (`/opt/arkime/etc/user_settings.json`)
Default user interface settings and preferences

### WISE Configuration (`/opt/arkime/etc/wise.ini.example`)
Web Intelligence Service Engine configuration template

## Key Features

### Packet Capture and Analysis
- **Full Packet Capture**: Complete packet storage and indexing
- **Session Reconstruction**: TCP/UDP session reassembly
- **Protocol Parsing**: Application-layer protocol analysis
- **Metadata Extraction**: Session metadata and statistics

### Search and Analysis Capabilities
- **Session Search**: Complex query capabilities across session metadata
- **PCAP Export**: Full packet download for detailed analysis
- **Visualization**: Network connection graphs and statistics
- **Drill-down Analysis**: From high-level views to packet details

### Integration with Malcolm
- **Zeek Correlation**: Links Arkime sessions with Zeek logs via Community ID
- **OpenSearch Integration**: Stores session data in Malcolm's OpenSearch cluster
- **Unified Interface**: Accessible through Malcolm's web interface
- **Cross-referencing**: Correlates with other Malcolm data sources

## Important Environment Variables

### Capture Configuration
- `ARKIME_CAPTURE_INTERFACE` - Network interface for packet capture
- `ARKIME_LIVE_CAPTURE` - Enable live packet capture
- `ARKIME_ROTATED_PCAP` - Process rotated PCAP files

### Performance Settings
- `ARKIME_PACKET_THREADS` - Number of packet processing threads
- `ARKIME_PCAP_READ_METHOD` - PCAP reading method optimization
- `ARKIME_COMPRESSION_TYPE` - PCAP compression type (none/gzip/zstd)

### Storage Configuration
- `ARKIME_INDEX_PATTERN` - OpenSearch index pattern
- `ARKIME_REPLICAS` - Number of index replicas
- `ARKIME_HISTORY_DAYS` - Data retention period

### Authentication
- `ARKIME_AUTH_MODE` - Authentication method (anonymous/basic/digest)
- `ARKIME_PASSWORD_SECRET` - Password encryption secret

## Viewer Configuration Views

### Pre-configured Views (`/opt/arkime/etc/views/`)
- `arkime_sessions.json` - Default session view
- `public_ip_addresses.json` - External IP communication
- `suricata_alerts.json` - Correlated Suricata alerts
- `suricata_logs.json` - Suricata log integration
- `uninventoried_internal_assets.json` - Unknown internal devices
- `uninventoried_observed_services.json` - Unidentified services
- `zeek_conn.json` - Zeek connection correlation
- `zeek_exclude_conn.json` - Non-connection Zeek events
- `zeek_logs.json` - All Zeek log correlation

## WISE Service (Web Intelligence Service Engine)

### Threat Intelligence Integration
- **Real-time Enrichment**: Live threat intelligence lookups
- **Multiple Sources**: Support for various threat intel feeds
- **Caching**: Intelligent caching for performance
- **Custom Sources**: Extensible source plugin architecture

### Source Plugins
- File-based intelligence feeds
- URL-based feeds
- Database integrations
- Custom API sources

### Zeek Log Integration (`wise/source.zeeklogs.js`)
- Integrates Zeek intelligence logs
- Provides context from Malcolm's Zeek analysis
- Real-time correlation capabilities

## Capture Modes

### Live Capture Mode
- **Real-time Processing**: Captures and indexes live network traffic
- **Interface Monitoring**: Monitors specified network interfaces
- **Load Balancing**: Supports multiple capture threads for high-speed networks

### Offline Analysis Mode
- **PCAP Processing**: Analyzes pre-captured PCAP files
- **Batch Processing**: Processes multiple files sequentially
- **Rotated PCAP**: Handles log rotation scenarios

### Hybrid Mode
- **Simultaneous Operation**: Live capture with offline processing
- **Unified Storage**: Common index structure for all data
- **Consistent Interface**: Same search capabilities across modes

## Performance Optimization

### Capture Performance
- **Packet Buffer Sizing**: Configurable buffer sizes for high-speed capture
- **Thread Allocation**: Multi-threaded packet processing
- **CPU Affinity**: CPU core assignment for optimal performance
- **Memory Management**: Intelligent memory usage for large captures

### Storage Optimization
- **Compression**: PCAP compression options (gzip, zstd)
- **Index Management**: Automated index lifecycle management
- **Sharding**: Configurable index sharding strategies
- **Retention Policies**: Automated data cleanup based on age/size

### Query Performance
- **Index Optimization**: Optimized field indexing for common searches
- **Caching**: Query result caching
- **Aggregation**: Efficient statistical aggregations
- **Parallel Processing**: Multi-threaded search execution

## Security Features

### Authentication Integration
- **Malcolm SSO**: Integration with Malcolm's authentication system
- **Role-based Access**: Fine-grained permission control
- **Session Management**: Secure session handling
- **API Security**: Protected API endpoints

### Data Protection
- **PCAP Security**: Controlled access to full packet data
- **Field-level Security**: Configurable field visibility
- **Audit Logging**: User action tracking
- **Network Isolation**: Secure network configuration

## Integration Points

### OpenSearch Integration
- **Index Structure**: Arkime-specific index patterns
- **Field Mapping**: Optimized field mappings for search performance
- **Cluster Health**: Integration with Malcolm's OpenSearch monitoring

### Zeek Correlation
- **Community ID**: Links Arkime sessions with Zeek connection logs
- **Metadata Enrichment**: Zeek analysis data enhances Arkime sessions
- **Unified Timeline**: Correlated view of network events

### Malcolm API Integration
- **Session Export**: API access to session data
- **PCAP Download**: Programmatic PCAP export
- **Search Integration**: API search capabilities
- **Statistics API**: Performance and usage metrics

## Common Configuration Examples

### Enable Live Capture
```ini
[default]
interfaceOps=UP
interface=eth0
pcapDir=/opt/arkime/raw
```

### Configure Performance Settings
```ini
packetThreads=4
pcapReadMethod=tpacketv3
maxFileSizeG=4
compressLevel=6
```

### Set Retention Policy
```ini
deleteCheckDays=30
optimizeDays=1
replicas=0
```

### Authentication Configuration
```ini
passwordSecret=your-secret-key
authMode=digest
userNameHeader=x-remote-user
```

## Health Monitoring

### System Health Checks
- Container health: `/usr/local/bin/container_health.sh`
- Service status monitoring via supervisord
- Capture statistics and performance metrics
- Storage utilization tracking

### Performance Metrics
- **Packet Capture Rates**: Packets per second statistics
- **Index Performance**: OpenSearch indexing rates
- **Query Performance**: Search response times
- **Storage Usage**: Disk space utilization

### Alerting and Monitoring
- **Capture Loss Detection**: Identifies dropped packets
- **Storage Warnings**: Disk space threshold alerts
- **Performance Degradation**: Response time monitoring
- **Service Availability**: Component health tracking

## Troubleshooting

### Common Issues

**Capture Problems**
- Interface permissions and configuration
- Network buffer sizing issues
- High packet loss scenarios
- Performance bottlenecks

**Storage Issues**
- OpenSearch connection problems
- Index mapping conflicts
- Disk space exhaustion
- Retention policy failures

**Performance Problems**
- Insufficient system resources
- Suboptimal configuration settings
- Network throughput limitations
- Index fragmentation

### Debugging Commands

```bash
# Check capture statistics
curl localhost:8005/api/stats

# View current sessions
curl localhost:8005/api/sessions

# Check OpenSearch health
curl localhost:8005/api/eshealth

# Monitor capture interfaces
arkime-capture --status
```

## Best Practices

### Deployment Configuration
- **Resource Planning**: Adequate CPU, memory, and storage allocation
- **Network Configuration**: Proper interface setup and buffer sizing
- **Security Hardening**: Authentication and access control implementation
- **Monitoring Setup**: Comprehensive health and performance monitoring

### Performance Tuning
- **Thread Optimization**: Match thread count to CPU cores
- **Storage Configuration**: Use fast storage for active indices
- **Compression Settings**: Balance compression ratio with CPU usage
- **Retention Policies**: Implement appropriate data lifecycle management

### Security Best Practices
- **Access Control**: Implement least-privilege access principles
- **Network Security**: Secure all communication channels
- **Data Protection**: Encrypt sensitive data at rest and in transit
- **Audit Compliance**: Maintain comprehensive audit logs

### Operational Procedures
- **Regular Maintenance**: Schedule index optimization and cleanup
- **Backup Strategies**: Implement data backup and recovery procedures
- **Capacity Planning**: Monitor growth trends and plan for expansion
- **Update Management**: Keep Arkime and dependencies up to date

## API Reference

### Session API
- `GET /api/sessions` - Search sessions
- `GET /api/session/{id}` - Get specific session details
- `GET /api/session/{id}/pcap` - Download session PCAP

### Statistics API
- `GET /api/stats` - System and capture statistics
- `GET /api/eshealth` - OpenSearch cluster health
- `GET /api/esstats` - OpenSearch performance statistics

### Configuration API
- `GET /api/config` - Current configuration settings
- `GET /api/fields` - Available search fields
- `GET /api/views` - Configured views

This reference provides comprehensive information for understanding, configuring, and maintaining Malcolm's Arkime component for effective packet capture analysis and network traffic investigation.