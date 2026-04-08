# Deployment Guide

## Option 1: Railway (Recommended for portfolio)

Railway offers a free tier with persistent storage — ideal for showcasing the portfolio.

1. Create account at [railway.app](https://railway.app)
2. New Project → Deploy from GitHub repo
3. Select `Asilvag20/n8n-ai-portfolio`
4. Add environment variables from `docker-compose.yml`
5. Add a custom domain in Railway settings

## Option 2: DigitalOcean Droplet (~$6/mo)

```bash
# On a fresh Ubuntu 22.04 droplet
apt update && apt install -y docker.io docker-compose git
git clone https://github.com/Asilvag20/n8n-ai-portfolio.git
cd n8n-ai-portfolio

# Edit docker-compose.yml:
# - Change N8N_HOST to your domain or IP
# - Change N8N_PROTOCOL to https (with reverse proxy)
# - Set strong passwords

docker-compose up -d
```

## Option 3: Render

1. [render.com](https://render.com) → New Web Service
2. Connect GitHub repo
3. Set build command: (none, uses Docker)
4. Set environment variables

## Custom Domain Setup

### With Nginx reverse proxy (DigitalOcean):

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Get free SSL cert
certbot --nginx -d your-domain.com
```

## Environment Variables for Production

```env
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=your-admin-user
N8N_BASIC_AUTH_PASSWORD=your-strong-password
N8N_HOST=your-domain.com
N8N_PORT=5678
N8N_PROTOCOL=https
WEBHOOK_URL=https://your-domain.com/
GENERIC_TIMEZONE=America/Bogota
```

## GitHub Pages (for documentation only)

If you want a static documentation site:

1. Go to repo Settings → Pages
2. Source: Deploy from branch `main`, folder `/docs`
3. Your docs will be at `https://Asilvag20.github.io/n8n-ai-portfolio`
