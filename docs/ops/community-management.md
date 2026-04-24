# Community Management — Фаза 8 Engagement

**Цель:** не потерять конверсию после публикации. Отвечаем на комменты/DM в течение 4 часов в рабочее время, 24 часов максимум.

**Агент:** `community-manager` (role — расширение `content-creator`)

---

## Почему это критично

В beauty-нише девушки пишут в Instagram DM вопросами «как записаться», «в каком салоне есть ваш сервис». Если отвечаем через 3 дня — она уже забыла и ушла к конкуренту.

Для B2B на LinkedIn: владелец салона оставил комментарий «а как у вас с оплатой?» → отвечаем через час → он на демо-колле через неделю. Не ответили → лид мёртв.

---

## Ежедневный ритуал (15 мин утро + 15 мин вечер)

### Утренний чек (9:00)
- [ ] Instagram DM — проверить inbox
- [ ] Instagram комменты под постами за последние 24 часа
- [ ] TikTok комменты (особенно под скетчами — часто обсуждения)
- [ ] LinkedIn уведомления + комментарии
- [ ] Twitter DM + mentions
- [ ] Facebook комментарии и сообщения Page

### Вечерний чек (19:00)
- Повтор, плюс:
- [ ] Ответить на комментарии, которые пришли в течение дня
- [ ] Пометить лиды в Linear как VIS-LEAD-XXX (B2B) / VIS-USER-XXX (B2C)

---

## Saved Replies — шаблоны ответов

Хранятся в `content/replies/` и используются в нативных DM-инструментах соцсетей (Instagram Saved Replies, FB Page quick replies).

### B2C — Instagram / TikTok DM

**«Как записаться?»** [RU]
> Привет! 🧡 Мы сейчас в закрытой бете в Варшаве. Оставь email — дадим доступ одной из первых: [link]

**«В каком салоне вы есть?»** [RU]
> Отличный вопрос! Первые салоны-партнёры в Варшаве подключаются в мае. Какой район тебе удобен — отмечу, когда откроем запись: [link]

**«Это бесплатно?»** [RU]
> Для клиентов — да, бронирование всегда бесплатно. Платят только салоны, и только процент с реально пришедших клиентов.

### B2B — LinkedIn / Facebook

**«How does pricing work?»** [EN]
> Good question. During beta — free for the first 3 months, then 2% per completed booking. No subscription, no setup fee. Want to jump on a 15-min demo call? Here's my calendar: [link]

**«Do you have API?»** [EN]
> Yes, for salon owners who want to integrate with their existing tools. Full docs here: [link]. What's your current stack?

**«Are you available in city X?»** [EN]
> Launching in Warsaw, Krakow, Wroclaw first (May–June 2026). City X is in the roadmap for Q3. Want me to ping you when we open there?

---

## Что эскалируем тебе (не отвечает агент)

| Ситуация | Action |
|----------|--------|
| Публичная негативная критика | В Linear как VIS-OPS-XXX, статус urgent, ответ только с твоего согласия |
| Вопрос по продукту, на который нет ответа в docs | Агент пишет «спрошу у founder, вернусь в течение дня» + Linear task |
| Лид B2B с серьёзным интересом (упоминает размер салона, готовность платить) | Лид в CRM/Linear, копия тебе в Telegram/email |
| Юридический / legal вопрос | Без ответа, эскалация сразу |
| Жалоба на баг / оплату | В Linear на dev-команду, клиенту — «спасибо, разбираемся, ответ за 24ч» |

---

## Метрики community management

Еженедельно в weekly report:
- **Response rate** (ответили в течение 24ч / всего получили) — таргет 95%
- **Avg response time** — таргет < 4 часов в рабочее время
- **DM → lead conversion** — сколько DM превратилось в лиды/демо/установки
- **Sentiment** (позитив / нейтрал / негатив) — простая ручная разметка

---

## Инструменты

- **Meta Business Suite** — единый inbox для FB + IG (бесплатный)
- **Metricool inbox** (позже) — единый inbox для всех соцсетей включая TikTok/LinkedIn
- **Saved replies** — нативные в каждой соцсети + master-копия в `content/replies/`

---

## Команда в Claude Code

```
проверь активность и ответь на комменты/DM за сегодня — используй saved replies из content/replies/
```

```
сделай weekly community report — response rate, avg time, leads, sentiment
```
