# Plant Master Server Specifications

## Overview
This document outlines the server specifications required for deploying the Plant Master application, a comprehensive manufacturing management system with real-time monitoring, work order management, and reporting capabilities.

## Architecture
- **Backend**: Django REST API with MySQL database
- **Frontend**: React.js SPA (Single Page Application)  
- **Communication**: HTTPS REST API with JWT authentication
- **Deployment**: Docker containerized with SSL/TLS encryption

---

## Minimum Server Requirements

### Production Environment

#### Hardware Specifications
| Component | Minimum | Recommended | Description |
|-----------|---------|-------------|-------------|
| **CPU** | 4 cores @ 2.4GHz | 8 cores @ 3.0GHz | Intel Xeon or AMD EPYC processors |
| **RAM** | 16 GB | 32 GB | For Django, React build, and database operations |
| **Storage** | 100 GB SSD | 500 GB NVMe SSD | Application, logs, and database storage |
| **Network** | 100 Mbps | 1 Gbps | For API communication and file transfers |

#### Operating System
- **Primary**: Ubuntu 22.04 LTS Server
- **Alternative**: CentOS 8+ / RHEL 8+
- **Windows**: Windows Server 2019/2022 (if required)

---

## Software Dependencies

### Core Runtime Environment
| Software | Version | Purpose |
|----------|---------|---------|
| **Docker** | 24.0+ | Container orchestration |
| **Docker Compose** | 2.20+ | Multi-container management |
| **Python** | 3.13.x | Backend runtime |
| **Node.js** | 19.6.0+ | Frontend build tooling |
| **MySQL** | 8.0+ | Primary database |
| **Redis** | 7.0+ | Caching and session storage |
| **Nginx** | 1.24+ | Reverse proxy and load balancer |

### SSL/TLS Requirements
- **SSL Certificates**: Valid certificates for HTTPS
- **Minimum TLS Version**: 1.2
- **Cipher Suites**: Modern cipher suites only
- **Certificate Authority**: Trusted CA or valid self-signed for development

---

## Database Requirements

### MySQL Configuration
```sql
-- Minimum MySQL settings for production
[mysqld]
innodb_buffer_pool_size = 8G        # 50-70% of available RAM
max_connections = 500
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 2
query_cache_size = 256M
thread_cache_size = 64
```

### Database Storage
- **Minimum Space**: 50 GB
- **Recommended**: 200 GB+ for logs and historical data
- **Backup Space**: Additional 100% of database size
- **IOPS**: Minimum 3000 IOPS for production workloads

---

## Application-Specific Requirements

### Backend API (Django/Python)
```yaml
Resources:
  CPU: 2-4 cores dedicated
  Memory: 8-16 GB
  Storage: 50 GB for application and logs
  
Key Dependencies:
  - Django 5.1.6
  - MySQL Client 2.2.6
  - Pandas/Numpy for data processing
  - Celery for background tasks
  - Channels for WebSocket support
```

### Frontend (React)
```yaml
Build Requirements:
  Node.js: 19.6.0+
  Memory: 4 GB allocated for build process
  Build Time: 5-15 minutes depending on hardware

Runtime (Static Files):
  Web Server: Nginx/Apache
  Storage: 500 MB for built assets
  CDN: Recommended for production
```

---

## Network and Security Requirements

### Port Configuration & Internet Access Strategy

#### **Public Internet Access (Required for End Users)**
| Port | Protocol | Service | Access Level | Purpose |
|------|----------|---------|--------------|---------|
| 443 | HTTPS | Web Application | **Public Internet** | Primary user access to Plant Master |
| 80 | HTTP | HTTP Redirect | **Public Internet** | Automatic redirect to HTTPS |

#### **CI/CD & Administration Access (Restricted Internet)**
| Port | Protocol | Service | Access Level | Purpose |
|------|----------|---------|--------------|---------|
| 22 | SSH | Server Management | **Restricted IPs Only** | Deployment scripts, server admin |
| 8080 | HTTP | CI/CD Webhooks | **CI/CD IPs Only** | GitHub/GitLab webhook endpoints |
| 2376 | TCP | Docker API | **CI/CD IPs Only** | Container deployment (optional) |

#### **Internal Services (No Internet Access)**
| Port | Protocol | Service | Access Level | Purpose |
|------|----------|---------|--------------|---------|
| 8000 | HTTP | Django API | **Internal Only** | Backend API (behind Nginx proxy) |
| 3306 | TCP | MySQL | **Internal Only** | Database server |
| 6379 | TCP | Redis | **Internal Only** | Cache and sessions |
| 5672 | TCP | RabbitMQ | **Internal Only** | Message queue (if using Celery) |

### Firewall Configuration

#### **Production Firewall Rules**
```bash
#!/bin/bash
# Plant Master Production Firewall Setup

# Reset firewall
ufw --force reset

# Default policies
ufw default deny incoming
ufw default allow outgoing

# === PUBLIC ACCESS (End Users) ===
# Web application access - open to internet
ufw allow 80/tcp comment "HTTP redirect to HTTPS"
ufw allow 443/tcp comment "HTTPS web application access"

# === CI/CD ACCESS (Restricted) ===
# Replace YOUR_CI_CD_IP with actual CI/CD server IPs
# Example: GitHub Actions, GitLab CI, Jenkins server IPs
ufw allow from YOUR_CI_CD_IP_1 to any port 22 comment "CI/CD SSH access"
ufw allow from YOUR_CI_CD_IP_2 to any port 22 comment "CI/CD SSH access"
ufw allow from YOUR_CI_CD_IP_1 to any port 8080 comment "CI/CD webhooks"

# === ADMINISTRATIVE ACCESS ===
# SSH access for system administrators (restrict to office IPs)
ufw allow from YOUR_OFFICE_IP to any port 22 comment "Admin SSH access"

# === BLOCK EVERYTHING ELSE ===
# Explicitly deny database and internal services
ufw deny 3306/tcp comment "Block MySQL from internet"
ufw deny 6379/tcp comment "Block Redis from internet"
ufw deny 8000/tcp comment "Block direct API access"

# Enable firewall
ufw enable
```

#### **Development/Testing Environment**
```bash
# More permissive for development
ufw allow 22/tcp comment "SSH access"
ufw allow 80/tcp comment "HTTP"
ufw allow 443/tcp comment "HTTPS"
ufw allow 8000/tcp comment "Django dev server"
ufw allow 3000/tcp comment "React dev server"
```

### CI/CD Pipeline Requirements

#### **Required Internet Access for CI/CD**
```yaml
Outbound Access Needed:
  - GitHub/GitLab: Pull source code and releases
  - Docker Hub/Registry: Pull base images and push builds
  - NPM Registry: Download Node.js packages for React builds
  - PyPI: Download Python packages for Django
  - SSL Certificate Authority: Certificate validation and renewal

Recommended CI/CD Workflow:
  1. CI server pulls code from repository
  2. CI server builds Docker images with dependencies
  3. CI server pushes to private container registry
  4. Production server pulls from private registry via SSH
  5. Deploy using Docker Compose with zero-downtime strategy
```

#### **CI/CD Security Requirements**
```yaml
Authentication & Access:
  - SSH Key-based authentication only (disable password auth)
  - Dedicated CI/CD service accounts with limited sudo privileges
  - IP whitelist for CI/CD servers and webhook endpoints
  - Separate deployment keys for different environments

Deployment Security:
  - Use CI/CD secrets management for environment variables
  - Deploy to staging environment first for validation
  - Automated security scanning of containers and dependencies  
  - Database migration scripts require manual review
  - Automated rollback capability for failed deployments
```

### Network Architecture
```
Internet Users
    │
    ├── Port 443 (HTTPS) ────► Nginx ────► Plant Master App
    ├── Port 80 (HTTP) ─────► Nginx ────► HTTPS Redirect
    │
CI/CD Servers ──── Port 22 (SSH) ────► Production Server
    │                    │
    └── Port 8080 ───────┘ (Webhooks)
    
Production Server (Internal Network):
    ├── Port 8000: Django API (Nginx proxy)
    ├── Port 3306: MySQL Database
    ├── Port 6379: Redis Cache
    └── Port 5672: RabbitMQ (optional)
```

### Security Hardening Requirements
- **Fail2Ban**: Protection against brute force attacks
- **Regular Updates**: Automated security patches for OS and containers
- **Backup Strategy**: Daily automated backups with off-site storage
- **Monitoring**: Application and system monitoring with alerting
- **Log Rotation**: Prevent disk space issues with proper log management
- **SSL/TLS**: Strong cipher suites and automatic certificate renewal
- **Container Security**: Regular vulnerability scanning and image updates

---

## Performance Optimization

### Application Level
```python
# Django Settings for Production
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'sql_mode': 'traditional',
            'charset': 'utf8mb4',
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
        }
    }
}

# Caching Configuration
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}
```

### Web Server (Nginx) Configuration
```nginx
# /etc/nginx/sites-available/plant-master
server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    # SSL Configuration
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Performance Settings
    client_max_body_size 100M;
    keepalive_timeout 65;
    gzip on;
    gzip_types text/plain text/css application/javascript;
    
    # API Proxy
    location /api/ {
        proxy_pass https://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # Static Files
    location / {
        root /var/www/plant-master/build;
        try_files $uri $uri/ /index.html;
    }
}
```

---

## Monitoring and Logging

### Log Storage Requirements
- **Application Logs**: 500 MB per day (rotating)
- **Access Logs**: 1 GB per day (rotating)
- **Error Logs**: 100 MB per day (rotating)
- **Retention**: 30 days minimum, 90 days recommended

### Monitoring Metrics
```yaml
Application Metrics:
  - Response time < 500ms (95th percentile)
  - Error rate < 1%
  - Database query time < 100ms average
  - Memory usage < 80%
  - CPU usage < 70% average

System Metrics:
  - Disk usage < 80%
  - Network latency < 50ms
  - Uptime > 99.5%
  - SSL certificate validity
```

---

## Backup and Disaster Recovery

### Backup Strategy
```bash
# Daily MySQL backup
mysqldump --single-transaction --routines --triggers \
  --all-databases > backup_$(date +%Y%m%d).sql

# Application files backup
tar -czf app_backup_$(date +%Y%m%d).tar.gz /app/

# Retention: 7 daily, 4 weekly, 12 monthly
```

### Recovery Requirements
- **RTO** (Recovery Time Objective): 4 hours
- **RPO** (Recovery Point Objective): 24 hours
- **Backup Location**: Off-site storage required
- **Testing**: Monthly backup restore testing

---

## Hosting Provider Requirements

### **Port Configuration Request**
Request the following port configuration from your hosting provider:

```yaml
Public Internet Access (Required):
  - Port 443 (HTTPS): Full internet access for end users
  - Port 80 (HTTP): Full internet access for HTTPS redirects

Restricted Access (CI/CD & Administration):
  - Port 22 (SSH): Access restricted to CI/CD server IPs and admin IPs
  - Port 8080 (Webhooks): Access restricted to CI/CD server IPs only
  - Port 2376 (Docker API): Optional, restricted to CI/CD IPs only

Internal Services (No Internet Access):
  - Port 8000 (Django API): Internal network only
  - Port 3306 (MySQL): Internal network only  
  - Port 6379 (Redis): Internal network only
  - Port 5672 (RabbitMQ): Internal network only
```

### **Network & Security Requirements**
```yaml
Firewall Configuration:
  - Advanced firewall rules support (UFW or equivalent)
  - IP whitelisting capabilities for CI/CD access
  - DDoS protection for public-facing ports
  - Intrusion detection and prevention systems

SSL/TLS Requirements:
  - SSL certificate installation support
  - Automatic certificate renewal capability
  - Modern TLS protocols (1.2+ minimum)
  - Custom cipher suite configuration

Outbound Internet Access:
  - GitHub/GitLab access for repository cloning
  - Docker Hub/Registry access for image pulls
  - NPM Registry access for Node.js packages
  - PyPI access for Python packages
  - Certificate Authority access for SSL validation
```

### **Infrastructure Requirements**
```yaml
Server Specifications:
  - Ubuntu 22.04 LTS (preferred) or equivalent
  - Root/sudo access for system configuration
  - Docker and Docker Compose installation support
  - Static IP address assignment
  - Minimum 16GB RAM, 4 CPU cores, 100GB SSD storage

Network Features:
  - Load balancer support (if multi-server deployment)
  - Content Delivery Network (CDN) integration
  - Database backup and restore capabilities
  - Log aggregation and monitoring tools access

Compliance & Support:
  - 24/7 technical support availability
  - SLA guarantee (99.9% uptime minimum)
  - Security incident response procedures
  - Regular security patching and maintenance windows
```

### **CI/CD Integration Requirements**
```yaml
Required for Automated Deployment:
  - SSH key-based authentication setup
  - Webhook endpoint configuration (Port 8080)
  - Container registry access (Docker Hub/private registry)
  - Environment variable management system
  - Automated backup verification before deployments

CI/CD IP Whitelist (To be provided by development team):
  - GitHub Actions IP ranges: [To be specified]
  - GitLab CI IP ranges: [To be specified]  
  - Office/Admin IP addresses: [To be specified]
  - Development team VPN IPs: [To be specified]
```

### **Monitoring & Maintenance**
```yaml
Required Monitoring Capabilities:
  - Server resource monitoring (CPU, RAM, disk, network)
  - Application performance monitoring setup
  - Log rotation and retention policies
  - Automated backup scheduling and verification
  - SSL certificate expiration monitoring
  - Security scan reports and vulnerability assessments

Maintenance Windows:
  - Scheduled maintenance windows for security updates
  - Emergency patch deployment procedures
  - Rollback capabilities for failed deployments
  - Database maintenance and optimization schedules
```

---

## Hosting Provider Requirements

### **Port Configuration Request**
Request the following port configuration from your hosting provider:

```yaml
Public Internet Access (Required):
  - Port 443 (HTTPS): Full internet access for end users
  - Port 80 (HTTP): Full internet access for HTTPS redirects

Restricted Access (CI/CD & Administration):
  - Port 22 (SSH): Access restricted to CI/CD server IPs and admin IPs
  - Port 8080 (Webhooks): Access restricted to CI/CD server IPs only
  - Port 2376 (Docker API): Optional, restricted to CI/CD IPs only

Internal Services (No Internet Access):
  - Port 8000 (Django API): Internal network only
  - Port 3306 (MySQL): Internal network only  
  - Port 6379 (Redis): Internal network only
  - Port 5672 (RabbitMQ): Internal network only
```

### **Network & Security Requirements**
```yaml
Firewall Configuration:
  - Advanced firewall rules support (UFW or equivalent)
  - IP whitelisting capabilities for CI/CD access
  - DDoS protection for public-facing ports
  - Intrusion detection and prevention systems

SSL/TLS Requirements:
  - SSL certificate installation support
  - Automatic certificate renewal capability
  - Modern TLS protocols (1.2+ minimum)
  - Custom cipher suite configuration

Outbound Internet Access:
  - GitHub/GitLab access for repository cloning
  - Docker Hub/Registry access for image pulls
  - NPM Registry access for Node.js packages
  - PyPI access for Python packages
  - Certificate Authority access for SSL validation
```

### **Infrastructure Requirements**
```yaml
Server Specifications:
  - Ubuntu 22.04 LTS (preferred) or equivalent
  - Root/sudo access for system configuration
  - Docker and Docker Compose installation support
  - Static IP address assignment
  - Minimum 16GB RAM, 4 CPU cores, 100GB SSD storage

Network Features:
  - Load balancer support (if multi-server deployment)
  - Content Delivery Network (CDN) integration
  - Database backup and restore capabilities
  - Log aggregation and monitoring tools access

Compliance & Support:
  - 24/7 technical support availability
  - SLA guarantee (99.9% uptime minimum)
  - Security incident response procedures
  - Regular security patching and maintenance windows
```

### **CI/CD Integration Requirements**
```yaml
Required for Automated Deployment:
  - SSH key-based authentication setup
  - Webhook endpoint configuration (Port 8080)
  - Container registry access (Docker Hub/private registry)
  - Environment variable management system
  - Automated backup verification before deployments

CI/CD IP Whitelist (To be provided by development team):
  - GitHub Actions IP ranges: [To be specified]
  - GitLab CI IP ranges: [To be specified]  
  - Office/Admin IP addresses: [To be specified]
  - Development team VPN IPs: [To be specified]
```

### **Monitoring & Maintenance**
```yaml
Required Monitoring Capabilities:
  - Server resource monitoring (CPU, RAM, disk, network)
  - Application performance monitoring setup
  - Log rotation and retention policies
  - Automated backup scheduling and verification
  - SSL certificate expiration monitoring
  - Security scan reports and vulnerability assessments

Maintenance Windows:
  - Scheduled maintenance windows for security updates
  - Emergency patch deployment procedures
  - Rollback capabilities for failed deployments
  - Database maintenance and optimization schedules
```

---

## Hosting Provider Requirements

### **Port Configuration Request**
Request the following port configuration from your hosting provider:

```yaml
Public Internet Access (Required):
  - Port 443 (HTTPS): Full internet access for end users
  - Port 80 (HTTP): Full internet access for HTTPS redirects

Restricted Access (CI/CD & Administration):
  - Port 22 (SSH): Access restricted to CI/CD server IPs and admin IPs
  - Port 8080 (Webhooks): Access restricted to CI/CD server IPs only
  - Port 2376 (Docker API): Optional, restricted to CI/CD IPs only

Internal Services (No Internet Access):
  - Port 8000 (Django API): Internal network only
  - Port 3306 (MySQL): Internal network only  
  - Port 6379 (Redis): Internal network only
  - Port 5672 (RabbitMQ): Internal network only
```

### **Network & Security Requirements**
```yaml
Firewall Configuration:
  - Advanced firewall rules support (UFW or equivalent)
  - IP whitelisting capabilities for CI/CD access
  - DDoS protection for public-facing ports
  - Intrusion detection and prevention systems

SSL/TLS Requirements:
  - SSL certificate installation support
  - Automatic certificate renewal capability
  - Modern TLS protocols (1.2+ minimum)
  - Custom cipher suite configuration

Outbound Internet Access:
  - GitHub/GitLab access for repository cloning
  - Docker Hub/Registry access for image pulls
  - NPM Registry access for Node.js packages
  - PyPI access for Python packages
  - Certificate Authority access for SSL validation
```

### **Infrastructure Requirements**
```yaml
Server Specifications:
  - Ubuntu 22.04 LTS (preferred) or equivalent
  - Root/sudo access for system configuration
  - Docker and Docker Compose installation support
  - Static IP address assignment
  - Minimum 16GB RAM, 4 CPU cores, 100GB SSD storage

Network Features:
  - Load balancer support (if multi-server deployment)
  - Content Delivery Network (CDN) integration
  - Database backup and restore capabilities
  - Log aggregation and monitoring tools access

Compliance & Support:
  - 24/7 technical support availability
  - SLA guarantee (99.9% uptime minimum)
  - Security incident response procedures
  - Regular security patching and maintenance windows
```

### **CI/CD Integration Requirements**
```yaml
Required for Automated Deployment:
  - SSH key-based authentication setup
  - Webhook endpoint configuration (Port 8080)
  - Container registry access (Docker Hub/private registry)
  - Environment variable management system
  - Automated backup verification before deployments

CI/CD IP Whitelist (To be provided by development team):
  - GitHub Actions IP ranges: [To be specified]
  - GitLab CI IP ranges: [To be specified]  
  - Office/Admin IP addresses: [To be specified]
  - Development team VPN IPs: [To be specified]
```

### **Monitoring & Maintenance**
```yaml
Required Monitoring Capabilities:
  - Server resource monitoring (CPU, RAM, disk, network)
  - Application performance monitoring setup
  - Log rotation and retention policies
  - Automated backup scheduling and verification
  - SSL certificate expiration monitoring
  - Security scan reports and vulnerability assessments

Maintenance Windows:
  - Scheduled maintenance windows for security updates
  - Emergency patch deployment procedures
  - Rollback capabilities for failed deployments
  - Database maintenance and optimization schedules
```

---

## Deployment Checklist

### Pre-Deployment
- [ ] Server provisioning completed
- [ ] Operating system hardened
- [ ] Docker and Docker Compose installed
- [ ] SSL certificates obtained and configured
- [ ] Database server installed and configured
- [ ] Firewall rules configured
- [ ] Monitoring tools installed

### Application Deployment
- [ ] Application containers built and tested
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] Static files built and deployed
- [ ] SSL/HTTPS functionality verified
- [ ] API endpoints tested
- [ ] User authentication tested

### Post-Deployment
- [ ] Performance benchmarks completed
- [ ] Monitoring alerts configured
- [ ] Backup procedures tested
- [ ] Documentation updated
- [ ] Team training completed

---

## Contact and Support

For technical support regarding Plant Master deployment:

- **Documentation**: Check project README and API documentation
- **Issues**: Create issues in the project repository
- **Emergency**: Contact system administrator team

---

**Document Version**: 1.1  
**Last Updated**: January 2025  
**Next Review**: July 2025  
**Latest Changes**: Added comprehensive CI/CD and port configuration requirements
