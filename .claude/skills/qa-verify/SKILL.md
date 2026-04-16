---
name: qa-verify
description: QA верификация задачи — проверка Acceptance Criteria по PR, approve/reject, перевод в Done.
---

# QA Verify Workflow

## Когда активировать

- "верифицируй {ISSUE_ID}", "проверь задачу {ISSUE_ID}", "qa review VIS-XXX"
- Разработчик написал "Передай QA: верифицируй VIS-XXX"

---

## Шаг 1 — Прочитай задачу и найди PR

```python
import json, urllib.request, os

def linear_api(query, variables=None):
    env_path = os.path.join(os.getcwd(), ".env")
    key = next(l.split("=",1)[1].strip() for l in open(env_path) if l.startswith("LINEAR_API_KEY="))
    req = urllib.request.Request(
        "https://api.linear.app/graphql",
        data=json.dumps({"query": query, "variables": variables or {}}).encode(),
        headers={"Authorization": key, "Content-Type": "application/json"}
    )
    with urllib.request.urlopen(req, timeout=15) as r:
        return json.loads(r.read())

result = linear_api("""
query($id: String!) {
  issue(id: $id) {
    id identifier title description
    state { name }
    comments { nodes { body createdAt } }
  }
}
""", {"id": "{ISSUE_IDENTIFIER}"})
issue = result["data"]["issue"]

# Найди PR URL в комментариях
pr_url = None
for comment in reversed(issue["data"]["issue"]["comments"]["nodes"]):
    if "PR" in comment["body"] or "github.com" in comment["body"]:
        import re
        match = re.search(r'https://github\.com/\S+/pull/\d+', comment["body"])
        if match:
            pr_url = match.group(0)
            break
```

Выведи статус:
```
🔍 Верифицирую {ISSUE_ID}: {title}
   Статус: {state}
   PR: {pr_url}
```

Если PR не найден — остановись:
```
⚠️ PR не найден в комментариях к {ISSUE_ID}.
   Убедись что разработчик создал PR и добавил ссылку в Linear.
```

---

## Шаг 2 — Получи PR и переключись на ветку

```bash
# Получи номер PR из URL (последняя часть)
PR_NUMBER=$(echo "{pr_url}" | grep -o '[0-9]*$')

# Получи имя ветки через gh
BRANCH=$(gh pr view $PR_NUMBER --json headRefName -q '.headRefName')

# Fetch и checkout ветки
git fetch origin "$BRANCH"
git checkout "$BRANCH"
git pull origin "$BRANCH"

echo "Ветка: $BRANCH"
```

---

## Шаг 3 — Запусти тесты

Запусти все тесты связанные с задачей:

### iOS
```bash
xcodebuild test \
  -scheme {SchemeName} \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  2>&1 | grep -E "(FAILED|PASSED|error:|✗|✓)" | tail -20
```

### Android
```bash
cd {project_root}/android
./gradlew test 2>&1 | grep -E "(FAILED|PASSED|BUILD)" | tail -10
```

### KMM
```bash
./gradlew {module}:commonTest 2>&1 | grep -E "(FAILED|PASSED|BUILD)" | tail -10
```

Результат:
```
✅ Тесты: {N} passed / ❌ {N} failed
```

Если тесты падают → сразу переходи к Шагу 5b (Reject).

---

## Шаг 4 — Проверь Acceptance Criteria

Из описания задачи возьми каждый AC и проверь вручную или через тесты:

```
Acceptance Criteria:
- [ ] AC 1: {описание} — проверяю...
- [ ] AC 2: {описание} — проверяю...
```

Для каждого AC:
- Запусти соответствующий тест если есть
- Или опиши как будет проверяться при ручном тесте

Итог:
```
✅ AC 1: пройден
✅ AC 2: пройден
❌ AC 3: не пройден — {причина}
```

---

## Шаг 5a — ✅ Approve (все AC пройдены)

```bash
# Снять Draft (PR готов к merge)
gh pr ready $PR_NUMBER

# Approve PR
gh pr review $PR_NUMBER --approve --body "✅ QA верификация пройдена.
Тесты: {N} passed.
Все Acceptance Criteria выполнены.
Готов к merge в develop."
```

```python
# Linear: Done
states = linear_api('{ workflowStates { nodes { id name } } }')
done_id = next(s["id"] for s in states["data"]["workflowStates"]["nodes"] if s["name"] == "Done")

linear_api("""
mutation($id: String!, $stateId: String!) {
  issueUpdate(id: $id, input: { stateId: $stateId }) { success }
}
""", {"id": issue["id"], "stateId": done_id})

linear_api("""
mutation($issueId: String!, $body: String!) {
  commentCreate(input: { issueId: $issueId, body: $body }) { success }
}
""", {"issueId": issue["id"], "body": f"✅ QA верификация пройдена. PR одобрен: {pr_url}\nГотов к merge в develop."})
```

---

## Шаг 5b — ❌ Reject (есть провалившиеся AC или тесты)

```bash
# PR остаётся Draft, запрашиваем изменения
gh pr review $PR_NUMBER --request-changes --body "❌ QA верификация не пройдена.

Проблемы:
{список провалившихся AC / упавших тестов}

Необходимо исправить до merge."
```

```python
# Linear: обратно In Progress
states = linear_api('{ workflowStates { nodes { id name } } }')
in_progress_id = next(s["id"] for s in states["data"]["workflowStates"]["nodes"] if s["name"] == "In Progress")

linear_api("""
mutation($id: String!, $stateId: String!) {
  issueUpdate(id: $id, input: { stateId: $stateId }) { success }
}
""", {"id": issue["id"], "stateId": in_progress_id})

linear_api("""
mutation($issueId: String!, $body: String!) {
  commentCreate(input: { issueId: $issueId, body: $body }) { success }
}
""", {"issueId": issue["id"], "body": f"❌ QA верификация не пройдена. Возвращено в разработку.\nПроблемы:\n{issues_list}"})
```

Сообщи разработчику:
```
claude --agent {dev-agent-id} "QA вернул задачу {ISSUE_ID}. Проблемы: {список}"
```

---

## Итоговый отчёт

### Approve:
```
✅ {ISSUE_ID} — {title}

🧪 Тесты:      {N} passed
📋 AC:         {N}/{N} выполнено
👁 PR:         одобрен → {pr_url}
📌 Linear:     Done ✓

PR готов к merge в develop.
```

### Reject:
```
❌ {ISSUE_ID} — {title}

🧪 Тесты:      {failed} failed
📋 AC:         {N}/{total} выполнено
👁 PR:         изменения запрошены
📌 Linear:     In Progress (возвращено)

Разработчик уведомлён.
```
