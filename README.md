# ChatGPT Proxy via Nginx + Docker Compose

Этот проект позволяет развернуть прокси для доступа к `chatgpt.com` через ваш домен, с автоматическим получением SSL сертификатов от Let's Encrypt.

---

## 📁 Структура проекта

```
.
├── docker-compose.yml
├── .env
├── nginx.conf.template
├── certs/               # хранятся сертификаты
├── webroot/             # для ACME challenge
└── certbot-logs/        # логи certbot
```

---

## ⚙️ Настройка `.env`

Создайте файл `.env` в корне проекта:

```dotenv
DOMAIN=your-domain
EMAIL=your-email@example.com
```

* `DOMAIN` — ваш домен, через который будет доступен прокси.
* `EMAIL` — email для регистрации у Let's Encrypt.

---

## 🐳 Запуск

### 1. Получение сертификата

Перед первым запуском Nginx необходимо получить сертификат:

```bash
docker compose run --rm certbot-init
```

### 2. Поднимаем все сервисы

```bash
docker compose up -d
```

* Nginx будет слушать порты `80` и `443`.
* Certbot автоматически обновляет сертификаты каждые 12 часов.

---

## 🔧 Проверка работы

1. Проверка доступности ACME challenge:

```bash
curl -v http://your-domain/.well-known/acme-challenge/test
```

2. Проверка логов Nginx:

```bash
docker compose logs nginx --tail 50
```

---

## ⚠️ Особенности

* Nginx **не стартует**, если сертификаты ещё не созданы.
* Используется шаблон `nginx.conf.template` с переменной `${DOMAIN}`, подставляемой через `envsubst`.
* Certbot работает в двух режимах:

  * `certbot-init` — инициализация сертификата.
  * `certbot` — автоматическое обновление сертификатов каждые 12 часов.

---

## 🔄 Обновление сертификатов

Для ручного обновления:

```bash
docker compose run --rm certbot-init
```

После этого можно перезапустить Nginx:

```bash
docker compose restart nginx
```

---

## 💡 Советы

* Все пути к сертификатам и ACME challenge смонтированы как volume, чтобы данные сохранялись между перезапусками контейнеров.
* Для добавления нового домена:

  1. Добавьте его в `.env` (`DOMAIN=newdomain.com`).
  2. Перегенерируйте `nginx.conf` с `envsubst`.
  3. Получите новый сертификат через `certbot-init`.

---

## 🔄 Автоматическая генерация default.conf

Создайте скрипт `generate-nginx-config.sh`:

```bash
#!/bin/sh

if [ -z "$DOMAIN" ]; then
  echo "Please set DOMAIN environment variable"
  exit 1
fi

envsubst '$${DOMAIN}' < ./nginx.conf.template > ./default.conf
```

Запуск:

```bash
export DOMAIN=your-domain
sh generate-nginx-config.sh
```

* После этого `default.conf` готов и можно запускать контейнер Nginx.

