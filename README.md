# nocodb Installation Guide

nocodb is a free and open-source open source Airtable alternative. NocoDB turns any database into a smart spreadsheet with API access, serving as an open-source alternative to Airtable or Microsoft Access

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum (2+ recommended)
  - RAM: 1GB minimum (2GB+ recommended)
  - Storage: 1GB for application
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default nocodb port)
  - Database connections
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install nocodb
sudo dnf install -y nocodb

# Enable and start service
sudo systemctl enable --now nocodb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
nocodb --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nocodb
sudo apt install -y nocodb

# Enable and start service
sudo systemctl enable --now nocodb

# Configure firewall
sudo ufw allow 8080

# Verify installation
nocodb --version
```

### Arch Linux

```bash
# Install nocodb
sudo pacman -S nocodb

# Enable and start service
sudo systemctl enable --now nocodb

# Verify installation
nocodb --version
```

### Alpine Linux

```bash
# Install nocodb
apk add --no-cache nocodb

# Enable and start service
rc-update add nocodb default
rc-service nocodb start

# Verify installation
nocodb --version
```

### openSUSE/SLES

```bash
# Install nocodb
sudo zypper install -y nocodb

# Enable and start service
sudo systemctl enable --now nocodb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
nocodb --version
```

### macOS

```bash
# Using Homebrew
brew install nocodb

# Start service
brew services start nocodb

# Verify installation
nocodb --version
```

### FreeBSD

```bash
# Using pkg
pkg install nocodb

# Enable in rc.conf
echo 'nocodb_enable="YES"' >> /etc/rc.conf

# Start service
service nocodb start

# Verify installation
nocodb --version
```

### Windows

```bash
# Using Chocolatey
choco install nocodb

# Or using Scoop
scoop install nocodb

# Verify installation
nocodb --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nocodb

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nocodb --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nocodb

# Start service
sudo systemctl start nocodb

# Stop service
sudo systemctl stop nocodb

# Restart service
sudo systemctl restart nocodb

# Check status
sudo systemctl status nocodb

# View logs
sudo journalctl -u nocodb -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nocodb default

# Start service
rc-service nocodb start

# Stop service
rc-service nocodb stop

# Restart service
rc-service nocodb restart

# Check status
rc-service nocodb status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nocodb_enable="YES"' >> /etc/rc.conf

# Start service
service nocodb start

# Stop service
service nocodb stop

# Restart service
service nocodb restart

# Check status
service nocodb status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nocodb
brew services stop nocodb
brew services restart nocodb

# Check status
brew services list | grep nocodb
```

### Windows Service Manager

```powershell
# Start service
net start nocodb

# Stop service
net stop nocodb

# Using PowerShell
Start-Service nocodb
Stop-Service nocodb
Restart-Service nocodb

# Check status
Get-Service nocodb
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nocodb_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name nocodb.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nocodb.example.com;

    ssl_certificate /etc/ssl/certs/nocodb.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nocodb.example.com.key;

    location / {
        proxy_pass http://nocodb_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName nocodb.example.com
    Redirect permanent / https://nocodb.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nocodb.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nocodb.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nocodb.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nocodb_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nocodb.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nocodb_backend

backend nocodb_backend
    balance roundrobin
    server nocodb1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nocodb:nocodb /etc/nocodb
sudo chmod 750 /etc/nocodb

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status nocodb

# View logs
sudo journalctl -u nocodb -f

# Monitor resource usage
top -p $(pgrep nocodb)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nocodb"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nocodb-backup-$DATE.tar.gz" /etc/nocodb /var/lib/nocodb

echo "Backup completed: $BACKUP_DIR/nocodb-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nocodb

# Restore from backup
tar -xzf /backup/nocodb/nocodb-backup-*.tar.gz -C /

# Start service
sudo systemctl start nocodb
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nocodb -n 100
sudo tail -f /var/log/nocodb/nocodb.log

# Check configuration
nocodb --version

# Check permissions
ls -la /etc/nocodb
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nocodb)

# Check disk I/O
iotop -p $(pgrep nocodb)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nocodb:
    image: nocodb:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/nocodb
      - ./data:/var/lib/nocodb
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nocodb

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nocodb

# Arch Linux
sudo pacman -Syu nocodb

# Alpine Linux
apk update && apk upgrade nocodb

# openSUSE
sudo zypper update nocodb

# FreeBSD
pkg update && pkg upgrade nocodb

# Always backup before updates
tar -czf /backup/nocodb-pre-update-$(date +%Y%m%d).tar.gz /etc/nocodb

# Restart after updates
sudo systemctl restart nocodb
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nocodb

# Clean old logs
find /var/log/nocodb -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nocodb
```

## Additional Resources

- Official Documentation: https://docs.nocodb.org/
- GitHub Repository: https://github.com/nocodb/nocodb
- Community Forum: https://forum.nocodb.org/
- Best Practices Guide: https://docs.nocodb.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
