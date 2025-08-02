# Nginx Setup on VPS
## 1. 📦 Install Nginx
```
sudo apt update
sudo apt install nginx -y
```
## 2. 🔧 Configure Nginx
Create a new config file for your site:
```bash
sudo nano /etc/nginx/sites-available/fba-frontend
```
# Basic HTTP Configuration (Port 80)
```nginx
server {
    listen 80;
    server_name smartchatbot.click;  # 🔁 Replace with your domain or VPS IP

    location / {
        proxy_pass http://localhost:3000;  # Container is mapped to port 3000
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
## Then enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/fba-frontend /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

## HTTPS with Let's Encrypt (Certbot)
### Install Certbot:
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```
### Issue Certificate:

#### Use Certbot in dry-run mode first
Test the process without requesting a real certificate:
```bash
sudo certbot certonly --nginx --dry-run -d smartchatbot.click
```

Unless you know you want to configure redirection manually, always use:
```bash
sudo certbot --nginx -d smartchatbot.click --redirect
```

```bash
sudo certbot --nginx -d smartchatbot.click
```
This will:

- Automatically edit your config for HTTPS

- Set up auto-renewal

## 🧾 Notes
Nginx listens on port 80 (HTTP) or 443 (HTTPS), and proxies to your container on localhost:3000

Your container in the docker run step must publish port 3000, which you already do:

```yaml
-p 3000:3000
```