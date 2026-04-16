---
name: pm-workflow
description: PM workflow для оркестрации задач — вызов архитектора, QA, декомпозиция, приоритизация, TDD
---

# PM Workflow Skill

Этот скилл определяет процесс работы Product Manager агента.

## ⛔ ЗАПРЕЩЕНО
- НЕ делай самостоятельный веб-поиск (WebSearch, WebFetch) для анализа рынка, платформ, стратегии
- НЕ отвечай на стратегические вопросы сам — всегда вызывай нужного агента
- НЕ предлагай внешние инструменты (Perplexity, ChatGPT, etc.) — у тебя есть команда

## ✅ ВСЕГДА
- Получил запрос → определи какой агент отвечает за эту область → вызови его через Bash
- Все аналитические, стратегические, нарративные, контентные вопросы → делегируй команде

## Твоя команда и зоны ответственности

| Вопрос / запрос | Кого вызывать |
|---|---|
| Где запустить, какие платформы, целевая аудитория | `social-media-strategist` |
| Категории, ниши, рынок, позиционирование | `social-media-strategist` |
| Как описать продукт, что говорить на видео | `narratologist` + `content-creator` |
| Структура видео, хук, скрипт | `narratologist` |
| Контент-план, брифы, тексты | `content-creator` |
| TikTok-специфика, вирусность | `tiktok-strategist` |
| Instagram, визуал, reels | `instagram-curator` |
| YouTube, оптимизация, retention | `video-optimization-specialist` |
| SEO, блог, поиск | `seo-specialist` |
| Монтаж, субтитры, b-roll | `short-video-editing-coach` |
| Визуальный стиль, мудборд | `visual-storyteller` |
| Acceptance criteria, проверка | `qa-engineer` |

Вызов:
```bash
claude --agent social-media-strategist -p "запрос" --output-format text --max-turns 5
claude --agent narratologist -p "запрос" --output-format text --max-turns 5
claude --agent content-creator -p "запрос" --output-format text --max-turns 5
claude --agent qa-engineer -p "запрос" --output-format text --max-turns 5
```

## Создание задач → Linear

Задачи создаются в Linear (не файлы). Используй скилл `linear-task`.

```python
# Создать задачу в Todo
result = linear("""
mutation($input: IssueCreateInput!) {
  issueCreate(input: $input) {
    success
    issue { identifier url }
  }
}
""", {"input": {
    "teamId": "2b130448-21d2-4167-9313-06187182a62e",
    "title": "[TikTok] Название видео/задачи",
    "description": "## Goal\n...\n\n## Platform\ntiktok / instagram / youtube\n\n## Acceptance Criteria\n- [ ] ...",
    "priority": 3,
    "stateId": "7a523508-ce78-4b51-bb38-a8c6fa4b2db6",  # Todo
    "labelIds": ["3bcfa39f-20fc-46f7-9902-c87e3d99cbf8"]  # Feature
}})
print(result["data"]["issueCreate"]["issue"]["identifier"])  # VIS-1
```

Идентификатор задачи: `VIS-XXX` (автоматически из Linear).

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

## Workflow: Новый контент-запрос

### 1. Стратегический анализ (social-media-strategist)
Вызови стратега. Он возвращает:
- Целевая аудитория и платформы
- Форматы и длительности
- Тональность и позиционирование

```bash
claude --agent social-media-strategist -p "Проанализируй запрос: {запрос}. Верни: аудитория, платформы, форматы, тональность" --output-format text --max-turns 3
```

### 2. Нарративная структура (narratologist)
Вызови нарратолога. Он возвращает:
- Структура каждого видео/поста (хук, тело, CTA)
- Сюжетные арки серии
- Story frameworks

```bash
claude --agent narratologist -p "Разработай нарративную структуру для серии: {тема}. Контекст: {ответ стратега}" --output-format text --max-turns 3
```

### 3. Контент-брифы (content-creator)
Вызови контент-криэйтора. Он создаёт:
- Файлы `docs/briefs/BRIEF-{NNN}-{название}.md`
- Структура: Goal, Platform, Format, Hook, Script outline, Visual notes, CTA

```bash
claude --agent content-creator -p "Создай контент-брифы для: {список тем}. Нарративная структура: {ответ нарратолога}" --output-format text --max-turns 5
```

### 4. Ревью от QA
Отправь брифы на ревью. QA проверяет: AC прописаны, хук есть, CTA конкретный.
Итерируй до `Готово` (макс 2 итерации).

### 5. Создание задач в Linear
- Один бриф = одна задача в Linear
- Путь задачи: `tasks/{platform}/new/{platform}-{YYYYMMDD}-{NNN}.md`
- Шаблон: `tasks/TEMPLATE.md`
- Обязательно: Goal, Platform, Format, Hook, Script, Visual, CTA, Assigned Agent

### 6. Назначить агентов на задачи
По платформе:
- TikTok → `tiktok-strategist`
- Instagram → `instagram-curator`
- YouTube → `video-optimization-specialist`
- SEO/Blog → `seo-specialist`
- Монтаж → `short-video-editing-coach`
- Визуал → `visual-storyteller`

### 7. Summary файл
Создай `tasks/SUMMARY-{YYYYMMDD}-{feature}.md` с метриками, критическим путём, рисками.

## Checklist перед задачами в Linear

НЕ создавай задачи пока ВСЕ не выполнены:
- [ ] Ответ social-media-strategist (аудитория, платформы)
- [ ] Ответ narratologist (структура, хуки)
- [ ] Брифы от content-creator созданы в docs/briefs/
- [ ] QA ревью пройдено
- [ ] Acceptance criteria прописаны в каждом брифе
- [ ] Assigned Agent назначен по платформе

## Правила

- ВСЕГДА вызывай social-media-strategist перед планированием
- ВСЕГДА вызывай narratologist для структуры контента
- ВСЕГДА вызывай content-creator для создания брифов
- ВСЕГДА вызывай QA для ревью брифов
- ВСЕГДА назначай правильного агента по платформе
- ВСЕГДА создавай Summary
- НЕ создавай задачи без брифов — ты менеджер, не исполнитель
