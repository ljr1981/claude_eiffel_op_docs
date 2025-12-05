# Oracle Cloud Infrastructure Deployment Guide

Step-by-step guide for deploying Eiffel web applications to Oracle Cloud Infrastructure (OCI).

## Overview

This guide covers deploying an Eiffel web application (like simple_showcase) to Oracle Cloud's Always Free tier compute instance.

**Target Setup:**
- Oracle Linux 8 on ARM64 (Ampere A1)
- Nginx reverse proxy with SSL
- Systemd service management
- Let's Encrypt SSL certificates

---

## Phase 1: Oracle Cloud Account Setup

### 1.1 Create Oracle Cloud Account

1. Go to https://cloud.oracle.com
2. Click "Sign Up" for free tier
3. Provide email, name, and payment method (not charged for free tier)
4. Select home region (choose closest to your users)
5. Complete verification

### 1.2 Access the Console

1. Log in to https://cloud.oracle.com
2. Navigate to the OCI Console
3. Familiarize yourself with the dashboard

---

## Phase 2: Network Configuration

### 2.1 Create Virtual Cloud Network (VCN)

1. **Networking** → **Virtual Cloud Networks** → **Start VCN Wizard**
2. Select "Create VCN with Internet Connectivity"
3. Configure:
   - VCN Name: `eiffel-vcn`
   - Compartment: (root)
   - VCN CIDR: `10.0.0.0/16`
   - Public Subnet CIDR: `10.0.0.0/24`
   - Private Subnet CIDR: `10.0.1.0/24`
4. Click **Create**

### 2.2 Configure Security List (Firewall Rules)

1. Go to **VCN** → **Security Lists** → **Default Security List**
2. Add **Ingress Rules**:

| Source CIDR | Protocol | Dest Port | Description |
|-------------|----------|-----------|-------------|
| 0.0.0.0/0 | TCP | 22 | SSH |
| 0.0.0.0/0 | TCP | 80 | HTTP |
| 0.0.0.0/0 | TCP | 443 | HTTPS |
| 0.0.0.0/0 | TCP | 8080 | Eiffel App (optional) |

3. Click **Add Ingress Rules**

---

## Phase 3: Compute Instance Creation

### 3.1 Create Compute Instance

1. **Compute** → **Instances** → **Create Instance**
2. Configure:
   - Name: `eiffel-server`
   - Compartment: (root)
   - Availability Domain: AD-1
   - **Image**: Oracle Linux 8 (ARM compatible)
   - **Shape**: VM.Standard.A1.Flex (Always Free)
     - OCPUs: 1-4 (free tier allows up to 4)
     - Memory: 6-24 GB (free tier allows up to 24 GB)
   - **Networking**: Select your VCN and public subnet
   - **Public IP**: Assign automatically

### 3.2 SSH Key Setup

1. Generate SSH key pair (if you don't have one):
   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/oracle_eiffel
   ```
2. Upload the **public key** (`oracle_eiffel.pub`) during instance creation
3. Save the **private key** securely

### 3.3 Note Instance Details

After creation, record:
- **Public IP**: e.g., `141.148.86.105`
- **Private IP**: e.g., `10.0.0.123`
- **Username**: `opc` (default for Oracle Linux)

---

## Phase 4: Server Configuration

### 4.1 Connect via SSH

```bash
ssh -i ~/.ssh/oracle_eiffel opc@141.148.86.105
```

### 4.2 Update System

```bash
sudo dnf update -y
sudo dnf upgrade -y
```

### 4.3 Install Required Packages

```bash
# Install Nginx
sudo dnf install -y nginx

# Install certbot for SSL
sudo dnf install -y certbot python3-certbot-nginx

# Install useful tools
sudo dnf install -y git curl wget htop
```

### 4.4 Configure Firewall (firewalld)

```bash
# Enable and start firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld

# Open HTTP and HTTPS ports
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify
sudo firewall-cmd --list-all
```

### 4.5 Create Application Directory

```bash
sudo mkdir -p /opt/eiffel-app
sudo chown opc:opc /opt/eiffel-app
```

---

## Phase 5: Application Deployment

### 5.1 Build Application Locally

On your development machine:

```bash
# Compile with finalization for production
ec -batch -config simple_showcase.ecf -target run_server -finalize -c_compile

# The executable is in:
# EIFGENs/run_server/F_code/simple_showcase.exe (Windows)
# EIFGENs/run_server/F_code/simple_showcase (Linux)
```

### 5.2 Cross-Compile for ARM64 (if needed)

If developing on x86_64 and deploying to ARM64:

1. Set up a Linux ARM64 build environment (Docker or actual ARM machine)
2. Install EiffelStudio for ARM64
3. Compile there

Or compile directly on the Oracle instance (slower but works).

### 5.3 Transfer Application

```bash
# From local machine
scp -i ~/.ssh/oracle_eiffel ./simple_showcase opc@141.148.86.105:/opt/eiffel-app/

# Transfer static files
scp -i ~/.ssh/oracle_eiffel -r ./www opc@141.148.86.105:/opt/eiffel-app/

# Transfer config
scp -i ~/.ssh/oracle_eiffel ./config.production.json opc@141.148.86.105:/opt/eiffel-app/config.json
```

### 5.4 Set Permissions

```bash
ssh -i ~/.ssh/oracle_eiffel opc@141.148.86.105

cd /opt/eiffel-app
chmod +x simple_showcase
```

---

## Phase 6: Systemd Service Setup

### 6.1 Create Service File

```bash
sudo nano /etc/systemd/system/eiffel-app.service
```

Contents:
```ini
[Unit]
Description=Eiffel Web Application
After=network.target

[Service]
Type=simple
User=opc
Group=opc
WorkingDirectory=/opt/eiffel-app
ExecStart=/opt/eiffel-app/simple_showcase
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

# Environment variables (if needed)
Environment=EIFFEL_ENV=production

[Install]
WantedBy=multi-user.target
```

### 6.2 Enable and Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable eiffel-app
sudo systemctl start eiffel-app

# Check status
sudo systemctl status eiffel-app

# View logs
sudo journalctl -u eiffel-app -f
```

---

## Phase 7: Nginx Reverse Proxy

### 7.1 Configure Nginx

```bash
sudo nano /etc/nginx/conf.d/eiffel-app.conf
```

Contents:
```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    # Redirect HTTP to HTTPS (after SSL setup)
    # return 301 https://$server_name$request_uri;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 86400;
    }

    # Static files (optional - can be served by Eiffel app)
    location /static/ {
        alias /opt/eiffel-app/www/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

### 7.2 Test and Reload Nginx

```bash
sudo nginx -t
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl reload nginx
```

---

## Phase 8: SSL Certificate Setup

### 8.1 Point Domain to Server

1. In your DNS provider, create an A record:
   - Host: `@` (or `www`)
   - Points to: Your Oracle instance public IP
   - TTL: 300 (5 minutes for testing)

2. Wait for DNS propagation (check with `nslookup your-domain.com`)

### 8.2 Obtain Let's Encrypt Certificate

```bash
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

Follow prompts:
- Enter email for renewal notices
- Agree to terms
- Choose to redirect HTTP to HTTPS (recommended)

### 8.3 Verify Auto-Renewal

```bash
sudo certbot renew --dry-run
```

### 8.4 Updated Nginx Config (with SSL)

Certbot modifies the config, but verify it looks like:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## Phase 9: Monitoring and Maintenance

### 9.1 View Application Logs

```bash
# Live logs
sudo journalctl -u eiffel-app -f

# Last 100 lines
sudo journalctl -u eiffel-app -n 100

# Logs since boot
sudo journalctl -u eiffel-app -b
```

### 9.2 Restart Application

```bash
sudo systemctl restart eiffel-app
```

### 9.3 Update Application

```bash
# Stop service
sudo systemctl stop eiffel-app

# Upload new binary
scp -i ~/.ssh/oracle_eiffel ./simple_showcase opc@141.148.86.105:/opt/eiffel-app/

# Set permissions
ssh -i ~/.ssh/oracle_eiffel opc@141.148.86.105 "chmod +x /opt/eiffel-app/simple_showcase"

# Start service
sudo systemctl start eiffel-app
```

### 9.4 Monitor System Resources

```bash
# CPU and memory
htop

# Disk usage
df -h

# Network connections
ss -tulpn
```

### 9.5 Setup Log Rotation

```bash
sudo nano /etc/logrotate.d/eiffel-app
```

Contents:
```
/var/log/eiffel-app/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 opc opc
    sharedscripts
    postrotate
        systemctl reload eiffel-app > /dev/null 2>&1 || true
    endscript
}
```

---

## Phase 10: Backup and Recovery

### 10.1 Backup Application

```bash
# Create backup script
cat > /opt/eiffel-app/backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/opc/backups"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/eiffel-app-$DATE.tar.gz /opt/eiffel-app
find $BACKUP_DIR -type f -mtime +7 -delete
EOF

chmod +x /opt/eiffel-app/backup.sh
```

### 10.2 Schedule Backups

```bash
crontab -e
```

Add:
```
0 2 * * * /opt/eiffel-app/backup.sh
```

---

## Troubleshooting

### Application Won't Start

```bash
# Check service status
sudo systemctl status eiffel-app

# Check logs
sudo journalctl -u eiffel-app -n 50

# Try running manually
cd /opt/eiffel-app && ./simple_showcase
```

### Cannot Connect to Server

1. Check security list rules in OCI Console
2. Check firewalld: `sudo firewall-cmd --list-all`
3. Check nginx: `sudo nginx -t && sudo systemctl status nginx`
4. Check app is listening: `ss -tulpn | grep 8080`

### SSL Certificate Issues

```bash
# Check certificate status
sudo certbot certificates

# Force renewal
sudo certbot renew --force-renewal

# Check nginx config
sudo nginx -t
```

### High Memory Usage

```bash
# Check memory
free -h

# Check process memory
ps aux --sort=-%mem | head -10

# Restart application
sudo systemctl restart eiffel-app
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start app | `sudo systemctl start eiffel-app` |
| Stop app | `sudo systemctl stop eiffel-app` |
| Restart app | `sudo systemctl restart eiffel-app` |
| View logs | `sudo journalctl -u eiffel-app -f` |
| Check status | `sudo systemctl status eiffel-app` |
| Reload nginx | `sudo systemctl reload nginx` |
| Renew SSL | `sudo certbot renew` |

---

## Oracle Cloud Free Tier Limits

| Resource | Free Tier Allowance |
|----------|---------------------|
| ARM Compute | 4 OCPUs, 24 GB RAM total |
| Block Storage | 200 GB |
| Object Storage | 20 GB |
| Outbound Data | 10 TB/month |

---

*Last updated: December 2025*
