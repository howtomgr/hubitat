# hubitat Installation Guide

hubitat is a free and open-source home automation hub. Hubitat provides local processing home automation hub

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
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for data
  - Network: Zigbee/Z-Wave
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default hubitat port)
  - None
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

# Install hubitat
sudo dnf install -y hubitat

# Enable and start service
sudo systemctl enable --now hubitat

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
hubitat --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install hubitat
sudo apt install -y hubitat

# Enable and start service
sudo systemctl enable --now hubitat

# Configure firewall
sudo ufw allow 80

# Verify installation
hubitat --version
```

### Arch Linux

```bash
# Install hubitat
sudo pacman -S hubitat

# Enable and start service
sudo systemctl enable --now hubitat

# Verify installation
hubitat --version
```

### Alpine Linux

```bash
# Install hubitat
apk add --no-cache hubitat

# Enable and start service
rc-update add hubitat default
rc-service hubitat start

# Verify installation
hubitat --version
```

### openSUSE/SLES

```bash
# Install hubitat
sudo zypper install -y hubitat

# Enable and start service
sudo systemctl enable --now hubitat

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
hubitat --version
```

### macOS

```bash
# Using Homebrew
brew install hubitat

# Start service
brew services start hubitat

# Verify installation
hubitat --version
```

### FreeBSD

```bash
# Using pkg
pkg install hubitat

# Enable in rc.conf
echo 'hubitat_enable="YES"' >> /etc/rc.conf

# Start service
service hubitat start

# Verify installation
hubitat --version
```

### Windows

```bash
# Using Chocolatey
choco install hubitat

# Or using Scoop
scoop install hubitat

# Verify installation
hubitat --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/hubitat

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
hubitat --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable hubitat

# Start service
sudo systemctl start hubitat

# Stop service
sudo systemctl stop hubitat

# Restart service
sudo systemctl restart hubitat

# Check status
sudo systemctl status hubitat

# View logs
sudo journalctl -u hubitat -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add hubitat default

# Start service
rc-service hubitat start

# Stop service
rc-service hubitat stop

# Restart service
rc-service hubitat restart

# Check status
rc-service hubitat status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'hubitat_enable="YES"' >> /etc/rc.conf

# Start service
service hubitat start

# Stop service
service hubitat stop

# Restart service
service hubitat restart

# Check status
service hubitat status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start hubitat
brew services stop hubitat
brew services restart hubitat

# Check status
brew services list | grep hubitat
```

### Windows Service Manager

```powershell
# Start service
net start hubitat

# Stop service
net stop hubitat

# Using PowerShell
Start-Service hubitat
Stop-Service hubitat
Restart-Service hubitat

# Check status
Get-Service hubitat
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream hubitat_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name hubitat.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name hubitat.example.com;

    ssl_certificate /etc/ssl/certs/hubitat.example.com.crt;
    ssl_certificate_key /etc/ssl/private/hubitat.example.com.key;

    location / {
        proxy_pass http://hubitat_backend;
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
    ServerName hubitat.example.com
    Redirect permanent / https://hubitat.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName hubitat.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/hubitat.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/hubitat.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend hubitat_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/hubitat.pem
    redirect scheme https if !{ ssl_fc }
    default_backend hubitat_backend

backend hubitat_backend
    balance roundrobin
    server hubitat1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R hubitat:hubitat /etc/hubitat
sudo chmod 750 /etc/hubitat

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status hubitat

# View logs
sudo journalctl -u hubitat -f

# Monitor resource usage
top -p $(pgrep hubitat)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/hubitat"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/hubitat-backup-$DATE.tar.gz" /etc/hubitat /var/lib/hubitat

echo "Backup completed: $BACKUP_DIR/hubitat-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop hubitat

# Restore from backup
tar -xzf /backup/hubitat/hubitat-backup-*.tar.gz -C /

# Start service
sudo systemctl start hubitat
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u hubitat -n 100
sudo tail -f /var/log/hubitat/hubitat.log

# Check configuration
hubitat --version

# Check permissions
ls -la /etc/hubitat
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep hubitat)

# Check disk I/O
iotop -p $(pgrep hubitat)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  hubitat:
    image: hubitat:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/hubitat
      - ./data:/var/lib/hubitat
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update hubitat

# Debian/Ubuntu
sudo apt update && sudo apt upgrade hubitat

# Arch Linux
sudo pacman -Syu hubitat

# Alpine Linux
apk update && apk upgrade hubitat

# openSUSE
sudo zypper update hubitat

# FreeBSD
pkg update && pkg upgrade hubitat

# Always backup before updates
tar -czf /backup/hubitat-pre-update-$(date +%Y%m%d).tar.gz /etc/hubitat

# Restart after updates
sudo systemctl restart hubitat
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/hubitat

# Clean old logs
find /var/log/hubitat -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/hubitat
```

## Additional Resources

- Official Documentation: https://docs.hubitat.org/
- GitHub Repository: https://github.com/hubitat/hubitat
- Community Forum: https://forum.hubitat.org/
- Best Practices Guide: https://docs.hubitat.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
