# ELK Stack with Advanced Veeam & OOTBI Log Parsing

> Transform chaotic syslog messages into actionable security and operational intelligence

This repository provides an enhanced ELK stack that builds on our [basic Docker setup](https://github.com/object1st/elk-stack) but adds intelligent parsing for Veeam malware detection and OOTBI storage monitoring.

## 🚀 Quick Start (Just Like Blog Part 2!)

```bash
# Clone the repository
git clone https://github.com/yourusername/elk-veeam-ootbi.git
cd elk-veeam-ootbi

# Start the stack (same as before!)
docker-compose up -d

# Check containers are running
docker-compose ps

# Access Kibana
open http://localhost:5601
```

**That's it!** Everything else is automatic.

## 📋 What's New vs Basic Setup

This builds on the foundation from Blog Part 2 but adds:

✅ **Veeam Malware Detection** - Automatically identifies ransomware threats  
✅ **OOTBI Storage Monitoring** - Proactive capacity alerting  
✅ **Smart Field Extraction** - Structured data from messy logs  
✅ **Business Impact Classification** - Know what matters most  
✅ **Error Handling** - Robust parsing that doesn't break  

## 🔄 Backwards Compatibility

**Everything from your original setup still works:**
- Same ports (5514 for syslog, 5601 for Kibana)
- Same Veeam configuration process
- Same basic job parsing
- Same index patterns

**But now you also get advanced parsing automatically!**

## 📊 Before vs After

### Before (Original Setup)
```json
{
  "message": "Mar 20 14:06:09 VBRSRV01 Veeam_MP: [origin enterpriseId=\"31023\"] [categoryId=0 instanceId=41600...]",
  "source_type": "veeam",
  "backup_status": "unknown"
}
```

### After (Enhanced Setup)
```json
{
  "message": "Mar 20 14:06:09 VBRSRV01 Veeam_MP: [origin enterpriseId=\"31023\"]...",
  "source_type": "veeam",
  "severity": "critical",
  "threat_type": "ransomware_indicator",
  "activity_type": "EncryptedData",
  "username": "TECH\\user1",
  "object_name": "VM01",
  "business_impact": "critical",
  "recommended_action": "isolate_system"
}
```

## �� Testing Your Setup

### Send a Test Veeam Malware Alert
```bash
echo '<14>Mar 20 14:06:09 VBRSRV01 Veeam_MP: [origin enterpriseId="31023"] [categoryId=0 instanceId=41600 DetectionTimeUTC="03/20/2025 13:05:41" OibID="0e54d3bf-add8-48eb-9122-fad3ac1e8fb3" ActivityType="EncryptedData" UserName="TECH\user1" ObjectName="VM01" VbrHostName="vbrsrv01.tech.local" Description="Potential malware activity detected"]' | nc -u localhost 5514
```

### Send a Test OOTBI Storage Alert
```bash
echo '<14>Jul 18 12:23:53 ootbiprod02 OOTBI: - - [ootbi_event@61062 Node="ootbiprod02" NodeId="E08F3180-B0DC-11EF-AC5B-799EABA66937" DateTime="18/07/2025 16:23:48 UTC" Source="Storage" EventId="2510" ProductVersion="1.6.70.11503" EDataVersion="1"] Cluster node "ootbiprod02" has less than 40% of free space.' | nc -u localhost 5514
```

### Check Results in Kibana
1. Open Kibana: http://localhost:5601
2. Go to **Discover**
3. Create data views for:
   - `veeam-logs-*` (enhanced Veeam logs)
   - `ootbi-logs-*` (new OOTBI logs)  
   - `parsing-failures-*` (any parsing errors)
4. Look for the new parsed fields!

## 📁 Simple Repository Structure

```
elk-veeam-ootbi/
├── docker-compose.yml              # Same as Blog 2, just updated
├── elasticsearch/
│   └── config/
│       └── elasticsearch.yml       # Enhanced but familiar config
├── logstash/
│   ├── config/
│   │   └── logstash.yml            # Simple Logstash config
│   └── pipeline/
│       └── logstash.conf           # Enhanced parsing (the magic!)
├── kibana/
│   └── config/
│       └── kibana.yml              # Basic Kibana config
└── README.md                       # This file
```

## 🔧 What's Different Under the Hood

### Enhanced Logstash Configuration
The main difference is in `logstash/pipeline/logstash.conf`:
- **Keeps** all your original Veeam job parsing
- **Adds** intelligent malware detection patterns
- **Adds** OOTBI storage monitoring
- **Adds** business impact classification
- **Adds** graceful error handling

### Smarter Output Routing
Instead of just `veeam-logs-*`, now you get:
- `veeam-logs-*` - Enhanced Veeam logs with threat detection
- `ootbi-logs-*` - OOTBI storage and system logs
- `parsing-failures-*` - Any logs that couldn't be parsed (for debugging)

### Same Docker Experience
- Same `docker-compose up -d` command
- Same port configuration
- Same service dependencies
- Same resource requirements

## 🛠️ Configuring Your Systems

### Veeam VBR Setup (Same as Blog 2!)
1. Go to **Options** → **Event Forwarding**
2. Add your Docker host IP
3. Use port **5514**
4. Veeam will send test messages automatically

### OOTBI Setup (New!)
1. Configure syslog forwarding to your Docker host
2. Use port **5514** 
3. Events will be automatically parsed and categorized

## 🎯 Key Features

### For Veeam Logs
- **Original job parsing** (Success/Warning/Error) - unchanged
- **NEW: Malware detection** - Automatic ransomware identification
- **NEW: User tracking** - Extract domain\username from security events
- **NEW: Threat classification** - Critical/Warning based on activity type
- **NEW: Business impact** - Understand the severity automatically

### For OOTBI Logs  
- **NEW: Storage monitoring** - Capacity alerts with thresholds
- **NEW: Node identification** - Track which nodes have issues
- **NEW: Event categorization** - Storage/System/General events
- **NEW: Severity mapping** - Critical (<20%), Warning (<40%), Info (>40%)

### Error Handling
- **Parsing failures** go to separate index for debugging
- **Original logs preserved** - nothing is lost
- **Graceful degradation** - unknown logs still get indexed

## 💡 Pro Tips

### Viewing Parsed Fields
In Kibana Discover, look for these new fields:
- `severity` - Critical, Warning, Info
- `business_impact` - Critical, High, Medium, Low  
- `threat_type` - ransomware_indicator, ransomware_confirmed
- `event_category` - security, storage, backup, general
- `recommended_action` - isolate_system, investigate, monitor

### Dashboard Ideas
Create visualizations using:
- **Security Overview**: Count by `threat_type` over time
- **Storage Health**: OOTBI events by `severity` and `node_name`
- **User Activity**: Veeam security events by `clean_username`
- **System Impact**: Events by `business_impact` level

## 🤝 Contributing

Found a log format we don't parse? Want to add a new source?
1. Fork the repo
2. Update `logstash/pipeline/logstash.conf`
3. Test with sample data  
4. Submit a pull request!

## 📚 Learn More

This setup is part of our ELK Stack blog series:
- **[Part 1: Introduction](your-blog-link)** - Why ELK matters for monitoring
- **[Part 2: Basic Docker Setup](https://github.com/object1st/elk-stack)** - Get started with Docker
- **[Part 3: Advanced Parsing](your-blog-link)** - This repository

## 🛟 Troubleshooting

### Containers Won't Start
```bash
# Check what's wrong
docker-compose logs

# Common fix: not enough memory
# Edit .env and reduce heap sizes
```

### Logs Not Appearing
```bash
# Check Logstash is receiving data
docker-compose logs logstash

# Check Elasticsearch has data
curl http://localhost:9200/_cat/indices
```

### Parsing Not Working
```bash
# Check for parsing failures
curl http://localhost:9200/parsing-failures-*/_search
```

## 📄 License

MIT License - see LICENSE file for details.

---

**🎉 Enjoy your enhanced ELK stack with intelligent log parsing!**

## 📋 What You Get

### **Intelligent Log Processing**
✅ **Veeam Malware Detection** - Automatic ransomware threat identification  
✅ **OOTBI Storage Monitoring** - Proactive capacity alerting  
✅ **Smart Categorization** - Business impact classification  
✅ **Automated Alerting** - Priority-based notifications  
✅ **Executive Dashboards** - Management-ready visualizations  

### **Production Features**
✅ **Index Lifecycle Management** - Automated data retention  
✅ **Security Templates** - Optimized field mappings  
✅ **Performance Tuning** - High-volume log processing  
✅ **Error Handling** - Robust parsing with failure recovery  
✅ **Monitoring Stack** - Health checks for your ELK infrastructure  

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Veeam/OOTBI   │───▶│    Logstash     │───▶│  Elasticsearch  │
│   Syslog Logs   │    │   (Port 5514)   │    │   (Port 9200)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │     Parsing     │    │     Kibana      │
                       │   Intelligence  │    │   (Port 5601)   │
                       └─────────────────┘    └─────────────────┘
```

## 📊 Sample Dashboards

The repository includes pre-built dashboards for:

- **Executive Overview** - High-level security and operational metrics
- **Security Operations** - Threat detection and user risk analysis  
- **Storage Management** - Capacity monitoring and trend analysis
- **System Health** - ELK stack performance monitoring

## 🔧 Configuration Examples

### Before (Raw Syslog)
```
Jul 18, 2025 @ 12:23:53.767 ootbiprod02 OOTBI: - - [ootbi_event@61062 Node="ootbiprod02"...] Cluster node has less than 40% free space.
```

### After (Structured Intelligence)
```json
{
  "node_name": "ootbiprod02",
  "event_category": "storage", 
  "severity": "warning",
  "business_impact": "medium",
  "alert_priority": "standard",
  "event_message": "Cluster node has less than 40% free space"
}
```

## 🚦 Prerequisites

- **Docker**: Version 20.10 or higher
- **Docker Compose**: Version 2.0 or higher  
- **Memory**: Minimum 4GB RAM available for containers
- **Disk Space**: 10GB+ for logs and indices
- **Network**: Port 5514 (syslog), 5601 (Kibana), 9200 (Elasticsearch)

## 📁 Repository Structure

```
elk-veeam-ootbi/
├── docker-compose.yml              # Main orchestration file
├── .env                           # Environment variables
├── README.md                      # This file
├── 
├── elasticsearch/
│   ├── config/
│   │   └── elasticsearch.yml      # ES configuration
│   └── templates/
│       └── syslog-template.json   # Index templates
├── 
├── logstash/
│   ├── config/
│   │   └── logstash.yml          # Logstash configuration
│   ├── pipeline/
│   │   └── logstash.conf         # Main parsing logic
│   └── patterns/
│       └── custom-patterns       # Custom grok patterns
├── 
├── kibana/
│   ├── config/
│   │   └── kibana.yml            # Kibana configuration
│   └── dashboards/
│       ├── executive-overview.json
│       ├── security-operations.json
│       ├── storage-management.json
│       └── system-health.json
├── 
├── sample-data/
│   ├── veeam-samples.log         # Test Veeam logs
│   ├── ootbi-samples.log         # Test OOTBI logs
│   └── send-test-logs.sh         # Script to send sample data
├── 
├── monitoring/
│   ├── metricbeat.yml            # Stack monitoring
│   └── heartbeat.yml             # Uptime monitoring
├── 
└── docs/
    ├── DEPLOYMENT.md             # Production deployment guide
    ├── TROUBLESHOOTING.md        # Common issues and solutions
    ├── CUSTOMIZATION.md          # Adapting for your environment
    └── SCALING.md                # Performance and scaling guide
```

## 🛠️ Installation Guide

### Step 1: Clone and Setup

```bash
git clone https://github.com/yourusername/elk-veeam-ootbi.git
cd elk-veeam-ootbi

# Copy environment template
cp .env.example .env

# Edit configuration (optional)
nano .env
```

### Step 2: Start the Stack

```bash
# Start all services
docker-compose up -d

# Watch the logs (optional)
docker-compose logs -f
```

### Step 3: Initialize Elasticsearch

```bash
# Apply index templates
./scripts/setup-templates.sh

# Set up index lifecycle policies
./scripts/setup-ilm.sh
```

### Step 4: Import Dashboards

```bash
# Import all dashboards
./scripts/import-dashboards.sh

# Or import individually
curl -X POST "localhost:5601/api/saved_objects/_import" \
  -H "Content-Type: application/json" \
  -d @kibana/dashboards/executive-overview.json
```

### Step 5: Test with Sample Data

```bash
# Send test logs
./sample-data/send-test-logs.sh

# Verify in Kibana
open http://localhost:5601/app/discover
```

## 🧪 Testing Your Setup

### Send Sample Veeam Malware Alert
```bash
echo '<14>Mar 20, 2025 @ 14:06:09.662 VBRSRV01 Veeam_MP: [origin enterpriseId="31023"] [categoryId=0 instanceId=41600 DetectionTimeUTC="03/20/2025 13:05:41" OibID="0e54d3bf-add8-48eb-9122-fad3ac1e8fb3" ActivityType="EncryptedData" UserName="TECH\user1" ObjectName="VM01" VbrHostName="vbrsrv01.tech.local" Description="Potential malware activity detected"]' | nc -u localhost 5514
```

### Send Sample OOTBI Storage Alert
```bash
echo '<14>Jul 18, 2025 @ 12:23:53.767 ootbiprod02 OOTBI: - - [ootbi_event@61062 Node="ootbiprod02" NodeId="E08F3180-B0DC-11EF-AC5B-799EABA66937" DateTime="18/07/2025 16:23:48 UTC" Source="Storage" EventId="2510" ProductVersion="1.6.70.11503" EDataVersion="1"] Cluster node "ootbiprod02" has less than 40% of free space.' | nc -u localhost 5514
```

### Verify Processing
1. Open Kibana: http://localhost:5601
2. Go to **Discover** → Select `syslog-*` data view
3. Look for your test messages with parsed fields

## 📈 Monitoring Your Stack

### Health Check
```bash
# Check all services
./scripts/health-check.sh

# Individual service status
docker-compose ps
curl http://localhost:9200/_cluster/health
curl http://localhost:5601/api/status
```

### Performance Monitoring
```bash
# Start Metricbeat for stack monitoring
docker-compose -f docker-compose.monitoring.yml up -d

# View stack performance in Kibana
open http://localhost:5601/app/monitoring
```

## 🔐 Security Configuration

### Change Default Passwords
```bash
# Generate new passwords
./scripts/generate-passwords.sh

# Update .env file with new credentials
nano .env

# Restart stack
docker-compose down && docker-compose up -d
```

### Enable TLS (Production)
```bash
# Generate certificates
./scripts/generate-certs.sh

# Update docker-compose for TLS
cp docker-compose.secure.yml docker-compose.yml

# Restart with security
docker-compose up -d
```

## 🚀 Production Deployment

### Resource Requirements

| Component | Minimum | Recommended | High-Volume |
|-----------|---------|-------------|-------------|
| **Elasticsearch** | 2GB RAM | 4GB RAM | 8GB+ RAM |
| **Logstash** | 1GB RAM | 2GB RAM | 4GB+ RAM |
| **Kibana** | 512MB RAM | 1GB RAM | 2GB RAM |
| **Total** | 3.5GB RAM | 7GB RAM | 14GB+ RAM |

### Scaling Configuration

For high-volume environments, use the production compose file:

```bash
# Use production configuration
cp docker-compose.production.yml docker-compose.yml

# Scale Logstash instances
docker-compose up -d --scale logstash=3

# Scale Elasticsearch cluster
docker-compose up -d --scale elasticsearch=3
```

## 🛠️ Customization

### Adding New Log Sources

1. **Create parsing patterns** in `logstash/patterns/`
2. **Update** `logstash/pipeline/logstash.conf`
3. **Add field mappings** to `elasticsearch/templates/`
4. **Test** with sample data
5. **Create dashboards** in Kibana

### Custom Grok Patterns
```ruby
# Add to logstash/patterns/custom-patterns
CUSTOM_TIMESTAMP %{MONTHDAY}/%{MONTHNUM}/%{YEAR} %{TIME}
MY_APP_LOG %{CUSTOM_TIMESTAMP:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}
```

### Environment-Specific Configuration
```bash
# Development
cp .env.development .env

# Production  
cp .env.production .env

# Testing
cp .env.testing .env
```

## 🐛 Troubleshooting

### Common Issues

#### Logs Not Appearing in Kibana
```bash
# Check Logstash processing
docker-compose logs logstash

# Verify Elasticsearch indexing
curl "localhost:9200/syslog-*/_count"

# Check for parsing failures
curl "localhost:9200/parsing-failures-*/_search"
```

#### Performance Issues
```bash
# Check resource usage
docker stats

# Tune JVM heap (in .env)
ES_JAVA_OPTS=-Xms2g -Xmx4g
LS_JAVA_OPTS=-Xms1g -Xmx2g

# Restart services
docker-compose restart
```

#### Grok Pattern Failures
```bash
# Test patterns in Kibana Dev Tools
POST _grok/debugger
{
  "pattern": "%{YOUR_PATTERN}",
  "text": "your sample log line"
}
```

See [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for detailed solutions.

## 📚 Documentation

- **[Deployment Guide](docs/DEPLOYMENT.md)** - Production deployment best practices
- **[Customization Guide](docs/CUSTOMIZATION.md)** - Adapting for your environment  
- **[Scaling Guide](docs/SCALING.md)** - Performance optimization and scaling
- **[Troubleshooting](docs/TROUBLESHOOTING.md)** - Common issues and solutions

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md).

### How to Contribute
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Areas for Contribution
- Additional log source parsers (Cisco, Palo Alto, Windows Event Logs)
- Enhanced dashboard templates
- Performance optimizations
- Documentation improvements
- Bug fixes and testing

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Elastic Stack](https://www.elastic.co/) for the amazing ELK platform
- [Veeam](https://www.veeam.com/) for backup and security capabilities
- [Object First](https://www.objectfirst.com/) for immutable backup storage
- Community contributors and testers

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/elk-veeam-ootbi/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/elk-veeam-ootbi/discussions)
- **Blog Series**: [Original Blog Post](https://yourblog.com/elk-series)
- **Community**: [Elastic Community](https://discuss.elastic.co/)

## 🗺️ Roadmap

### v2.0 (Coming Soon)
- [ ] Machine Learning anomaly detection
- [ ] Advanced correlation rules
- [ ] Multi-tenant support
- [ ] Cloud deployment templates (AWS, Azure, GCP)

### v2.1 (Future)
- [ ] SIEM integration (Splunk, QRadar)
- [ ] Advanced threat intelligence feeds
- [ ] Custom alerting workflows
- [ ] Mobile dashboard support

---

**⭐ If this repository helps you, please give it a star!**

**🐛 Found a bug?** [Report it here](https://github.com/yourusername/elk-veeam-ootbi/issues)

**💡 Have an idea?** [Start a discussion](https://github.com/yourusername/elk-veeam-ootbi/discussions)
