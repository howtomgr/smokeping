# smokeping Installation Guide

smokeping is a free and open-source latency measurement. SmokePing measures latency, packet loss and bandwidth

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
  - Storage: 2GB for RRDs
  - Network: ICMP/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default smokeping port)
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

# Install smokeping
sudo dnf install -y smokeping

# Enable and start service
sudo systemctl enable --now smokeping

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
smokeping --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install smokeping
sudo apt install -y smokeping

# Enable and start service
sudo systemctl enable --now smokeping

# Configure firewall
sudo ufw allow 80

# Verify installation
smokeping --version
```

### Arch Linux

```bash
# Install smokeping
sudo pacman -S smokeping

# Enable and start service
sudo systemctl enable --now smokeping

# Verify installation
smokeping --version
```

### Alpine Linux

```bash
# Install smokeping
apk add --no-cache smokeping

# Enable and start service
rc-update add smokeping default
rc-service smokeping start

# Verify installation
smokeping --version
```

### openSUSE/SLES

```bash
# Install smokeping
sudo zypper install -y smokeping

# Enable and start service
sudo systemctl enable --now smokeping

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
smokeping --version
```

### macOS

```bash
# Using Homebrew
brew install smokeping

# Start service
brew services start smokeping

# Verify installation
smokeping --version
```

### FreeBSD

```bash
# Using pkg
pkg install smokeping

# Enable in rc.conf
echo 'smokeping_enable="YES"' >> /etc/rc.conf

# Start service
service smokeping start

# Verify installation
smokeping --version
```

### Windows

```bash
# Using Chocolatey
choco install smokeping

# Or using Scoop
scoop install smokeping

# Verify installation
smokeping --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/smokeping

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
smokeping --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable smokeping

# Start service
sudo systemctl start smokeping

# Stop service
sudo systemctl stop smokeping

# Restart service
sudo systemctl restart smokeping

# Check status
sudo systemctl status smokeping

# View logs
sudo journalctl -u smokeping -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add smokeping default

# Start service
rc-service smokeping start

# Stop service
rc-service smokeping stop

# Restart service
rc-service smokeping restart

# Check status
rc-service smokeping status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'smokeping_enable="YES"' >> /etc/rc.conf

# Start service
service smokeping start

# Stop service
service smokeping stop

# Restart service
service smokeping restart

# Check status
service smokeping status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start smokeping
brew services stop smokeping
brew services restart smokeping

# Check status
brew services list | grep smokeping
```

### Windows Service Manager

```powershell
# Start service
net start smokeping

# Stop service
net stop smokeping

# Using PowerShell
Start-Service smokeping
Stop-Service smokeping
Restart-Service smokeping

# Check status
Get-Service smokeping
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream smokeping_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name smokeping.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name smokeping.example.com;

    ssl_certificate /etc/ssl/certs/smokeping.example.com.crt;
    ssl_certificate_key /etc/ssl/private/smokeping.example.com.key;

    location / {
        proxy_pass http://smokeping_backend;
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
    ServerName smokeping.example.com
    Redirect permanent / https://smokeping.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName smokeping.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/smokeping.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/smokeping.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend smokeping_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/smokeping.pem
    redirect scheme https if !{ ssl_fc }
    default_backend smokeping_backend

backend smokeping_backend
    balance roundrobin
    server smokeping1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R smokeping:smokeping /etc/smokeping
sudo chmod 750 /etc/smokeping

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
sudo systemctl status smokeping

# View logs
sudo journalctl -u smokeping -f

# Monitor resource usage
top -p $(pgrep smokeping)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/smokeping"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/smokeping-backup-$DATE.tar.gz" /etc/smokeping /var/lib/smokeping

echo "Backup completed: $BACKUP_DIR/smokeping-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop smokeping

# Restore from backup
tar -xzf /backup/smokeping/smokeping-backup-*.tar.gz -C /

# Start service
sudo systemctl start smokeping
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u smokeping -n 100
sudo tail -f /var/log/smokeping/smokeping.log

# Check configuration
smokeping --version

# Check permissions
ls -la /etc/smokeping
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
top -p $(pgrep smokeping)

# Check disk I/O
iotop -p $(pgrep smokeping)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  smokeping:
    image: smokeping:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/smokeping
      - ./data:/var/lib/smokeping
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update smokeping

# Debian/Ubuntu
sudo apt update && sudo apt upgrade smokeping

# Arch Linux
sudo pacman -Syu smokeping

# Alpine Linux
apk update && apk upgrade smokeping

# openSUSE
sudo zypper update smokeping

# FreeBSD
pkg update && pkg upgrade smokeping

# Always backup before updates
tar -czf /backup/smokeping-pre-update-$(date +%Y%m%d).tar.gz /etc/smokeping

# Restart after updates
sudo systemctl restart smokeping
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/smokeping

# Clean old logs
find /var/log/smokeping -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/smokeping
```

## Additional Resources

- Official Documentation: https://docs.smokeping.org/
- GitHub Repository: https://github.com/smokeping/smokeping
- Community Forum: https://forum.smokeping.org/
- Best Practices Guide: https://docs.smokeping.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
