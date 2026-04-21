## Создание git в проекте (Python)
Зайти в папку проекта (`cd папка_проекта` в bash или `Open Folder` в VSCode).
Находясь в папке проекта в терминале ввести:
```bash
git init
```
В папке будет создан `.git`  
Создать файл `.gitignore`, например:
```ini
.env
*.pyc
.venv/
имя_папки/__pycache__/
имя_папки_с_логами/*.log
```
> Слэши в unix-стиле: `/`

Создать файл `requirements.txt` со всеми зависимостями проекта: 
```bash
pip freeze > requirements.txt
```
Создать репозиторий на github.com (https://github.com/new)  
Связать локальный и удаленный репозиторий. Подключение к github.com уже должно быть настроено. Для подключения по ssh:
```bash
git branch -M main
git remote add origin git@github.com:пользователь_github/название_репозитория.git
```
Добавить файлы в индекс и commit:
```bash
git add .
git commit -m "комментарий"
```
Отправить на GitHub:
```bash
git push -u origin main
```
> Затем можно просто `git push`

Удалить файлы из индекса, но оставить на диске:
```bash
git rm -r cached папка/файл
```
