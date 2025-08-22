# Malcolm Zeek Component Reference

## Overview
Zeek serves as Malcolm's primary network protocol analyzer, processing both live traffic and PCAP files to generate structured logs for network communications, security events, and protocol-specific metadata.

## Container Configuration
- **Base Image**: Debian 12-slim
- **Zeek Version**: 7.2.2-0
- **User**: zeeker (non-privileged)
- **Process Manager**: supervisord

## Key Files and Directories

### Configuration Files
- `/opt/zeek/share/zeek/site/local.zeek` - Main site policy and configuration
- `/opt/zeek/share/zeek/site/extractor.zeek` - File extraction configuration
- `/opt/zeek/share/zeek/site/guess.zeek` - Protocol identification heuristics
- `/opt/zeek/share/zeek/site/kafka.zeek` - Kafka integration (optional)

### Custom Extensions
- `/opt/zeek/share/zeek/site/custom/` - User-defined scripts and packages
- `/opt/zeek/share/zeek/packages/` - Installed Zeek packages

### Intelligence Data
- `/opt/zeek/share/zeek/intel/` - Threat intelligence feeds
- `/opt/zeek/bin/zeek_intel_setup.sh` - Intelligence management script

## Supported Protocols

### Standard Internet Protocols
- HTTP/HTTPS, DNS, SMTP, FTP, SSH, Kerberos
- SSL/TLS, IPSec, RADIUS, SNMP, Syslog

### Industrial Control System (ICS) Protocols
- **Building Automation**: BACnet
- **Serial Protocols**: BSAP, ROC Plus
- **Power Systems**: C1222, DNP3, Synchrophasor
- **Manufacturing**: EtherNet/IP (ENIP), EtherCAT, Modbus, Profinet
- **Process Control**: HART-IP, OPC UA Binary, S7comm
- **Specialized**: GE-SRTP, Genisys, Omron FINS

## Key Environment Variables

### Protocol Control
- `ZEEK_DISABLE_<PROTOCOL>` - Disable specific protocol analyzers
- `ZEEK_DISABLE_SPICY_<PROTOCOL>` - Disable Spicy-based parsers
- `ZEEK_DISABLE_ICS_ALL` - Disable all ICS protocols

### File Extraction
- `ZEEK_EXTRACTOR_MODE` - Extraction mode: none/known/mapped/all/notcommtxt
- `ZEEK_EXTRACTOR_OVERRIDE_FILE` - Override MIME type mappings
- `EXTRACTED_FILE_MAX_BYTES` - Maximum file size for extraction (default: 134217728)

### Intelligence Configuration
- `ZEEK_INTEL_FEED_SINCE` - Intelligence feed time window
- `ZEEK_INTEL_REFRESH_CRON_EXPRESSION` - Cron schedule for intel updates
- `ZEEK_INTEL_ITEM_EXPIRATION` - Intelligence item expiration time

### Performance Tuning
- `ZEEK_LIVE_CAPTURE` - Enable live packet capture
- `ZEEK_AUTO_ANALYZE_NODE_COUNT` - Number of worker processes
- `PCAP_PROCESSOR_THREADS` - Parallel processing threads

## Operational Modes

### Offline Analysis (Default)
- Processes PCAP files from `/pcap/processed/`
- Generates logs to `/zeek/upload/`
- Automatic file cleanup after processing

### Live Capture Mode
- Captures from network interfaces
- Real-time log generation to `/zeek/live/`
- AF_PACKET support with load balancing

## Supervisord Services
- `pcap-zeek` - PCAP file processing
- `intel-initialization` - Intelligence feed setup
- `cron` - Scheduled tasks
- `live-zeek` - Live capture (when enabled)

## Third-Party Plugins

### Security Analysis
- CVE-2020-0601, CVE-2021-44228, CVE-2022-26809 detectors
- Malware detection: Agent Tesla, AsyncRAT, QuasarRAT, StrRAT
- HASSH SSH fingerprinting
- JA4 TLS analysis

### ICS Security
- EternalSafety exploit detection
- Industrial protocol anomaly detection
- OT-specific threat indicators

## Log Output Types

### Core Network Logs
- `conn.log` - Connection summaries
- `dns.log` - DNS queries and responses
- `http.log` - HTTP transactions
- `ssl.log` - SSL/TLS connections
- `files.log` - File transfers

### Security Logs
- `intel.log` - Threat intelligence matches
- `signatures.log` - Signature matches
- `notice.log` - Security notices
- `weird.log` - Protocol anomalies

### ICS Protocol Logs
- `modbus.log`, `dnp3.log`, `enip.log`
- `profinet.log`, `s7comm.log`, `bacnet.log`
- Over 20 industrial protocol-specific logs

## Integration Points

### Malcolm Pipeline
1. Zeek generates logs → `/zeek/upload/` or `/zeek/live/`
2. Filebeat monitors directories → Tags with `_filebeat_zeek*`
3. Logstash processes → Structured parsing and enrichment
4. OpenSearch indexes → Search and visualization

### File Extraction
- Extracted files stored in `/zeek/extract_files/`
- Metadata correlation with network events
- Integration with Malcolm's file scanning pipeline

### Asset Management
- Device identification via protocol analysis
- Service discovery and inventory
- Network segment mapping

## Common Configuration Examples

### Enable All ICS Protocols
```bash
# Remove or set to false
ZEEK_DISABLE_ICS_ALL=false
```

### Configure File Extraction
```bash
ZEEK_EXTRACTOR_MODE=mapped
EXTRACTED_FILE_MAX_BYTES=268435456  # 256MB
```

### Live Capture Setup
```bash
ZEEK_LIVE_CAPTURE=true
ZEEK_LIVE_CAPTURE_NETSNIFF=true
PCAP_IFACE=eth0
```

### Intelligence Configuration
```bash
ZEEK_INTEL_REFRESH_CRON_EXPRESSION="0 2 * * *"  # Daily at 2 AM
ZEEK_INTEL_ITEM_EXPIRATION=-1min  # Immediate expiration
```

## Health Monitoring
- Container health check: `/usr/local/bin/container_health.sh`
- Process monitoring via supervisord
- Log analysis for capture loss
- Performance metrics collection

## Troubleshooting

### Common Issues
1. **Permission Errors**: Ensure proper volume mounting and user permissions
2. **Memory Issues**: Increase container memory limits for large PCAP files
3. **Plugin Errors**: Check package installation and dependency resolution
4. **Intelligence Failures**: Verify network connectivity and feed sources

### Debugging Commands
```bash
# Check Zeek installation
zeek --version

# Validate configuration
zeek -C local.zeek

# Test script syntax
zeek -r test.pcap local.zeek

# Check plugin status
zkg list
```

## Performance Optimization

### Memory Management
- Configure `PCAP_PROCESSOR_THREADS` based on available CPU cores
- Use memory-mapped I/O for large PCAP files
- Enable log rotation to prevent disk space issues

### CPU Optimization
- Set appropriate worker node count
- Use CPU affinity for live capture
- Enable load balancing for high-speed interfaces

### Storage Optimization
- Configure log rotation and compression
- Use SSD storage for active log directories
- Implement tiered storage for archived logs

## Security Considerations
- Runs as non-privileged user after initialization
- File extraction size limits prevent resource exhaustion
- Configurable password logging controls
- SSL certificate validation options
- Intelligence feed sanitization