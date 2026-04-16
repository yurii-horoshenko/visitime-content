# VisiTimeContent

## Стек
Не определён

## Агенты (`claude --agent <name>`)

| Агенты | Модель |
|--------|--------|
| social-media-strategist, tiktok-strategist | `cloud-max` |
| qa-engineer | `cloud-code` |
| content-creator, instagram-curator, narratologist, product-manager, seo-specialist, short-video-editing-coach, video-optimization-specialist, visual-storyteller | `cloud-smart` |

## Задачи
Linear — задачи ведутся там. Ключ: `LINEAR_API_KEY` в `.env`.
Team: `VisiTimeContent` | ID: `2b130448-21d2-4167-9313-06187182a62e` | Key: `VIS` | Задачи: `VIS-XXX`
Статусы: `Backlog` → `Todo` → `In Progress` → `Done`

## Workflow
PM → social-media-strategist (стратегия) → narratologist (структура) → content-creator (брифы в docs/briefs/) → QA (ревью) → задачи в Linear → платформенный агент (`In Progress`) → QA (`Done`)

## Compact Instructions
При компакте сохранять:
- Текущая задача и статус
- Изменённые файлы и ключевые решения
Выбрасывать: tool calls, промежуточные чтения, отладочный вывод.
