## APScheduler
Установить:
```bash
pip install apscheduler
```
Импортировать (для асинхронного использования):
```python
from apscheduler.schedulers.asyncio import AsyncIOScheduler
```
Создать объект планировщика:
```python
scheduler = AsyncIOScheduler()
```
> Опционально (для aiogram) добавить в workflow для использования в хэндлерах
> ```python
> dp.workflow_data['scheduler'] = scheduler
> ```

## Основные методы
| Метод | Описание | Пример |
|-------|----------|--------|
| `add_job(func, trigger, **kwargs)` | Добавить задачу | `scheduler.add_job(job, 'interval', minutes=5, id='my_job_1')` |
| `reschedule_job(job_id, trigger, **kwargs)` | Перепланировать задачу | `scheduler.reschedule_job(job_id='my_job_1', trigger='interval', minutes=10)` |
| `remove_job(job_id)` | Удалить задачу по ID | `scheduler.remove_job('my_job_1')` |
| `get_job(job_id)` | Получить задачу по ID | `job = scheduler.get_job('my_job_1')` |
| `get_jobs()` | Получить список объектов Job — всех активных задач в планировщике| `jobs = scheduler.get_jobs()` |
| `pause_job(job_id)` | Приостановить задачу | `scheduler.pause_job('my_job_1')` |
| `resume_job(job_id)` | Возобновить задачу | `scheduler.resume_job('my_job_1')` |
| `modify_job(job_id, **kwargs)` | Изменить задачу | `scheduler.modify_job('my_job_1', minutes=10)` |
| `shutdown()` | Остановить планировщик задач, завершить все работающие задачи, освободить ресурсы | `scheduler.shutdown()` |

## Параметры `add_job()`:
| Параметр | Описание | Пример |
|----------|----------|--------|
| `func` | Функция для выполнения | `my_function` |
| `trigger` | Тип триггера (`interval`, `date`, `cron`) | `'interval'` |
| `args` | Позиционные аргументы (кортеж) | `args=(url, user_id)` |
| `kwargs` | Именованные аргументы (словарь) | `kwargs={'url': URL, 'delay': 5}` |
| `id` | Уникальный идентификатор задачи | `id='my_job_1'` |
| `name` | Имя задачи | `name='Проверка сайта'` |
| `replace_existing` | Заменить существующую с таким же ID | `replace_existing=True` |
| `max_instances` | Макс. одновременных запусков | `max_instances=3` |
| `misfire_grace_time` | Время на запуск после пропуска (сек) | `misfire_grace_time=300` |


## Триггеры (параметры планирования)
| Триггер | Описание | Пример |
|---------|----------|--------|
| `interval` | Повторение через интервал | `minutes=5`, `hours=2`, `days=1` |
| `date` | Однократное выполнение | `run_date='2024-01-01 10:00:00'` |
| `cron` | По cron-расписанию | `hour=10, minute=30`, `day_of_week='mon-fri'` |


## Примеры использования

```python
#BackgroundScheduler - работает в фоне (отдельный поток) - Веб-приложения, боты, сервисы
from apscheduler.schedulers.background import BackgroundScheduler
from datetime import datetime

scheduler = BackgroundScheduler()
scheduler.start()

# Interval — каждые 5 минут
scheduler.add_job(
    my_function,
    'interval',
    minutes=5,
    id='job_interval'
)

# Date — однократно в указанное время
scheduler.add_job(
    my_function,
    'date',
    run_date=datetime(2024, 1, 1, 10, 0, 0),
    id='job_date'
)

# Cron — каждый день в 10:30
scheduler.add_job(
    my_function,
    'cron',
    hour=10,
    minute=30,
    id='job_cron'
)

# Cron — по будням в 9:00
scheduler.add_job(
    my_function,
    'cron',
    day_of_week='mon-fri',
    hour=9,
    minute=0
)
```

## Атрибуты объекта Job

| Атрибут | Описание | Пример |
|---------|----------|--------|
| `job.id` | Уникальный идентификатор задачи | `'my_job_1'` |
| `job.name` | Имя задачи (по умолчанию имя функции) | `'my_function'` |
| `job.func` | Ссылка на функцию | `<function my_func>` |
| `job.args` | Позиционные аргументы | `(1, 2, 3)` |
| `job.kwargs` | Именованные аргументы | `{'url': 'http://example.com'}` |
| `job.trigger` | Триггер задачи | `IntervalTrigger(...)` |
| `job.next_run_time` | Время следующего запуска | `datetime.datetime(...)` |
| `job.pending` | Задача в ожидании (ещё не запускалась) | `True` / `False` |
| `job.max_instances` | Макс. количество одновременных запусков | `1` |
| `job.misfire_grace_time` | Время на запуск после пропуска (сек) | `60` |

# Хранение задач APScheduler в БД с помощью SQLAlchemy
Установить: `pip install sqlalchemy`  
Если нужно прокидывать объект Scheduler между модулями, лучше создать отдельный модуль, например, `init_scheduler.py`:
```python
from pathlib import Path
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore

scheduler = None

def init_scheduler():
    global scheduler
    db_path = Path(__file__).parent.parent / "database" / "apscheduler_jobs.db"
    db_path.parent.mkdir(exist_ok=True)
    url = f"sqlite:///{db_path.absolute()}"
    jobstore = {'default': SQLAlchemyJobStore(url=url)}
    scheduler = AsyncIOScheduler(jobstores=jobstore)
```
