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
# Один элемент
element = driver.find_element(By.ID, "username")

# Все элементы (возвращает список)
elements = driver.find_elements(By.CLASS_NAME, "item")
```




## самый короткий и надёжный способ установить Chrome на VPS:
# Удали Chromium из Snap
sudo snap remove chromium

# Установи Google Chrome одной командой
sudo apt update && sudo apt install -y wget gnupg
wget -qO- https://dl.google.com/linux/linux_signing_key.pub | sudo gpg --dearmor -o /usr/share/keyrings/google-chrome-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-keyring.gpg] http://dl.google.com/linux/chrome/deb stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
sudo apt update
sudo apt install -y google-chrome-stable

После этого браузер будет тут:
/usr/bin/google-chrome


Вот простая последовательность команд для установки совместимого chromedriver под Google Chrome 145.0.7632.159:

# 1. Скачать chromedriver
wget -O /tmp/chromedriver.zip https://storage.googleapis.com/chrome-for-testing-public/145.0.7632.159/linux64/chromedriver-linux64.zip

# 2. Распаковать
unzip /tmp/chromedriver.zip -d /tmp

# 3. Установить в систему
sudo mv /tmp/chromedriver-linux64/chromedriver /usr/bin/

# 4. Сделать исполняемым
sudo chmod +x /usr/local/chromedriver

# 5. Проверить
/usr/local/chromedriver --version

