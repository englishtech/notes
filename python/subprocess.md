# `subprocess`

Модуль `subprocess` позволяет запускать внешние программы и скрипты из Python. Это как открыть терминал и написать команду, но автоматически, с возможностью перехватить результат и обработать ошибки.

**Аналогия:** Python — это менеджер. Он может дать задачу сотруднику (внешней программе), подождать результат, проверить, всё ли сделано правильно, и получить отчёт (вывод программы).

---

## Основной метод: `subprocess.run()`

Это самый простой и рекомендуемый способ. Запускает команду, ждёт завершения, возвращает результат.

```python
import subprocess

# Запуск простой команды
result = subprocess.run(["python", "script.py"])
```

### Ключевые параметры

| Параметр | Что делает | Пример |
|----------|-----------|--------|
| `args` | Список команды и аргументов | `["python", "script.py", "--arg"]` |
| `capture_output=True` | Перехватить stdout и stderr | `result.stdout`, `result.stderr` |
| `text=True` | Вернуть строки вместо байтов | `result.stdout` будет `str`, не `bytes` |
| `check=True` | Выбросить исключение, если код ≠ 0 | `subprocess.CalledProcessError` |
| `timeout=N` | Убить процесс через N секунд | `subprocess.TimeoutExpired` |
| `cwd` | Рабочая директория | `cwd="/path/to/dir"` |

---

## Базовые примеры

### Пример 1: Запуск и проверка результата

```python
import subprocess

# Запускаем Python-скрипт
result = subprocess.run(["python", "hello.py"])

# Проверяем код возврата (0 = успех)
if result.returncode == 0:
    print("Скрипт выполнен успешно")
else:
    print(f"Ошибка: код {result.returncode}")
```

**Пояснение:** `returncode` — это число, которое программа возвращает при завершении. 0 означает успех, любое другое число — ошибка.

---

### Пример 2: Перехват вывода программы

```python
import subprocess

# capture_output=True перехватывает stdout и stderr
# text=True возвращает строки, а не байты
result = subprocess.run(
    ["python", "script.py"],
    capture_output=True,
    text=True
)

print("Вывод программы:")
print(result.stdout)

if result.stderr:
    print("Ошибки:")
    print(result.stderr)
```

**Пояснение:** Без `text=True` вывод будет в байтах (`b'Hello\n'`). С `text=True` — обычная строка (`'Hello\n'`).

---

### Пример 3: Обработка ошибок

```python
import subprocess

try:
    # check=True выбросит исключение, если скрипт упал
    result = subprocess.run(
        ["python", "bad_script.py"],
        capture_output=True,
        text=True,
        check=True
    )
    print("Успех!")
    
except subprocess.CalledProcessError as e:
    # e.returncode — код ошибки
    # e.stderr — текст ошибки из программы
    print(f"Скрипт упал с кодом {e.returncode}")
    print(f"Причина: {e.stderr}")
```

**Пояснение:** Без `check=True` программа просто вернёт `returncode ≠ 0`, и нужно проверять его вручную. С `check=True` исключение выбрасывается автоматически.

---

### Пример 4: Таймаут (защита от зависания)

```python
import subprocess

try:
    # Если скрипт не завершится за 5 секунд — убить его
    result = subprocess.run(
        ["python", "slow_script.py"],
        timeout=5,
        capture_output=True,
        text=True
    )
    
except subprocess.TimeoutExpired:
    print("Скрипт завис и был убит через 5 секунд")
```

**Пояснение:** Полезно для сетевых запросов или тяжёлых вычислений, которые могут зависнуть навсегда.

---

### Пример 5: Передача данных в программу

```python
import subprocess

# Передаём текст в stdin программы
result = subprocess.run(
    ["python", "process_input.py"],
    input="Привет, мир!",
    capture_output=True,
    text=True
)

print(result.stdout)  # Вывод программы, обработавшей наш текст
```

**Пояснение:** Параметр `input` передаёт данные в стандартный ввод (stdin) программы. Работает только с `text=True` или если передать байты.

---

## Продвинутый уровень: `subprocess.Popen`

Если нужен полный контроль (асинхронность, потоковый вывод), используйте `Popen`.

```python
import subprocess

# Запускаем процесс, не ждём завершения
process = subprocess.Popen(
    ["python", "long_task.py"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True
)

# Делаем что-то параллельно...
print("Процесс запущен, работаем дальше...")

# Ждём завершения и получаем результат
stdout, stderr = process.communicate()

print(f"Код возврата: {process.returncode}")
print(f"Вывод: {stdout}")
```

**Пояснение:** `Popen` запускает процесс и сразу возвращает управление. `communicate()` ждёт завершения и собирает весь вывод.

---

## Частые ошибки

### Ошибка 1: Забыть `text=True`
```python
# ❌ Вывод в байтах: b'Hello\n'
result = subprocess.run(["echo", "Hello"], capture_output=True)

# ✅ Вывод в строках: 'Hello\n'
result = subprocess.run(["echo", "Hello"], capture_output=True, text=True)
```

### Ошибка 2: Не проверить `returncode`
```python
# ❌ Скрипт упал, но мы этого не знаем
result = subprocess.run(["python", "bad.py"])

# ✅ Проверяем результат
result = subprocess.run(["python", "bad.py"], check=True)  # Выбросит исключение
```

### Ошибка 3: Передать строку вместо списка
```python
# ❌ Ошибка: shell=True опасен и медленен
subprocess.run("python script.py --arg value")

# ✅ Правильно: список аргументов
subprocess.run(["python", "script.py", "--arg", "value"])
```

---

## Краткая шпаргалка

```python
# Базовый запуск
subprocess.run(["команда", "аргумент"])

# Перехват вывода
subprocess.run([...], capture_output=True, text=True)

# Обработка ошибок
subprocess.run([...], check=True)

# Таймаут
subprocess.run([...], timeout=10)

# Передача данных
subprocess.run([...], input="текст", text=True)
```

---

## Когда что использовать

| Задача | Решение |
|--------|---------|
| Простой запуск скрипта | `subprocess.run()` |
| Нужен вывод программы | `capture_output=True, text=True` |
| Обработка ошибок | `check=True` + `try/except` |
| Защита от зависания | `timeout=N` |
| Параллельная работа | `subprocess.Popen` |
| Передать данные в программу | `input="текст"` |
