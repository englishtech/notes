# Библиотека logging

Модуль для записи событий приложения: ошибки, отладка, аудит.
Логирует в консоль, файлы, на почту, по сети.

## Уровни логирования (чем выше число — тем важнее)
| Уровень | Число | Когда использовать |
|---|---|---|
| `DEBUG` | 10 | Детали для разработчика |
| `INFO` | 20 | Нормальная работа |
| `WARNING` | 30 | Подозрительно, но работает |
| `ERROR` | 40 | Ошибка операции |
| `CRITICAL` | 50 | Приложение падает |

**Правило:** если установлен уровень `N`, в лог попадают все сообщения ≥ N.
Например, `setLevel(logging.INFO)` пропустит INFO, WARNING, ERROR, CRITICAL.

---

## Пошаговое создание логгера

### Шаг 1. Получить логгер
| Метод | Что делает | Пример |
|---|---|---|
| `getLogger(name)` | Создаёт/получает логгер по имени | `logger = logging.getLogger("app")` |

### Шаг 2. Задать минимальный уровень
| Метод | Что делает | Пример |
|---|---|---|
| `setLevel()` | Минимальный уровень для логгера | `logger.setLevel(logging.INFO)` |

### Шаг 3. Создать форматтер
Определяет, как будет выглядеть строка в логе.

```python
fmt = logging.Formatter(
    '%(asctime)s | %(levelname)s | %(name)s | %(message)s',
    datefmt='%d.%m.%Y %H:%M:%S'
)
```

| Параметр | Значение | Пример вывода |
|---|---|---|
| `%(asctime)s` | Время события | `03.09.2026 14:30:15` |
| `%(levelname)s` | Уровень | `ERROR` |
| `%(name)s` | Имя логгера | `app` |
| `%(message)s` | Само сообщение | `Ошибка БД` |
| `%(filename)s` | Имя файла | `main.py` |
| `%(lineno)d` | Номер строки | `42` |
| `datefmt` | Формат даты | `%d.%m.%Y %H:%M:%S` |

Результат: `03.03.2023 14:30:15 | ERROR | app | Ошибка БД`

### Шаг 4. Создать хэндлеры и привязать форматтер
Хэндлер — это **куда** пишем лог.

| Метод | Что делает | Пример |
|---|---|---|
| `addHandler()` | Подключает хэндлер к логгеру | `logger.addHandler(handler)` |
| `setFormatter()` | Задаёт формат строки для хэндлера | `handler.setFormatter(fmt)` |

#### Виды хэндлеров
| Хэндлер | Куда пишет | Пример создания |
|---|---|---|
| `StreamHandler` | В консоль (терминал) | `logging.StreamHandler(sys.stdout)` |
| `FileHandler` | В файл (без ограничений) | `logging.FileHandler("app.log", encoding="utf-8")` |
| `RotatingFileHandler` | В файл с ротацией по размеру | `RotatingFileHandler("app.log", maxBytes=5*1024*1024, backupCount=5)` |
| `TimedRotatingFileHandler` | В файл с ротацией по времени | `TimedRotatingFileHandler("app.log", when="midnight", backupCount=7)` |
| `SMTPHandler` | На почту по SMTP | `SMTPHandler(mailhost=("smtp.mail.com", 1234), ...)` |

#### Ротация файлов (RotatingFileHandler)
```python
from logging.handlers import RotatingFileHandler

fh = RotatingFileHandler(
    "app.log",
    maxBytes=5*1024*1024,  # 5 МБ — при достижении создаётся новый файл
    backupCount=5,         # хранить 5 старых файлов (app.log.1, app.log.2, ...)
    encoding="utf-8"
)
```
При достижении 5 МБ: `app.log` → `app.log.1`, `app.log.1` → `app.log.2`, и т.д.
Старые файлы сверх `backupCount` удаляются. Итого на диске: ~30 МБ.

#### Почтовый хэндлер (SMTPHandler)
```python
from logging.handlers import SMTPHandler

mail_handler = SMTPHandler(
    mailhost=("smtp.mail.com", 1234),   # сервер и порт
    fromaddr="logs@domain.ru",           # от кого
    toaddrs=["info@domain.ru"],          # кому (список)
    subject="⚠️ Ошибка в проекте",       # тема письма
    credentials=("logs@domain.ru", "pass"),  # логин/пароль
    secure=()                            # включает STARTTLS
)
mail_handler.setLevel(logging.ERROR)  # слать только ERROR и выше
```

### Шаг 5. Писать сообщения
| Метод | Уровень | Пример |
|---|---|---|
| `logger.debug()` | DEBUG | `logger.debug("Значение x = %s", x)` |
| `logger.info()` | INFO | `logger.info("Сервер запущен")` |
| `logger.warning()` | WARNING | `logger.warning("Мало места на диске")` |
| `logger.error()` | ERROR | `logger.error("Не удалось подключиться к БД")` |
| `logger.critical()` | CRITICAL | `logger.critical("Приложение не может работать")` |
| `logger.exception()` | ERROR + traceback | `logger.exception("Ошибка:")` (внутри `except`) |

---

## Готовый пример: `config/logger.py`

```python
import logging, sys
from logging.handlers import RotatingFileHandler, SMTPHandler

def get_logger(name: str) -> logging.Logger:
    logger = logging.getLogger(name)
    if logger.handlers: return logger

    logger.setLevel(logging.INFO)
    fmt = logging.Formatter('%(asctime)s | %(levelname)s | %(name)s | %(message)s', datefmt='%d.%m.%Y %H:%M:%S')

    # Вывод в терминал
    sh = logging.StreamHandler(sys.stdout)
    sh.setFormatter(fmt)
    logger.addHandler(sh)

    # Вывод в файл: макс 5 МБ, храним текущий + 5 старых (итого ~30 МБ)
    fh = RotatingFileHandler(LOG_DIR / "app.log", maxBytes=5*1024*1024, backupCount=5, encoding="utf-8")
    fh.setFormatter(fmt)
    logger.addHandler(fh)

    # Отправка ошибок (ERROR и выше) на почту
    mail_handler = SMTPHandler(
        mailhost=(EMAIL_SERV, EMAIL_PORT),
        fromaddr=EMAIL_ADDR,
        toaddrs=[EMAIL_ADDR],
        subject="Ошибка",
        credentials=(EMAIL_ADDR, EMAIL_PASS),
        secure=()
    )
    mail_handler.setLevel(logging.ERROR)
    mail_handler.setFormatter(fmt)
    logger.addHandler(mail_handler)

    return logger
```

## Пример использования в любом файле проекта

```python
from config.logger import get_logger

logger = get_logger(__name__)  # имя = имя модуля, например "app.main"

def load_data():
    logger.info("Загрузка данных началась")
    try:
        data = fetch_from_api()
        logger.info("Загружено %d записей", len(data))
        return data
    except ConnectionError:
        logger.exception("Не удалось подключиться к API")  # ERROR + полный traceback
        return []
```

**Что произойдёт при `logger.exception(...)`:**
1. Строка появится в терминале
2. Строка запишется в `logs/app.log`
3. На почту придёт письмо с темой `Ошибка` и полным traceback

---

## Советы
- В продакшене: `INFO` для файла/терминала, `ERROR` для почты — не будет спама
- Не использовать `print()` для ошибок — только логгер
- Всегда указывать `encoding="utf-8"` для файловых хэндлеров
- Проверка `if logger.handlers: return` защищает от дублирования строк

---

## Самый простой вариант: `basicConfig`

Если нужен быстрый логгер без хэндлеров, форматтеров и прочего - можно использовать `basicConfig`. Идеально для небольших скриптов.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s | %(levelname)s | %(message)s',
    datefmt='%d.%m.%Y %H:%M:%S',
    handlers=[
        logging.FileHandler("app.log", encoding="utf-8"),
        logging.StreamHandler()
    ]
)

logging.info("Скрипт запущен")
logging.error("Что-то пошло не так")
```

### Параметры `basicConfig`
| Параметр | Что делает | Пример |
|---|---|---|
| `level` | Минимальный уровень | `level=logging.INFO` |
| `format` | Формат строки | `format='%(levelname)s: %(message)s'` |
| `datefmt` | Формат даты | `datefmt='%H:%M:%S'` |
| `filename` | Писать в файл (вместо консоли) | `filename="app.log"` |
| `filemode` | Режим открытия файла | `filemode="a"` (append, по умолчанию) |
| `handlers` | Список хэндлеров | `handlers=[FileHandler(...), StreamHandler()]` |
