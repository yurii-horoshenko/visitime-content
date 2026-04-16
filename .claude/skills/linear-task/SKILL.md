---
name: linear-task
description: Работа с задачами Linear — читать, создавать, обновлять статус через GraphQL API
---

# Linear Task Skill

API: `https://api.linear.app/graphql`
Auth: `LINEAR_API_KEY` из `.env` (без Bearer prefix)
Проект: VisiTimeContent · тикеты `VIS-XXX`

## IDs (VisiTimeContent)

```
Team:    2b130448-21d2-4167-9313-06187182a62e
States:  Backlog   → 59aa9866-9784-4bc9-af7b-0942b4f77bd3
         Todo      → 7a523508-ce78-4b51-bb38-a8c6fa4b2db6
         In Prog   → dc2365fa-7e05-47b7-a254-740e31799ba4
         Done      → 1e868d27-fe00-48fe-bb73-77e8ecad4ed4
         Canceled  → dfa25e0d-572f-401a-9a3c-c84e371874f5
Labels:  Bug       → 8f1f4d7a-a77d-4776-848f-450eaf39c014
         Feature   → 3bcfa39f-20fc-46f7-9902-c87e3d99cbf8
         Improvement → ac809b6a-3edb-409f-8d98-7833eef79f73
```

## Хелпер функция

```python
import json, urllib.request

def linear(query, variables=None):
    env = {}
    with open('/Users/gi_part/Standart/Projects/VisiTimeContent/.env') as f:
        for line in f:
            if '=' in line:
                k, v = line.strip().split('=', 1)
                env[k] = v
    req = urllib.request.Request(
        "https://api.linear.app/graphql",
        data=json.dumps({"query": query, "variables": variables or {}}).encode(),
        headers={"Authorization": env["LINEAR_API_KEY"], "Content-Type": "application/json"}
    )
    with urllib.request.urlopen(req, timeout=15) as r:
        return json.loads(r.read())
```

## Читать задачу

```python
result = linear("""
query($id: String!) {
  issue(id: $id) {
    id identifier title description priority
    state { name }
    labels { nodes { name } }
  }
}
""", {"id": "VIS-1"})
issue = result["data"]["issue"]
print(issue["identifier"], issue["title"])
print(issue["description"])
```

## Создать задачу

```python
result = linear("""
mutation($input: IssueCreateInput!) {
  issueCreate(input: $input) {
    success
    issue { identifier url }
  }
}
""", {"input": {
    "teamId": "2b130448-21d2-4167-9313-06187182a62e",
    "title": "Название задачи",
    "description": "## Goal\n...\n\n## Acceptance Criteria\n- [ ] ...",
    "priority": 3,  # 1=Urgent 2=High 3=Medium 4=Low
    "stateId": "7a523508-ce78-4b51-bb38-a8c6fa4b2db6",  # Todo
    "labelIds": ["3bcfa39f-20fc-46f7-9902-c87e3d99cbf8"]  # Feature
}})
print(result["data"]["issueCreate"]["issue"]["identifier"])  # VIS-1
print(result["data"]["issueCreate"]["issue"]["url"])
```

## Обновить статус

```python
linear("""
mutation($id: String!, $stateId: String!) {
  issueUpdate(id: $id, input: { stateId: $stateId }) {
    success
  }
}
""", {"id": "<issue-uuid>", "stateId": "dc2365fa-7e05-47b7-a254-740e31799ba4"})  # In Progress
```

## Добавить комментарий

```python
linear("""
mutation($issueId: String!, $body: String!) {
  commentCreate(input: { issueId: $issueId, body: $body }) {
    success
  }
}
""", {"issueId": "<issue-uuid>", "body": "✓ Реализовано."})
```

## Использование агентами

1. **PM** — создаёт задачи через API (stateId: Todo), назначает лейбл Feature/Bug/Improvement
2. **Content Creator / Platform Agents** — читают задачу, переводят в `In Progress`
3. **QA Engineer** — переводит в `Done` или комментирует замечания
