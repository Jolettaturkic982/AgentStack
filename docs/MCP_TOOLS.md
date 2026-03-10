# MCP — справочник инструментов (Tools)

Полный список MCP tools по категориям. Быстрый старт: [MCP_QUICKSTART.md](MCP_QUICKSTART.md). Обзор и API: [MCP_OVERVIEW.md](MCP_OVERVIEW.md).

---

## Доступные Tools

### 🔐 Auth Tools (4 tools)

#### `auth.quick_auth`
Быстрая аутентификация пользователя.

**Параметры:**
- `email` (string) - Email пользователя
- `password` (string) - Пароль

**Возвращает:** Токен доступа и информацию о пользователе

#### `auth.create_user`
Создание нового пользователя.

**Параметры:**
- `email` (string)
- `password` (string)
- `display_name` (string, optional)

#### `auth.assign_role`
Назначение роли пользователю.

**Параметры:**
- `user_id` (int)
- `role` (string)
- `project_id` (int, optional)

#### `auth.get_profile`
Получение профиля пользователя.

#### `auth.update_profile`
Обновление профиля пользователя.

---

### ⚙️ Logic Engine Tools (9 tools)

#### `logic.create`
Создание правила логики.

#### `logic.update`
Обновление правила логики.

#### `logic.delete`
Удаление правила логики.

#### `logic.get`
Получение информации о правиле.

#### `logic.list`
Список всех правил проекта.

#### `logic.execute`
Выполнение правила логики.

#### `logic.get_processors`
Получение списка доступных процессоров.

#### `logic.get_commands`
Получение списка доступных команд.

#### `logic.flush_batch`
Очистка батч-очереди правил.

---

### 💳 Payments Tools (4 tools)

#### `payments.create_payment`
Создание платежа.

**Параметры:**
- `amount` (float)
- `currency` (string)
- `payment_method` (string)
- `description` (string, optional)

#### `payments.get_status`
Получение статуса платежа.

**Параметры:**
- `payment_id` (string)

#### `payments.refund`
Возврат платежа.

**Параметры:**
- `payment_id` (string)
- `amount` (float, optional) - частичный возврат

#### `payments.list_transactions`
Список транзакций.

**Параметры:**
- `limit` (int, optional)
- `offset` (int, optional)

#### `payments.get_balance`
Получение баланса.

---

### 📁 Projects Tools (8 tools)

#### Основные CRUD операции

##### `projects.get_projects`
Получение списка проектов.

**Параметры:**
- `is_active` (bool, optional)
- `limit` (int, optional)
- `offset` (int, optional)

##### `projects.get_project`
Получение проекта по ID.

**Параметры:**
- `project_id` (int)

##### `projects.create_project`
Создание проекта (требует авторизацию).

**Параметры:**
- `name` (string) - Обязательно
- `description` (string, optional)
- `config` (dict, optional)
- `data` (dict, optional)
- `preset_id` (string, optional)
- `settings` (dict, optional)

##### `projects.create_project_anonymous`
**⭐ НОВОЕ:** Создание проекта без авторизации (для AI агентов).

**Параметры:**
- `name` (string) - Обязательно
- `description` (string, optional)
- `config` (dict, optional)
- `data` (dict, optional)
- `preset_id` (string, optional)
- `settings` (dict, optional)

**Возвращает:**
```json
{
  "project_id": 1025,
  "user_id": 1037,
  "user_api_key": "anon_ask_...",
  "project_api_key": "ask_...",
  "session_token": "...",
  "project": {...},
  "_auth_info": {...}
}
```

**Особенности:**
- Не требует авторизации
- **Автоматически создает анонимного пользователя** с `is_anonymous: true`
- Генерирует API ключ с префиксом `anon_ask_` (ANONYMOUS tier)
- Возвращает `user_api_key`, `project_api_key`, `session_token`, `user_id`
- Пользователь сразу аутентифицирован и готов к использованию

**⚠️ ОГРАНИЧЕНИЯ ANONYMOUS ТАРИФА:**
- **1 проект максимум** - Нельзя создавать дополнительные проекты
- **1 API ключ** - Нельзя создавать дополнительные ключи
- **1,000 вызовов Logic Engine/месяц** (вместо 10,000 у FREE)
- **20 MB JSON хранилища** (вместо 100 MB у FREE)
- **20 триггеров** (вместо 50 у FREE)
- **Платежи отключены** - Нельзя обрабатывать платежи

**Конвертация в полный аккаунт:**
- Используйте `auth.convert_anonymous_user` для конвертации в FREE тариф
- После конвертации все лимиты увеличиваются до FREE tier
- Пользователь может войти с email/password
- Можно создавать дополнительные проекты и API ключи

##### `projects.update_project`
Обновление проекта.

**Параметры:**
- `project_id` (int)
- `name` (string, optional)
- `description` (string, optional)
- `config` (dict, optional)
- `data` (dict, optional)
- `is_active` (bool, optional)

##### `projects.delete_project`
Удаление проекта.

**Параметры:**
- `project_id` (int)

#### Статистика и аналитика

##### `projects.get_stats`
Получение статистики проекта.

**Параметры:**
- `project_id` (int)

**Возвращает:** Статистику по проекту (пользователи, активность, использование)

#### Управление пользователями

##### `projects.get_users`
Получение списка пользователей проекта.

**Параметры:**
- `project_id` (int)
- `is_active` (bool, optional)
- `role` (string, optional)
- `limit` (int, optional)
- `offset` (int, optional)

##### `projects.add_user`
**⭐ ТРЕБУЕТ Professional подписку**

Добавление пользователя в проект.

**Параметры:**
- `project_id` (int)
- `user_id` (int, optional)
- `email` (string, optional)
- `role` (string) - По умолчанию "member"

**Ограничения:**
- Требуется Professional подписка (pro, professional, enterprise)
- Проверка на уровне API endpoint
- Ошибка 403 при отсутствии подписки

##### `projects.remove_user`
**⭐ ТРЕБУЕТ Professional подписку**

Удаление пользователя из проекта.

**Параметры:**
- `project_id` (int)
- `user_id` (int)

##### `projects.update_user_role`
Обновление роли пользователя в проекте.

**Параметры:**
- `project_id` (int)
- `user_id` (int)
- `role` (string)

##### `projects.attach_to_user`
**⭐ НОВОЕ:** Прикрепление анонимного проекта к пользователю.

**Параметры:**
- `project_id` (int)
- `auth_key` (string) - Ключ из анонимного создания
- `user_id` (int, optional)
- `email` (string, optional)

**Процесс:**
1. Проверяет соответствие `auth_key` API ключам проекта
2. Обновляет `project.user_id` на нового владельца
3. Создает записи в `data_projects_user`
4. Удаляет старый анонимный ключ
5. Создает новый API ключ для владельца

**Возвращает:**
```json
{
  "success": true,
  "project_id": 1025,
  "new_api_key": "ask_...",
  "message": "Project successfully attached to user"
}
```

#### Управление API ключами

##### `projects.get_api_keys`
Получение списка API ключей проекта.

**Параметры:**
- `project_id` (int)

**Использует:** `GET /api/apikeys/keys?project_id={project_id}`

##### `projects.create_api_key`
Создание API ключа для проекта.

**Параметры:**
- `project_id` (int)
- `name` (string)
- `scopes` (array, optional)
- `expires_at` (string, optional)

**Использует:** `POST /api/apikeys/keys` с `project_id` в body

##### `projects.delete_api_key`
Удаление API ключа проекта.

**Параметры:**
- `project_id` (int)
- `key_id` (string)

**Использует:** `DELETE /api/apikeys/keys/{key_id}?project_id={project_id}`

#### Настройки проекта

##### `projects.get_settings`
Получение настроек проекта (извлекает `config`).

**Параметры:**
- `project_id` (int)

##### `projects.update_settings`
Обновление настроек проекта (обновляет `config`).

**Параметры:**
- `project_id` (int)
- `settings` (dict)

#### Активность проекта

##### `projects.get_activity`
Получение активности проекта (логи).

**Параметры:**
- `project_id` (int)
- `limit` (int, optional)
- `offset` (int, optional)

**Использует:** `GET /api/projects/{project_id}/logs`

---

### ⏰ Scheduler Tools (11 tools)

#### `scheduler.schedule_task`
Планирование задачи.

**Параметры:**
- `project_id` (int)
- `run_at` (string) - ISO datetime
- `tool` (string)
- `payload` (dict)

#### `scheduler.cancel_task`
Отмена задачи.

**Параметры:**
- `task_id` (string)

#### `scheduler.get_task`
Получение информации о задаче.

**Параметры:**
- `task_id` (string)

#### `scheduler.list_tasks`
Список задач.

**Параметры:**
- `project_id` (int, optional)

#### `scheduler.update_task`
Обновление задачи.

**Параметры:**
- `task_id` (string)
- `task_data` (dict)

---

### 📊 Analytics Tools (2 tools)

#### `analytics.get_usage`
Получение статистики использования.

**Параметры:**
- `project_id` (int, optional)
- `start_date` (string, optional)
- `end_date` (string, optional)

#### `analytics.set_budget`
Установка бюджета.

**Параметры:**
- `project_id` (int)
- `budget` (float)
- `period` (string) - "monthly", "yearly"

#### `analytics.get_metrics`
Получение метрик.

**Параметры:**
- `project_id` (int, optional)
- `metric_type` (string, optional)

#### `analytics.export_data`
Экспорт данных аналитики.

**Параметры:**
- `project_id` (int)
- `format` (string) - "json", "csv"
- `start_date` (string, optional)
- `end_date` (string, optional)

#### `analytics.get_dashboard`
Получение данных для дашборда.

**Параметры:**
- `project_id` (int, optional)

---

### 🔑 API Keys Tools (3 tools)

#### `api_keys.create_key`
Создание API ключа.

**Параметры:**
- `project_id` (int)
- `name` (string)
- `scopes` (array, optional)

#### `api_keys.revoke_key`
Отзыв API ключа.

**Параметры:**
- `key_id` (string)

#### `api_keys.list_keys`
Список API ключей.

**Параметры:**
- `project_id` (int, optional)

#### `api_keys.update_key`
Обновление API ключа.

**Параметры:**
- `key_id` (string)
- `key_data` (dict)

#### `api_keys.get_usage`
Использование API ключа.

**Параметры:**
- `key_id` (string)

---

### ⚙️ Rules Tools (6 tools)

#### `rules.create_rules`
Создание правил.

**Параметры:**
- `project_id` (int)
- `json_rules` (dict)
- `name` (string, optional)
- `description` (string, optional)

#### `rules.test_rules`
Тестирование правил.

**Параметры:**
- `rule_id` (string)
- `test_data` (dict)

#### `rules.activate_rules`
Активация правил.

**Параметры:**
- `rule_id` (string)

#### `rules.list_rules`
Список правил.

**Параметры:**
- `project_id` (int, optional)

#### `rules.update_rules`
Обновление правил.

**Параметры:**
- `rule_id` (string)
- `rule_data` (dict)

#### `rules.delete_rules`
Удаление правил.

**Параметры:**
- `rule_id` (string)

---

### 🔔 Webhooks Tools (5 tools)

#### `webhooks.create_endpoint`
Создание webhook endpoint.

**Параметры:**
- `project_id` (int)
- `url` (string)
- `events` (array)
- `secret` (string, optional)

#### `webhooks.list_endpoints`
Список webhook endpoints.

**Параметры:**
- `project_id` (int, optional)

#### `webhooks.update_endpoint`
Обновление webhook endpoint.

**Параметры:**
- `endpoint_id` (string)
- `endpoint_data` (dict)

#### `webhooks.delete_endpoint`
Удаление webhook endpoint.

**Параметры:**
- `endpoint_id` (string)

#### `webhooks.test_endpoint`
Тестирование webhook endpoint.

**Параметры:**
- `endpoint_id` (string)
- `test_data` (dict, optional)

---

### 📬 Notifications Tools (4 tools)

#### `notifications.send_notification`
Отправка уведомления.

**Параметры:**
- `project_id` (int)
- `user_id` (int, optional)
- `type` (string)
- `message` (string)
- `data` (dict, optional)

#### `notifications.list_notifications`
Список уведомлений.

**Параметры:**
- `project_id` (int)
- `user_id` (int, optional)
- `limit` (int, optional)

#### `notifications.create_template`
Создание шаблона уведомления.

**Параметры:**
- `project_id` (int)
- `name` (string)
- `template` (string)
- `variables` (array, optional)

#### `notifications.update_template`
Обновление шаблона уведомления.

**Параметры:**
- `template_id` (string)
- `template_data` (dict)

---

### 💰 Wallets Tools (4 tools)

#### `wallets.get_balance`
Получение баланса кошелька.

**Параметры:**
- `project_id` (int)
- `wallet_id` (int, optional)

#### `wallets.list_transactions`
Список транзакций кошелька.

**Параметры:**
- `project_id` (int)
- `wallet_id` (int, optional)
- `limit` (int, optional)

#### `wallets.create_wallet`
Создание кошелька.

**Параметры:**
- `project_id` (int)
- `name` (string)
- `type` (string) - "business", "personal"
- `currency` (string)

#### `wallets.update_wallet`
Обновление кошелька.

**Параметры:**
- `wallet_id` (int)
- `wallet_data` (dict)

---

### 🎁 Buff Management Tools (10 tools)

**Что такое бафы?** Бафы (Buffs) - это система временных и постоянных эффектов, которые изменяют лимиты, функции и ресурсы пользователей и проектов. 

**Временные эффекты (автоматически истекают и откатываются):**
- Триальные периоды (7-30 дней) - бесплатные пробные версии функций
- Промо-акции и события - ограниченные по времени предложения
- Временные бонусы - дополнительные ресурсы на короткий срок

**Постоянные эффекты (не истекают, требуют ручного управления):**
- Подписки - ежемесячные/годовые планы с возможностью продления
- Разовые покупки - одноразовые улучшения (например, увеличение лимитов)
- Пакеты улучшений - накопительные постоянные бонусы

Система баффов идеально подходит для управления триальными периодами, промо-акциями, подписками и разовыми покупками в вашем SaaS, игре или приложении.

#### `buffs.create_buff`
Создание шаблона баффа в состоянии PENDING.

**Параметры:**
- `entity_id` (int, required) - ID сущности (user или project)
- `entity_kind` (string, required) - Тип сущности: "user" или "project"
- `name` (string, required) - Название баффа
- `type` (string, required) - Тип баффа (trial, promo, subscription, purchase)
- `category` (string, required) - Категория для фильтрации
- `duration_days` (int, optional, default: 30) - Длительность в днях
- `priority` (int, optional, default: 100) - Приоритет (0-3000)
- `effects` (object, optional) - Эффекты в формате {"data.limits.api_calls": 10000}
- `config` (object, optional) - Конфигурация (persistent, revert_on_expire, etc.)
- `project_id` (int, optional) - ID проекта (для user buffs)

**Пример:**
```json
{
  "tool": "buffs.create_buff",
  "params": {
    "entity_id": 1,
    "entity_kind": "user",
    "name": "7-Day Premium Trial",
    "type": "trial",
    "category": "subscription",
    "duration_days": 7,
    "effects": {
      "data.limits.api_calls": 10000
    }
  }
}
```

#### `buffs.apply_buff`
Применение баффа к сущности (активация PENDING баффа).

**Параметры:**
- `buff_id` (string, required) - UUID баффа
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `project_id` (int, optional) - ID проекта

**Пример:**
```json
{
  "tool": "buffs.apply_buff",
  "params": {
    "buff_id": "uuid-here",
    "entity_id": 123,
    "entity_kind": "user",
    "project_id": 1
  }
}
```

#### `buffs.extend_buff`
Продление активного баффа.

**Параметры:**
- `buff_id` (string, required) - UUID баффа
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `additional_days` (int, optional) - Дополнительные дни (если не указано, продлевает на исходную длительность)
- `project_id` (int, optional) - ID проекта

**Пример:**
```json
{
  "tool": "buffs.extend_buff",
  "params": {
    "buff_id": "uuid-here",
    "entity_id": 123,
    "entity_kind": "user",
    "additional_days": 30
  }
}
```

#### `buffs.revert_buff`
Откат активного баффа с валидацией успешности.

**Параметры:**
- `buff_id` (string, required) - UUID баффа
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `project_id` (int, optional) - ID проекта

**Важно:** Откат проверяет что ресурсы могут быть восстановлены. Если бафф добавил 100 золота, при откате должно быть отнято 100 золота. Если откат невозможен (недостаточно ресурсов), выбрасывается ошибка.

**Пример:**
```json
{
  "tool": "buffs.revert_buff",
  "params": {
    "buff_id": "uuid-here",
    "entity_id": 123,
    "entity_kind": "user",
    "project_id": 1
  }
}
```

#### `buffs.cancel_buff`
Принудительная отмена баффа в любом состоянии.

**Параметры:**
- `buff_id` (string, required) - UUID баффа
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `project_id` (int, optional) - ID проекта

**Важно:** 
- Для PENDING баффов - обычные права
- Для ACTIVE баффов - требуются admin/owner права
- Для ACTIVE баффов сначала выполняется revert, затем удаление

**Пример:**
```json
{
  "tool": "buffs.cancel_buff",
  "params": {
    "buff_id": "uuid-here",
    "entity_id": 123,
    "entity_kind": "user"
  }
}
```

#### `buffs.get_buff`
Получение информации о конкретном баффе.

**Параметры:**
- `buff_id` (string, required) - UUID баффа
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `project_id` (int, optional) - ID проекта

**Возвращает:** Полную информацию о баффе включая состояние, эффекты, конфигурацию, сроки действия.

#### `buffs.list_active_buffs`
Список активных баффов для сущности.

**Параметры:**
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `category` (string, optional) - Фильтр по категории
- `project_id` (int, optional) - ID проекта
- `include_ecosystem` (bool, optional) - Включать экосистемные баффы (для проектов)

**Пример:**
```json
{
  "tool": "buffs.list_active_buffs",
  "params": {
    "entity_id": 123,
    "entity_kind": "user",
    "category": "subscription",
    "project_id": 1
  }
}
```

#### `buffs.get_effective_limits`
Получение эффективных лимитов с учетом всех активных баффов.

**Параметры:**
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `project_id` (int, optional) - ID проекта

**Возвращает:** Эффективные лимиты после применения всех активных баффов.

**Пример:**
```json
{
  "tool": "buffs.get_effective_limits",
  "params": {
    "entity_id": 123,
    "entity_kind": "user",
    "project_id": 1
  }
}
```

#### `buffs.apply_temporary_effect`
Быстрое применение временного эффекта (создает и применяет в одном шаге).

**Параметры:**
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `name` (string, required) - Название эффекта
- `duration_days` (int, required) - Длительность в днях
- `effects` (object, required) - Эффекты
- `priority` (int, optional, default: 100) - Приоритет
- `project_id` (int, optional) - ID проекта

**Пример:**
```json
{
  "tool": "buffs.apply_temporary_effect",
  "params": {
    "entity_id": 123,
    "entity_kind": "user",
    "name": "Weekend Bonus",
    "duration_days": 2,
    "effects": {
      "data.limits.api_calls": 5000
    }
  }
}
```

#### `buffs.apply_persistent_effect`
Быстрое применение постоянного эффекта (создает и применяет в одном шаге).

**Параметры:**
- `entity_id` (int, required) - ID сущности
- `entity_kind` (string, required) - Тип сущности
- `name` (string, required) - Название эффекта
- `effects` (object, required) - Эффекты
- `priority` (int, optional, default: 300) - Приоритет
- `project_id` (int, optional) - ID проекта

**Пример:**
```json
{
  "tool": "buffs.apply_persistent_effect",
  "params": {
    "entity_id": 123,
    "entity_kind": "user",
    "name": "Lifetime Premium",
    "effects": {
      "data.features.premium": true,
      "data.limits.api_calls": 1000000
    }
  }
}
```

### Workflows и комбинации инструментов

**Workflow 1: Полный цикл триального периода**
```json
[
  {"tool": "buffs.create_buff", "params": {...}},
  {"tool": "buffs.apply_buff", "params": {"buff_id": "<from previous>"}},
  {"tool": "buffs.list_active_buffs", "params": {...}}
]
```

**Workflow 2: Продление подписки**
```json
[
  {"tool": "buffs.list_active_buffs", "params": {"category": "subscription"}},
  {"tool": "buffs.extend_buff", "params": {"buff_id": "<subscription_buff_id>"}},
  {"tool": "buffs.get_buff", "params": {"buff_id": "<subscription_buff_id>"}}
]
```

**Workflow 3: Отмена ошибочного баффа**
```json
[
  {"tool": "buffs.get_buff", "params": {...}},
  {"tool": "buffs.revert_buff", "params": {...}},
  {"tool": "buffs.cancel_buff", "params": {...}}
]
```

### Сценарии использования

**Что такое бафы?** Бафы (Buffs) - это система временных и постоянных эффектов, которые изменяют лимиты, функции и ресурсы пользователей и проектов. Идеально подходят для управления триалами, промо-акциями, подписками и разовыми покупками.

**Временные эффекты (автоматически истекают и откатываются):**
- **Триальные периоды (7-30 дней)** - бесплатные пробные версии функций для новых пользователей
- **Промо-акции и события** - ограниченные по времени предложения (например, "Черная пятница")
- **Временные бонусы** - дополнительные ресурсы на короткий срок (например, удвоение опыта на выходных)
- **Сезонные предложения** - праздничные акции и специальные события

**Постоянные эффекты (не истекают, требуют ручного управления):**
- **Подписки (с продлением)** - ежемесячные/годовые планы с возможностью автоматического продления
- **Разовые покупки** - одноразовые улучшения (например, увеличение лимитов API calls)
- **Пакеты улучшений** - накопительные постоянные бонусы (можно купить несколько раз)
- **Lifetime upgrades** - пожизненные улучшения без срока действия

**Примеры синергии с другими системами:**
- **Buffs + Logic Engine** - автоматическое применение триалов при регистрации пользователя
- **Buffs + Payments** - продажа подписок и разовых покупок через платежную систему
- **Buffs + Scheduler** - автоматическое продление подписок по расписанию
- **Buffs + Analytics** - отслеживание эффективности промо-кампаний и конверсии триалов
- **Buffs + Notifications** - уведомления о новых бафах, истечении триалов и промо-акциях

**Подробные примеры комплексных проектов:** См. [examples/mcp_complex_projects.md](examples/mcp_complex_projects.md), [examples/mcp_buffs_workflows.md](examples/mcp_buffs_workflows.md), [examples/mcp_buffs_temporary.md](examples/mcp_buffs_temporary.md), [examples/mcp_buffs_persistent.md](examples/mcp_buffs_persistent.md).

---