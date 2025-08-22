# Malcolm Network Traffic Analysis System - Comprehensive Overview

## Executive Summary

Malcolm is a sophisticated, open-source network traffic analysis platform designed for comprehensive network security monitoring and incident response. Developed by Battelle Energy Alliance in cooperation with CISA, Malcolm provides an integrated suite of tools for processing, analyzing, and visualizing network traffic data from both packet captures (PCAP) and live network monitoring.

## System Architecture

### High-Level Components

Malcolm operates as a containerized microservices architecture with the following core components:

1. **Data Ingestion Layer**
   - Arkime (packet capture and session analysis)
   - Zeek (network protocol analysis)
   - Suricata (intrusion detection/prevention)
   - Filebeat (log forwarding and aggregation)

2. **Data Processing Layer**
   - Logstash (data parsing, enrichment, and transformation)
   - Redis (caching and message queuing)

3. **Data Storage Layer**
   - OpenSearch (primary data indexing and search)
   - PostgreSQL (metadata and configuration storage)

4. **Analysis and Visualization Layer**
   - OpenSearch Dashboards (data visualization and analytics)
   - Arkime Viewer (packet-level analysis interface)
   - Malcolm API (programmatic access and integration)

5. **Infrastructure Layer**
   - Nginx (reverse proxy and authentication)
   - Keycloak (identity and access management)
   - NetBox (network asset management)

### Container Orchestration

Malcolm supports multiple orchestration platforms:
- **Docker Compose** (primary deployment method)
- **Kubernetes** (enterprise and cloud deployments)
- **Podman** (rootless container environments)

## Key Features and Capabilities

### 1. Multi-Protocol Analysis

**Network Protocols:**
- Standard Internet protocols (HTTP/HTTPS, DNS, SMTP, FTP, etc.)
- Security protocols (Kerberos, RADIUS, IPSec, etc.)
- Industrial Control System (ICS) protocols:
  - Modbus, DNP3, Profinet, S7comm, EtherNet/IP
  - BACnet, BSAP, C1222, HART-IP, OPC UA
  - GE-SRTP, Genisys, Omron FINS, ROC Plus

**Data Sources:**
- PCAP files (offline analysis)
- Live network capture (real-time monitoring)
- Zeek logs (structured network analysis)
- Suricata alerts (IDS/IPS events)
- Third-party log sources (Syslog, EVTX, etc.)

### 2. Advanced Security Analysis

**Threat Detection:**
- CVE-specific detection (Log4j, Zerologon, EternalSafety, etc.)
- Malware identification (Agent Tesla, AsyncRAT, Quasar, etc.)
- Network behavior analysis
- Industrial-specific threat detection

**Intelligence Integration:**
- STIX/TAXII threat intelligence feeds
- MISP (Malware Information Sharing Platform)
- Commercial threat intelligence (Mandiant, etc.)
- Custom threat intelligence integration

**Behavioral Analysis:**
- JA4 TLS fingerprinting
- HASSH SSH fingerprinting
- Domain generation algorithm (DGA) detection
- Entropy analysis for malicious domains

### 3. Network Asset Management

**Automatic Discovery:**
- Device identification via MAC address OUI lookup
- Service discovery and port scanning detection
- Network segment identification
- Asset inventory management through NetBox integration

**Enrichment Capabilities:**
- GeoIP location mapping
- DNS resolution and reverse lookups
- Cross-reference with known asset databases
- Industrial device classification

### 4. Visualization and Analysis

**Pre-built Dashboards:**
- Network overview and statistics
- Protocol-specific analysis dashboards
- Security event monitoring
- Industrial control system monitoring
- Geographic traffic visualization

**Interactive Analysis:**
- Arkime session browser for packet-level analysis
- OpenSearch Dashboards for flexible data exploration
- PCAP export and download capabilities
- Real-time alerting and monitoring

## Deployment Modes

### 1. Standalone Deployment
- Single-host deployment using Docker Compose
- Suitable for small networks, home labs, and incident response
- Resource requirements: 16GB RAM minimum, 32GB recommended

### 2. Distributed Deployment
- Multi-host Kubernetes deployment
- Horizontal scaling of processing components
- High availability and load balancing
- Enterprise-grade for large network environments

### 3. Sensor-Based Deployment (Hedgehog)
- Dedicated sensor appliances for network monitoring
- Real-time packet capture and forwarding
- Edge processing capabilities
- Automated ISO building for sensor deployment

### 4. Cloud Deployment
- AWS, Azure, GCP compatibility
- Container registry integration (ghcr.io)
- Infrastructure as Code support
- Elastic scaling capabilities

## Data Processing Pipeline

### 1. Data Ingestion
```
Network Traffic → Zeek/Suricata Analysis → Structured Logs → Filebeat → Logstash
```

### 2. Data Enrichment
```
Raw Logs → Protocol Parsing → GeoIP Enhancement → Asset Correlation → Threat Intelligence → ECS Normalization
```

### 3. Data Storage
```
Enriched Data → Index Routing → OpenSearch Storage → Dashboard Visualization
```

### 4. Analysis Workflow
```
OpenSearch Data ↔ Arkime Sessions ↔ PCAP Files ↔ Malcolm API ↔ External Tools
```

## Security and Authentication

### Authentication Methods
- Local user management (htpasswd)
- LDAP/Active Directory integration
- Keycloak SSO with SAML/OIDC support
- Role-based access control (RBAC)

### Security Features
- TLS encryption for all communications
- Certificate-based authentication
- Network segmentation and isolation
- Audit logging and event tracking
- Read-only operational modes

### Data Protection
- Configurable data retention policies
- Encrypted communication channels
- Secure credential storage
- GDPR compliance capabilities

## Configuration Management

### Environment-Based Configuration
- 200+ environment variables for customization
- Docker environment files (.env)
- Kubernetes ConfigMaps and Secrets
- Runtime configuration updates

### Key Configuration Areas
- Network interface monitoring
- Protocol analyzer settings
- Data retention and storage policies
- Authentication and authorization
- Performance tuning parameters
- Integration endpoints

### Management Scripts
- `./scripts/configure` - Interactive configuration wizard
- `./scripts/auth_setup` - Authentication configuration
- `./scripts/install.py` - Automated installation
- `./scripts/control.py` - Runtime management

## Performance and Scalability

### Resource Requirements
- **Minimum**: 16GB RAM, 4 CPU cores, 200GB storage
- **Recommended**: 32GB+ RAM, 8+ CPU cores, 1TB+ storage
- **Enterprise**: 64GB+ RAM, 16+ CPU cores, 10TB+ storage

### Optimization Features
- Multi-threaded processing
- Configurable worker pools
- Memory-mapped file processing
- Intelligent data compression
- Automated index lifecycle management

### Monitoring and Health Checks
- Container health monitoring
- Performance metrics collection
- Capacity planning tools
- Resource utilization dashboards

## Industrial Control System (ICS) Focus

### Specialized Capabilities
- 20+ ICS protocol analyzers
- Industrial device identification
- OT network monitoring
- Safety instrumented system (SIS) awareness
- Purdue model network segmentation analysis

### Compliance and Standards
- NIST Cybersecurity Framework alignment
- IEC 62443 industrial security standards
- NERC CIP compliance features
- Industrial IoT (IIoT) monitoring

## Integration Ecosystem

### External Integrations
- SIEM platforms (Splunk, QRadar, Sentinel)
- Threat intelligence platforms
- Network management systems
- Security orchestration tools

### API Capabilities
- RESTful API for data access
- OpenSearch API proxy
- Bulk data export/import
- Real-time event streaming

### Data Export Formats
- JSON/JSONL for structured data
- PCAP for packet-level analysis
- CSV for spreadsheet analysis
- STIX for threat intelligence sharing

## Development and Customization

### Extensibility
- Custom Zeek scripts and packages
- Logstash plugin development
- Dashboard customization
- Integration scripting (Python/Ruby)

### Development Environment
- Local development setup
- Container-based testing
- Continuous integration pipelines
- Documentation generation tools

### Community and Support
- Open-source Apache 2.0 license
- Active GitHub repository
- Community-driven development
- Government and academic partnerships

## Use Cases and Applications

### Network Security Operations Centers (SOCs)
- 24/7 network monitoring
- Incident response and forensics
- Threat hunting and analysis
- Compliance reporting

### Industrial Networks
- Operational technology (OT) monitoring
- Critical infrastructure protection
- Safety system monitoring
- Regulatory compliance

### Research and Education
- Network protocol research
- Cybersecurity education
- Malware analysis
- Academic research projects

### Incident Response
- Portable analysis platform
- Offline PCAP analysis
- Forensic investigation
- Evidence collection and analysis

## Future Development

### Roadmap Priorities
- Enhanced cloud-native features
- Advanced machine learning integration
- Improved industrial protocol support
- Enhanced visualization capabilities
- Performance optimization

### Community Contributions
- Plugin ecosystem expansion
- Additional protocol analyzers
- Dashboard template library
- Integration adapters

## Conclusion

Malcolm represents a comprehensive, enterprise-grade network traffic analysis platform that combines the power of multiple open-source tools into a cohesive, easily deployable solution. Its particular strength in industrial control system analysis, combined with robust general-purpose network monitoring capabilities, makes it an ideal choice for organizations requiring comprehensive network visibility and security monitoring.

The platform's container-based architecture, extensive configuration options, and strong focus on security make it suitable for deployment across a wide range of environments, from small office networks to large-scale industrial facilities and enterprise data centers.