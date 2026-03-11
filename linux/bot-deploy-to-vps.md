## Создание виртуального окружения проекта на VPS-сервере
Перейти в папку пользователя `../home/<пользователь>`  
Создать папку проекта:
```bash
mkdir ~/имя_бота
```
Перейти в папку проекта:
```bash
cd ~/имя_бота
```
Создать виртуальное окружение:
```bash
python3 -m venv .venv
```
Активировать виртуальное окружение:
```bash
source .venv/bin/activate
```
В терминале появится `(.venv)`  
Установить зависимости. Если есть файл `requirements.txt`:
```bash
python3 -m pip install -r requirements.txt
```
Запустить проект:
```bash
python3 имя_бота.py
```


## Загрузка файлов проекта на VPS-сервер
Для загрузки файлов с локального компьютера, запустить терминал (`bash`).
Не подключаясь к VPS-серверу, ввести:
```bash
scp -r D:/Python/../папка_проекта_на_компьютере/* пользователь@ip_адрес:~/папка_проекта_на_vps/
```
Ввести пароль, если подключение по паролю, дождаться загрузки файлов проекта.  
Файл `.env` загрузить отдельно (видимо, из-за наличия `.gitignore`)
```bash
scp -r D:/Python/../папка_проекта_на_компьютере/.env пользователь@ip_адрес:~/папка_проекта_на_vps/
```

## Деплой проекта на VPS-сервере
В папке `../etc/systemd/system` создать файл `<имя_бота>.service`. Они имеет вид:
```ini
[Unit]
Description=<описание_бота>
After=network.target

[Service]
Type=simple
User=<пользователь>
WorkingDirectory=/home/<пользователь>/<папка_бота>
ExecStart=/home/<пользователь>/<папка_бота>/.venv/bin/python3 <имя_бота>.py  # Запуск Python из виртуального окружения
Restart=always
RestartSec=3

StandardOutput=append:/var/log/<имя_бота>.log                                # Стандартный вывод в консоль пишется в лог
StandardError=append:/var/log/<имя_бота>.log                                 # Стандартный вывод ошибок пишется в лог
# StandardOutput=journal                                                     # Или в журнал
# StandardError=journal
SyslogIdentifier=<имя_бота>

[Install]
WantedBy=multi-user.target
```


## Команды systemd
| Команда | Описание |
|---------|----------|
| `sudo systemctl daemon-reload` | Перечитать конфиги после изменения `.service`-файла |
| `sudo systemctl enable <имя>.service` | Добавить сервис в автозагрузку (запуск при старте системы) |
| `sudo systemctl start <имя>.service` | Запустить сервис |
| `sudo systemctl restart <имя>.service` | Перезапустить сервис (остановить и запустить заново) |
| `sudo systemctl status <имя>.service` | Показать статус сервиса (работает/упал, логи, время запуска) |
| `sudo systemctl stop <имя>.service` | Остановить сервис |
| `sudo systemctl disable <имя>.service` | Убрать из автозагрузки |
| `sudo journalctl -u <имя>.service -f` | Показать логи сервиса в реальном времени |
| `tail -f /var/log/<имя_бота>.log` | Показать логи сервиса в реальном времени, если в <имя_бота>.service указана запись в файл |
