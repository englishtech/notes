# `io`
Библиотека io — это стандартная библиотека Python для работы с потоками данных.

BytesIO — это объект, который ведёт себя как файл, но хранится в оперативной памяти, а не на диске. 
Он позволяет работать с бинарными данными так, будто это обычный файл, но без необходимости создавать файл на диске. 
Это удобно, когда нужно создать файл в памяти и сразу передать его, например, для скачивания через веб-интерфейс или отправки по сети.
## Основные классы

| Класс | Описание | Пример данных |
|-------|----------|---------------|
| `BytesIO` | Буфер для бинарных данных (bytes) | `b"текст"` |
| `StringIO` | Буфер для текстовых данных (str) | `"текст"` |

---

## Основные методы

| Метод | Описание | Пример |
|-------|----------|--------|
| `.write(data)` | Записать данные в буфер | `buffer.write(b"привет")` |
| `.read()` | Прочитать все данные | `buffer.read()` |
| `.seek(pos)` | Переместить указатель | `buffer.seek(0)` |
| `.tell()` | Текущая позиция указателя | `buffer.tell()` |
| `.getvalue()` | Получить всё содержимое | `buffer.getvalue()` |
| `.truncate()` | Обрезать буфер | `buffer.truncate()` |
| `.close()` | Закрыть буфер | `buffer.close()` |

---

## BytesIO — работа с бинарными данными

```python
from io import BytesIO

# Создание
buffer = BytesIO()

# Запись
buffer.write(b"Hello, World!")
buffer.write(bytes([1, 2, 3]))

# Переместить указатель в начало
buffer.seek(0)

# Чтение
content = buffer.read()  # b'Hello, World!\x01\x02\x03'

# Получить значение
data = buffer.getvalue()

# Закрыть
buffer.close()
```

---

## StringIO — работа с текстом

```python
from io import StringIO

# Создание
buffer = StringIO()

# Запись
buffer.write("Привет, ")
buffer.write("мир!")

# Переместить в начало
buffer.seek(0)

# Чтение
text = buffer.read()  # "Привет, мир!"

# Получить значение
content = buffer.getvalue()

# Закрыть
buffer.close()
```

---

## Контекстный менеджер

```python
from io import BytesIO

with BytesIO() as buffer:
    buffer.write(b"Данные")
    buffer.seek(0)
    print(buffer.read())
# Автоматически закрывается
```

---

## Практические примеры

### 1. Создать файл в памяти для скачивания

```python
from io import BytesIO

buffer = BytesIO()
buffer.write(b"Данные для файла")
buffer.seek(0)

# Streamlit
st.download_button("Скачать", data=buffer, file_name="file.txt")
```

---

### 2. Сохранить изображение в память

```python
from io import BytesIO
from PIL import Image

img = Image.new('RGB', (100, 100), color='red')
buffer = BytesIO()
img.save(buffer, format='PNG')
buffer.seek(0)
```

---

### 3. Работа с текстом как с файлом

```python
from io import StringIO
import csv

buffer = StringIO()
writer = csv.writer(buffer)
writer.writerow(['Имя', 'Возраст'])
writer.writerow(['Анна', 25])

buffer.seek(0)
print(buffer.read())
```

---

## Важно помнить:

1. **`seek(0)`** — перемещает указатель в начало перед чтением
2. **`getvalue()`** — возвращает всё содержимое без изменения позиции указателя
3. **Бинарные данные** → `BytesIO`
4. **Текст** → `StringIO`
5. Можно использовать как обычный файл (`.read()`, `.write()`, `.close()`)
