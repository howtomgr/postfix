# postfix Installation Guide

postfix is a free and open-source mail transfer agent. Postfix provides fast, secure mail transfer agent as alternative to Sendmail

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
  - Storage: 1GB for mail
  - Network: SMTP protocols
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 25 (default postfix port)
  - Submission 587, SMTPS 465
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

# Install postfix
sudo dnf install -y postfix

# Enable and start service
sudo systemctl enable --now postfix

# Configure firewall
sudo firewall-cmd --permanent --add-port=25/tcp
sudo firewall-cmd --reload

# Verify installation
postconf -d mail_version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install postfix
sudo apt install -y postfix

# Enable and start service
sudo systemctl enable --now postfix

# Configure firewall
sudo ufw allow 25

# Verify installation
postconf -d mail_version
```

### Arch Linux

```bash
# Install postfix
sudo pacman -S postfix

# Enable and start service
sudo systemctl enable --now postfix

# Verify installation
postconf -d mail_version
```

### Alpine Linux

```bash
# Install postfix
apk add --no-cache postfix

# Enable and start service
rc-update add postfix default
rc-service postfix start

# Verify installation
postconf -d mail_version
```

### openSUSE/SLES

```bash
# Install postfix
sudo zypper install -y postfix

# Enable and start service
sudo systemctl enable --now postfix

# Configure firewall
sudo firewall-cmd --permanent --add-port=25/tcp
sudo firewall-cmd --reload

# Verify installation
postconf -d mail_version
```

### macOS

```bash
# Using Homebrew
brew install postfix

# Start service
brew services start postfix

# Verify installation
postconf -d mail_version
```

### FreeBSD

```bash
# Using pkg
pkg install postfix

# Enable in rc.conf
echo 'postfix_enable="YES"' >> /etc/rc.conf

# Start service
service postfix start

# Verify installation
postconf -d mail_version
```

### Windows

```bash
# Using Chocolatey
choco install postfix

# Or using Scoop
scoop install postfix

# Verify installation
postconf -d mail_version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/postfix

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
postconf -d mail_version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable postfix

# Start service
sudo systemctl start postfix

# Stop service
sudo systemctl stop postfix

# Restart service
sudo systemctl restart postfix

# Check status
sudo systemctl status postfix

# View logs
sudo journalctl -u postfix -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add postfix default

# Start service
rc-service postfix start

# Stop service
rc-service postfix stop

# Restart service
rc-service postfix restart

# Check status
rc-service postfix status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'postfix_enable="YES"' >> /etc/rc.conf

# Start service
service postfix start

# Stop service
service postfix stop

# Restart service
service postfix restart

# Check status
service postfix status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start postfix
brew services stop postfix
brew services restart postfix

# Check status
brew services list | grep postfix
```

### Windows Service Manager

```powershell
# Start service
net start postfix

# Stop service
net stop postfix

# Using PowerShell
Start-Service postfix
Stop-Service postfix
Restart-Service postfix

# Check status
Get-Service postfix
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream postfix_backend {
    server 127.0.0.1:25;
}

server {
    listen 80;
    server_name postfix.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name postfix.example.com;

    ssl_certificate /etc/ssl/certs/postfix.example.com.crt;
    ssl_certificate_key /etc/ssl/private/postfix.example.com.key;

    location / {
        proxy_pass http://postfix_backend;
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
    ServerName postfix.example.com
    Redirect permanent / https://postfix.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName postfix.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/postfix.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/postfix.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:25/
    ProxyPassReverse / http://127.0.0.1:25/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend postfix_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/postfix.pem
    redirect scheme https if !{ ssl_fc }
    default_backend postfix_backend

backend postfix_backend
    balance roundrobin
    server postfix1 127.0.0.1:25 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R postfix:postfix /etc/postfix
sudo chmod 750 /etc/postfix

# Configure firewall
sudo firewall-cmd --permanent --add-port=25/tcp
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
sudo systemctl status postfix

# View logs
sudo journalctl -u postfix -f

# Monitor resource usage
top -p $(pgrep postfix)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/postfix"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/postfix-backup-$DATE.tar.gz" /etc/postfix /var/lib/postfix

echo "Backup completed: $BACKUP_DIR/postfix-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop postfix

# Restore from backup
tar -xzf /backup/postfix/postfix-backup-*.tar.gz -C /

# Start service
sudo systemctl start postfix
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u postfix -n 100
sudo tail -f /var/log/postfix/postfix.log

# Check configuration
postconf -d mail_version

# Check permissions
ls -la /etc/postfix
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 25

# Test connectivity
telnet localhost 25

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep postfix)

# Check disk I/O
iotop -p $(pgrep postfix)

# Check connections
ss -an | grep 25
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  postfix:
    image: postfix:latest
    ports:
      - "25:25"
    volumes:
      - ./config:/etc/postfix
      - ./data:/var/lib/postfix
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update postfix

# Debian/Ubuntu
sudo apt update && sudo apt upgrade postfix

# Arch Linux
sudo pacman -Syu postfix

# Alpine Linux
apk update && apk upgrade postfix

# openSUSE
sudo zypper update postfix

# FreeBSD
pkg update && pkg upgrade postfix

# Always backup before updates
tar -czf /backup/postfix-pre-update-$(date +%Y%m%d).tar.gz /etc/postfix

# Restart after updates
sudo systemctl restart postfix
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/postfix

# Clean old logs
find /var/log/postfix -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/postfix
```

## Additional Resources

- Official Documentation: https://docs.postfix.org/
- GitHub Repository: https://github.com/postfix/postfix
- Community Forum: https://forum.postfix.org/
- Best Practices Guide: https://docs.postfix.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
