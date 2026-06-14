# Развёртывание Next.js + FastAPI на Google Cloud Ubuntu Server

## 1. Подключение к серверу

```bash
ssh USER@SERVER_IP
```

---

## 2. Обновление системы

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 3. Установка Docker

```bash
curl -fsSL https://get.docker.com | sh
```

Проверка:

```bash
docker --version
```

---

## 4. Установка Docker Compose

```bash
sudo apt install docker-compose-plugin -y
```

Проверка:

```bash
docker compose version
```

---

## 5. Добавление пользователя в группу Docker

```bash
sudo usermod -aG docker $USER
```

Перезайти на сервер:

```bash
exit
```

Затем снова подключиться:

```bash
ssh USER@SERVER_IP
```

Проверка:

```bash
docker ps
```

---

## 6. Установка Nginx

```bash
sudo apt install nginx -y
```

Запуск:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

Проверка:

```bash
sudo systemctl status nginx
```

---

## 7. Настройка Firewall в Google Cloud

Создать правило Firewall:

* Direction: Ingress
* Source IPv4: 0.0.0.0/0
* Protocols and ports:

```text
tcp:80,tcp:443
```

---

## 8. Настройка DNS

Добавить A-записи:

```text
A     testsage.uz           SERVER_IP
A     www.testsage.uz       SERVER_IP
A     api.testsage.uz       SERVER_IP
A     www.api.testsage.uz   SERVER_IP
```

Проверить:

```bash
dig testsage.uz
dig api.testsage.uz
```

---

## 9. Клонирование проекта

```bash
git clone REPOSITORY_URL app
cd app
```

---

## 10. Запуск Docker-контейнеров

Сборка и запуск:

```bash
docker compose up -d --build
```

Проверка:

```bash
docker ps
```

Логи:

```bash
docker logs -f frontend
docker logs -f backend
```

---

## 11. Установка Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

---

## 12. Создание конфигурации Nginx

Создать файл:

```bash
sudo nano /etc/nginx/sites-available/testsage
```

Содержимое:

```nginx
upstream fastapi_backend {
    server localhost:8000;
    keepalive 32;
}

upstream nextjs_frontend {
    server localhost:3000;
    keepalive 32;
}

server {
    listen 80;
    server_name testsage.uz www.testsage.uz api.testsage.uz www.api.testsage.uz;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.testsage.uz www.api.testsage.uz;

    ssl_certificate /etc/letsencrypt/live/testsage.uz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/testsage.uz/privkey.pem;

    location /media/ {
        alias /var/www/media/;
        expires 30d;
        access_log off;
    }

    location / {
        proxy_pass http://fastapi_backend;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443 ssl http2;
    server_name testsage.uz www.testsage.uz;

    ssl_certificate /etc/letsencrypt/live/testsage.uz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/testsage.uz/privkey.pem;

    location / {
        proxy_pass http://nextjs_frontend;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 13. Активация конфигурации

```bash
sudo ln -s /etc/nginx/sites-available/testsage /etc/nginx/sites-enabled/
```

Удалить дефолтный сайт:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

Проверить:

```bash
sudo nginx -t
```

Перезапустить:

```bash
sudo systemctl reload nginx
```

---

## 14. Получение SSL сертификата

```bash
sudo certbot --nginx \
-d testsage.uz \
-d www.testsage.uz \
-d api.testsage.uz \
-d www.api.testsage.uz
```

Проверка:

```bash
sudo certbot certificates
```

---

## 15. Проверка автообновления сертификатов

```bash
sudo certbot renew --dry-run
```

---

## 16. Полезные команды

Контейнеры:

```bash
docker ps
docker compose up -d
docker compose down
docker compose restart
docker logs -f frontend
docker logs -f backend
```

Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
```

SSL:

```bash
sudo certbot certificates
sudo certbot renew --dry-run
```

Проверка сайтов:

```bash
curl https://testsage.uz
curl https://api.testsage.uz
```


docker compose run certbot certonly \
  --webroot \
  --webroot-path=/var/www/certbot \
  -d testsage.uz \
  -d www.testsage.uz \
-d api.testsage.uz \
  --email shukhratbekovb@gmail.com \
  --agree-tos