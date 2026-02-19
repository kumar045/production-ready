* ✅ Production **Nginx reverse proxy config**
* ✅ SSL with **Let’s Encrypt**
* ✅ Separate domain/subdomain for frontend & backend
* ✅ Full DNS setup steps
* ✅ Docker-compatible setup


# 🌍 Architecture

Example domains:

* Frontend → `example.com`
* Backend API → `api.example.com`

Flow:

```
User → Domain → Nginx (SSL) → Docker services
```

---

# 🐳 Updated docker-compose.yml (Production Style)

⚠ Remove port exposure. Nginx will handle traffic.

```yaml
version: "3.9"

services:
  backend:
    build: ./backend
    container_name: fastapi_backend
    restart: always
    networks:
      - app_network

  frontend:
    build: ./frontend
    container_name: nextjs_frontend
    restart: always
    networks:
      - app_network

  nginx:
    image: nginx:latest
    container_name: nginx_reverse_proxy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/certbot/conf:/etc/letsencrypt
      - ./nginx/certbot/www:/var/www/certbot
    depends_on:
      - frontend
      - backend
    networks:
      - app_network

networks:
  app_network:
    driver: bridge
```

---

# 📂 Create Nginx Config Folder

```bash
mkdir -p nginx/conf.d
```

Create:

```
nginx/conf.d/default.conf
```

---

# 🔐 Nginx Reverse Proxy Config (SSL Ready)

```nginx
# HTTP → Redirect to HTTPS
server {
    listen 80;
    server_name example.com api.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# -------------------------
# Frontend (Next.js)
# -------------------------
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://frontend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# -------------------------
# Backend (FastAPI)
# -------------------------
server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location / {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Replace:

```
example.com
api.example.com
```

with your real domain.

---

# 🔐 Step 1: Point Domain to VPS (DNS Setup)

Go to your domain provider:

Examples:

* GoDaddy
* Namecheap
* Cloudflare

Add these DNS records:

| Type | Name | Value (Your VPS IP) |
| ---- | ---- | ------------------- |
| A    | @    | YOUR_SERVER_IP      |
| A    | api  | YOUR_SERVER_IP      |

Example:

```
example.com      → 123.45.67.89
api.example.com  → 123.45.67.89
```

⏳ Wait 5–15 minutes for DNS propagation.

You can check:

```bash
ping example.com
```

---

# 🔐 Step 2: Install Certbot in Docker

Add certbot service temporarily.

Run:

```bash
docker compose up -d nginx
```

Now generate SSL:

```bash
docker run --rm \
  -v $(pwd)/nginx/certbot/conf:/etc/letsencrypt \
  -v $(pwd)/nginx/certbot/www:/var/www/certbot \
  certbot/certbot certonly \
  --webroot \
  --webroot-path=/var/www/certbot \
  -d example.com \
  -d api.example.com \
  --email your@email.com \
  --agree-tos \
  --no-eff-email
```

If successful:

Certificates will be saved in:

```
nginx/certbot/conf/live/
```

---

# 🔁 Restart Everything

```bash
docker compose down
docker compose up -d --build
```

Now visit:

```
https://example.com
https://api.example.com
```

---

# 🔄 Auto Renew SSL

Add cron job on server:

```bash
crontab -e
```

Add:

```bash
0 3 * * * docker run --rm -v $(pwd)/nginx/certbot/conf:/etc/letsencrypt -v $(pwd)/nginx/certbot/www:/var/www/certbot certbot/certbot renew
```

---

# 🛡 Production Security Improvements (Recommended)

Add inside SSL server block:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;

add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
```

---

# 🧠 Important: Update Frontend API URL

Inside Next.js `.env`:

```
NEXT_PUBLIC_API_URL=https://api.example.com
```

---

# 🚀 Final Architecture

```
Internet
   ↓
Domain DNS
   ↓
VPS (Port 80/443 open)
   ↓
Nginx (SSL)
   ↓
Docker network
   ├── Next.js
   └── FastAPI
```
