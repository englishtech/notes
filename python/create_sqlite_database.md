## Создание базы данных SQLite

- Лучше создать отдельную функцию для подключения к БД, например:
```python
def get_db_connection():
    conn = sqlite3.connect(DB_PATH)   # Файл базы данных
    conn.row_factory = sqlite3.Row    # Позволяет обращаться к столбцам по имени (row['data'], а не row[0]
    return conn
```
> Если файл базы данных будет находится в другой папке, путь можно указать, например:  
> `DB_PATH = Path(__file__).parent.parent / "database" / "mydatabase.db"`  
- Создать файла БД:
```python
def init_db():
    """
    Аннотация
    """
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS название (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL,
                data TEXT NOT NULL,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        conn.commit()
```
