## Деплой бота на VPS-сервере
В папке `../etc/systemd/system` создать файл `<имя_бота>.service`. Они имеет вид:


```ini
[Unit]
Description=<описание_бота>
After=network.target

[Service]
Type=simple
User=<пользователь>
WorkingDirectory=/home/<пользователь>/<папка_бота>
ExecStart=/home/<пользователь>/<папка_бота>/.venv/bin/python3 <имя_бота>.py
Restart=always
RestartSec=3

StandardOutput=append:/var/log/<имя_бота>.log
StandardError=append:/var/log/<имя_бота>.log
# StandardOutput=journal
# StandardError=journal
SyslogIdentifier=<имя_бота>

[Install]
WantedBy=multi-user.target
```
