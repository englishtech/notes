# Создание базы данных SQLite

- Установить и импортировать библиотеку:
```python
import sqlite3
```
- Лучше создать отдельную функцию для подключения к БД, например:
```python
def get_db_connection():
    conn = sqlite3.connect(DB_PATH)   # Файл базы данных
    conn.row_factory = sqlite3.Row    # Позволяет обращаться к столбцам по имени (row['data'], а не row[0]
    return conn
```
> Если файл базы данных будет находится в другой папке, путь можно указать, например:  
> `DB_PATH = Path(__file__).parent.parent / "database" / "mydatabase.db"`


## Создание файла БД
```python
def init_db(): 
    """ 
    Аннотация
    """
    with get_db_connection() as conn:                              # Открываем соединение с БД (автоматически закроется)
        cursor = conn.cursor()                                     # Создаём курсор для выполнения SQL-запросов
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS название (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL,
                data TEXT NOT NULL,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        conn.commit()                                              # Сохраняем изменения в базе данных
```
> SQL-запрос создает таблицу, если ее еще нет.  
> Столбец id: целое число, первичный ключ, автоинкремент  
> Столбец user_id: целое число, обязательное поле  
> Столбец data: текст, обязательное поле  
> Столбец updated_at: время, по умолчанию текущее
> 
> conn.commit() - нужно при создании БД. При дальнейшем сохранении/чтении данных контекстный менеджер `with ...` делает commit сам.
> conn.close()  - не нужно в контекстном менеджере.

 
## Пример функции записи данных в БД SQLite
```python
def add_record(user_id, data):
    """
    Записывает данные в таблицу
    
    Аргументы:
        user_id: ID пользователя
        data: данные для сохранения (строка)
    """
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute(
            'INSERT INTO название (user_id, data) VALUES (?, ?)',
            (user_id, data)
        )
        conn.commit()
```

## Пример функции чтения данных из БД SQLite
```python
def get_records(user_id):
    """
    Читает все записи пользователя из таблицы
    
    Аргументы:
        user_id: ID пользователя
    
    Возвращает:
        Список кортежей с записями
    """
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute(
            'SELECT * FROM название WHERE user_id = ?',
            (user_id,)
        )
        return cursor.fetchall()
```


## Объекты и методы
| Элемент | Описание | Пример |
|---------|----------|--------|
| `conn` | Соединение с базой данных | `conn = sqlite3.connect('db.db')` |
| `cursor` | Объект для выполнения SQL-запросов | `cursor = conn.cursor()` |
| `cursor.execute(sql)` | Выполнить один запрос | `cursor.execute('SELECT * FROM table')` |
| `cursor.fetchall()` | Получить все строки результата в виде списка кортежей `[(1, 'John'), (2, 'Jane')]` | `rows = cursor.fetchall()` |
| `cursor.fetchone()` | Получить одну строку результата в виде кортежа `(1, 'John')` или `None` | `row = cursor.fetchone()` |
| `cursor.fetchmany(n)` | Получить `n` строк в виде списка кортежей `[(1, 'John'), (2, 'Jane')]` | `rows = cursor.fetchmany(n)` |
| `conn.commit()` | Сохранить изменения в БД | `conn.commit()` |
| `conn.close()` | Закрыть соединение | `conn.close()` |

---


## SQL — основные команды
| Команда | Описание | Пример |
|---------|----------|--------|
| `SELECT` | Выбрать данные | `SELECT name FROM users` |
| `FROM` | Из какой таблицы | `SELECT * FROM users` |
| `WHERE` | Условие фильтрации | `SELECT * FROM users WHERE id = 1` |
| `INSERT INTO` | Добавить запись | `INSERT INTO users (name) VALUES ('John')` |
| `UPDATE` | Обновить запись | `UPDATE users SET name = 'Jane' WHERE id = 1` |
| `DELETE FROM` | Удалить запись | `DELETE FROM users WHERE id = 1` |
| `CREATE TABLE` | Создать таблицу | `CREATE TABLE users (id INTEGER, name TEXT)` |

---


## Вопросительные знаки `?`
**Зачем нужны:**
- Защита от SQL-инъекций
- Автоматическое экранирование значений  
**Как использовать:**
```python
# ❌ Плохо (уязвимо к инъекциям)
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# ✅ Хорошо (безопасно)
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
```


## Примеры запросов
```python
# Чтение всех записей
cursor.execute("SELECT * FROM users")
all_users = cursor.fetchall()

# Чтение одной записи
cursor.execute("SELECT * FROM users WHERE id = ?", (1,))
user = cursor.fetchone()

# Получить все значения столбца data для пользователя
cursor.execute("SELECT data FROM название WHERE user_id = ?", (user_id,))
# Каждая строка — кортеж (значение,), извлекаем первое значение
return [row[0] for row in cursor.fetchall()]

# Вставка записи
cursor.execute(
    "INSERT INTO users (name, age) VALUES (?, ?)",
    ("Alice", 30)
)
conn.commit()

# Обновление записи
cursor.execute(
    "UPDATE users SET age = ? WHERE name = ?",
    (31, "Alice")
)
conn.commit()

# Удаление записи
cursor.execute("DELETE FROM users WHERE id = ?", (1,))
conn.commit()
```
