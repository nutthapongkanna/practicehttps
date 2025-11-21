
# Docker + Nginx + Certbot Setup (www.test.com)

## 1. Install Docker & Docker Compose

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install     ca-certificates     curl     gnupg     lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg]   https://download.docker.com/linux/debian   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker

sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)"     -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker compose version
```

---

## 2. Docker Compose Example

Create `docker-compose.yaml`:

```yaml
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./html:/usr/share/nginx/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./data/certbot:/var/www/certbot
      - ./letsencrypt:/etc/letsencrypt

  certbot:
    image: certbot/certbot
    volumes:
      - ./data/certbot:/var/www/certbot
      - ./letsencrypt:/etc/letsencrypt
    entrypoint: /bin/sh -c
```

---

## 3. Nginx Config (`nginx/default.conf`)

```nginx
server {
    listen 80;
    server_name www.test.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

---

## 4. Generate SSL Certificate

```bash
docker compose up -d

docker compose exec certbot   certbot certonly --webroot   --webroot-path=/var/www/certbot   -d www.test.com   --email your-email@gmail.com --agree-tos --no-eff-email
```

---

## 5. HTTPS Nginx Config

```nginx
server {
    listen 80;
    server_name www.test.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name www.test.com;

    ssl_certificate /etc/letsencrypt/live/www.test.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.test.com/privkey.pem;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

---

## 6. Restart Nginx

```bash
docker compose restart nginx
```

---

## 7. Test SSL

```bash
curl -I https://www.test.com
```

---

## Done! 🎉 You now have HTTPS on Docker + Nginx + Certbot.
