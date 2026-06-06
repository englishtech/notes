# 📚 Модуль `string` в Python

Модуль `string` содержит полезные константы и классы для работы с текстом. Не требует установки — входит в стандартную библиотеку Python.

```python
import string
```

## 🔤 Константы

| Константа | Значение | Пример |
|-----------|----------|--------|
| `string.ascii_letters` | Латинские буквы (верхний + нижний регистр) | `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ` |
| `string.ascii_lowercase` | Строчные латинские | `abcdefghijklmnopqrstuvwxyz` |
| `string.ascii_uppercase` | Заглавные латинские | `ABCDEFGHIJKLMNOPQRSTUVWXYZ` |
| `string.digits` | Цифры 0-9 | `0123456789` |
| `string.hexdigits` | Шестнадцатеричные цифры | `0123456789abcdefABCDEF` |
| `string.octdigits` | Восьмеричные цифры | `01234567` |
| `string.punctuation` | Знаки препинания | `!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~` |
| `string.whitespace` | Пробельные символы | пробел, `\t`, `\n`, `\r`, `\v`, `\f` |
| `string.printable` | Все печатные символы | digits + letters + punctuation + whitespace |

## 🇷🇺 Кириллица (нет в модуле — добавляем вручную)

```python
RU_LETTERS = "абвгдеёжзийклмнопрстуфхцчшщъыьэюяАБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ"
```

## 💡 Практические примеры

### 1. Проверка символов через `set` (быстро)
```python
import string

valid_chars = set(string.ascii_letters + string.digits)
text = "Hello123!"
clean = [c for c in text if c in valid_chars]  # ['H','e','l','l','o','1','2','3']
```

### 2. Генерация случайного пароля
```python
import random, string

alphabet = string.ascii_letters + string.digits + string.punctuation
password = ''.join(random.choices(alphabet, k=12))
# Пример: 'aB3$kL9#mP@x'
```

### 3. Удаление пунктуации из текста
```python
text = "Привет, мир! Как дела?"
clean = text.translate(str.maketrans('', '', string.punctuation))
# 'Привет мир Как дела'
```

### 4. Подсчёт категорий символов
```python
text = "Hello, World! 123"

letters = sum(1 for c in text if c in string.ascii_letters)  # 10
digits  = sum(1 for c in text if c in string.digits)         # 3
spaces  = sum(1 for c in text if c in string.whitespace)     # 4
punct   = sum(1 for c in text if c in string.punctuation)    # 2
```

### 5. Транслитерация / замена символов
```python
# maketrans создаёт таблицу замен
table = str.maketrans(
    string.ascii_lowercase,
    string.ascii_uppercase
)
"hello".translate(table)  # 'HELLO'
```

## ⚠️ Чего в модуле НЕТ

- ❌ Кириллицы (русских букв) — нужно объявлять вручную
- ❌ Функций обработки строк — модуль содержит только **константы** и класс `Template`
- ❌ Методов `.upper()`, `.split()` — это методы самого `str`, а не `string`

## 🎯 Класс `string.Template`

Простой шаблонизатор для подстановки переменных:

```python
from string import Template

t = Template("Привет, $name! Тебе $age лет.")
result = t.substitute(name="Алекс", age=30)
# 'Привет, Алекс! Тебе 30 лет.'

# Безопасная подстановка (не падает на пропущенных)
t.safe_substitute(name="Алекс")
# 'Привет, Алекс! Тебе $age лет.'
```

---

> 💡 **Совет:** для проверок символов всегда оборачивать константы в `set()` — это даёт проверку за O(1) вместо O(n).
