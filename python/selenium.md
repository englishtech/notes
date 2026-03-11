# Selenium
## Установка

```bash
pip install selenium webdriver-manager
```

## Быстрый старт
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

options = webdriver.ChromeOptions()                         # Создаёт объект для настройки браузера
options.add_argument("--headless=new")                      # Запуск без окна (без графики)
options.add_argument("--no-sandbox")                        # Отключает проверки безопасности (нужно на VPS)
options.add_argument("--disable-gpu")                       # Отключает графический процессор (экономит ресурсы)
options.add_argument("--disable-dev-shm-usage")             # Использует /tmp вместо /dev/shm (для VPS)
options.add_argument("--disable-infobars")                  # Скрывает предупреждение "автоматизированный браузер"
options.add_argument("--log-level=0")                       # Самый низкий уровень логирования
# options.add_argument("--incognito")                       # Режим инкогнито
options.add_argument("--disable-application-cache")         # Отключает кэш приложения
options.add_argument("--disk-cache-size=0")                 # Отключает дисковый кэш
options.add_argument("--media-cache-size=0")                # Отключает медиа-кэш
options.add_argument(                                       # Подменяет User-Agent
    "--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36")
options.binary_location = r"C:\Program Files\BraveSoftware\Brave-Browser\Application\brave.exe"
# options.page_load_strategy = 'eager'                      # Ждёт только готовности DOM (без картинок и ресурсов), быстрее
service = Service(ChromeDriverManager().install())          # Скачивает/находит в кэше chromedriver, создаёт сервис

# для Google Chrome на VPS:
# options.binary_location = "/usr/bin/google-chrome"
# service = Service(executable_path="/usr/lib/chromium-browser/chromedriver")

browser = webdriver.Chrome(service=service, options=options)  # Запускает браузер с настройками
# Чтобы браузер сам закрывался, можно через менеджер контекста:
# with webdriver.Chrome(service=service, options=chrome_options) as browser:

browser.get(URL)                                              # Открывает страницу по адресу URL (str)

### Основной код ###

browser.quit()                                                # Закрывает браузер
```
## Основные методы поиска

```python
from selenium.webdriver.common.by import By

# Один элемент
element = driver.find_element(By.ID, "username")

# Все элементы (возвращает список)
elements = driver.find_elements(By.CLASS_NAME, "item")
```
## Селекторы

| Селектор | Синтаксис | Пример в коде | Пример в DOM |
|----------|-----------|---------------|--------------|
| `By.ID` | `#id` | `find_element(By.ID, "header")` | `<div id="header">...</div>` |
| `By.CLASS_NAME` | `.class` | `find_element(By.CLASS_NAME, "btn")` | `<button class="btn">...</button>` |
| `By.TAG_NAME` | `tag` | `find_element(By.TAG_NAME, "a")` | `<a href="...">...</a>` |
| `By.CSS_SELECTOR` | CSS | `find_element(By.CSS_SELECTOR, "div.item")` | `<div class="item">...</div>` |

## CSS-селекторы (частые случаи)
```python
# По классу
".btn-primary"

# По тегу и классу
"div.item"

# По атрибуту
"[type='submit']"
"[href='/home']"

# Дочерний элемент
"ul li"           # любой потомок
"ul > li"         # прямой потомок

# Псевдоклассы
"p:nth-of-type(2)"   # второй <p>
"input:first-child"  # первый дочерний
```
## Работа с элементами

| Метод | Назначение | Пример |
|-------|------------|--------|
| `.click()` | Клик по элементу | `button.click()` |
| `.text` | Текст элемента | `print(el.text)` |
| `.get_attribute("attr")` | Значение атрибута | `el.get_attribute("href")` |
| `.send_keys("text")` | Ввод текста | `input.send_keys("hello")` |
| `.is_displayed()` | Видим ли элемент | `el.is_displayed()` |
| `.is_enabled()` | Доступен ли элемент | `el.is_enabled()` |

## Примеры
```python
# Клик по кнопке
driver.find_element(By.ID, "submit").click()

# Получить текст
title = driver.find_element(By.TAG_NAME, "h1").text
print(title)

# Получить ссылку
link = driver.find_element(By.CSS_SELECTOR, "a.main").get_attribute("href")

# Ввести текст
driver.find_element(By.ID, "search").send_keys("python")
```

----
## Установка Chrome на VPS:
Удалить Chromium из Snap
```bash
sudo snap remove chromium
```

Установить Google Chrome одной командой
```bash
sudo apt update && sudo apt install -y wget gnupg
wget -qO- https://dl.google.com/linux/linux_signing_key.pub | sudo gpg --dearmor -o /usr/share/keyrings/google-chrome-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-keyring.gpg] http://dl.google.com/linux/chrome/deb stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
sudo apt update
sudo apt install -y google-chrome-stable
```

После этого браузер будет тут: `/usr/bin/google-chrome`

## Установка совместимого chromedriver под Google Chrome (версии 145.0.7632.159):

1. Скачать chromedriver
```bash
wget -O /tmp/chromedriver.zip https://storage.googleapis.com/chrome-for-testing-public/145.0.7632.159/linux64/chromedriver-linux64.zip
```
2. Распаковать
```bash
unzip /tmp/chromedriver.zip -d /tmp
```
3. Установить в систему
```bash
sudo mv /tmp/chromedriver-linux64/chromedriver /usr/bin/
```
4. Сделать исполняемым
```bash
sudo chmod +x /usr/local/chromedriver
```
5. Проверить
```bash
/usr/local/chromedriver --version
```

