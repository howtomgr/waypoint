# waypoint Installation Guide

waypoint is a free and open-source build, deploy, release across platforms. HashiCorp Waypoint provides a consistent workflow to build, deploy, and release applications across any platform

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
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum
  - Storage: 1GB for installation
  - Network: Platform API access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9701 (default waypoint port)
  - Port 9702 for UI
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

# Install waypoint
sudo dnf install -y waypoint

# Enable and start service
sudo systemctl enable --now waypoint

# Configure firewall
sudo firewall-cmd --permanent --add-port=9701/tcp
sudo firewall-cmd --reload

# Verify installation
waypoint version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install waypoint
sudo apt install -y waypoint

# Enable and start service
sudo systemctl enable --now waypoint

# Configure firewall
sudo ufw allow 9701

# Verify installation
waypoint version
```

### Arch Linux

```bash
# Install waypoint
sudo pacman -S waypoint

# Enable and start service
sudo systemctl enable --now waypoint

# Verify installation
waypoint version
```

### Alpine Linux

```bash
# Install waypoint
apk add --no-cache waypoint

# Enable and start service
rc-update add waypoint default
rc-service waypoint start

# Verify installation
waypoint version
```

### openSUSE/SLES

```bash
# Install waypoint
sudo zypper install -y waypoint

# Enable and start service
sudo systemctl enable --now waypoint

# Configure firewall
sudo firewall-cmd --permanent --add-port=9701/tcp
sudo firewall-cmd --reload

# Verify installation
waypoint version
```

### macOS

```bash
# Using Homebrew
brew install waypoint

# Start service
brew services start waypoint

# Verify installation
waypoint version
```

### FreeBSD

```bash
# Using pkg
pkg install waypoint

# Enable in rc.conf
echo 'waypoint_enable="YES"' >> /etc/rc.conf

# Start service
service waypoint start

# Verify installation
waypoint version
```

### Windows

```bash
# Using Chocolatey
choco install waypoint

# Or using Scoop
scoop install waypoint

# Verify installation
waypoint version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/waypoint

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
waypoint version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable waypoint

# Start service
sudo systemctl start waypoint

# Stop service
sudo systemctl stop waypoint

# Restart service
sudo systemctl restart waypoint

# Check status
sudo systemctl status waypoint

# View logs
sudo journalctl -u waypoint -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add waypoint default

# Start service
rc-service waypoint start

# Stop service
rc-service waypoint stop

# Restart service
rc-service waypoint restart

# Check status
rc-service waypoint status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'waypoint_enable="YES"' >> /etc/rc.conf

# Start service
service waypoint start

# Stop service
service waypoint stop

# Restart service
service waypoint restart

# Check status
service waypoint status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start waypoint
brew services stop waypoint
brew services restart waypoint

# Check status
brew services list | grep waypoint
```

### Windows Service Manager

```powershell
# Start service
net start waypoint

# Stop service
net stop waypoint

# Using PowerShell
Start-Service waypoint
Stop-Service waypoint
Restart-Service waypoint

# Check status
Get-Service waypoint
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream waypoint_backend {
    server 127.0.0.1:9701;
}

server {
    listen 80;
    server_name waypoint.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name waypoint.example.com;

    ssl_certificate /etc/ssl/certs/waypoint.example.com.crt;
    ssl_certificate_key /etc/ssl/private/waypoint.example.com.key;

    location / {
        proxy_pass http://waypoint_backend;
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
    ServerName waypoint.example.com
    Redirect permanent / https://waypoint.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName waypoint.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/waypoint.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/waypoint.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9701/
    ProxyPassReverse / http://127.0.0.1:9701/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend waypoint_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/waypoint.pem
    redirect scheme https if !{ ssl_fc }
    default_backend waypoint_backend

backend waypoint_backend
    balance roundrobin
    server waypoint1 127.0.0.1:9701 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R waypoint:waypoint /etc/waypoint
sudo chmod 750 /etc/waypoint

# Configure firewall
sudo firewall-cmd --permanent --add-port=9701/tcp
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
sudo systemctl status waypoint

# View logs
sudo journalctl -u waypoint -f

# Monitor resource usage
top -p $(pgrep waypoint)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/waypoint"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/waypoint-backup-$DATE.tar.gz" /etc/waypoint /var/lib/waypoint

echo "Backup completed: $BACKUP_DIR/waypoint-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop waypoint

# Restore from backup
tar -xzf /backup/waypoint/waypoint-backup-*.tar.gz -C /

# Start service
sudo systemctl start waypoint
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u waypoint -n 100
sudo tail -f /var/log/waypoint/waypoint.log

# Check configuration
waypoint version

# Check permissions
ls -la /etc/waypoint
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9701

# Test connectivity
telnet localhost 9701

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep waypoint)

# Check disk I/O
iotop -p $(pgrep waypoint)

# Check connections
ss -an | grep 9701
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  waypoint:
    image: waypoint:latest
    ports:
      - "9701:9701"
    volumes:
      - ./config:/etc/waypoint
      - ./data:/var/lib/waypoint
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update waypoint

# Debian/Ubuntu
sudo apt update && sudo apt upgrade waypoint

# Arch Linux
sudo pacman -Syu waypoint

# Alpine Linux
apk update && apk upgrade waypoint

# openSUSE
sudo zypper update waypoint

# FreeBSD
pkg update && pkg upgrade waypoint

# Always backup before updates
tar -czf /backup/waypoint-pre-update-$(date +%Y%m%d).tar.gz /etc/waypoint

# Restart after updates
sudo systemctl restart waypoint
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/waypoint

# Clean old logs
find /var/log/waypoint -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/waypoint
```

## Additional Resources

- Official Documentation: https://docs.waypoint.org/
- GitHub Repository: https://github.com/waypoint/waypoint
- Community Forum: https://forum.waypoint.org/
- Best Practices Guide: https://docs.waypoint.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
