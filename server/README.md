# Конфигурация сервера (VPS SpaceWeb 80.93.52.144)

Здесь хранятся боевые конфиги nginx для восстановления при переносе сервера.

## Файлы

| Файл | Куда копировать на сервере |
|------|---------------------------|
| `nginx-plenkinaokna.ru.conf` | `/etc/nginx/sites-available/plenkinaokna.ru` |
| `nginx-podkovamsk.ru.conf` | `/etc/nginx/sites-available/podkovamsk.ru` |

## Восстановление на новом сервере

```bash
cp server/nginx-plenkinaokna.ru.conf /etc/nginx/sites-available/plenkinaokna.ru
cp server/nginx-podkovamsk.ru.conf   /etc/nginx/sites-available/podkovamsk.ru
ln -sf /etc/nginx/sites-available/plenkinaokna.ru /etc/nginx/sites-enabled/
ln -sf /etc/nginx/sites-available/podkovamsk.ru   /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

## Что учтено в конфиге plenkinaokna.ru

- **HTTP/2** (`listen 443 ssl http2`) — критично: все ресурсы страницы идут
  в одном TLS-соединении (без этого мобильные операторы РФ резали
  повторные соединения — сайт «вечно грузился» с телефонов)
- **301-редиректы** старых URL предыдущей версии сайта (Vercel/Next.js)
  на новые страницы — для склейки в поисковиках. Добавляйте новые сюда
  и в конфиг на сервере одновременно
- Сертификаты Let's Encrypt: `/etc/letsencrypt/live/plenkinaokna.ru/`
- Корень сайта: `/var/www/plenkin`

## Шрифты

`fonts/inter-*.woff2` обязаны быть в репозитории — nginx раздаёт их локально
(зависимость от cdn.jsdelivr.net удалена, он нестабилен в РФ).

## Деплой (как делается)

1. Правки заливаются в этот репозиторий (main)
2. rsync содержимого (без `.git`, `vercel.json`, `README.md`, `DEPLOY.md`, `server/`) в `/var/www/plenkin/`
3. Перед деплоем — бэкап: `tar -czf /root/plenkin-backup-$(date +%Y%m%d-%H%M%S).tar.gz -C /var/www plenkin`
