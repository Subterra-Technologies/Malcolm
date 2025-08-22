# Malcolm Logstash Component Reference

## Overview
Logstash serves as Malcolm's central data processing engine, transforming and enriching network traffic data from multiple sources before indexing it into OpenSearch. It implements a sophisticated multi-pipeline architecture for handling different data types and processing stages.

## Container Configuration
- **Base Image**: Elastic Logstash 8.x
- **User**: logstash (non-privileged)
- **Process Manager**: supervisord
- **Configuration**: `/usr/share/logstash/config/logstash.yml`

## Pipeline Architecture

### Multi-Pipeline Structure
- **input**: Data ingestion from Beats
- **zeek-parse**: Zeek log processing (60+ log types)
- **suricata-parse**: Suricata IDS/IPS log processing
- **beats-parse**: System log processing
- **enrichment**: Central data enrichment
- **output**: Primary OpenSearch output
- **external**: Secondary/external output

## Key Directories and Files

### Pipeline Configurations
- `/usr/share/logstash/pipeline/pipelines.yml` - Pipeline definitions
- `/usr/share/logstash/pipeline/*/` - Individual pipeline configurations

### Custom Scripts
- `/usr/share/logstash/ruby/` - Custom Ruby scripts for data processing
- `/usr/share/logstash/maps/` - YAML mapping files for enrichment

### SSL/TLS Configuration
- `/usr/share/logstash/certs/` - Certificate files for secure communication

## Data Flow Architecture

```
Filebeat → Input Pipeline → Parse Pipelines → Enrichment Pipeline → Output Pipelines → OpenSearch
```

### 1. Input Collection (`input/`)
- **Port**: 5044 (Beats input)
- **SSL/TLS**: Certificate-based encryption
- **Routing**: Tag-based pipeline routing

### 2. Parse Pipelines

#### Zeek Pipeline (`zeek/`)
Processes 60+ Zeek log types:
- **Network Analysis**: conn, dns, http, ssl, ssh, ftp
- **Security Events**: intel, signatures, notice, weird
- **File Analysis**: files, pe, x509
- **ICS Protocols**: modbus, dnp3, enip, s7comm, bacnet, profinet

#### Suricata Pipeline (`suricata/`)
- **Alerts**: IDS/IPS security alerts
- **Flow Records**: Network flow information
- **Statistical Data**: Performance and detection metrics

#### Beats Pipeline (`beats/`)
- **System Logs**: Syslog, journald entries
- **Windows Events**: Converted EVTX files
- **Application Logs**: Various service logs

### 3. Enrichment Pipeline (`enrichment/`)

#### Core Enrichment Stages

**11_lookups.conf - Lookups and GeoIP**
- MAC address OUI lookup for device manufacturer identification
- GeoIP enhancement (country, city, coordinates, ASN)
- Reverse DNS resolution for public IP addresses
- Network direction classification (internal/external/inbound/outbound)
- ICS device detection from MAC addresses

**21_netbox.conf - NetBox Integration**
- Automatic device discovery and population
- Asset enrichment with device and network segment information
- Service discovery and association
- Fuzzy string matching for device identification

**23_severity.conf - Severity Scoring**
- Risk assessment and severity score assignment
- Threat categorization by security relevance
- Insecure protocol identification
- Non-standard port detection
- File transfer risk classification

**20_enriched_to_ecs.conf - ECS Normalization**
- Elastic Common Schema (ECS) field mapping
- Standardized field naming across log sources
- Creation of `related.*` fields for correlation

**97_arkimize.conf - Arkime Compatibility**
- Data transformation for Arkime integration
- Field mapping for packet capture analysis tools

### 4. Output Processing (`output/`)
- Unique document ID generation for deduplication
- Index routing and naming
- Dual output support (local and remote OpenSearch)

## Ruby Scripts and Custom Processing

### Network Enrichment Scripts

**`netbox_enrich.rb`**
- NetBox API integration
- Device auto-population and enrichment
- Intelligent caching with TTL
- IP-to-device and MAC-to-device mappings

**`freq_lookup.rb`**
- Domain name entropy calculation
- DGA (Domain Generation Algorithm) detection
- Frequency analysis service integration
- Result caching for performance

**`mac_lookup.rb`**
- Binary search on MAC address ranges
- Manufacturer identification
- Support for standard and ICS device databases

### Data Processing Utilities

**Hash and Array Processing**
- `compact_event.rb` - Event compaction and cleanup
- `dotted_hash_unflatten.rb` - Nested hash expansion
- `hash_merge.rb` - Deep hash merging
- `make_unique_array.rb` - Array deduplication

**Specialized Processors**
- `smbsplit.rb` - SMB protocol parsing
- `hashhash_to_hasharray.rb` - Hash structure transformation
- `dedoubleslash.rb` - URL normalization

## Mapping Files and Configuration

### Core Mapping Files (`maps/`)

**Severity and Classification**
- `malcolm_severity.yaml` - Configurable risk scoring system
- `zeek_log_ecs_categories.yaml` - ECS category mappings
- `service_ports.yaml` - Standard port assignments

**Protocol Analysis**
- `conn_states.yaml` - TCP connection state mappings
- `ip_protocol_*.yaml` - IP protocol number/name mappings
- `http_result_codes.yaml` - HTTP status code meanings

**ICS and Industrial Systems**
- `ics_macs.yaml` - Industrial device MAC address ranges
- `vm_macs.yaml` - Virtual machine MAC identifiers

**Security Intelligence**
- `mitre_attack_*` - MITRE ATT&CK framework mappings
- `notice_*.yaml` - Zeek notice categorization
- `zeek_intel_indicator_types.yaml` - Threat intelligence types

**System Integration**
- `syslog_*.yaml` - Syslog facility and severity mappings
- `winlog_*.yaml` - Windows event log mappings
- `ntp_modes.yaml` - NTP mode descriptions

## Environment Variables

### Performance Configuration
- `LS_JAVA_OPTS` - JVM memory and performance settings
- `LOGSTASH_API_PORT` - Management API port (default: 9600)
- `PIPELINE_WORKERS` - Number of worker threads per pipeline

### Input Configuration
- `LOGSTASH_HOST` - Hostname for service discovery
- `LOGSTASH_LJ_PORT` - Lumberjack input port (default: 5044)

### Output Configuration
- `OPENSEARCH_URL` - Primary OpenSearch cluster URL
- `OPENSEARCH_SSL_CERTIFICATE_VERIFICATION` - SSL verification setting
- `OPENSEARCH_CREDS_CONFIG_FILE` - Authentication configuration

### Enrichment Settings
- `NETBOX_POPULATE` - Enable NetBox integration
- `NETBOX_ENRICH_URL_CACHE_SIZE` - NetBox cache size
- `FREQ_LOOKUP` - Enable frequency analysis
- `SEVERITY_SCORING` - Enable severity calculation

## Performance Optimization

### Memory Management
- JVM heap sizing based on available memory
- Pipeline-specific memory allocation
- Persistent queue configuration for reliability

### Threading and Concurrency
- Multi-threaded pipeline processing
- Worker pool optimization
- Thread-safe Ruby script execution

### Caching Strategies
- LRU caches for frequent lookups
- TTL-based expiration for dynamic data
- Persistent caching for static data

### Batch Processing
- Configurable batch sizes for output
- Bulk indexing optimization
- Queue management for high throughput

## Security Features

### SSL/TLS Configuration
- Certificate-based authentication
- Encrypted communication channels
- Configurable cipher suites

### Data Protection
- Sensitive field filtering
- Configurable password logging controls
- Audit trail maintenance

### Access Control
- Pipeline-level security
- Input validation and sanitization
- Output authorization controls

## Monitoring and Health Checks

### Health Monitoring
- Container health check: `/usr/local/bin/container_health.sh`
- Pipeline status monitoring
- Queue depth tracking
- Processing rate metrics

### Performance Metrics
- Event processing rates
- Memory utilization
- Queue statistics
- Error rates and types

### Debugging and Troubleshooting
- Pipeline debug logging
- Event tracing capabilities
- Performance profiling tools
- Configuration validation

## Common Configuration Examples

### Enable NetBox Integration
```yaml
NETBOX_POPULATE: "true"
NETBOX_ENRICH: "true"
NETBOX_URL: "http://netbox:8080/netbox"
```

### Configure Severity Scoring
```yaml
SEVERITY_SCORING: "true"
MALCOLM_SEVERITY_CONFIG_FILE: "/usr/share/logstash/maps/malcolm_severity.yaml"
```

### Optimize for High Throughput
```yaml
LS_JAVA_OPTS: "-Xms8g -Xmx8g"
PIPELINE_WORKERS: "8"
PIPELINE_BATCH_SIZE: "500"
```

### Enable Frequency Analysis
```yaml
FREQ_LOOKUP: "true"
FREQ_URL: "http://freq:10004"
```

## Integration Points

### Input Sources
- Filebeat (primary log forwarder)
- Direct TCP/UDP inputs
- HTTP inputs for webhooks
- Kafka (optional streaming)

### Output Destinations
- OpenSearch (primary)
- Elasticsearch (compatible)
- External SIEM systems
- File outputs for archiving

### External Services
- NetBox API for asset management
- Frequency analysis service
- GeoIP databases
- Threat intelligence feeds

## Troubleshooting

### Common Issues
1. **Memory Errors**: Increase JVM heap size
2. **Connection Failures**: Check SSL certificates and network connectivity
3. **Parsing Errors**: Validate input data format and pipeline configuration
4. **Performance Issues**: Optimize worker counts and batch sizes

### Debugging Commands
```bash
# Check pipeline status
curl localhost:9600/_node/stats/pipelines

# Monitor processing rates
curl localhost:9600/_node/stats/events

# View pipeline configuration
curl localhost:9600/_node/pipelines

# Check hot threads
curl localhost:9600/_node/hot_threads
```

### Log Analysis
- Pipeline-specific log files
- Error pattern analysis
- Performance bottleneck identification
- Queue overflow detection

## Best Practices

### Configuration Management
- Version control for pipeline configurations
- Environment-specific variable management
- Regular configuration validation
- Backup and recovery procedures

### Performance Tuning
- Right-size JVM heap based on available memory
- Monitor queue depths and processing rates
- Optimize batch sizes for your data volume
- Use appropriate number of workers per pipeline

### Security Hardening
- Enable SSL/TLS for all communications
- Implement proper access controls
- Regular security updates
- Audit log review procedures