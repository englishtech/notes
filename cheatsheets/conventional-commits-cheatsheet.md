# Conventional Commits Cheatsheet

Стандарт описания коммитов для автоматической генерации CHANGELOG, семантического версионирования и удобной истории проекта.

## 📋 Формат

```
<type>(<scope>): <subject>

<body>

<footer>
```

- **type** — тип изменения (обязательно)
- **scope** — область/модуль (опционально)
- **subject** — краткое описание (обязательно)
- **body** — детальное описание (опционально)
- **footer** — метаданные (опционально)

---

## ️ Типы коммитов

| Тип | Описание | Пример |
|-----|----------|--------|
| **`feat`** | Новая функциональность | `feat(auth): add OAuth2 login` |
| **`fix`** | Исправление бага | `fix(api): resolve null pointer in /users` |
| **`docs`** | Изменения документации | `docs(readme): update installation guide` |
| **`style`** | Форматирование, пробелы, отступы (не логика!) | `style(css): fix indentation in main.scss` |
| **`refactor`** | Реструктуризация кода без смены поведения | `refactor(parser): extract validation logic` |
| **`perf`** | Улучшение производительности | `perf(query): add index on created_at` |
| **`test`** | Добавление/изменение тестов | `test(auth): add unit tests for JWT` |
| **`chore`** | Вспомогательные изменения (зависимости, конфиги) | `chore(deps): bump requests to 2.32.0` |
| **`build`** | Изменения системы сборки | `build(webpack): update config for production` |
| **`ci`** | Изменения CI/CD | `ci(github): add workflow for tests` |
| **`revert`** | Откат предыдущего коммита | `revert: "feat(search): add fuzzy matching"` |

> **feat** — добавление новой логики, функции, утилиты или возможности.  
> **fix** — только если функция чинит конкретный баг (например, "устраняет падение при пустом JSON").  
> **refactor** — если функция просто вынесла повторяющийся код без изменения внешнего поведения.
---

## 📝 Правила оформления

### 1. Subject (заголовок)
- ✅ **Императив**: `add`, `fix`, `update` (не `added`, `fixing`)
- ✅ **Без точки** в конце
- ✅ **Первая буква заглавная** (опционально, но рекомендуется)
- ✅ **≤ 72 символа** (чтобы влезало в `git log --oneline`)

**Примеры:**
```
✅ feat: add user profile page
❌ feat: added user profile page.
❌ feat: Adding user profile page
```

### 2. Scope (область)
Указывает на модуль/компонент:
- `feat(api): ...` — бэкенд API
- `fix(ui): ...` — фронтенд интерфейс
- `docs(readme): ...` — документация
- `test(auth): ...` — тесты модуля авторизации

### 3. Body (тело)
- Отделяется **пустой строкой** от заголовка
- Объясняет **почему**, а не **что** (что уже видно из кода)
- Перенос строк на **72 символе**

### 4. Footer (подвал)
- Ссылки на issues: `Closes #123`, `Fixes #456`
- Breaking changes: `BREAKING CHANGE: API endpoint /v1/users removed`

---

## 🎯 Примеры

### Простой коммит
```bash
git commit -m "feat: add dark mode toggle"
```

### С областью
```bash
git commit -m "fix(parser): handle empty CSV rows"
```

### С телом и футером
```bash
git commit -m "feat(news): implement sentiment analysis

Add VADER-based sentiment scoring for each news article.
Scores range from -1 (negative) to +1 (positive).

Closes #42
```

### Breaking Change
```bash
git commit -m "refactor(api): migrate to v2 endpoints

BREAKING CHANGE: /api/v1/users endpoint removed. 
Use /api/v2/users instead.

Migration guide: https://wiki.example.com/v2-migration
```

---

## 🔧 Сленг и сокращения

| Сокращение | Значение | Когда использовать |
|------------|----------|-------------------|
| **WIP** | Work In Progress | Черновой коммит (не для main) |
| **HOTFIX** | Срочный фикс | Критический баг в проде |
| **TEMP** | Temporary | Временное решение (удалить позже) |
| **FIXME** | Нужно исправить | Костыль с техдолгом |
| **XXX** | Внимание! | Потенциальная проблема |

**Примеры:**
```bash
git commit -m "WIP: draft of new dashboard"
git commit -m "HOTFIX: resolve payment gateway timeout"
git commit -m "TEMP: disable rate limiting for testing"
```

---

## ⚡ Быстрые команды

```bash
# Фича
git commit -m "feat(module): add new feature"

# Фикс бага
git commit -m "fix(module): resolve issue #123"

# Документация
git commit -m "docs(readme): update examples"

# Рефакторинг
git commit -m "refactor(core): simplify validation logic"

# Тесты
git commit -m "test(auth): add edge cases"

# Зависимости
git commit -m "chore(deps): upgrade django to 4.2"
```

---

## ❌ Антипаттерны

```bash
❌ git commit -m "fix bug"           # Слишком общо
❌ git commit -m "wip"               # Не описывает что сделано
❌ git commit -m "Update file.txt"   # Бессмысленно
❌ git commit -m "asdfasdf"          # Просто нет
❌ git commit -m "Fixed bug in parser module and added tests"  # Слишком длинно

✅ git commit -m "fix(parser): handle null values in CSV"
✅ git commit -m "test(parser): add null value edge cases"
```

---

## 🔗 Ресурсы

- [Conventional Commits Spec](https://www.conventionalcommits.org)
- [Commitizen](https://commitizen-tools.github.io/commitizen/)
- [Angular Commit Guidelines](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit)
- [Keep a Changelog](https://keepachangelog.com)
