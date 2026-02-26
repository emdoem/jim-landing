# VPS Deployment Guide

## Quick Setup

### 1. Upload Files to VPS

Create directory structure:
```bash
ssh user@your-vps-ip
mkdir -p /var/www/html
exit
```

Upload files:
```bash
# Upload HTML
scp index.html user@your-vps-ip:/var/www/html/

# Create media directories and upload content
ssh user@your-vps-ip "mkdir -p /var/www/html/photo /var/www/html/video"
scp -r photo/* user@your-vps-ip:/var/www/html/photo/
scp -r vid/* user@your-vps-ip:/var/www/html/video/
```

### 2. Install Nginx

```bash
ssh user@your-vps-ip

# Update and install
sudo apt update
sudo apt install nginx -y

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 3. Configure Nginx

Copy `nginx.conf` from this repo to VPS:
```bash
scp nginx.conf user@your-vps-ip:/tmp/nginx.conf
```

On VPS:
```bash
sudo cp /tmp/nginx.conf /etc/nginx/sites-available/swing-dancers.conf
sudo ln -s /etc/nginx/sites-available/swing-dancers.conf /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### 4. Set Permissions

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

### 5. Enable HTTPS (Optional but Recommended)

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal is automatically configured
```

### 6. Test

Visit `http://your-ip` or `https://your-domain.com` to verify the site works.

## Updating Content

To add new photos/videos:
```bash
scp index.html user@your-vps-ip:/var/www/html/
scp photo/new-image.jpg user@your-vps-ip:/var/www/html/photo/
scp vid/new-video.mp4 user@your-vps-ip:/var/www/html/video/
```

## Performance Tips

1. **Image Optimization**: Convert images to WebP for faster loading
2. **Caching**: Already configured in nginx.conf (30 days videos, 1 year images)
3. **Cloudflare (Optional)**: Add in front for free CDN and DDoS protection

## Troubleshooting

Check Nginx status:
```bash
sudo systemctl status nginx
```

View logs:
```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

Restart after changes:
```bash
sudo systemctl reload nginx
```

## Alternative: Use rsync for easier updates

Create a sync script:
```bash
#!/bin/bash
# sync.sh

VPS_USER="your-user"
VPS_HOST="your-ip"

rsync -avz --progress index.html ${VPS_USER}@${VPS_HOST}:/var/www/html/
rsync -avz --progress photo/ ${VPS_USER}@${VPS_HOST}:/var/www/html/photo/
rsync -avz --progress vid/ ${VPS_USER}@${VPS_HOST}:/var/www/html/video/
```

Make executable and run:
```bash
chmod +x sync.sh
./sync.sh
```