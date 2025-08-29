# ChatGPT Proxy Docker

Обратный прокси для доступа к chatgpt.com и его поддоменам через ваш собственный домен с SSL от Let’s Encrypt, полностью работающий внутри Docker.

## Структура проекта

```
chatgpt-proxy-docker/
├── docker-compose.yml
├── nginx.conf.template
├── certs/
├── certbot-logs/
└── webroot/
```

## Возможности

- Полное проксирование chatgpt.com и всех поддоменов (auth, api, cdn и др.)
- Переписывание ссылок внутри HTML/JS/CSS через sub_filter
- HTTPS с сертификатом Let’s Encrypt для вашего домена
- Автоматическое обновление сертификатов и перезагрузка Nginx
- Домен задается через переменную окружения DOMAIN в docker-compose.yml

## Настройка

1. Клонируйте проект или создайте папку
2. Правьте docker-compose.yml, задав DOMAIN и EMAIL
3. Получите первый сертификат:
```bash
docker compose run --rm certbot-init
```
4. Запустите сервисы:
```bash
docker compose up -d
```
5. Проверьте работу: https://myproxy.example.com

## Обновление сертификатов

- Сертификаты автоматически проверяются каждые 12 часов
- Если сертификат обновлен → Nginx автоматически перезагружается
- Проверить тестовое обновление вручную:
```bash
docker compose run --rm certbot renew --dry-run
```

## Перезапуск

```bash
docker compose restart
```

Логи:
```bash
docker compose logs -f nginx
docker compose logs -f certbot
```
