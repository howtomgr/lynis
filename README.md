# lynis Installation Guide

lynis is a free and open-source security auditing tool. Lynis performs security auditing and hardening for Unix-like systems, providing compliance testing and system hardening suggestions

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
  - RAM: 128MB minimum
  - Storage: 50MB for installation
  - Network: None required
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port N/A (default lynis port)
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

# Install lynis
sudo dnf install -y lynis

# Enable and start service
sudo systemctl enable --now lynis

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
lynis --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install lynis
sudo apt install -y lynis

# Enable and start service
sudo systemctl enable --now lynis

# Configure firewall
sudo ufw allow N/A

# Verify installation
lynis --version
```

### Arch Linux

```bash
# Install lynis
sudo pacman -S lynis

# Enable and start service
sudo systemctl enable --now lynis

# Verify installation
lynis --version
```

### Alpine Linux

```bash
# Install lynis
apk add --no-cache lynis

# Enable and start service
rc-update add lynis default
rc-service lynis start

# Verify installation
lynis --version
```

### openSUSE/SLES

```bash
# Install lynis
sudo zypper install -y lynis

# Enable and start service
sudo systemctl enable --now lynis

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
sudo firewall-cmd --reload

# Verify installation
lynis --version
```

### macOS

```bash
# Using Homebrew
brew install lynis

# Start service
brew services start lynis

# Verify installation
lynis --version
```

### FreeBSD

```bash
# Using pkg
pkg install lynis

# Enable in rc.conf
echo 'lynis_enable="YES"' >> /etc/rc.conf

# Start service
service lynis start

# Verify installation
lynis --version
```

### Windows

```bash
# Using Chocolatey
choco install lynis

# Or using Scoop
scoop install lynis

# Verify installation
lynis --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/lynis

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
lynis --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable lynis

# Start service
sudo systemctl start lynis

# Stop service
sudo systemctl stop lynis

# Restart service
sudo systemctl restart lynis

# Check status
sudo systemctl status lynis

# View logs
sudo journalctl -u lynis -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add lynis default

# Start service
rc-service lynis start

# Stop service
rc-service lynis stop

# Restart service
rc-service lynis restart

# Check status
rc-service lynis status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'lynis_enable="YES"' >> /etc/rc.conf

# Start service
service lynis start

# Stop service
service lynis stop

# Restart service
service lynis restart

# Check status
service lynis status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start lynis
brew services stop lynis
brew services restart lynis

# Check status
brew services list | grep lynis
```

### Windows Service Manager

```powershell
# Start service
net start lynis

# Stop service
net stop lynis

# Using PowerShell
Start-Service lynis
Stop-Service lynis
Restart-Service lynis

# Check status
Get-Service lynis
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream lynis_backend {
    server 127.0.0.1:N/A;
}

server {
    listen 80;
    server_name lynis.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name lynis.example.com;

    ssl_certificate /etc/ssl/certs/lynis.example.com.crt;
    ssl_certificate_key /etc/ssl/private/lynis.example.com.key;

    location / {
        proxy_pass http://lynis_backend;
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
    ServerName lynis.example.com
    Redirect permanent / https://lynis.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName lynis.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/lynis.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/lynis.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:N/A/
    ProxyPassReverse / http://127.0.0.1:N/A/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend lynis_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/lynis.pem
    redirect scheme https if !{ ssl_fc }
    default_backend lynis_backend

backend lynis_backend
    balance roundrobin
    server lynis1 127.0.0.1:N/A check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R lynis:lynis /etc/lynis
sudo chmod 750 /etc/lynis

# Configure firewall
sudo firewall-cmd --permanent --add-port=N/A/tcp
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
sudo systemctl status lynis

# View logs
sudo journalctl -u lynis -f

# Monitor resource usage
top -p $(pgrep lynis)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/lynis"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/lynis-backup-$DATE.tar.gz" /etc/lynis /var/lib/lynis

echo "Backup completed: $BACKUP_DIR/lynis-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop lynis

# Restore from backup
tar -xzf /backup/lynis/lynis-backup-*.tar.gz -C /

# Start service
sudo systemctl start lynis
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u lynis -n 100
sudo tail -f /var/log/lynis/lynis.log

# Check configuration
lynis --version

# Check permissions
ls -la /etc/lynis
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep N/A

# Test connectivity
telnet localhost N/A

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep lynis)

# Check disk I/O
iotop -p $(pgrep lynis)

# Check connections
ss -an | grep N/A
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  lynis:
    image: lynis:latest
    ports:
      - "N/A:N/A"
    volumes:
      - ./config:/etc/lynis
      - ./data:/var/lib/lynis
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update lynis

# Debian/Ubuntu
sudo apt update && sudo apt upgrade lynis

# Arch Linux
sudo pacman -Syu lynis

# Alpine Linux
apk update && apk upgrade lynis

# openSUSE
sudo zypper update lynis

# FreeBSD
pkg update && pkg upgrade lynis

# Always backup before updates
tar -czf /backup/lynis-pre-update-$(date +%Y%m%d).tar.gz /etc/lynis

# Restart after updates
sudo systemctl restart lynis
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/lynis

# Clean old logs
find /var/log/lynis -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/lynis
```

## Additional Resources

- Official Documentation: https://docs.lynis.org/
- GitHub Repository: https://github.com/lynis/lynis
- Community Forum: https://forum.lynis.org/
- Best Practices Guide: https://docs.lynis.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
