# etcd Installation Guide

etcd is a free and open-source distributed key-value store. etcd provides a reliable way to store data across a cluster of machines, serving as the backbone for Kubernetes and other distributed systems

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
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 8GB for data
  - Network: Low-latency cluster network
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 2379 (default etcd port)
  - Port 2380 for peer communication
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

# Install etcd
sudo dnf install -y etcd

# Enable and start service
sudo systemctl enable --now etcd

# Configure firewall
sudo firewall-cmd --permanent --add-port=2379/tcp
sudo firewall-cmd --reload

# Verify installation
etcd --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install etcd
sudo apt install -y etcd

# Enable and start service
sudo systemctl enable --now etcd

# Configure firewall
sudo ufw allow 2379

# Verify installation
etcd --version
```

### Arch Linux

```bash
# Install etcd
sudo pacman -S etcd

# Enable and start service
sudo systemctl enable --now etcd

# Verify installation
etcd --version
```

### Alpine Linux

```bash
# Install etcd
apk add --no-cache etcd

# Enable and start service
rc-update add etcd default
rc-service etcd start

# Verify installation
etcd --version
```

### openSUSE/SLES

```bash
# Install etcd
sudo zypper install -y etcd

# Enable and start service
sudo systemctl enable --now etcd

# Configure firewall
sudo firewall-cmd --permanent --add-port=2379/tcp
sudo firewall-cmd --reload

# Verify installation
etcd --version
```

### macOS

```bash
# Using Homebrew
brew install etcd

# Start service
brew services start etcd

# Verify installation
etcd --version
```

### FreeBSD

```bash
# Using pkg
pkg install etcd

# Enable in rc.conf
echo 'etcd_enable="YES"' >> /etc/rc.conf

# Start service
service etcd start

# Verify installation
etcd --version
```

### Windows

```bash
# Using Chocolatey
choco install etcd

# Or using Scoop
scoop install etcd

# Verify installation
etcd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/etcd

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
etcd --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable etcd

# Start service
sudo systemctl start etcd

# Stop service
sudo systemctl stop etcd

# Restart service
sudo systemctl restart etcd

# Check status
sudo systemctl status etcd

# View logs
sudo journalctl -u etcd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add etcd default

# Start service
rc-service etcd start

# Stop service
rc-service etcd stop

# Restart service
rc-service etcd restart

# Check status
rc-service etcd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'etcd_enable="YES"' >> /etc/rc.conf

# Start service
service etcd start

# Stop service
service etcd stop

# Restart service
service etcd restart

# Check status
service etcd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start etcd
brew services stop etcd
brew services restart etcd

# Check status
brew services list | grep etcd
```

### Windows Service Manager

```powershell
# Start service
net start etcd

# Stop service
net stop etcd

# Using PowerShell
Start-Service etcd
Stop-Service etcd
Restart-Service etcd

# Check status
Get-Service etcd
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream etcd_backend {
    server 127.0.0.1:2379;
}

server {
    listen 80;
    server_name etcd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name etcd.example.com;

    ssl_certificate /etc/ssl/certs/etcd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/etcd.example.com.key;

    location / {
        proxy_pass http://etcd_backend;
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
    ServerName etcd.example.com
    Redirect permanent / https://etcd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName etcd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/etcd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/etcd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:2379/
    ProxyPassReverse / http://127.0.0.1:2379/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend etcd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/etcd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend etcd_backend

backend etcd_backend
    balance roundrobin
    server etcd1 127.0.0.1:2379 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R etcd:etcd /etc/etcd
sudo chmod 750 /etc/etcd

# Configure firewall
sudo firewall-cmd --permanent --add-port=2379/tcp
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
sudo systemctl status etcd

# View logs
sudo journalctl -u etcd -f

# Monitor resource usage
top -p $(pgrep etcd)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/etcd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/etcd-backup-$DATE.tar.gz" /etc/etcd /var/lib/etcd

echo "Backup completed: $BACKUP_DIR/etcd-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop etcd

# Restore from backup
tar -xzf /backup/etcd/etcd-backup-*.tar.gz -C /

# Start service
sudo systemctl start etcd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u etcd -n 100
sudo tail -f /var/log/etcd/etcd.log

# Check configuration
etcd --version

# Check permissions
ls -la /etc/etcd
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 2379

# Test connectivity
telnet localhost 2379

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep etcd)

# Check disk I/O
iotop -p $(pgrep etcd)

# Check connections
ss -an | grep 2379
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  etcd:
    image: etcd:latest
    ports:
      - "2379:2379"
    volumes:
      - ./config:/etc/etcd
      - ./data:/var/lib/etcd
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update etcd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade etcd

# Arch Linux
sudo pacman -Syu etcd

# Alpine Linux
apk update && apk upgrade etcd

# openSUSE
sudo zypper update etcd

# FreeBSD
pkg update && pkg upgrade etcd

# Always backup before updates
tar -czf /backup/etcd-pre-update-$(date +%Y%m%d).tar.gz /etc/etcd

# Restart after updates
sudo systemctl restart etcd
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/etcd

# Clean old logs
find /var/log/etcd -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/etcd
```

## Additional Resources

- Official Documentation: https://docs.etcd.org/
- GitHub Repository: https://github.com/etcd/etcd
- Community Forum: https://forum.etcd.org/
- Best Practices Guide: https://docs.etcd.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
