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

