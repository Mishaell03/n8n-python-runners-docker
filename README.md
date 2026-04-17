# n8n + External Runners (Python/JS)

Этот проект поднимает:
- `n8n` (`n8nio/n8n:1.121.0`)
- `n8n-runners` (кастомный образ на базе `n8nio/runners:1.121.0`)

Python runner работает во внешнем контейнере `runners`.

## Быстрый старт

Требования:
- Docker
- Docker Compose (plugin `docker compose`)

Запуск:

```bash
docker compose up -d --build
```

Проверить статус:

```bash
docker compose ps
```

Открыть n8n:
- http://localhost:5678

Логи:

```bash
docker compose logs -f n8n runners
```

Остановка:

```bash
docker compose down
```

Остановка с удалением volume (полный сброс данных n8n):

```bash
docker compose down -v
```

## Где добавлять Python-библиотеки

Чтобы библиотека работала в Python Code node, нужно сделать **2 шага**.

1. Установить пакет в образ runner

Файл: `runners/Dockerfile`

Сейчас там:

```dockerfile
RUN cd /opt/runners/task-runner-python && uv pip install numpy pandas requests
```

Добавляйте свои пакеты в эту команду, например:

```dockerfile
RUN cd /opt/runners/task-runner-python && uv pip install numpy pandas requests python-dateutil openpyxl
```

2. Разрешить импорт пакета в runner-конфиге

Файл: `runners/n8n-task-runners.json`

Сейчас в блоке `python -> env-overrides`:

```json
"N8N_RUNNERS_EXTERNAL_ALLOW": "numpy,pandas,requests"
```

Добавьте те же пакеты в список, например:

```json
"N8N_RUNNERS_EXTERNAL_ALLOW": "numpy,pandas,requests,python_dateutil,openpyxl"
```

Важно:
- После изменения зависимостей/конфига нужен ребилд контейнера:

```bash
docker compose up -d --build runners
```

или полностью:

```bash
docker compose up -d --build
```

## Полезные замечания

- Обязательно замените дефолтный токен `super-secret-token` в `docker-compose.yml`:
  - `- N8N_RUNNERS_AUTH_TOKEN=super-secret-token`
  - Поставьте свой длинный случайный токен и используйте **одно и то же** значение в секциях `n8n` и `runners`.
- Токен связи между сервисами задаётся переменной `N8N_RUNNERS_AUTH_TOKEN` и должен совпадать в `n8n` и `runners`.
- Часовой пояс сейчас задан как `Europe/Berlin` в `docker-compose.yml`. Если нужен другой (например `Europe/Moscow`), поменяйте `TZ` и `GENERIC_TIMEZONE` в обоих сервисах.
- Данные n8n хранятся в volume `n8n_data`.
