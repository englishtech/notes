# FastAPI
## 1. Основы Web и HTTP

Web работает по схеме «Клиент-Сервер»: клиент отправляет HTTP-запрос, сервер возвращает ответ. Запросы состоят из URL, метода (GET/POST), заголовков и тела. Для обмена данными используется универсальный формат JSON, который легко маппится на словари Python. Для тестирования API используются Postman, Thunder Client или встроенный Swagger UI.

- **Клиент/Сервер:** Инициатор запроса и обработчик ответа.
- **HTTP-методы:** GET (получить), POST (создать), PUT (обновить), DELETE (удалить).
- **Эндпоинт (ручка):** Конкретный URL для доступа к функции API.
- **JSON:** Текстовый формат данных (ключи в двойных кавычках, `true/false/null`).

```python
from fastapi import FastAPI
app = FastAPI()

# GET: клиент запрашивает данные
@app.get("/tasks")
def get_tasks():
    return [{"id": 1, "title": "Изучить HTTP"}] # Ответ в JSON

# POST: клиент отправляет данные в теле запроса
@app.post("/tasks")
def create_task(task: dict): # task — это JSON из тела
    return {"status": "created", "data": task}
```

## 2. Окружение, установка и структура приложения

Для изоляции зависимостей каждого проекта используется виртуальное окружение (`venv`). FastAPI — это фреймворк для описания логики API («повар»), а Uvicorn — ASGI-сервер, который принимает сетевые запросы и передаёт их в приложение («официант»). Минимальное приложение состоит из импорта `FastAPI` и создания экземпляра `app`, которому можно задать метаданные для автодокументации.

- **venv** — изолированное окружение Python для проекта, избегает конфликтов версий библиотек.
- **FastAPI** — фреймворк для создания API, отвечает за бизнес-логику и валидацию.
- **Uvicorn** — ASGI-сервер, принимает HTTP-запросы из сети и передаёт их в приложение.
- **ASGI** — стандарт асинхронного взаимодействия сервера и приложения.
- **app** — экземпляр `FastAPI()`, центральная переменная, в которой регистрируются эндпоинты.

```python
# main.py — точка входа в приложение

from fastapi import FastAPI

# Создание экземпляра приложения с метаданными
# Они отобразятся в автодокументации /docs
app = FastAPI(
    title="Task Manager API",
    description="Учебное приложение для курса по FastAPI",
    version="1.0.0"
)

# Запуск из терминала:
# uvicorn main:app --reload
# где main — имя файла, app — переменная экземпляра
```

**Шпаргалка по командам:**
```bash
# Создание окружения
python -m venv venv

# Активация
venv\Scripts\activate          # Windows (cmd)
.venv\Scripts\Activate.ps1     # Windows (PowerShell)
source venv/bin/activate       # macOS/Linux

# Установка
pip install fastapi uvicorn

# Запуск сервера (с автоперезагрузкой при изменениях)
uvicorn main:app --reload

# Игнорирование окружения в Git
echo venv/ > .gitignore
```

## 3. Uvicorn и автоматическая документация

Uvicorn — это ASGI-сервер, который принимает HTTP-запросы и передаёт их в FastAPI-приложение. Запуск осуществляется командой `uvicorn main:app --reload`, где флаг `--reload` автоматически перезагружает сервер при изменениях кода. FastAPI автоматически генерирует документацию по стандарту OpenAPI, доступную через Swagger UI (`/docs`) для интерактивного тестирования и ReDoc (`/redoc`) для чтения.

- **Uvicorn** — ASGI-сервер, принимает сетевые запросы и передаёт их в приложение.
- **ASGI** — стандарт асинхронного взаимодействия сервера и Python-приложения.
- **--reload** — флаг автоперезагрузки сервера при изменении кода (только для разработки).
- **Workers** — количество копий приложения для продакшена (несовместимо с `--reload`).
- **OpenAPI** — стандарт описания API в формате JSON (схема всех эндпоинтов).
- **Swagger UI** (`/docs`) — интерактивная документация с кнопкой "Try it out" для тестирования.
- **ReDoc** (`/redoc`) — документация для чтения, удобна для передачи клиентам.

```python
# main.py — приложение с запуском из кода

from fastapi import FastAPI
import uvicorn

# Создание приложения с метаданными
app = FastAPI(
    title="Task Manager API",
    description="Учебное приложение",
    version="1.0.0"
)

# Эндпоинт для демонстрации в документации
@app.get("/")
def read_root():
    return {"message": "Привет, Uvicorn!"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    # item_id: int — подсказка типа автоматически попадёт в документацию
    return {"item_id": item_id, "q": q}

# Запуск из кода (альтернатива терминалу)
if __name__ == "__main__":
    # "main:app" — строка с именем файла и переменной
    # reload=True — автоперезагрузка (как --reload в терминале)
    uvicorn.run("main:app", host="127.0.0.1", port=8000, reload=True)

# Команды запуска из терминала:
# uvicorn main:app --reload              # базовый запуск
# uvicorn main:app --port 8888           # сменить порт
# uvicorn main:app --host 0.0.0.0        # доступ из локальной сети
# uvicorn main:app --workers 4           # продакшен (без --reload)

# Документация:
# http://127.0.0.1:8000/docs    — Swagger UI (тестирование)
# http://127.0.0.1:8000/redoc   — ReDoc (чтение)
```

## 4. 5 главных HTTP-методов в FastAPI

REST API использует 5 основных HTTP-методов для работы с ресурсами: GET (получение), POST (создание), PUT (полное обновление), PATCH (частичное обновление), DELETE (удаление). FastAPI предоставляет декораторы `@app.get`, `@app.post` и т.д. для регистрации эндпоинтов. Валидация данных обеспечивается моделями Pydantic, а ошибки возвращаются через `HTTPException`.

- **GET** — получение данных, не изменяет состояние сервера.
- **POST** — создание нового ресурса, данные передаются в теле запроса.
- **PUT** — полная замена ресурса (все поля обязательны).
- **PATCH** — частичное обновление (поля опциональны через `Optional`).
- **DELETE** — удаление ресурса по идентификатору.
- **HTTPException** — исключение для возврата HTTP-ошибок с кодом (404, 400 и т.д.).
- **BaseModel (Pydantic)** — класс для описания схемы данных с автоматической валидацией.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# Модель для создания (все поля обязательны)
class User(BaseModel):
    id: int
    name: str
    age: int

# Модель для PATCH (все поля опциональны)
class UserUpdate(BaseModel):
    name: Optional[str] = None
    age: Optional[int] = None

# "База данных" в памяти
users_db = [
    {"id": 1, "name": "Alice", "age": 25},
    {"id": 2, "name": "Bob", "age": 30},
]

# 🔵 GET — получение всех или одного пользователя
@app.get("/users")
def get_all_users():
    return users_db

@app.get("/users/{user_id}")
def get_user(user_id: int):
    for user in users_db:
        if user["id"] == user_id:
            return user
    raise HTTPException(status_code=404, detail="User not found")

# 🟢 POST — создание нового пользователя
@app.post("/users")
def create_user(user: User):
    users_db.append(user.dict())
    return {"message": "User created", "user": user}

# 🟠 PUT — полная замена пользователя
@app.put("/users/{user_id}")
def update_user_complete(user_id: int, updated_user: User):
    for i, user in enumerate(users_db):
        if user["id"] == user_id:
            users_db[i] = updated_user.dict()
            return {"message": "User updated completely"}
    raise HTTPException(status_code=404, detail="User not found")

# 🟣 PATCH — частичное обновление
@app.patch("/users/{user_id}")
def update_user_partial(user_id: int, user_update: UserUpdate):
    for user in users_db:
        if user["id"] == user_id:
            if user_update.name is not None:
                user["name"] = user_update.name
            if user_update.age is not None:
                user["age"] = user_update.age
            return {"message": "User updated partially", "user": user}
    raise HTTPException(status_code=404, detail="User not found")

# 🔴 DELETE — удаление пользователя
@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    for i, user in enumerate(users_db):
        if user["id"] == user_id:
            del users_db[i]
            return {"message": f"User {user_id} deleted"}
    raise HTTPException(status_code=404, detail="User not found")
```

**Таблица методов:**

| Метод | Действие | Тело запроса | Идемпотентность |
|-------|----------|--------------|-----------------|
| GET | Получить | Нет | ✅ Да |
| POST | Создать | Да (обязательно) | ❌ Нет |
| PUT | Заменить целиком | Да (все поля) | ✅ Да |
| PATCH | Обновить частично | Да (нужные поля) | ❌ Нет |
| DELETE | Удалить | Нет | ✅ Да |
**Частые ошибки:**
- `Module not found` → вы не в папке с `main.py`
- `Address already in use` → порт занят, используйте `--port 8001`
- `no attribute 'app'` → переменная называется иначе, укажите `main:my_app`

## 5. Path и Query параметры, типизация и валидация

В FastAPI параметры делятся на Path (часть URL, `{id}`) и Query (после `?`, `?limit=10`). FastAPI автоматически конвертирует строки из URL в нужные типы (`int`, `float`, `bool`) и валидирует данные — при ошибке возвращает статус 422 с подробным JSON. Необязательные параметры объявляются через `Тип | None = None`.

- **Path Parameter** (`/users/{id}`) — обязательная часть URL, идентифицирует ресурс.
- **Query Parameter** (`/users?limit=10`) — опциональные настройки после `?`, для фильтрации/пагинации.
- **Type Hints** (`id: int`) — подсказки типов, включают автоконвертацию и валидацию.
- **422 Unprocessable Entity** — статус ответа при ошибке валидации данных.
- **Optional / `| None`** — синтаксис для необязательных параметров (`str | None = None`).

```python
from fastapi import FastAPI

app = FastAPI()

# "База данных" в памяти
fake_tasks_db = [
    {"task_id": 1, "task_name": "Изучить Python"},
    {"task_id": 2, "task_name": "Подключить БД"},
    {"task_id": 3, "task_name": "Выучить FastAPI"},
    {"task_id": 4, "task_name": "Написать тесты"},
]

# 🔵 Path-параметр: {task_id} — часть URL, обязательный
# Тип int включает автоконвертацию и валидацию
@app.get("/tasks/{task_id}")
async def get_task(task_id: int):
    for task in fake_tasks_db:
        if task["task_id"] == task_id:
            return task
    return {}  # позже заменим на HTTPException(404)

# 🟢 Query-параметры: limit, offset, keyword — после знака ?
# Значения по умолчанию делают параметры необязательными
@app.get("/tasks")
async def get_tasks(
    limit: int = 10,              # пагинация: сколько взять
    offset: int = 0,              # пагинация: сколько пропустить
    keyword: str | None = None    # поиск: необязательный фильтр
):
    result = fake_tasks_db
    
    # Фильтрация по ключевому слову (если передано)
    if keyword is not None:
        result = [
            task for task in result
            if keyword.lower() in task["task_name"].lower()
        ]
    
    # Пагинация — срез после фильтрации
    return result[offset : offset + limit]

# Примеры запросов:
# GET /tasks/2                      → задача с id=2
# GET /tasks/hello                  → 422 ошибка (не int)
# GET /tasks?limit=2                → первые 2 задачи
# GET /tasks?offset=2&limit=1       → пропустить 2, взять 1
# GET /tasks?keyword=python         → поиск по слову
# GET /tasks?keyword=БД&limit=5     → фильтр + пагинация
```

**Правило выбора Path vs Query:**

| Параметр | Где | Когда использовать |
|----------|-----|-------------------|
| Path `/users/{id}` | В URL | Идентификатор ресурса (обязательный) |
| Query `?limit=10` | После `?` | Фильтры, сортировка, пагинация (опциональные) |

## 6. Pydantic-схемы, POST-запросы и фильтрация ответов

Pydantic-модели (наследники `BaseModel`) заменяют словари — они автоматически валидируют типы, защищают от опечаток и генерируют документацию. Метод POST принимает данные в теле запроса (Body), а FastAPI сам распознаёт Pydantic-модель как источник данных. Для защиты от утечки секретных полей используется `response_model` или аннотация `-> STask`, а принцип DRY реализуется через наследование схем.

- **BaseModel** — базовый класс Pydantic для описания схемы данных.
- **Field(...)** — тонкая настройка поля: ограничения (`min_length`, `ge`, `le`) и описание для Swagger.
- **`...` (Ellipsis)** — маркер обязательного поля при использовании `Field`.
- **Request Body** — тело POST-запроса, распознаётся по типу Pydantic-модели в аргументе функции.
- **model_dump()** — метод конвертации Pydantic-объекта обратно в словарь (в v1 — `.dict()`).
- **response_model** — параметр декоратора для фильтрации и валидации ответа.
- **Наследование схем** — вынос общих полей в `Base`-класс (принцип DRY).

```python
# schemas.py — иерархия моделей
from pydantic import BaseModel, Field

# Базовая схема: общие поля
class STaskBase(BaseModel):
    name: str = Field(..., min_length=2, max_length=100, description="Название задачи")
    description: str | None = Field(None, max_length=300)

# Для создания (наследует базу, ничего не добавляем)
class STaskAdd(STaskBase):
    pass

# Для ответа (добавляем id, который присваивает сервер)
class STask(STaskBase):
    id: int

# main.py — эндпоинты
from fastapi import FastAPI

app = FastAPI()
tasks: list[dict] = []

# POST: тело запроса автоматически валидируется по STaskAdd
@app.post("/tasks", response_model=STask)  # или -> STask
async def create_task(task: STaskAdd):
    task_dict = task.model_dump()          # объект → словарь
    task_dict["id"] = len(tasks) + 1       # сервер присваивает ID
    tasks.append(task_dict)
    return task_dict                       # response_model отфильтрует лишнее

# GET: возвращаем список по схеме STask
@app.get("/tasks", response_model=list[STask])
async def get_tasks():
    return tasks
```

**Ключевые правила:**
- `name: str` — обязательное поле
- `name: str = "default"` — необязательное со значением по умолчанию
- `name: str | None = None` — nullable (может быть `null`)
- `name: str = Field(..., min_length=2)` — обязательное с валидацией
- Если имя аргумента **в пути `{}`** → Path, **нет в пути** → Query, **тип — Pydantic-модель** → Body.

## 7. HTTP-статусы и модуль status

HTTP-статусы — это трёхзначные коды, которыми сервер сообщает клиенту результат запроса. Они делятся на классы: 2xx (успех), 4xx (ошибка клиента), 5xx (ошибка сервера). В FastAPI статус успешного ответа задаётся параметром `status_code` в декораторе, а вместо «магических чисел» рекомендуется использовать именованные константы из модуля `fastapi.status`.

- **2xx (200, 201, 204)** — успешные ответы: OK, Created, No Content.
- **4xx (400, 401, 403, 404, 422)** — ошибки клиента: плохой запрос, нет авторизации, доступ запрещён, не найдено, ошибка валидации.
- **5xx (500, 502, 503)** — ошибки сервера: внутренняя поломка, перегрузка.
- **status_code** — параметр декоратора, задающий код успешного ответа.
- **HTTPException** — исключение для ручного возврата ошибок с нужным статусом.
- **Магические числа** — «голые» цифры в коде, которые трудно понять без контекста.

```python
from fastapi import FastAPI, status, HTTPException

app = FastAPI()
tasks: list[dict] = []

# POST: успешное создание → 201 Created
# Тело запроса — обычный словарь (без Pydantic-схемы)
@app.post("/tasks", status_code=status.HTTP_201_CREATED)
async def create_task(task: dict):
    task["id"] = len(tasks) + 1
    tasks.append(task)
    return task

# GET: поиск задачи с обработкой ошибки 404
@app.get("/tasks/{task_id}")
async def get_task(task_id: int):
    for task in tasks:
        if task["id"] == task_id:
            return task
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="Задача не найдена"
    )

# DELETE: успешное удаление → 204 No Content (без тела ответа)
@app.delete("/tasks/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_task(task_id: int):
    for i, task in enumerate(tasks):
        if task["id"] == task_id:
            del tasks[i]
            return None
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="Задача не найдена"
    )
```

**Шпаргалка по статусам:**

| Код | Константа | Когда использовать |
|-----|-----------|-------------------|
| 200 | `HTTP_200_OK` | GET, PUT, PATCH — по умолчанию |
| 201 | `HTTP_201_CREATED` | POST — создание ресурса |
| 204 | `HTTP_204_NO_CONTENT` | DELETE — удаление без ответа |
| 400 | `HTTP_400_BAD_REQUEST` | Логическая ошибка в данных |
| 401 | `HTTP_401_UNAUTHORIZED` | Не авторизован (нет токена) |
| 403 | `HTTP_403_FORBIDDEN` | Доступ запрещён (мало прав) |
| 404 | `HTTP_404_NOT_FOUND` | Ресурс не найден |
| 422 | `HTTP_422_UNPROCESSABLE_ENTITY` | Ошибка валидации Pydantic (автоматически) |
| 500 | `HTTP_500_INTERNAL_SERVER_ERROR` | Внутренняя ошибка сервера |

## 8. Обработка ошибок (HTTPException)

Ошибки нельзя возвращать через `return` (это даст статус 200 OK). Для прерывания выполнения и возврата корректного HTTP-статуса используется `raise HTTPException`. FastAPI автоматически формирует JSON-ответ со стандартным ключом `detail`.

- **`raise`** — оператор Python, мгновенно останавливающий функцию.
- **`HTTPException`** — класс FastAPI для возврата HTTP-ошибки с кодом и сообщением.
- **`detail`** — стандартный ключ в JSON-ответе ошибки, содержащий описание проблемы.

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()
tasks = [{"id": 1, "name": "Тест"}]

@app.get("/tasks/{task_id}")
async def get_task(task_id: int):
    for task in tasks:
        if task["id"] == task_id:
            return task  # ✅ Успех: вернёт задачу и статус 200
    
    # ❌ Прерываем выполнение, возвращаем 404
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail=f"Задача с ID {task_id} не найдена"
    )
```

## 9. PUT vs PATCH, метод DELETE и статус 204

Метод **PUT** предполагает полную замену ресурса (все поля обязательны, недостающие затираются), тогда как **PATCH** используется для точечного, частичного обновления только переданных полей. Метод **DELETE** удаляет ресурс по идентификатору в URL без тела запроса. Стандартным успешным ответом для удаления является статус **204 No Content**, который подразумевает пустое тело ответа. GET и DELETE являются идемпотентными операциями (повторный вызов не меняет результат), в отличие от POST.

- **PUT** — полная перезапись объекта. Если не передать поле, оно будет удалено/обнулено.
- **PATCH** — частичное обновление. Меняются только те поля, которые пришли в запросе.
- **Идемпотентность** — свойство операции, при котором многократное выполнение дает тот же результат, что и однократное (GET, DELETE, PUT — идемпотентны; POST — нет).
- **Soft Delete** — «мягкое» удаление: запись не стирается физически, а помечается флагом (например, `is_deleted=True`).
- **204 No Content** — успешный HTTP-статус, означающий, что сервер выполнил запрос, но возвращать данные нечего (пустой Body).

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()
tasks = [{"id": 1, "name": "Купить хлеб", "is_done": False}]

# 💡 PUT vs PATCH (концепция):
# При PUT {"name": "Молоко"} поле is_done затрется (станет null).
# При PATCH {"name": "Молоко"} поле is_done останется False.

# ✅ DELETE: Удаление ресурса по ID из пути
@app.delete("/tasks/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_task(task_id: int):
    for index, task in enumerate(tasks):
        if task["id"] == task_id:
            tasks.pop(index)  # Физическое удаление из списка
            return None       # Возврат None обеспечивает пустое тело ответа (204)
    
    # Если цикл завершился, а совпадений нет
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail=f"Задача с ID {task_id} не найдена"
    )
```

> **Важно:** При статусе `204` тело ответа должно быть пустым. Если вернуть словарь `{"msg": "ok"}`, некоторые клиенты могут проигнорировать его или выдать ошибку парсинга, так как стандарт HTTP запрещает Body для 204.

## 10. Декомпозиция проекта — APIRouter и пакеты

При росте проекта один `main.py` превращается в «спагетти-код». FastAPI решает это через `APIRouter` — «мини-приложение», которое группирует эндпоинты с общим префиксом и тегами. Файлы раскладываются по папкам-пакетам (`routers/`, `schemas/`), помеченным `__init__.py`. Импорты становятся точечными (`from schemas.task import ...`), а архитектура строится по принципу однонаправленных зависимостей: **схемы → роутеры → main**.

- **APIRouter** — класс для группировки эндпоинтов в отдельном файле (аналог `FastAPI`, но без запуска сервера).
- **prefix** — общий префикс путей роутера (например, `/tasks`), приклеивается ко всем его эндпоинтам.
- **tags** — список тегов для группировки эндпоинтов в Swagger UI.
- **include_router()** — метод `app`, подключающий роутер к основному приложению.
- **Пакет (Package)** — папка с файлом `__init__.py`, которую Python распознаёт как модуль для импорта.
- **`__init__.py`** — маркер пакета (обычно пустой файл). Без него импорт из папки невозможен.
- **Абсолютный импорт** — путь от корня проекта: `from routers.task import router`.
- **Круговая зависимость (Circular Import)** — ошибка, когда два файла импортируют друг друга. Решается разделением на уровни.

```
my_fastapi_project/
├── main.py                 # Точка входа
├── routers/
│   ├── __init__.py         # Пустой файл-маркер
│   └── task.py             # Роутер задач
└── schemas/
    ├── __init__.py         # Пустой файл-маркер
    └── task.py             # Pydantic-схемы
```

```python
# ===== schemas/task.py =====
# Нижний уровень: знает только о Pydantic, ни о чём больше
from pydantic import BaseModel

class STaskAdd(BaseModel):
    name: str
    description: str | None = None

class STask(STaskAdd):
    id: int


# ===== routers/task.py =====
# Средний уровень: знает о схемах, но НЕ знает о main
from fastapi import APIRouter, HTTPException, status
from schemas.task import STaskAdd, STask   # ← абсолютный импорт

router = APIRouter(prefix="/tasks", tags=["Задачи"])
tasks: list[dict] = []

@router.post("", response_model=STask, status_code=status.HTTP_201_CREATED)
async def create_task(task: STaskAdd):
    task_dict = task.model_dump()
    task_dict["id"] = len(tasks) + 1
    tasks.append(task_dict)
    return task_dict

@router.get("/{task_id}", response_model=STask)
async def get_task(task_id: int):
    for task in tasks:
        if task["id"] == task_id:
            return task
    raise HTTPException(status_code=404, detail="Задача не найдена")


# ===== main.py =====
# Верхний уровень: знает обо всех, но никто не знает о нём
from fastapi import FastAPI
from routers.task import router as tasks_router   # ← абсолютный импорт

app = FastAPI(title="Task Manager")
app.include_router(tasks_router)   # «втыкаем удлинитель в розетку»
```

**Правила архитектуры:**

| Уровень | Что содержит | Что импортирует |
|---------|--------------|-----------------|
| `schemas/` | Pydantic-модели | Ничего из проекта |
| `routers/` | Эндпоинты, бизнес-логика | Только `schemas/` |
| `main.py` | Сборка приложения | Только `routers/` |

> ⚠️ **Главное правило:** импорты идут строго **снизу вверх**. Никогда не импортируйте `app` из `main.py` в роутер — это вызовет круговую зависимость.
