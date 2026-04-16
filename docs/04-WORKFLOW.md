# VisiTimeContent — Workflow

## Точка входа

Ты (заказчик) общаешься с **Product Manager** (бизнес-задачи, декомпозиция) или **Software Architect** (техзадачи).
PM — главная точка входа. Он вызывает Architect и QA автоматически.

```bash
claude --agent product-manager "Описание задачи"
```
```bash
claude --agent software-architect "Описание задачи"
```

## Цепочка агентов

```
       ┌─────────────────────┐
       │     Заказчик (ты)   │
       └─────────┬───────────┘
                 │
                 ▼
    ┌────────────────────────────────────────┐
    │   Product Manager                      │
    └───────────────────┬────────────────────┘
                   │
                   ▼
    ┌────────────────────────────────────────┐
    │   Software Architect                   │
    └───────────────────┬────────────────────┘
                   │
                   ▼
    ┌────────────────────────────────────────┐
    │   Content Creator                      │
    │   Instagram Curator                    │
    │   Narratologist                        │
    │   SEO Specialist                       │
    │   Short-Video Editing Coach            │
    │   Social Media Strategist              │
    │   TikTok Strategist                    │
    │   Video Optimization Specialist        │
    │   Visual Storyteller                   │
    └───────────────────┬────────────────────┘
                   │
                   ▼
    ┌────────────────────────────────────────┐
    │   QA Engineer                          │
    └───────────────────┬────────────────────┘
                   │
           ┌───────┴───────┐
           ▼               ▼
      reviewed/       in-progress/
      (ОК → done/)   (доработка)
```

## Кто за что отвечает

| Агент | Роль | Когда обращаться | Команда |
|-------|------|-----------------|---------|
| **Content Creator** | Разработка | Задачи по проекту | `claude --agent content-creator` |
| **Instagram Curator** | Разработка | Задачи по проекту | `claude --agent instagram-curator` |
| **Narratologist** | Разработка | Задачи по проекту | `claude --agent narratologist` |
| **Product Manager** | Бизнес-требования, приоритеты, user stories | Новая бизнес-фича, что важнее? | `claude --agent product-manager` |
| **QA Engineer** | Use cases, TDD тесты, ревью документации, качество | Напиши use cases, тесты, проверь документацию | `claude --agent qa-engineer` |
| **SEO Specialist** | Разработка | Задачи по проекту | `claude --agent seo-specialist` |
| **Short-Video Editing Coach** | Разработка | Задачи по проекту | `claude --agent short-video-editing-coach` |
| **Social Media Strategist** | Разработка | Задачи по проекту | `claude --agent social-media-strategist` |
| **Software Architect** | Архитектура, техрешения, декомпозиция | Новая фича, рефакторинг, как лучше сделать? | `claude --agent software-architect` |
| **TikTok Strategist** | Разработка | Задачи по проекту | `claude --agent tiktok-strategist` |
| **Video Optimization Specialist** | Разработка | Задачи по проекту | `claude --agent video-optimization-specialist` |
| **Visual Storyteller** | Разработка | Задачи по проекту | `claude --agent visual-storyteller` |

## Жизненный цикл задачи

### 1. Создание (ты → Architect/PM)
```bash
claude --agent software-architect "Создай задачу: описание"
```
Результат: `tasks/{platform}/new/{platform}-{YYYYMMDD}-{NNN}.md`

### 2. Проектирование (Architect)
```bash
claude --agent software-architect "Спроектируй {task_id}"
```
Заполняет: Technical Details, Affected Files, API контракты.

### 3. Тесты — TDD (Reviewer)
```bash
claude --agent qa-engineer "Напиши тест-кейсы для {task_id}"
```
Заполняет: Test Cases. Тесты пишутся **ДО** кода.

### 4. Реализация (Developer)
```bash
claude --agent content-creator "Реализуй {task_id}"
```
- Читает Technical Details + Test Cases
- Пишет код → тесты зелёные
- Перемещает в `tasks/{platform}/in-progress/`

### 5. Ревью (Reviewer)
```bash
claude --agent qa-engineer "Сделай ревью {task_id}"
```
- ОК → `reviewed/` → `done/`
- Замечания → назад в `in-progress/`

## Быстрые сценарии

### "Починить баг"
```bash
claude --agent content-creator "Баг: описание проблемы"
```

### "Как лучше сделать?"
```bash
claude --agent software-architect "Как лучше реализовать X?"
```

### "Проверь код"
```bash
claude --agent qa-engineer "Проверь последний коммит"
```

## Правила

- **Documentation-first**: сначала Technical Details → потом код
- **TDD**: сначала Test Cases → потом реализация
- **Статусы через папки**: перемещай файл задачи при смене статуса
- **Один агент — одна задача**: не давай одну задачу нескольким агентам одновременно
- **Каждый агент автономен**: `cd VisiTimeContent && claude --agent {id}` работает без зависимостей
