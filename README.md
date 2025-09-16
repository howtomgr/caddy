# Caddy Installation Guide

Caddy is a free and open-source fast, multi-platform web server with automatic HTTPS. Written in Go, Caddy is a powerful web server that automatically obtains and renews SSL certificates, serving as a modern alternative to nginx or Apache with built-in HTTPS automation

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
  - CPU: 1 core minimum (2+ for high traffic)
  - RAM: 128MB minimum (512MB+ recommended)
  - Storage: 50MB for installation
  - Network: Internet access for automatic HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.13+ (High Sierra or newer)
  - Windows: Windows 7+
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default Caddy port)
  - Port 2019 for admin API
- **Dependencies**:
  - None (standalone binary)
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

# Install Caddy
sudo dnf install -y caddy

# Enable and start service
sudo systemctl enable --now caddy

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
caddy version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install Caddy
sudo apt install -y caddy

# Enable and start service
sudo systemctl enable --now caddy

# Configure firewall
sudo ufw allow 80/443

# Verify installation
caddy version
```

### Arch Linux

```bash
# Install Caddy
sudo pacman -S caddy

# Enable and start service
sudo systemctl enable --now caddy

# Verify installation
caddy version
```

### Alpine Linux

```bash
# Install Caddy
apk add --no-cache caddy

# Enable and start service
rc-update add caddy default
rc-service caddy start

# Verify installation
caddy version
```

### openSUSE/SLES

```bash
# Install Caddy
sudo zypper install -y caddy

# Enable and start service
sudo systemctl enable --now caddy

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
caddy version
```

### macOS

```bash
# Using Homebrew
brew install caddy

# Start service
brew services start caddy

# Verify installation
caddy version
```

### FreeBSD

```bash
# Using pkg
pkg install caddy

# Enable in rc.conf
echo 'caddy_enable="YES"' >> /etc/rc.conf

# Start service
service caddy start

# Verify installation
caddy version
```

### Windows

```bash
# Using Chocolatey
choco install caddy

# Or using Scoop
scoop install caddy

# Verify installation
caddy version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/caddy

# Set up basic configuration
# Configuration details will vary based on your specific needs
# See official documentation for detailed configuration options

# Test configuration
caddy validate --config /etc/caddy/Caddyfile
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable caddy

# Start service
sudo systemctl start caddy

# Stop service
sudo systemctl stop caddy

# Restart service
sudo systemctl restart caddy

# Check status
sudo systemctl status caddy

# View logs
sudo journalctl -u caddy -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add caddy default

# Start service
rc-service caddy start

# Stop service
rc-service caddy stop

# Restart service
rc-service caddy restart

# Check status
rc-service caddy status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'caddy_enable="YES"' >> /etc/rc.conf

# Start service
service caddy start

# Stop service
service caddy stop

# Restart service
service caddy restart

# Check status
service caddy status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start caddy
brew services stop caddy
brew services restart caddy

# Check status
brew services list | grep caddy
```

### Windows Service Manager

```powershell
# Start service
net start caddy

# Stop service
net stop caddy

# Using PowerShell
Start-Service caddy
Stop-Service caddy
Restart-Service caddy

# Check status
Get-Service caddy
```

## Advanced Configuration

### Advanced Caddy Configuration

See the official documentation for advanced configuration options including:
- High availability setup
- Performance tuning
- Security hardening
- Integration with other services


## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream caddy_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name caddy.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name caddy.example.com;

    ssl_certificate /etc/ssl/certs/caddy.example.com.crt;
    ssl_certificate_key /etc/ssl/private/caddy.example.com.key;

    location / {
        proxy_pass http://caddy_backend;
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
    ServerName caddy.example.com
    Redirect permanent / https://caddy.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName caddy.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/caddy.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/caddy.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend caddy_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/caddy.pem
    redirect scheme https if !{ ssl_fc }
    default_backend caddy_backend

backend caddy_backend
    balance roundrobin
    server caddy1 127.0.0.1:80/443 check
```

## Security Configuration

### Security Best Practices

```bash
# Set appropriate permissions
sudo chown -R caddy:caddy /etc/caddy
sudo chmod 750 /etc/caddy

# Configure firewall rules
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

Not applicable

## Performance Optimization

### 8. Performance Tuning

```bash
# System tuning for Caddy
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Monitor performance
caddy list-modules
```

## Monitoring

### Monitoring Setup

```bash
# Basic monitoring
sudo systemctl status caddy
sudo journalctl -u caddy -f

# Set up health checks
curl -f http://localhost:80/health || exit 1
```

## 9. Backup and Restore

### Backup Procedures

```bash
#!/bin/bash
# Backup script
BACKUP_DIR="/backup/caddy"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf /backup/caddy-$(date +%Y%m%d).tar.gz /etc/caddy /var/lib/caddy

# Restore procedure
# Stop service, restore files, restart service
sudo systemctl stop caddy
# Restore backed up files
sudo systemctl start caddy
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u caddy -f
sudo tail -f /var/log/caddy/caddy.log

# Check configuration
caddy validate --config /etc/caddy/Caddyfile

# Check permissions
ls -la /etc/caddy
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443

# Test connectivity
telnet localhost 80/443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep caddy)

# Check disk I/O
iotop -p $(pgrep caddy)

# Check network connections
ss -an | grep 80/443
```



## Integration Examples

### Example Integration

```yaml
# Docker Compose example
version: '3.8'
services:
  caddy:
    image: caddy:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/caddy
      - ./data:/var/lib/caddy
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update caddy

# Debian/Ubuntu
sudo apt update && sudo apt upgrade caddy

# Arch Linux
sudo pacman -Syu caddy

# Alpine Linux
apk update && apk upgrade caddy

# openSUSE
sudo zypper update caddy

# FreeBSD
pkg update && pkg upgrade caddy

# Always backup before updates
tar -czf /backup/caddy-$(date +%Y%m%d).tar.gz /etc/caddy /var/lib/caddy

# Restart after updates
sudo systemctl restart caddy
```

### Regular Maintenance Tasks

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/caddy

# Clean old logs
find /var/log/caddy -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/caddy

# Verify configuration
caddy validate --config /etc/caddy/Caddyfile

# Test functionality
caddy list-modules
```



## Additional Resources

- [Official Documentation](https://caddyserver.com/docs/)
- [GitHub Repository](https://github.com/caddyserver/caddy)
- [Community Forum](https://caddy.community/)
- [Best Practices Guide](https://caddyserver.com/docs/best-practices)


---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
