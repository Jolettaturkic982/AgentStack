# MCP — обзор и архитектура

- [Обзор](#обзор)
- [Архитектура](#архитектура)
- [API Endpoints](#api-endpoints)

**Другие разделы:** [Быстрый старт](MCP_QUICKSTART.md) · [Справочник tools](MCP_TOOLS.md) · [Особенности и примеры](MCP_FEATURES_EXAMPLES.md)

---

## Обзор

**MCP (Microservice Control Plane)** - это универсальный сервер для управления всеми сервисами AgentStack через единый интерфейс. MCP предоставляет AI агентам (Google Studio, Cursor, и другим) полный доступ к функциональности платформы через стандартизированный набор инструментов.

### Основные характеристики

- **Версия протокола**: 2.0
- **Всего доступных tools**: 62+
- **Поддержка streaming**: Да
- **Idempotency**: Да
- **Job tracking**: Да
- **Порт**: 8000 (через API Gateway)

### Ключевые возможности

- ✅ Полное управление проектами (включая анонимное создание)
- ✅ Аутентификация и авторизация
- ✅ Управление платежами
- ✅ Планировщик задач
- ✅ Аналитика и метрики
- ✅ Управление API ключами
- ✅ Rules Engine (серверная логика)
- ✅ Webhooks
- ✅ Уведомления
- ✅ Кошельки
- ✅ Buff System (временные и постоянные эффекты)

---

## Архитектура

### Структура проекта

```
mcp/
├── main.py              # FastAPI приложение, middleware, регистрация
├── routes.py            # API endpoints для tools
├── tools.py             # Реализация всех MCP tools
├── sdk_wrapper.py       # SDK обертка для работы с проектами
└── dependencies.py      # Зависимости
```

### Компоненты

1. **FastAPI Application** (`main.py`)
   - CORS middleware
   - Metrics middleware
   - Correlation ID tracking
   - Health checks
   - Prometheus metrics

2. **Routes** (`routes.py`)
   - `GET /mcp/tools` - список всех доступных tools
   - `GET /mcp/discovery` - discovery endpoint для клиентов
   - `POST /mcp/tools/{tool_name}` - выполнение tool
   - `POST /mcp/tools/{tool_name}/stream` - streaming выполнение
   - `GET /mcp/jobs/{job_id}` - статус задачи

3. **Tools Registry** (`tools.py`)
   - Все доступные tools зарегистрированы в `MCP_TOOLS`
   - Pydantic модели для валидации запросов
   - Обработка ошибок и логирование

4. **SDK Wrapper** (`sdk_wrapper.py`)
   - Обертка над HTTP API для работы с проектами
   - Использует существующие endpoints из `agentstack-core`
   - Единообразный интерфейс для всех операций

---

## API Endpoints

**Публичные эндпоинты (без X-API-Key):** вызовы без заголовка аутентификации допускаются для `GET /mcp/tools` (список tools) и для вызова инструмента `projects.create_project_anonymous` (создание анонимного проекта и получение ключей). Все остальные запросы требуют `X-API-Key`.

### Discovery и метаданные

#### `GET /mcp/tools`
Возвращает список всех доступных tools, сгруппированных по категориям. **Может вызываться без X-API-Key.**

**Ответ:**
```json
{
  "tools": {
    "auth": [...],
    "payments": [...],
    "projects": [...],
    ...
  },
  "version": "2.0",
  "total_tools": 60,
  "description": "Full AgentStack MCP integration..."
}
```

#### `GET /mcp/discovery`
Discovery endpoint для автоматического обнаружения возможностей.

**Ответ:**
```json
{
  "protocol_version": "2.0",
  "capabilities": {
    "streaming": true,
    "idempotency": true,
    "job_tracking": true
  },
  "services": {
    "auth": {...},
    "projects": {...},
    ...
  }
}
```

### Выполнение tools

#### `POST /mcp/tools/{tool_name}`
Выполняет указанный tool.

**Запрос:**
```json
{
  "tool": "projects.create_project",
  "params": {
    "name": "My Project",
    "description": "Project description"
  },
  "idempotency_key": "optional-key"
}
```

**Ответ:**
```json
{
  "success": true,
  "data": {...},
  "error": null,
  "trace_id": "uuid"
}
```

#### `POST /mcp/tools/{tool_name}/stream`
Выполняет tool с streaming ответом (Server-Sent Events).

**Ответ:** SSE stream с событиями:
- `status: started`
- `status: processing`
- `status: completed`
- `status: error`

#### `GET /mcp/jobs/{job_id}`
Получает статус асинхронной задачи.

---