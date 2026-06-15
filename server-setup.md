# Развёртывание Next.js + FastAPI на Google Cloud Ubuntu Server

Домен: **newposm.uz** | IP: **34.32.142.97**

---

## 1. Подключение к серверу

```bash
ssh USER@34.32.142.97
```

---

## 2. Обновление системы

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 3. Установка Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Перезайти на сервер:

```bash
exit
# затем снова подключиться
ssh USER@34.32.142.97
docker ps   # проверка
```

---

## 4. Установка Docker Compose

```bash
sudo apt install docker-compose-plugin -y
docker compose version   # проверка
```

---

## 5. Настройка Firewall в Google Cloud

Открыть только эти порты (MinIO не нужен публично — он работает через nginx):

```
tcp:80, tcp:443
```

---

## 6. Клонирование проекта

```bash
git clone REPOSITORY_URL app
cd app
```

---

## 7. Получение SSL сертификата (первый запуск)

Nginx не стартует без SSL сертификатов. Поэтому сначала поднимаем только HTTP,
получаем сертификат, потом запускаем полный стек.

### Шаг 7.1 — Временно заменить nginx конфиг на HTTP-only

```bash
cp nginx/conf.d/app.conf nginx/conf.d/app.conf.bak
cp app.conf nginx/conf.d/app.conf
```

### Шаг 7.2 — Создать папки для certbot и запустить nginx

```bash
mkdir -p certbot/www certbot/conf
docker compose up -d nginx
```

Проверить что nginx работает:
```bash
curl http://newposm.uz        # должно вернуть: ok
curl http://api.newposm.uz    # должно вернуть: ok
```

### Шаг 7.3 — Получить SSL сертификат

```bash
docker compose run --rm certbot certonly \
  --webroot \
  --webroot-path=/var/www/certbot \
  -d newposm.uz \
  -d www.newposm.uz \
  -d api.newposm.uz \
  --email sanjaranvarov55563@gmail.com \
  --agree-tos \
  --no-eff-email
```

### Шаг 7.4 — Восстановить production nginx конфиг

```bash
cp nginx/conf.d/app.conf.bak nginx/conf.d/app.conf
```

---

## 8. Запуск всего стека

```bash
docker compose up -d --build
```

Проверить что все контейнеры работают:

```bash
docker ps
```

Должны быть запущены: `nginx`, `frontend`, `backend`, `postgres`, `minio`

---

## 9. Проверка сайтов

```bash
curl https://newposm.uz
curl https://api.newposm.uz/todos
```

---

## 10. Автообновление SSL сертификатов

Добавить в crontab:

```bash
crontab -e
```

Добавить строку:

```
0 3 * * * cd /home/USER/app && docker compose run --rm certbot renew --quiet && docker compose kill -s HUP nginx
```

---

## 11. Полезные команды

```bash
# Контейнеры
docker ps
docker compose up -d --build
docker compose down
docker compose restart
docker logs -f frontend
docker logs -f backend
docker logs -f nginx

# Перезагрузить nginx без остановки
docker compose kill -s HUP nginx

# Проверить SSL сертификат
docker compose run --rm certbot certificates
```
