# ELK Stack Tutorial - Docker Lab Environment

> **Part of the SIEM & Monitoring Blog Series by Geoffrey Burke**

This repository contains everything you need to set up a functional ELK (Elasticsearch, Logstash, Kibana) test environment using Docker. This is a hands-on companion to the blog post ["Hands-On Practice with the ELK Stack"](https://objectfirst.com/blog/hands-on-practice-with-the-elk-stack/) and provides a ready-to-run environment for learning how the ELK stack processes and visualizes log data.

## 🎯 What You'll Learn

- How to deploy a complete ELK stack using Docker Compose
- Configure Logstash to parse Veeam Backup & Replication syslog messages
- Create data views and explore logs in Kibana
- Set up syslog forwarding from applications to ELK

## 📋 Prerequisites

- **Ubuntu Server VM** (or any Linux environment with Docker support)
- **Docker** and **Docker Compose** installed
- Basic familiarity with command line operations
- *(Optional)* Veeam Backup & Replication for testing syslog forwarding

## 🚀 Quick Start

### 1. Install Docker (if not already installed)

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

**Important:** Log out and log back in after installation, then verify:

```bash
docker --version
docker compose version
```

### 2. Clone and Run

```bash
git clone https://github.com/ober73/elk-stack.git
cd elk-stack
docker compose up -d
```

That's it! Your ELK stack is now running.

### 3. Access Kibana

Open your browser and navigate to:
- **If using VM with hostname:** `http://your-vm-hostname:5601`
- **If using IP address:** `http://your-vm-ip:5601`
- **If running locally:** `http://localhost:5601`

## 📁 Repository Structure

```
elk-stack/
├── README.md                           # This file
├── docker-compose.yml                  # Docker services configuration
├── elasticsearch/
│   └── elasticsearch.yml               # Elasticsearch configuration
└── logstash/
    └── pipeline/
        └── logstash.conf               # Logstash pipeline configuration
```

## ⚙️ Configuration Details

### Elasticsearch
- **Port:** 9200
- **Configuration:** Single-node cluster with security disabled (test environment only)
- **Data persistence:** Uses Docker volumes for data storage

### Logstash
- **TCP Input:** Port 5000 (JSON format)
- **Syslog Input:** Port 5514 (UDP)
- **Pipeline:** Pre-configured to parse Veeam logs and general syslog messages
- **Output:** Sends to Elasticsearch with date-based indices

### Kibana
- **Port:** 5601
- **Configuration:** Auto-connects to Elasticsearch
- **Features:** Full Kibana interface for data exploration and visualization

## 🔧 Testing with Sample Data

### Option 1: Configure Veeam VBR (if available)

1. In Veeam VBR, go to **Menu > Options**
2. Click the **Event Forwarding** tab
3. Add your ELK server:
   - **Server:** Your Docker host IP/hostname
   - **Port:** 5514
   - **Protocol:** UDP

Veeam will automatically send a test message when you save the configuration.

### Option 2: Send Test Syslog Messages

```bash
# Send a test syslog message
echo '<14>$(date --rfc-3339=seconds) your-hostname Veeam.Backup.Manager[1234]: Job [Test Backup Job] completed with Success' | nc -u -w1 your-docker-host 5514

# Send a test JSON message via TCP
echo '{"message": "Test log entry", "level": "info", "timestamp": "'$(date -Iseconds)'"}' | nc your-docker-host 5000
```

## 📊 Exploring Data in Kibana

1. **Access Kibana** at `http://your-host:5601`
2. **Create a Data View:**
   - Go to **Menu > Discover**
   - Click **"Create data view"**
   - Use index pattern: `veeam-logs-*` or `logs-*`
   - Set timestamp field: `@timestamp`
3. **Explore your logs** in the Discover interface

## 🛠️ Troubleshooting

### Container Issues
```bash
# Check container status
docker compose ps

# View logs
docker compose logs elasticsearch
docker compose logs logstash
docker compose logs kibana

# Restart services
docker compose restart
```

### Common Problems

**Elasticsearch won't start:**
- Check available memory (requires at least 1GB)
- Verify ports 9200 aren't already in use: `netstat -tulpn | grep 9200`

**Logstash not receiving logs:**
- Verify firewall allows ports 5000 (TCP) and 5514 (UDP)
- Check logstash logs: `docker compose logs logstash`

**Kibana connection issues:**
- Ensure Elasticsearch is healthy: `curl http://localhost:9200/_cluster/health`
- Wait for all services to fully start (can take 2-3 minutes)

## 🔄 Reset Environment

To completely reset your environment and start fresh:

```bash
docker compose down --volumes --remove-orphans
docker system prune -a --volumes -f
```

**⚠️ Warning:** This will delete all data, indices, and configurations.

## 🎓 Learning Exercises

Once your environment is running, try these exercises:

1. **Create custom log entries** and observe how Logstash processes them
2. **Build visualizations** in Kibana dashboards
3. **Modify the Logstash configuration** to parse different log formats
4. **Experiment with Elasticsearch queries** using the Dev Tools console

## 📚 Additional Resources

- [Official ELK Documentation](https://www.elastic.co/guide/)
- [Logstash Configuration Reference](https://www.elastic.co/guide/en/logstash/current/configuration.html)
- [Kibana User Guide](https://www.elastic.co/guide/en/kibana/current/index.html)
- **Blog Series:** [SIEM & Monitoring with ELK](your-blog-series-url)

## ⚠️ Important Notes

- **This is a test environment only** - security is disabled for simplicity
- **Not suitable for production** - lacks authentication, encryption, and proper security configurations
- **Resource usage** - Monitor your system resources as ELK can be memory-intensive

## 🤝 Contributing

Found an issue or have suggestions? Feel free to:
- Open an issue
- Submit a pull request
- Reach out via the blog comments

---

**Happy logging!** 🚀

*This lab environment is part of the SIEM & Monitoring Blog Series. Stay tuned for the next post covering ElasticSecurity and Fluentd alternatives.*
