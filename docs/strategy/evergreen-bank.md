# Evergreen Content Bank — страховка от творческого кризиса

**Цель:** всегда иметь 10–15 готовых к публикации «вечнозелёных» постов. Когда заболел / аврал с разработкой / нет идей — достаём из банка.

**Папка:** `content/evergreen/ready/` — готовые к публикации · `content/evergreen/drafts/` — в работе

---

## Что такое evergreen

Пост актуален через месяц и через год. Не привязан к дате, trending sound'у или текущему событию.

**✅ Evergreen темы:**
- «Как работает фильтр по языку мастера»
- «5 причин почему VisiTime удобнее Instagram-поиска»
- «Что должно быть в профиле хорошего мастера»
- «Как solo-провайдеру получить первых клиентов»
- «Feature walkthrough: онлайн-бронирование за 28 секунд»
- «Feature walkthrough: управление командой мастеров»
- «Feature walkthrough: CRM для бизнеса»
- «Как защищены отзывы в VisiTime»

**❌ Не evergreen:**
- «Итоги этой недели»
- «Тренды весны 2026»
- «Как мы зафиксили баг вчера»
- «Happy New Year»

---

## Минимум в банке

| Формат | Минимум в запасе | Актуальный запас |
|--------|-----------------|------------------|
| **IG carousels RU** | 5 | 0 → заполнить |
| **LinkedIn posts EN** | 4 | 0 → заполнить |
| **Instagram Reels скринкасты** | 3 | 0 → заполнить |
| **TikTok features/use cases RU** | 3 | 0 → заполнить |
| **Twitter threads EN** | 3 | 0 → заполнить |

**Итого ~18 единиц** — покрывает 2 недели если совсем ничего не делать.

---

## План наполнения банка

**Раз в 2 недели** — отдельная «рабочая сессия evergreen» по понедельникам:
- `content-creator` + `instagram-curator` готовят 3–5 evergreen постов
- Складывают в `content/evergreen/drafts/`
- `qa-engineer` проверяет → перемещает в `ready/`

---

## Когда использовать

### Плановое использование
- Раз в 2–3 недели в основной план вставляем 1–2 evergreen — они стабилизируют охваты

### Аварийное использование
- Ты в отпуске / болеешь / деплой горит → агент достаёт из банка по расписанию
- Планируемый пост провалился на этапе 4 (ты не успел записать видео) → замена из банка

### Репурпосинг evergreen
- Evergreen пост сработавший хорошо → репурпосится в 4 формата (см. `content-repurposing.md`)

---

## Тематический список evergreen для банка

### Instagram carousels RU (5)
1. «Как выбрать мастера по языку — пошагово в VisiTime»
2. «6 категорий услуг, о которых ты не думал, что их можно забронировать онлайн»
3. «Solo-мастер: как превратить DM-хаос в профессиональную витрину»
4. «Все записи дня на одной карте — пример из жизни»
5. «Как VisiTime защищает от фейковых отзывов»

### LinkedIn posts EN (4)
1. «Why language filter matters: the immigrant founder's perspective»
2. «5 reasons solo service providers lose clients (and how one platform fixes all)»
3. «Building a universal service booking platform — lessons after 1 month»
4. «From DM chaos to online storefront: the home-based masters playbook»

### Instagram Reels / TikTok / YT Shorts (3)
1. «Бронирование за 28 секунд» (скринкаст)
2. «Регистрация solo-провайдера за 3 минуты» (скринкаст EN)
3. «Что внутри приложения VisiTime — тур за 60 секунд»

### TikTok features/use cases RU (3)
1. «3 фильтра, которые спасут от плохого мастера»
2. «Как девочки находят клиенток без Instagram-сторис»
3. «Все записи на неделю в одном приложении — это возможно»

### Twitter threads EN (3)
1. «I'm a solo founder building a universal service booking app. Here are 10 lessons from month 1.»
2. «How 13 AI agents replaced my team and tripled my velocity. Thread.»
3. «The immigrant pain that launched a startup: finding service providers who speak your language.»

---

## Команды в Claude Code

```
наполни evergreen bank — сделай 5 постов по темам из evergreen-bank.md
```

```
покажи текущий evergreen bank и что не хватает до минимума
```

```
используй evergreen пост на замену запланированному посту #N
```
