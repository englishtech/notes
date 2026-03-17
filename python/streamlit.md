# Streamlit — создание веб-интерфейсов на Python

**Streamlit** — библиотека для быстрого создания веб-приложений на Python без знания веб-разработки.

### Принцип работы:

| Аспект | Описание |
|--------|----------|
| **Где работает** | В браузере по адресу `http://localhost:8501` |
| **Как запускается** | `streamlit run app.py` — запускает локальный сервер |
| **Перезагрузка страницы** | Весь скрипт выполняется заново (сверху вниз) |
| **Состояние виджетов** | Хранится в `st.session_state` |
| **Hot reload** | При изменении кода страница обновляется автоматически (нужно включить) |


## Установка и запуск
```bash
# Установка
pip install streamlit

# Запуск
streamlit run app.py

# Запуск на другом порту
streamlit run app.py --server.port 8502
```
> Стабильная версия `streamlit==1.40.1, pyarrow==14.0.1`

## Виджеты
| Виджет | Описание | Пример |
|--------|----------|--------|
| `st.title()` | Заголовок страницы | `st.title("Заголовок")` |
| `st.header()` | Подзаголовок | `st.header("Подзаголовок")` |
| `st.write()` | Вывод текста/данных | `st.write("Текст")` |
| `st.text_input()` | Поле ввода текста | `name = st.text_input("Имя")` |
| `st.number_input()` | Числовое поле | `age = st.number_input("Возраст", min_value=0)` |
| `st.slider()` | Ползунок | `val = st.slider("Значение", 0, 100)` |
| `st.selectbox()` | Выпадающий список | `opt = st.selectbox("Выбор", ["А", "Б"])` |
| `st.multiselect()` | Множественный выбор | `tags = st.multiselect("Теги", ["1", "2"])` |
| `st.checkbox()` | Чекбокс | `checked = st.checkbox("Согласен")` |
| `st.radio()` | Радиокнопки | `choice = st.radio("Выбор", ["А", "Б"])` |
| `st.button()` | Кнопка | `if st.button("Нажми"): ...` |
| `st.file_uploader()` | Загрузка файла | `file = st.file_uploader("Файл")` |
| `st.dataframe()` | Таблица данных | `st.dataframe(df)` |
| `st.table()` | Статическая таблица | `st.table(df)` |
| `st.image()` | Изображение | `st.image("photo.jpg")` |
| `st.success()` | Успешное сообщение | `st.success("Готово!")` |
| `st.error()` | Сообщение об ошибке | `st.error("Ошибка!")` |
| `st.warning()` | Предупреждение | `st.warning("Внимание!")` |
| `st.info()` | Информационное сообщение | `st.info("Инфо")` |

---

## Session State
**`st.session_state`** — словарь для хранения данных между перезагрузками страницы.

| Без Session State | С использованием Session State |
|-------------------|-------------------------------|
| Все переменные сбрасываются при каждом взаимодействии, streamlit заново читает весь код python | Данные сохраняются между повторным чтением кода, например, при любом пользовательском вводе |
| Нельзя хранить состояние приложения | Можно сохранять счётчики, списки, настройки |

### Инициализация
```python
import streamlit as st

# Проверка и инициализация
if 'counter' not in st.session_state:
    st.session_state.counter = 0

# Использование
st.session_state.counter += 1
st.write(f"Счётчик: {st.session_state.counter}")
```
### Сохранение списка данных

```python
if 'items' not in st.session_state:
    st.session_state.items = []

new_item = st.text_input("Добавить:")
if st.button("Добавить") and new_item:
    st.session_state.items.append(new_item)

st.write("Список:")
for item in st.session_state.items:
    st.write(f"- {item}")
```

### Полезные методы
| Метод | Описание | Пример |
|-------|----------|--------|
| `get()` | Получить значение или значение по умолчанию | `st.session_state.get('count', 0)` |
| `keys()` | Получить все ключи | `st.session_state.keys()` |
| `items()` | Получить пары ключ-значение | `st.session_state.items()` |
| `clear()` | Очистить всё состояние | `st.session_state.clear()` |

---

## Forms
**Форма** — группа виджетов, которые отправляются **одновременно** при нажатии кнопки `form_submit_button`.
| Без формы | С формой |
|-----------|----------|
| Каждое изменение виджета вызывает перезапуск скрипта | Скрипт перезапускается только при нажатии кнопки |
| Много лишних обновлений | Одно обновление после заполнения всех полей |

### Синтаксис
```python
import streamlit as st

st.title("Регистрация")

with st.form("registration_form"):
    name = st.text_input("Имя")
    email = st.text_input("Email")
    age = st.number_input("Возраст", min_value=0, max_value=120)
    subscribe = st.checkbox("Подписаться на рассылку")
    
    submit = st.form_submit_button("Зарегистрироваться")

if submit:
    st.success(f"Привет, {name}! Регистрация успешна.")
    st.write(f"Email: {email}")
    st.write(f"Возраст: {age}")
    st.write(f"Подписка: {'Да' if subscribe else 'Нет'}")
```
### Комбинация с `session_state`:
```python
with st.form("login_form"):
    username = st.text_input("Логин")
    password = st.text_input("Пароль", type="password")
    submit = st.form_submit_button("Войти")

if submit:
    if username == "admin" and password == "123":
        st.session_state.logged_in = True
        st.success("Вход выполнен!")
    else:
        st.error("Неверный логин или пароль")
```
