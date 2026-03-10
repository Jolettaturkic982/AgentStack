# Работа с данными экосистемы AgentStack

**См. также:** [MCP и экосистема — индекс](MCP_AND_ECOSYSTEM.md), [DNA Key-Value API](architecture/DNA_KEY_VALUE_API.md), [примеры комплексных проектов](examples/mcp_complex_projects.md).

В экосистеме AgentStack **нет версий в путях API** (используются пути вида `/api/...`). Хранение и чтение данных проектов и пользователей реализовано через существующие эндпоинты и MCP tools — отдельный «Ecosystem API» не используется.

---

## Где реализована работа с данными

| Задача | Эндпоинт или инструмент | Описание |
|--------|-------------------------|----------|
| Данные проекта (чтение/запись по пути) | `GET /api/projects/{project_id}/data`<br>`PATCH /api/projects/{project_id}/data` | Опционально `?path=...` для GET; body `{ "path": "...", "value": ... }` для PATCH. Требуется аутентификация и доступ к проекту. |
| Данные текущего пользователя в проекте | `GET /api/projects/{project_id}/users/me/data`<br>`PATCH /api/projects/{project_id}/users/me/data` | То же формат path/value. Данные хранятся в `data_projects_user` для пары project_id + user_id. |
| Key-value (project.data.* / user.data.*) | `GET /api/dna/data`<br>`POST /api/dna/data` | Query: `key` (обязательно), `project_id` (для `project.data.*`). Body: `{ "key", "value", "project_id" }`. См. [DNA_KEY_VALUE_API.md](architecture/DNA_KEY_VALUE_API.md). |
| Проект целиком (включая data) через MCP | `projects.get_project`, `projects.update_project` | Поле `data` в запросе/ответе. Удобно для агентов и плагинов. См. [MCP_SERVER_CAPABILITIES.md](MCP_SERVER_CAPABILITIES.md). |

Базовый URL для облака: **https://agentstack.tech**. Аутентификация: сессия (REST) или заголовок `X-API-Key` (REST/MCP).

---

## Быстрый старт

1. **Получить API ключ**
   - Через дашборд [agentstack.tech](https://agentstack.tech): создать проект и взять API ключ.
   - Либо без регистрации: один запрос к MCP — `projects.create_project_anonymous` с именем проекта; в ответе будут `user_api_key`, `project_id`. См. [MCP_SERVER_CAPABILITIES.md](MCP_SERVER_CAPABILITIES.md#projectscreate_project_anonymous).

2. **Читать и писать данные проекта**
   - REST:  
     `GET https://agentstack.tech/api/projects/{project_id}/data` — всё `project.data` или по `?path=ключ`.  
     `PATCH https://agentstack.tech/api/projects/{project_id}/data` с телом `{ "path": "ключ", "value": <значение> }`.
   - Либо key-value:  
     `GET https://agentstack.tech/api/dna/data?key=project.data.ключ&project_id={project_id}`  
     `POST https://agentstack.tech/api/dna/data` с телом `{ "key": "project.data.ключ", "value": <значение>, "project_id": <id> }`.
   - MCP: `projects.get_project`, `projects.update_project` (поле `data`).

3. **Данные пользователя в проекте**
   - REST:  
     `GET /api/projects/{project_id}/users/me/data` (опционально `?path=...`),  
     `PATCH /api/projects/{project_id}/users/me/data` с `{ "path": "...", "value": ... }`.
   - Key-value: ключ `user.data.<path>`, `GET/POST /api/dna/data` (см. [DNA_KEY_VALUE_API.md](architecture/DNA_KEY_VALUE_API.md)).

---

## Пример: хранение данных мобильной игры

Сохраняем прогресс игрока (уровень, монеты, инвентарь) в данных текущего пользователя в проекте — в `user.data.game.progress`.

### Вариант 1: REST (path/value)

**Запись прогресса:**
```http
PATCH https://agentstack.tech/api/projects/{project_id}/users/me/data
Authorization: Bearer <token>
Content-Type: application/json

{
  "path": "game.progress",
  "value": {
    "level": 5,
    "coins": 1000,
    "inventory": ["sword", "potion"]
  }
}
```

**Чтение прогресса:**
```http
GET https://agentstack.tech/api/projects/{project_id}/users/me/data?path=game.progress
Authorization: Bearer <token>
```

### Вариант 2: DNA Key-Value API

**Запись:**
```http
POST https://agentstack.tech/api/dna/data
Authorization: Bearer <token>
Content-Type: application/json

{
  "key": "user.data.game.progress",
  "value": { "level": 5, "coins": 1000, "inventory": ["sword", "potion"] },
  "project_id": 1025
}
```

**Чтение:**
```http
GET https://agentstack.tech/api/dna/data?key=user.data.game.progress&project_id=1025
Authorization: Bearer <token>
```

### Вариант 3: MCP (для агентов и плагинов)

Использовать tools для работы с проектом и данными пользователя по [MCP_SERVER_CAPABILITIES.md](MCP_SERVER_CAPABILITIES.md) (например, получение/обновление проекта с полем `data` или вызовы, которые записывают в user.data).

---

## Rules / Logic

Правила и логика (when/then, триггеры) доступны через API Logic Engine: префикс **`/api/logic`**. Список и создание правил, выполнение — см. документацию по Logic Engine и [CONTEXT_FOR_AI.md](plugins/CONTEXT_FOR_AI.md) (домен Rules Engine → `logic.*`).

---

**Версия:** 0.2 — гайд по реальным эндпоинтам, без версионирования в путях.  
**Источник правды по путям:** код `agentstack-core` (projects_endpoints, dna_api_endpoints, core_app).
