# Metrics Feedback Loop — Фаза 0

**Цель:** каждый понедельник смотрим метрики прошлой недели и **корректируем план** текущей. Без этого pipeline работает вслепую.

**Агент:** `analytics-specialist` (роль — расширение `seo-specialist` + data-анализ)

---

## Понедельничный ритуал (30 мин)

Запускается **до** Фазы 1 планирования недели. Результат идёт в `product-manager` как инпут.

### Шаг 1 — Собрать метрики по каналам (15 мин, автоматически)

Агент пулит через API:

| Канал | Что собираем |
|-------|--------------|
| **Instagram** | Reach, impressions, likes, comments, saves, shares, profile visits, link clicks |
| **TikTok** | Views, likes, comments, shares, average watch time, follows from video |
| **LinkedIn** | Impressions, unique views, reactions, comments, reshares, click-through |
| **Twitter/X** | Impressions, engagements, profile clicks, url clicks |
| **YouTube** | Views, watch time, subscribers gained, CTR, avg view duration |
| **Facebook** | Reach, reactions, comments, shares, page follows |
| **Google Analytics** | Sessions, source/medium, conversions (install, signup) |

### Шаг 2 — Ранжировать посты (5 мин)

Каждому посту присваивается оценка по двум осям:

| Категория | Engagement Rate | Действие |
|-----------|----------------|----------|
| 🔥 **Hit** | ≥ 2× среднего | Кандидат на **repurposing** (4 других формата) + **boost** (Ads) |
| ✅ **Solid** | 1–2× среднего | Оставляем в плане, тема «работает» |
| 😐 **Average** | 0.5–1× среднего | Возможно неудачное время или крючок — пересмотреть |
| ❌ **Flop** | < 0.5× среднего | Тему/формат не повторяем, разбираемся почему |

**Среднее** — скользящее по последним 4 неделям в этом канале. Первые 2 недели — сравниваем с baseline benchmarks (IG ER 1%, TikTok ER 5%).

### Шаг 3 — 3 рекомендации (10 мин)

Агент выдаёт короткий отчёт:

```markdown
# Week 2 Analysis → Week 3 Adjustments

## Highlights
- Hit: пост #4 «No-show cost» на LinkedIn (ER 4.2% vs avg 1.8%) — **repurpose в IG carousel + TikTok data-видео**
- Flop: пост #2 «5 красных флагов» IG carousel (ER 0.3%) — **причина:** обложка не продала, слишком общая тема

## 3 изменения на следующую неделю
1. **Удвоить data-посты** на LinkedIn (замечено: цифры дают 2× ER)
2. **Переделать обложки** IG каруселей — тестируем обложку с лицом vs текст-only
3. **Сократить скринкасты** с 45с до 25с — CTR падает после 20с
```

### Шаг 4 — Корректировка плана (5 мин)

Рекомендации идут в `product-manager`, он обновляет `docs/calendar/NN-week.md` и переставляет задачи в Linear.

---

## Weekly Report шаблон

Сохраняется в `docs/calendar/weekly-reports/week-NN.md`:

```markdown
# Week NN Report (YYYY-MM-DD to YYYY-MM-DD)

## Top-line
- Reach: X (+/- %)
- Engagement rate avg: X%
- Followers gained: +X
- Link clicks: X → installs: X → signups: X
- Community: X DMs answered, X leads

## Best post
- Пост #N «Title» — ER X%, reach Y
- Why worked: [hypothesis]

## Worst post
- Пост #N «Title» — ER X%
- Why flopped: [hypothesis]

## Adjustments for next week
1. ...
2. ...
3. ...

## KPI vs goal
| KPI | Week | Goal (8 weeks) | Status |
|-----|------|---------------|--------|
| IG followers | X | 1000 | Y% |
| TikTok followers | X | 2000 | Y% |
| B2B waitlist | X | 50 | Y% |
| B2C installs | X | 500 | Y% |
```

---

## Триггер-правила (автоматические)

Если метрика пересекает порог — срабатывает автоматика:

| Триггер | Action | Кто делает |
|---------|--------|-----------|
| Пост ≥ 2× avg ER в первые 48ч | Запуск **repurposing** (Фаза 3 повторная в 4 формата) | `content-creator` |
| Пост ≥ 2× avg ER в первые 48ч **И** есть link clicks | Кандидат на **Meta/TikTok Ads boost** €50–100 | Notification тебе + задача в Linear |
| Followers growth < 5%/нед 2 недели подряд | Алерт «стратегия нуждается в пересмотре» | Задача в Linear для тебя |
| Community response time > 8ч | Алерт «community management отстаёт» | Задача в Linear |

---

## Команды в Claude Code

```
сделай weekly report — метрики прошлой недели и 3 рекомендации на следующую
```

```
проанализируй пост #N — почему сработал/не сработал, что учесть
```

```
какие посты прошлой недели — кандидаты на repurposing и paid boost
```
