---
name: agent-task-picker
description: Показывает задачи агента из Linear по его метке, спрашивает какие взять в работу
---

# Agent Task Picker

Этот скилл даёт агенту команду **"мои задачи"** — показывает задачи из Linear по метке агента и берёт выбранные в работу.

## Конфигурация

Читается из файлов проекта:
- `LINEAR_API_KEY` — из `.env` в корне проекта
- Лейблы — запрашиваются динамически из Linear (не хардкодятся)

## Когда активировать

Когда пользователь говорит:
- "мои задачи" / "список задач" / "что мне делать"
- "покажи задачи" / "задачи из linear" / "task list"
- "возьми задачу" / "take task"

## Алгоритм выполнения

### Шаг 1 — Получи лейблы из Linear и спроси фильтр

Не читай лейблы из yaml — запроси напрямую из Linear:

```python
import json, urllib.request, os

project_root = os.getcwd()

def linear_api(query, variables=None):
    env_path = os.path.join(project_root, ".env")
    key = ""
    for line in open(env_path):
        if line.startswith("LINEAR_API_KEY="):
            key = line.split("=", 1)[1].strip()
    req = urllib.request.Request(
        "https://api.linear.app/graphql",
        data=json.dumps({"query": query, "variables": variables or {}}).encode(),
        headers={"Authorization": key, "Content-Type": "application/json"}
    )
    with urllib.request.urlopen(req, timeout=15) as r:
        return json.loads(r.read())

# Получи все лейблы проекта
result = linear_api("{ issueLabels { nodes { id name } } }")
all_labels = [l["name"] for l in result["data"]["issueLabels"]["nodes"]]
```

Покажи лейблы пользователю и спроси фильтр:

```
🏷️ Лейблы в Linear: ios · android · kmm · backend · Bug · Feature

Показать задачи для (введи название, номер или "все"):
  1. ios
  2. android
  3. kmm
  4. backend
  5. Bug
  6. Feature
  7. все
```

Жди ответа. Примеры: `1`, `ios`, `1,2`, `ios,android`, `все`, `all`.
Преобразуй выбор в список лейблов для фильтра.

### Шаг 2 — Найди задачи по выбранным лейблам

```python
# selected_labels — список выбранных пользователем лейблов
result = linear_api("""
query($labels: [String!]!) {
  issues(
    filter: {
      labels: { name: { in: $labels } }
      state: { name: { in: ["Backlog", "Todo"] } }
    }
    orderBy: priority
  ) {
    nodes {
      id
      identifier
      title
      priority
      state { name }
      labels { nodes { name } }
      description
    }
  }
}
""", {"labels": selected_labels})

issues = result["data"]["issues"]["nodes"]
```

### Шаг 4 — Покажи список задач

Выведи пронумерованный список. Если выбрано несколько платформ — добавляй колонку **Платформа**:

```
📋 Мои задачи [ios · android] — 34 шт.

#  | ID      | Платформа | Приоритет | Статус | Название
---|---------|-----------|-----------|--------|---------------------------
1  | VIS-87  | ios       | 🔴 Urgent | Todo   | Убрать Sign Up из LoginScreen
2  | VIS-100 | android   | 🔴 High   | Todo   | Реализовать ActivateCodeScreen
3  | VIS-051 | ios       | 🟡 Medium | Todo   | Add pagination to feed
```

Если выбрана одна платформа — колонку Платформа не показывай:

```
📋 Мои задачи [ios] — 12 шт.

#  | ID      | Приоритет | Статус | Название
---|---------|-----------|--------|---------------------------
1  | VIS-87  | 🔴 Urgent | Todo   | Убрать Sign Up из LoginScreen
```

Платформу задачи определяй по её меткам в Linear.
Приоритеты: 1=🔴 Urgent, 2=🔴 High, 3=🟡 Medium, 4=🟢 Low

### Шаг 5 — Получи выбор пользователя

Жди ответа. Варианты:
- `все` / `all` → берём все задачи
- `1,3` / `1, 3` → берём задачи с номерами 1 и 3
- `отмена` / `cancel` / `нет` → ничего не делать

### Шаг 6 — Переведи выбранные в In Progress

```python
# Получи ID статуса "In Progress"
states = linear_api('{ workflowStates { nodes { id name } } }')
in_progress_id = next(
    s["id"] for s in states["data"]["workflowStates"]["nodes"]
    if s["name"] == "In Progress"
)

# Переведи каждую выбранную задачу
for issue in selected_issues:
    linear_api("""
    mutation($id: String!, $stateId: String!) {
      issueUpdate(id: $id, input: { stateId: $stateId }) {
        success
      }
    }
    """, {"id": issue["id"], "stateId": in_progress_id})
```

### Шаг 7 — Подтверди и начни работу

```
✅ Взято в работу:
   VIS-042 — Implement auth token refresh
   VIS-063 — Update error messages

Начинаю с VIS-042. Читаю описание задачи...
```

Прочитай полное описание первой задачи и приступай к реализации.

## Конфигурация в agent yaml

При создании агента через factory — добавляй поле `linear_label` в `aifactory/agents/{id}.yaml`:

```yaml
name: "Mobile Developer"
model: sonnet
linear_label: ios        # метка в Linear для фильтрации задач
skills:
  - agent-task-picker
  - systematic-debugging
```

## Поддерживаемые метки

Стандартные метки (совпадают с платформами проекта):
`ios`, `android`, `backend`, `frontend`, `kmm`, `ml`, `api`, `data`, `gameplay`, `engine`

Кастомные метки — любые, которые есть в Linear проекта.
