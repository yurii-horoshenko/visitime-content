# Trend & Competitor Watching — еженедельный разведывательный дайджест

**Цель:** раз в неделю получать короткую сводку «что происходит на рынке» → использовать в планах следующих недель.

**Агент:** `trend-watcher` (новый) · запускается каждый понедельник автоматически

---

## Три источника

### 1. Конкуренты на польском рынке

Кого мониторим:
- **Booksy** (booksy.com) — доминирующий игрок в PL
- **Versum** (versum.com)
- **LocalBookings** / Treatwell (глобал, есть PL)
- **Indywidualne** салоны с сильным Instagram — 5–10 флагманов

Что смотрим:
- Какие фичи анонсируют
- Какие тарифы меняют
- Какой контент публикуют, что залетает (саве-мемы, видео-типы)
- Что появляется нового на их сайтах (новые страны, новые продукты)

### 2. Beauty-тренды в соцсетях

Источники:
- TikTok Discover + `#nails`, `#manicure`, `#fryzjer`, `#saloonkrasoty`
- Instagram Explore по beauty-хэштегам
- Google Trends — запросы по beauty PL+RU
- Pinterest Predicts (раз в год, но важно)

Что вычленяем:
- **Тренды контента** — какой формат видео сейчас залетает (например: «get ready with me», «stitched reactions», «before/after»)
- **Тренды услуг** — какие процедуры набирают популярность (мы можем делать посты «как найти мастера X в приложении»)
- **Тренды звуков** — trending audio в TikTok (для `tiktok-strategist`)

### 3. SaaS / startup тренды

Источники:
- ProductHunt — еженедельный топ
- Hacker News frontpage
- Indie Hackers — milestone-посты
- Dev.to top trending

Что вычленяем:
- Интересные сценарии Build in Public — что делают другие основатели
- Tech-темы для LinkedIn (например: хайп вокруг конкретного AI-инструмента)
- Идеи для «how we built» постов

---

## Weekly Digest — формат

Понедельник 08:00 → в `docs/ops/trend-digest/YYYY-MM-DD.md`:

```markdown
# Trend Digest — 2026-05-04

## 🏢 Competitors
- Booksy запустил новую фичу X — подумать, надо ли нам
- Versum повысили цены на 20% — возможность для нашего pricing-поста
- 3 конкурента опубликовали похожий контент про фильтры поиска — **не делаем дубликат**, ищем свой угол

## 💄 Beauty trends this week
- На TikTok залетает формат «a day in the life of a nail tech» — добавить в план
- Тренд «hair gloss treatment» в салонах Варшавы — пост-объяснение
- Trending audio для скетчей: [link, expires in 2 weeks]

## 💡 Startup / SaaS
- Indie Hacker $30k MRR SaaS с 1 разработчиком — backstory для Build in Public inspiration
- Hacker News обсуждает Kotlin Multiplatform — хороший момент для нашего KMM-поста

## Рекомендации на следующую неделю
1. **Добавить пост** про hair gloss trend (Beauty Tips)
2. **Использовать trending audio** для скетча «Клиент не берёт трубку»
3. **Не делать** пост про generic «зачем booking app» (конкуренты засрали тему, подождём 2 недели)
```

---

## Когда это ценно

- **До планирования недели** — чтобы план учитывал контекст рынка
- **Во время кризиса вдохновения** — есть список «что сейчас работает»
- **При запуске рекламы** — знаем что у конкурентов работает, копируем подход

---

## Команды в Claude Code

```
сделай weekly trend digest — что у конкурентов, в beauty, в startup-сцене
```

```
мониторь Booksy — что они выкладывают последние 2 недели, что работает лучше всего
```

```
найди trending audio в TikTok Poland для скетча про салонные боли
```
