# VisiTimeContent

## Стек
Не определён

## Агенты (`claude --agent <name>`)

| Агенты | Модель |
|--------|--------|
| qa-engineer | `cloud-code` |
| social-media-strategist, software-architect, tiktok-strategist | `cloud-max` |
| content-creator, instagram-curator, narratologist, product-manager, seo-specialist, short-video-editing-coach, video-optimization-specialist, visual-storyteller | `cloud-smart` |

## Задачи
Linear — задачи ведутся там. Ключ: `LINEAR_API_KEY` в `.env`.
Team: `VisiTimeContent` | ID: `2b130448-21d2-4167-9313-06187182a62e` | Key: `VIS` | Задачи: `VIS-XXX`
Статусы: `Backlog` → `Todo` → `In Progress` → `Done`

## Workflow
PM (Linear задача) → Architect (Technical Details) → QA (Test Cases) → Dev (реализует → `In Progress`) → QA (`Done`)

## Compact Instructions
При компакте сохранять:
- Текущая задача и статус
- Изменённые файлы и ключевые решения
Выбрасывать: tool calls, промежуточные чтения, отладочный вывод.
