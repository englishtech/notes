# Nginx — краткая памятка

## Что это такое

**Nginx** — это высокопроизводительный веб-сервер и обратный прокси (reverse proxy). Принимает входящие HTTP-запросы, обрабатывает их и передаёт дальше — либо отдаёт статические файлы, либо перенаправляет на ваше приложение.

## Зачем нужен

- **Reverse proxy** — скрывает ваше приложение за собой, принимает запросы на 80/443 портах и передаёт их на внутренний порт (например, `localhost:8000`).
- **SSL/TLS** — централизованное управление HTTPS-сертификатами.
- **Балансировка нагрузки** — распределение трафика между несколькими серверами.
- **Статика** — отдача картинок, CSS, JS напрямую, без нагрузки на приложение.
- **Кэширование, сжатие (gzip), защита от базовых атак.**

## Почему Nginx

- **Быстрый** — асинхронная архитектура, держит тысячи одновременных соединений.
- **Лёгкий** — потребляет минимум памяти.
- **Гибкий** — конфигурация через простые текстовые файлы.
- **Стандарт индустрии** — используется на миллионах сайтов, от блогов до Netflix.

---

## Установка (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install nginx -y
```

Проверка:
```bash
sudo systemctl status nginx
```

Основные команды управления:
```bash
sudo systemctl start nginx     # запустить
sudo systemctl stop nginx      # остановить
sudo systemctl restart nginx   # перезапустить
sudo systemctl reload nginx    # перечитать конфиги без обрыва соединений
sudo systemctl enable nginx    # автозапуск при загрузке ОС
```

---

## Структура конфигов

- `/etc/nginx/nginx.conf` — главный конфиг
- `/etc/nginx/sites-available/` — **доступные** конфиги сайтов
- `/etc/nginx/sites-enabled/` — **активные** конфиги (символические ссылки)
- `/var/log/nginx/` — логи (`access.log`, `error.log`)

---

## Настройка сайта (Reverse Proxy)

Допустим, ваше приложение работает на `localhost:8000`, а домен `example.com` уже указывает на IP вашего сервера.

### 1. Создайте конфиг

```bash
sudo nano /etc/nginx/sites-available/example.com
```

### 2. Вставьте содержимое

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    # Обратный прокси на ваше приложение
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Статика (если есть)
    location /static/ {
        alias /var/www/example.com/static/;
    }
}
```

### 3. Активируйте сайт

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```

### 4. Проверьте синтаксис и перезагрузите

```bash
sudo nginx -t                 # проверка конфига
sudo systemctl reload nginx   # применение
```

---

## Подключение HTTPS (Let's Encrypt)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot **автоматически** изменит ваш конфиг: добавит SSL, перенаправление с HTTP на HTTPS и настроит автообновление сертификатов.

Проверка автообновления:
```bash
sudo certbot renew --dry-run
```

---

## Полезные команды

| Команда | Назначение |
|---|---|
| `sudo nginx -t` | Проверить синтаксис конфигов |
| `sudo systemctl reload nginx` | Применить изменения без перезапуска |
| `tail -f /var/log/nginx/error.log` | Смотреть ошибки в реальном времени |
| `tail -f /var/log/nginx/access.log` | Смотреть входящие запросы |
| `sudo ufw allow 'Nginx Full'` | Открыть порты 80 и 443 в файрволе |

---

## Частые ошибки

- **502 Bad Gateway** — приложение на целевом порту не запущено или недоступно.
- **403 Forbidden** — нет прав на файлы статики или не настроен `index`.
- **404 Not Found** — неверный `server_name` или путь в `alias`/`root`.
- **Конфиг не применяется** — забыли `sudo nginx -t` и `reload`, или создали файл в `sites-available`, но не сделали symlink в `sites-enabled`.

---

> 💡 **Совет:** всегда запускайте `sudo nginx -t` перед `reload` — это спасёт от падения сервера из-за опечатки в конфиге.
