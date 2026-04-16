---
name: tdd-workflow
description: TDD процесс выполнения задачи — сначала тесты, потом реализация. Red → Green → Done.
---

# TDD Workflow

Когда берёшь задачу в работу — **всегда** следуй этому порядку. Никакого кода до тестов.

## Когда активировать

- Пользователь говорит "выполни задачу", "возьми в работу", "реализуй VIS-XXX"
- Задача взята через `agent-task-picker`

---

## Шаг 1 — Прочитай задачу полностью

Получи задачу из Linear вместе с Test Cases от QA:

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
    labels { nodes { name } }
    comments { nodes { body createdAt } }
  }
}
""", {"id": "{ISSUE_IDENTIFIER}"})  # напр. "VIS-42"
issue = result["data"]["issue"]
```

Выведи структуру задачи:
```
📋 VIS-42: {title}
   Платформа: {labels}
   
   Описание:
   {description}
   
   Test Cases от QA:
   {test cases из описания или комментариев}
```

Если Test Cases отсутствуют в задаче — **остановись** и сообщи:
```
⚠️ В задаче VIS-42 нет Test Cases от QA.
   Workflow: сначала QA должен написать тесты, потом разработка.
   Передай задачу QA: claude --agent qa-engineer "Напиши Test Cases для VIS-42"
```

Test Cases есть → сразу поставь задачу `In Progress` и создай ветку:

```python
# Linear: In Progress
states = linear_api('{ workflowStates { nodes { id name } } }')
in_progress_id = next(s["id"] for s in states["data"]["workflowStates"]["nodes"] if s["name"] == "In Progress")
linear_api("""
mutation($id: String!, $stateId: String!) {
  issueUpdate(id: $id, input: { stateId: $stateId }) { success }
}
""", {"id": issue["id"], "stateId": in_progress_id})
```

```bash
# Создай feature ветку от актуального develop
ISSUE_ID="{ISSUE_IDENTIFIER}"  # напр. VIS-42
BRANCH="feature/${ISSUE_ID}-$(echo '{title}' | tr '[:upper:] ' '[:lower:]-' | tr -cd '[:alnum:]-' | cut -c1-40)"
git checkout develop 2>/dev/null || git checkout main
git pull origin $(git branch --show-current)
git checkout -b "$BRANCH"
echo "Ветка: $BRANCH"
```

---

## Шаг 2 — Прочитай связанную документацию

Перед написанием тестов прочитай `docs/00-index.md` и найди релевантные файлы:
- Архитектуру компонента который меняешь (`docs/04-WORKFLOW.md` — если вопросы по процессу)
- Требования (requirements/)
- Существующие тесты рядом с файлами которые будешь менять

---

## Шаг 3 — 🔴 RED: Напиши unit-тесты

**Сначала тесты — никакой реализации.**

Определи платформу из лейблов задачи и пиши тесты на нужном языке:

### iOS (Swift) — XCTest
```swift
// {FeatureName}Tests.swift
import XCTest
@testable import {ModuleName}

final class {FeatureName}Tests: XCTestCase {
    
    // MARK: - Setup
    var sut: {ClassName}!
    
    override func setUp() {
        super.setUp()
        sut = {ClassName}()
    }
    
    override func tearDown() {
        sut = nil
        super.tearDown()
    }
    
    // Один тест = один Test Case от QA
    func test_{scenario}() {
        // Given
        
        // When
        
        // Then
        XCTAssertEqual(...)
    }
}
```

### Android (Kotlin) — JUnit4 / Kotest
```kotlin
// {FeatureName}Test.kt
import org.junit.Test
import org.junit.Assert.*
import io.mockk.mockk

class {FeatureName}Test {
    
    private val sut = {ClassName}()
    
    @Test
    fun `test {scenario}`() {
        // Given
        
        // When
        val result = sut.{method}()
        
        // Then
        assertEquals(expected, result)
    }
}
```

### KMM (Kotlin Multiplatform)
```kotlin
// commonTest/kotlin/{FeatureName}Test.kt
import kotlin.test.Test
import kotlin.test.assertEquals

class {FeatureName}Test {
    @Test
    fun test_{scenario}() {
        // Given / When / Then
    }
}
```

Покажи созданные файлы:
```
✍️ Написаны тесты:
   └── {path/to/FeatureNameTests.swift}  ({N} тестов)
```

---

## Шаг 4 — 🔴 Запусти тесты (ожидаем падение)

Запусти тесты — они должны упасть (реализации ещё нет):

### iOS
```bash
xcodebuild test \
  -scheme {SchemeName} \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:{TargetName}/{FeatureName}Tests \
  2>&1 | grep -E "(FAILED|PASSED|error:|✗|✓)"
```

### Android
```bash
cd {project_root}/android
./gradlew test --tests "*.{FeatureName}Test" 2>&1 | grep -E "(FAILED|PASSED|BUILD)"
```

### KMM
```bash
cd {project_root}
./gradlew {module}:commonTest 2>&1 | grep -E "(FAILED|PASSED|BUILD)"
```

Ожидаемый результат:
```
🔴 RED — {N} тестов упали. Ожидаемо — реализации ещё нет.
   Продолжаю к реализации...
```

Если тесты **прошли** на этом этапе — остановись:
```
⚠️ Тесты прошли до реализации. Возможно:
   - Функциональность уже реализована
   - Тесты написаны некорректно (проверяют не то)
   Проверь тесты перед продолжением.
```

---

## Шаг 5 — ✅ GREEN: Реализуй фичу

Теперь пиши минимальный код чтобы тесты прошли.

- Читай Affected Files из описания задачи
- Вноси только изменения необходимые для задачи
- Не рефакторь то что не просили

---

## Шаг 6 — ✅ Запусти тесты снова (ожидаем успех)

```bash
# Та же команда что в Шаге 4
```

Ожидаемый результат:
```
✅ GREEN — все {N} тестов прошли.
```

Если тесты падают — анализируй ошибку и фикси реализацию. Не меняй тесты под реализацию.

Если после 3 итераций тесты всё ещё падают — сообщи пользователю с деталями ошибки.

---

## Шаг 7 — 👁 Code Review

Тесты прошли — вызови code-reviewer перед тем как двигаться дальше:

```bash
BASE_SHA=$(git rev-parse HEAD~1)
HEAD_SHA=$(git rev-parse HEAD)

claude --agent code-reviewer -p "Проверь реализацию задачи {ISSUE_ID}.
Что реализовано: {краткое описание}
BASE_SHA: ${BASE_SHA}
HEAD_SHA: ${HEAD_SHA}
Верни: критические проблемы, важные замечания, вердикт (Ready / Needs Fix)." \
--output-format text --max-turns 5
```

**По результату:**
- `Ready` → переходи к Шагу 8
- `Needs Fix` → исправь Critical и Important замечания → перезапусти тесты → повтори Code Review

---

## Шаг 8 — 🔒 Security Review (условно)

Запускай **только** если задача касается:
- Аутентификации / авторизации
- Платежей и подписок
- Персональных данных пользователей
- Deep links / URL схем
- Сетевых запросов с токенами

```bash
claude --agent security-engineer -p "Проверь задачу {ISSUE_ID} на уязвимости.
Изменённые файлы: {changed_files}
BASE_SHA: ${BASE_SHA} HEAD_SHA: ${HEAD_SHA}
Верни: уязвимости, риски, вердикт." \
--output-format text --max-turns 5
```

Если уязвимости найдены — исправь до продолжения.

---

## Шаг 9 — Git: commit + push + Draft PR

Ветка уже создана в Шаге 1. Закоммить, запушить, открыть Draft PR:

```bash
# Финальный коммит (или серия коммитов если уже были промежуточные)
git add {changed_files}
git commit -m "{ISSUE_ID}: {title}

TDD: {N} тестов. Code Review: ✅"

# Push
git push -u origin "$BRANCH"

# Draft PR — QA верифицирует, потом снимет Draft
# GITHUB_REVIEWER — GitHub username code-reviewer или команды
PR_URL=$(gh pr create \
  --base develop \
  --draft \
  --title "{ISSUE_ID}: {title}" \
  --reviewer "{GITHUB_REVIEWER}" \
  --body "## Задача
Linear: {ISSUE_URL}

## Изменения
{краткое описание}

## TDD
- Написано тестов: {N}
- RED → GREEN: ✅
- Code Review: ✅
- Security: {✅ / не требовался}

## Affected Files
{changed_files}

---
> Ожидает верификации QA. После approve QA — PR готов к merge." \
  --output url 2>/dev/null)

echo "PR (Draft): $PR_URL"
```

Если `gh` не установлен — создай PR вручную на GitHub как **Draft**.

---

## Шаг 10 — Обнови задачу в Linear

```python
# Переведи в In Review
states = linear_api('{ workflowStates { nodes { id name } } }')
in_review_id = next(
    (s["id"] for s in states["data"]["workflowStates"]["nodes"] if s["name"] == "In Review"),
    None
)
# Если статуса "In Review" нет — оставляем In Progress (уже выставлен в Шаге 1)
if in_review_id:
    linear_api("""
mutation($id: String!, $stateId: String!) {
  issueUpdate(id: $id, input: { stateId: $stateId }) { success }
}
""", {"id": issue["id"], "stateId": in_review_id})

# Добавь комментарий со ссылкой на PR
linear_api("""
mutation($issueId: String!, $body: String!) {
  commentCreate(input: { issueId: $issueId, body: $body }) { success }
}
""", {"issueId": issue["id"], "body": f"🔀 PR открыт (Draft): {PR_URL}\nTDD: {N} тестов. Code Review: ✅\nОжидает верификации QA."})
```

---

## Итоговый отчёт

```
✅ VIS-42 — {title}

🔴 RED:     {N} тестов написано → упали (ожидаемо)
✅ GREEN:    {N} тестов прошли после реализации
👁 Review:  Code Review пройден
🔒 Security: {пройден / не требовался}
🔀 PR:      {PR_URL} → develop

Изменённые файлы:
  ├── {path/to/Feature.swift}      (реализация)
  └── {path/to/FeatureTests.swift}  (тесты)

Linear: статус → In Review ✓ (был In Progress с Шага 1)
Передай QA: claude --agent qa-engineer "Верифицируй VIS-42 по Acceptance Criteria"
```
