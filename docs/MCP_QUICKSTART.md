# MCP — быстрый старт и настройка

- [Быстрый старт](#быстрый-старт)
- [Настройка в Cursor](#настройка-в-cursor)
- [Отладка и примеры](#отладка-подключения)

**Другие разделы:** [Обзор и архитектура](MCP_OVERVIEW.md) · [Справочник tools](MCP_TOOLS.md) · [Особенности и примеры](MCP_FEATURES_EXAMPLES.md)

---

## Быстрый старт

MCP доступен в облачной экосистеме **[agentstack.tech](https://agentstack.tech)** — локальный запуск сервера не требуется.

1. **Получите API ключ:** зарегистрируйтесь на [agentstack.tech](https://agentstack.tech) и создайте проект в дашборде, либо создайте анонимный проект одним запросом (см. ниже).
2. **Настройте плагин** (Cursor, Claude, VS Code или GPT): укажите MCP URL `https://agentstack.tech/mcp` и ваш API ключ. Готово — 60+ tools доступны в чате.

**Проверка доступности:**
```bash
# Список доступных tools (без ключа)
curl https://agentstack.tech/mcp/tools
```

---

## Настройка в Cursor

### Способ 1: HTTP MCP Server (рекомендуется)

Cursor поддерживает MCP серверы через HTTP. Настройте его следующим образом:

#### Шаг 1: Создайте конфигурационный файл

Создайте файл `cursor-mcp-config.json` в домашней директории или в настройках Cursor:

**Windows:**
```
%APPDATA%\Cursor\User\globalStorage\mcp-config.json
```

**macOS:**
```
~/Library/Application Support/Cursor/User/globalStorage/mcp-config.json
```

**Linux:**
```
~/.config/Cursor/User/globalStorage/mcp-config.json
```

#### Шаг 2: Добавьте конфигурацию MCP сервера

```json
{
  "mcpServers": {
    "agentstack": {
      "type": "http",
      "baseUrl": "https://agentstack.tech/mcp",
      "headers": {
        "X-API-Key": "your-api-key-here"
      },
      "tools": {
        "enabled": true,
        "autoDiscover": true
      }
    }
  }
}
```

#### Шаг 3: Перезапустите Cursor

После добавления конфигурации перезапустите Cursor, чтобы изменения вступили в силу.

### Способ 2: Через настройки Cursor UI

1. Откройте Cursor
2. Перейдите в **Settings** → **Features** → **MCP Servers**
3. Нажмите **Add Server**
4. Заполните форму:
   - **Name**: `agentstack`
   - **Type**: `HTTP`
   - **Base URL**: `https://agentstack.tech/mcp`
   - **API Key**: ваш API ключ (если требуется)
5. Сохраните настройки

### Способ 3: Использование через Chat (без настройки)

Если MCP сервер запущен, вы можете использовать его напрямую через HTTP API в чате Cursor:

```
Создай проект через MCP:
POST https://agentstack.tech/mcp/tools/projects.create_project_anonymous
Body: {"tool": "projects.create_project_anonymous", "params": {"name": "Test Project"}}
```

### Получение API ключа

Если требуется API ключ для аутентификации:

1. **Создайте анонимный проект** (получите API ключ):
```bash
curl -X POST https://agentstack.tech/mcp/tools/projects.create_project_anonymous \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "projects.create_project_anonymous",
    "params": {
      "name": "My Project"
    }
  }'
```

2. **Используйте полученный `api_key`** в конфигурации Cursor

### Использование в Cursor Chat

После настройки вы можете использовать MCP tools прямо в чате Cursor:

#### Пример 1: Создание проекта

```
Создай новый проект через MCP с названием "My Test Project"
```

Cursor автоматически вызовет `projects.create_project_anonymous` с параметрами.

#### Пример 2: Получение списка проектов

```
Покажи все мои проекты через MCP
```

#### Пример 3: Управление пользователями

```
Добавь пользователя user@example.com в проект 1025 с ролью member
```

**Примечание:** Для `add_user` и `remove_user` требуется Professional подписка.

### Отладка подключения

Если MCP сервер не работает в Cursor:

1. **Проверьте доступность MCP:**
```bash
curl https://agentstack.tech/mcp/tools
```

2. **Проверьте логи Cursor:**
   - Откройте Developer Tools (Ctrl+Shift+I / Cmd+Option+I)
   - Перейдите в Console
   - Ищите ошибки подключения к MCP

3. **Проверьте конфигурацию:**
   - Убедитесь, что `baseUrl` = `https://agentstack.tech/mcp`
   - Убедитесь, что API ключ указан верно (без лишних пробелов)

### Примеры команд для Cursor

#### Создание и настройка проекта

```
Создай анонимный проект "My AI Project" через MCP и покажи API ключ
```

#### Работа с пользователями

```
Покажи всех пользователей проекта 1025 через MCP
```

#### Управление API ключами

```
Создай новый API ключ для проекта 1025 с именем "Development Key"
```

#### Получение статистики

```
Покажи статистику проекта 1025 через MCP
```

#### Просмотр активности

```
Покажи последние 10 событий активности проекта 1025
```

### Альтернатива: Использование через HTTP клиент

Если настройка MCP в Cursor не работает, вы можете использовать HTTP клиент прямо в коде:

```python
import httpx

async def create_project_via_mcp(name: str):
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "https://agentstack.tech/mcp/tools/projects.create_project_anonymous",
            json={
                "tool": "projects.create_project_anonymous",
                "params": {"name": name}
            }
        )
        return response.json()
```

MCP доступен по адресу **https://agentstack.tech/mcp**. Дополнительные переменные окружения не требуются.

### Практические примеры работы в Cursor

#### Сценарий 1: Создание проекта для тестирования

**Запрос в Cursor:**
```
Создай новый тестовый проект через MCP с названием "Test Project" и опиши, что получилось
```

**Что происходит:**
1. Cursor вызывает `projects.create_project_anonymous`
2. Получает `project_id`, `api_key`, `auth_key`
3. Показывает результат в чате

**Ответ Cursor:**
```
✅ Проект успешно создан!

Project ID: 1025
API Key: ask_abc123xyz...
Auth Key: ask_abc123xyz...

Теперь вы можете использовать этот API ключ для работы с проектом.
```

#### Сценарий 2: Прикрепление проекта к пользователю

**Запрос в Cursor:**
```
Прикрепи проект 1025 к пользователю с email user@example.com. Используй auth_key: ask_abc123xyz...
```

**Что происходит:**
1. Cursor вызывает `projects.attach_to_user`
2. Проект переходит в собственность пользователя
3. Создается новый API ключ для владельца

#### Сценарий 3: Получение информации о проекте

**Запрос в Cursor:**
```
Покажи информацию о проекте 1025: статистику, пользователей, настройки
```

**Что происходит:**
1. Cursor вызывает несколько tools:
   - `projects.get_project` - основная информация
   - `projects.get_stats` - статистика
   - `projects.get_users` - список пользователей
   - `projects.get_settings` - настройки
2. Агрегирует результаты и показывает в удобном виде

#### Сценарий 4: Управление API ключами

**Запрос в Cursor:**
```
Создай новый API ключ для проекта 1025 с именем "Production Key" и покажи его
```

**Что происходит:**
1. Cursor вызывает `projects.create_api_key`
2. Получает новый ключ
3. Показывает его (⚠️ важно сохранить, т.к. показывается только один раз)

#### Сценарий 5: Просмотр активности

**Запрос в Cursor:**
```
Покажи последние 20 событий активности проекта 1025
```

**Что происходит:**
1. Cursor вызывает `projects.get_activity` с `limit=20`
2. Показывает список событий с временными метками

### Советы по использованию

1. **Всегда сохраняйте API ключи** - они показываются только один раз при создании
2. **Используйте анонимное создание** для быстрого тестирования
3. **Прикрепляйте проекты** к реальным пользователям для production
4. **Проверяйте подписку** перед добавлением пользователей (требуется Professional)
5. **Используйте trace_id** для отладки проблем

### Troubleshooting

#### Проблема: "Tool not found"

**Решение:**
- Убедитесь, что MCP сервер запущен
- Проверьте, что tool name правильный (с точкой: `projects.create_project`)
- Проверьте конфигурацию в Cursor

#### Проблема: "Connection refused"

**Решение:**
- Проверьте, что сервер запущен на порту 8000
- Проверьте firewall настройки
- Убедитесь, что `baseUrl` в конфигурации правильный

#### Проблема: "403 Forbidden" при добавлении пользователя

**Решение:**
- Это нормально - требуется Professional подписка
- Обновите подписку пользователя или используйте другой метод

#### Проблема: "Invalid auth_key" при прикреплении

**Решение:**
- Убедитесь, что используете правильный `auth_key` из анонимного создания
- Проверьте, что проект еще не прикреплен к другому пользователю

---