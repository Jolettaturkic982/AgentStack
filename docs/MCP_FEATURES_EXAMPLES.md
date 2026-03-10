# MCP — особенности реализации и примеры

- [Особенности реализации](#особенности-реализации)
- [Примеры использования](#примеры-использования)
- [Итоговая статистика](#итоговая-статистика)

**Другие разделы:** [Быстрый старт](MCP_QUICKSTART.md) · [Обзор](MCP_OVERVIEW.md) · [Справочник tools](MCP_TOOLS.md)

---

## Особенности реализации

### 1. Анонимное создание проектов

**Проблема:** AI агенты (Google Studio, Cursor) не могут создавать проекты без предварительной регистрации пользователя.

**Решение:** `projects.create_project_anonymous`

- ✅ Не требует авторизации
- ✅ **Автоматически создает анонимного пользователя** с `is_anonymous: true`
- ✅ Генерирует API ключ с префиксом `anon_ask_` (ANONYMOUS tier)
- ✅ Возвращает `user_api_key`, `project_api_key`, `session_token`, `user_id`
- ✅ Пользователь сразу аутентифицирован и готов к использованию
- ✅ Позволяет сразу начать работу с проектом

**⚠️ Ограничения ANONYMOUS тарифа:**
- 1 проект максимум (нельзя создавать дополнительные)
- 1 API ключ (нельзя создавать дополнительные)
- 1,000 вызовов Logic Engine/месяц (вместо 10,000 у FREE)
- 20 MB JSON хранилища (вместо 100 MB у FREE)
- 20 триггеров (вместо 50 у FREE)
- Платежи отключены

**Конвертация:**
- Используйте `auth.convert_anonymous_user` для конвертации в FREE тариф
- После конвертации все лимиты увеличиваются до FREE tier

**Использование:**
```json
{
  "tool": "projects.create_project_anonymous",
  "params": {
    "name": "Test Project",
    "description": "Created by AI agent"
  }
}
```

### 2. Прикрепление проектов к пользователям

**Проблема:** Анонимные проекты нужно привязать к реальному пользователю.

**Решение:** `projects.attach_to_user`

- ✅ Требует авторизацию (user_id нового владельца)
- ✅ Проверяет `auth_key` из анонимного создания
- ✅ Transfer ownership:
  - Обновляет `project.user_id`
  - Создает записи в `data_projects_user`
  - Удаляет старый анонимный ключ
  - Создает новый ключ для владельца

**Использование:**
```json
{
  "tool": "projects.attach_to_user",
  "params": {
    "project_id": 1025,
    "auth_key": "ask_...",
    "user_id": 123
  }
}
```

### 3. Ограничения по подпискам

**Управление пользователями проектов** (`add_user`, `remove_user`) требует **Professional подписку**.

**Реализация:**
- Проверка на уровне API endpoint (`agentstack-core/endpoints/projects_endpoints.py`)
- Использует `SubscriptionService.get_user_subscription()`
- Проверяет `plan_type` в ['pro', 'professional', 'enterprise']
- Возвращает 403 Forbidden при отсутствии подписки
- Ошибка пробрасывается через MCP без изменений

**Ошибка:**
```json
{
  "success": false,
  "error": "HTTP 403: Professional subscription required for adding/removing project users. Please upgrade your subscription."
}
```

### 4. SDK Wrapper

**Архитектура:**
- `ProjectsSDKWrapper` - обертка над HTTP API
- Использует `shared.clients.http.request` для запросов
- Единообразный интерфейс для всех операций
- Автоматическое построение заголовков (Authorization, X-Project-ID)
- Использует существующие endpoints (не дублирует функциональность)

**Особенности:**
- API ключи управляются через `/api/apikeys/keys` (не дубликаты)
- Настройки извлекаются из `config` проекта
- Активность получается из `/api/projects/{id}/logs`

### 5. Обработка ошибок

**Стратегия:**
- Ошибки от SDK/API пробрасываются напрямую
- HTTP статусы не конвертируются
- Сохранение `trace_id` для отладки
- Использование `httpx.HTTPStatusError` для HTTP ошибок

**Формат ошибки:**
```json
{
  "success": false,
  "error": "HTTP 403: Insufficient permissions",
  "trace_id": "uuid"
}
```

### 6. Валидация запросов

**Pydantic модели:**
- Все новые методы проектов используют Pydantic модели
- Автоматическая валидация параметров
- Type-safe интерфейс

**Пример:**
```python
class CreateProjectRequest(BaseModel):
    name: str
    description: Optional[str] = None
    config: Optional[Dict[str, Any]] = None
    ...
```

### 7. Демо режим

**Поддержка:**
- Проверка `demo_context.is_readonly()`
- Блокировка write операций в read-only режиме
- Проверка capabilities через `DemoCapabilities`

---

## Примеры использования

### Пример 1: Анонимное создание проекта

```bash
curl -X POST https://agentstack.tech/mcp/tools/projects.create_project_anonymous \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "projects.create_project_anonymous",
    "params": {
      "name": "My AI Project",
      "description": "Created by AI agent"
    }
  }'
```

**Ответ:**
```json
{
  "success": true,
  "data": {
    "project_id": 1025,
    "api_key": "ask_abc123...",
    "auth_key": "ask_abc123...",
    "project": {
      "id": 1025,
      "name": "My AI Project",
      ...
    }
  },
  "trace_id": "uuid"
}
```

### Пример 2: Прикрепление проекта к пользователю

```bash
curl -X POST https://agentstack.tech/mcp/tools/projects.attach_to_user \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "tool": "projects.attach_to_user",
    "params": {
      "project_id": 1025,
      "auth_key": "ask_abc123...",
      "user_id": 123
    }
  }'
```

**Ответ:**
```json
{
  "success": true,
  "data": {
    "success": true,
    "project_id": 1025,
    "new_api_key": "ask_xyz789...",
    "message": "Project 1025 successfully attached to user 123"
  },
  "trace_id": "uuid"
}
```

### Пример 3: Получение списка проектов

```bash
curl -X POST https://agentstack.tech/mcp/tools/projects.get_projects \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "tool": "projects.get_projects",
    "params": {
      "is_active": true,
      "limit": 10
    }
  }'
```

### Пример 4: Добавление пользователя (требует Professional)

```bash
curl -X POST https://agentstack.tech/mcp/tools/projects.add_user \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "tool": "projects.add_user",
    "params": {
      "project_id": 1025,
      "email": "user@example.com",
      "role": "member"
    }
  }'
```

**При отсутствии подписки:**
```json
{
  "success": false,
  "error": "HTTP 403: Professional subscription required for adding/removing project users. Please upgrade your subscription.",
  "trace_id": "uuid"
}
```

### Пример 5: Streaming выполнение

```bash
curl -X POST https://agentstack.tech/mcp/tools/projects.create_project/stream \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "tool": "projects.create_project",
    "params": {
      "name": "Streaming Project"
    }
  }'
```

**Ответ (SSE):**
```
data: {"status": "started", "trace_id": "uuid"}

data: {"status": "processing", "progress": 50}

data: {"status": "completed", "result": {...}}
```

---

## Итоговая статистика

### По категориям

- **Auth**: 4 tools
- **Projects**: 8 tools
- **Logic Engine**: 9 tools
- **Processors**: 3 tools
- **Commands**: 2 tools
- **Payments**: 4 tools
- **Scheduler**: 11 tools ⭐ (самая большая категория)
- **Analytics**: 2 tools
- **API Keys**: 3 tools
- **Buffs**: 10 tools
- **Assets**: 4 tools

**Всего: 60+ tools**

### Новые возможности (последнее обновление)

1. ✅ Анонимное создание проектов
2. ✅ Прикрепление проектов к пользователям
3. ✅ Полный набор методов управления проектами
4. ✅ Интеграция с существующими endpoints
5. ✅ Проверка подписки для управления пользователями

---

## Заключение

MCP сервер предоставляет **полный доступ** ко всей функциональности AgentStack через единый интерфейс. Особое внимание уделено:

- **Удобству для AI агентов** (анонимное создание)
- **Гибкости** (прикрепление проектов)
- **Безопасности** (проверка подписок, валидация)
- **Надежности** (обработка ошибок, trace_id)
- **Производительности** (streaming, кэширование)

Все tools используют существующие endpoints, что обеспечивает консистентность и отсутствие дублирования кода.

---

**Версия документа:** 1.0  
**Дата обновления:** 2025-01-29  
**Автор:** AgentStack Development Team
