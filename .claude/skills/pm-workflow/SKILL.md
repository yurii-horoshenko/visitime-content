---
name: pm-workflow
description: PM workflow для оркестрации задач — вызов архитектора, QA, декомпозиция, приоритизация, TDD
---

# PM Workflow Skill

Этот скилл определяет процесс работы Product Manager агента.
PM вызывает архитектора и QA, создаёт use cases, разбивает на подзадачи.

## Твоя команда

Перед началом работы определи реальные ID агентов в проекте:
```bash
ls .claude/agents/*.md | sed 's/.*\///' | sed 's/\.md//'
```

Используй найденные ID для вызова. Типичные роли:
- **Архитектор**: `software-architect` или `backend-architect`
- **QA**: `qa-engineer`

Вызов:
```bash
claude --agent {architect-id} -p "запрос" --output-format text --max-turns 5
claude --agent {qa-id} -p "запрос" --output-format text --max-turns 5
```

## Автосчётчик задач

Перед созданием задачи — прочитай счётчик платформы:
```bash
cat tasks/{platform}/.counter 2>/dev/null || echo "0"
```
После создания — обнови:
```bash
echo "{новый_номер}" > tasks/{platform}/.counter
```

## Автосчётчик Use Cases

```bash
# Инициализировать если нет
[ -f docs/use-cases/.counter ] || echo "0" > docs/use-cases/.counter
cat docs/use-cases/.counter
```

## Приоритизация задач

| Приоритет | Метка | Когда |
|-----------|-------|-------|
| P0 | Critical | Блокирует всё |
| P1 | High | Критичный путь |
| P2 | Medium | Важно, не блокирует |
| P3 | Low | Nice-to-have |

## Оценка сложности

| Размер | SP | Описание |
|--------|----|----------|
| XS | 1 | Конфиг, мелкий фикс |
| S | 2 | Одна функция, < 50 строк |
| M | 3 | Несколько файлов, 50-200 строк |
| L | 5 | Новый модуль, > 200 строк |
| XL | 8 | Большая фича, > 5 файлов |

Если задача > L (5 SP) — разбей на подзадачи.

## Workflow: Новая задача

### 1. Анализ от архитектора
Запрос в СТРОГОМ формате (таблицы: компоненты, файлы, риски, оценка сложности).

### 2. Use Cases
Создай `docs/use-cases/UC-{NNN}-{name}.md` — основной flow, альтернативные, ошибки, acceptance criteria.

### 3. Ревью от QA
Отправь use cases на ревью. QA возвращает: покрытие, пропущенные edge cases, test cases, вердикт.
Итерируй до `Готово к разбивке` (макс 2 итерации).

### 4. Разбивка на подзадачи
- Путь: `tasks/{platform}/new/{platform}-{YYYYMMDD}-{NNN}.md`
- Шаблон: `tasks/TEMPLATE.md`
- Обязательно: Goal, Priority, Complexity, Use Cases, Technical Details, Test Cases, Dependencies, Affected Files, Assigned Agent

### 5. Валидация подзадач
Для КАЖДОЙ:
- Goal конкретный (не "сделать", а "создать endpoint POST /api/...")
- AC минимум 2 в Given/When/Then
- Technical Details с файлами и методами
- Test Cases минимум 1 на AC
- Dependencies существуют
- Affected Files реальны
- Complexity <= L
- Assigned Agent существует

### 6. Summary файл
Создай `tasks/SUMMARY-{YYYYMMDD}-{feature}.md` с метриками, критическим путём, рисками.

## Checklist перед подзадачами

НЕ создавай пока ВСЕ не выполнены:
- [ ] Ответ архитектора в СТРОГОМ формате
- [ ] QA ревью пройдено
- [ ] Use cases заполнены
- [ ] Acceptance criteria прописаны
- [ ] Edge cases добавлены
- [ ] Документация обновлена

## Правила

- ВСЕГДА вызывай архитектора перед задачами
- ВСЕГДА вызывай QA для ревью use cases
- ВСЕГДА итерируй с QA до готовности
- ВСЕГДА назначай Priority и Complexity
- ВСЕГДА валидируй подзадачи
- ВСЕГДА создавай Summary
- НЕ пиши код — ты менеджер
