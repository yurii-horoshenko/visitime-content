# Контент-календарь VisiTime

## Документы

1. **[00-overview-8-weeks.md](00-overview-8-weeks.md)** — общий план на 8 недель, 4 фазы (Warmup → Validation → Momentum → Pre-launch)
2. **[01-week-01-02.md](01-week-01-02.md)** — детальный план первых 2 недель: **14 постов** по дням, платформы, агенты, файлы
3. **[02-shooting-plan.md](02-shooting-plan.md)** — план съёмок: почти весь контент без камеры, **1 короткий съёмочный час** покрывает видео недель 1–2

## Связанные документы

- `docs/strategy/01-content-strategy.md` — общая стратегия (цели, KPI, аудитории, языки RU/EN)
- `docs/pillars/content-pillars.md` — **6 рубрик:** Стройка на виду · AI & разработка · Фичи VisiTime · Use Cases · Категории услуг · Закулисье
- `docs/brand/voice-guide.md` — единый голос бренда
- `docs/briefs/briefs-build-in-public.md` — существующие сценарии 11 эпизодов
- `services-dashboard-v2.html` — статус маркетинговых аккаунтов
- `content-plan.html` — визуальный план публикаций
- `workflow-dashboard.html` — 9 фаз pipeline

## Следующие шаги

1. **Завести аккаунты** по services-dashboard (Cloudflare Email → GA4 → Meta BM → FB/IG/TikTok)
2. **Запустить pipeline** через агентов Claude (workflow в workflow-dashboard.html):
   - `product-manager` → декомпозиция 14 постов недели 1-2 в Linear
   - `social-media-strategist` + `tiktok-strategist` → уточнить хуки
   - `narratologist` → 5-актовая структура для видео
   - `content-creator` → брифы в `docs/briefs/week-01/` с опорой на brand voice guide
   - `qa-engineer` → ревью
3. **Проверить готовность фичей** в билде до публикации killer-постов (фильтр языка, фильтр гендера) — см. Risk-чек в content-plan.html
4. **Setup продакшн** — нужна короткая съёмка только для 1 поста (Use Case «девочка с ресницами»)

## Ритм обновления

- **Понедельник:** Фаза 0 — analytics прошлой недели + 3 рекомендации + trend digest
- **Пятница:** брифы на следующую неделю готовы
- **Воскресенье:** план на неделю + готовые материалы в `content/`
