# Настройка доступов к соцсетям — пошаговое руководство

**Цель:** получить токены для автоматической публикации через API от имени VisiTime.

**Время:** ~4 часа на все платформы. Priority 1 (LinkedIn + Meta) — 2 часа, остальное можно позже.

**Куда вписывать:** все токены идут в файл `.env` в корне проекта (уже в `.gitignore`, не коммитится).

---

## 🎯 Приоритет — что настроить в первую очередь

| Priority | Платформа | Зачем | Сложность | Время |
|----------|-----------|-------|-----------|-------|
| **1** | **LinkedIn** | 5+ постов в плане (Build in Public, AI) | Средняя | 30 мин |
| **1** | **Meta (FB + IG)** | ~8 постов (IG основной RU-канал) | Средняя | 45 мин |
| **2** | **YouTube** | 3-4 поста (Shorts + tutorials) | Средняя | 30 мин |
| **2** | **Google Business Profile** | Local SEO | Лёгкая | 15 мин |
| **3** | **Twitter/X** | Треды в плане — но API платный $200/мес | Дорого | 15 мин |
| **4** | **TikTok** | Много постов — но approval 2-4 недели | Долго | 30 мин + wait |

**Рекомендация на старт:** настраиваем только Priority 1+2. Twitter публикуем вручную (5 мин в неделю). TikTok через Metricool до approval.

---

## 🟦 LinkedIn

### Что нужно
- Создать developer app
- Дать app разрешение постить от имени Yurii (личный) + Company Page (после создания)
- Получить Access Token (60 дней, с refresh-возможностью)

### Шаги

**1. Создай developer app:**
- Открой https://www.linkedin.com/developers/apps
- «Create app» → имя: `VisiTime Content Publisher`
- LinkedIn Page: пока любая (можно твоя личная), потом переключим на VisiTime Company Page
- Verify через email

**2. Добавь продукты (Products tab):**
- ✅ **Share on LinkedIn** — для публикации от твоего имени
- ✅ **Sign In with LinkedIn using OpenID Connect** — для идентификации
- 📝 **Advertising API** — запросить (нужен для Company Page posting)
- 📝 **Community Management API** — запросить (для Company Page мониторинга)

**3. Настрой OAuth 2.0 (Auth tab):**
- Authorized redirect URLs: `http://localhost:3000/callback` (для локальной авторизации)
- Scopes:
  - `w_member_social` — постинг от личного профиля
  - `w_organization_social` — постинг от Company Page
  - `r_organization_social` — чтение от Company Page
  - `openid`, `profile`, `email`

**4. Получи Access Token:**
- Нужна одноразовая OAuth-ссылка для авторизации. Простейший способ:
  - Использовать **LinkedIn API Postman collection** (бесплатно)
  - Или [скрипт](https://github.com/linkedin-developers/linkedin-api-python-client) на Python
  - Или через n8n node «LinkedIn» — автоматический OAuth flow

**5. Узнай свой Person URN:**
- GET `https://api.linkedin.com/v2/userinfo` с Access Token
- В ответе поле `sub` — это твой URN: `urn:li:person:XXXXXX`

**Что вписать в `.env`:**
```
LINKEDIN_CLIENT_ID=<из app Credentials>
LINKEDIN_CLIENT_SECRET=<из app Credentials>
LINKEDIN_ACCESS_TOKEN=<полученный OAuth-token>
LINKEDIN_REFRESH_TOKEN=<если есть>
LINKEDIN_PERSON_URN=urn:li:person:XXXXXX
LINKEDIN_COMPANY_PAGE_ID=<ID Company Page, после создания>
```

### ⚠️ Важно
- Access Token живёт 60 дней. Нужно будет обновлять.
- Advertising API approval для Company Page занимает 1-2 недели. До approval постим с личного профиля.
- Company Page создаётся вручную через https://www.linkedin.com/company/setup/new/ с личного аккаунта Yurii.

---

## 🟨 Meta (Facebook Page + Instagram Business)

### Что нужно
- Создать app в Meta for Developers
- Подключить Facebook Page и Instagram Business аккаунт
- Получить long-lived Page Access Token (60 дней, продлевается автоматически)

### Шаги

**1. Создай Facebook Page (если нет):**
- https://www.facebook.com/pages/create
- Категория: «Software» или «App Page»
- Имя: VisiTime

**2. Создай Instagram Business Account:**
- В Instagram app: Settings → Switch to Professional Account → Business
- Свяжи с FB Page (обязательно для API)

**3. Создай Meta app:**
- Открой https://developers.facebook.com/apps/
- «Create App» → Type: «Business» → имя: `VisiTime Publisher`

**4. Добавь продукты (Dashboard):**
- ✅ **Instagram Graph API** (Content Publishing)
- ✅ **Facebook Login for Business**
- ✅ **Pages API**

**5. Настрой разрешения (App Review):**
Нужны permissions:
- `pages_manage_posts` — постинг в FB Page
- `pages_read_engagement` — чтение метрик
- `pages_show_list` — список Pages
- `instagram_basic` — базовая инфа IG
- `instagram_content_publish` — постинг в IG

Эти permissions в **Development mode** работают сразу для админов app. Для production нужен **App Review** (1-2 недели).

**6. Получи Page Access Token:**
- https://developers.facebook.com/tools/explorer/
- Выбери свою app → User Token → выбери permissions выше
- Получи User Access Token
- Конвертируй в Long-Lived: `GET /oauth/access_token?grant_type=fb_exchange_token&client_id={APP_ID}&client_secret={APP_SECRET}&fb_exchange_token={USER_TOKEN}`
- Получи Page Access Token: `GET /me/accounts` с long-lived user token → в ответе `access_token` у нужной page

**7. Узнай Page ID и IG Business Account ID:**
- Page ID: `GET /me/accounts` — в ответе у каждой page поле `id`
- IG Business Account: `GET /{page-id}?fields=instagram_business_account`

**Что вписать в `.env`:**
```
META_APP_ID=<из app Settings → Basic>
META_APP_SECRET=<из app Settings → Basic>
META_PAGE_ACCESS_TOKEN=<long-lived page token>
META_PAGE_ID=<FB Page ID>
META_IG_BUSINESS_ACCOUNT_ID=<IG Business ID>
```

### ⚠️ Важно
- В Development mode работает только для админов app — для постинга на production-аудиторию нужен App Review
- IG Content Publishing: только **feed photos/videos, Reels, carousels**. Stories постить через API пока нельзя.
- Long-lived Page Token **не истекает** (если сам владелец не менял пароль). Это лучше LinkedIn.

---

## 🟥 YouTube (+ Google Business Profile — один OAuth)

### Что нужно
- Google Cloud project с enabled YouTube Data API v3
- OAuth 2.0 credentials
- Refresh token для долгоживущего доступа

### Шаги

**1. Создай Google Cloud project:**
- https://console.cloud.google.com/
- New Project → имя: `VisiTime Publisher`

**2. Enable APIs:**
- APIs & Services → Library
- Enable: **YouTube Data API v3**
- Enable: **Google My Business API** (для GBP)

**3. OAuth consent screen:**
- External → fill required fields (app name VisiTime, support email, logo)
- Scopes:
  - `https://www.googleapis.com/auth/youtube.upload`
  - `https://www.googleapis.com/auth/youtube`
  - `https://www.googleapis.com/auth/business.manage` (для GBP)

**4. Create OAuth credentials:**
- APIs & Services → Credentials → Create → OAuth client ID
- Type: **Desktop app** (проще всего для CLI)
- Скачай `client_secret.json` → положи в `secrets/google-client-secret.json`

**5. Получи Refresh Token:**
- Запусти один раз локально Python-скрипт с google-auth-oauthlib
- Или через n8n OAuth node
- Или вручную через OAuth Playground: https://developers.google.com/oauthplayground
- Refresh Token — **вечный** пока пользователь не отозвал

**6. Узнай Channel ID:**
- GET `https://www.googleapis.com/youtube/v3/channels?mine=true&part=id` с access token

**Что вписать в `.env`:**
```
GOOGLE_CLIENT_ID=<из OAuth credentials>
GOOGLE_CLIENT_SECRET=<из OAuth credentials>
GOOGLE_REFRESH_TOKEN=<полученный refresh token>
YOUTUBE_CHANNEL_ID=<твой channel ID>
GMB_ACCOUNT_ID=<Google Business Profile account>
GMB_LOCATION_ID=<VisiTime location>
```

### ⚠️ Важно
- YouTube API имеет quota: **10,000 units/day**. Upload видео = ~1600 units. Хватит на 6 видео/день — нам более чем достаточно.
- Один refresh token работает для YouTube + GBP — удобно.

---

## ⚫ Twitter/X (ОПЦИОНАЛЬНО — платный!)

### Ситуация
- Free tier **не позволяет постить**
- Basic tier: **$200/мес** — 3,000 твитов/мес
- Pro tier: $5,000/мес

### Рекомендация
**На старте не подключаем API.** Публикуем 4 твита в неделю вручную (5 мин).

### Если всё же нужно:
- https://developer.twitter.com/ → apply for Basic tier ($200/mo)
- Create app → v2 credentials
- Scopes: `tweet.read`, `tweet.write`, `users.read`

**В `.env`:** `TWITTER_API_KEY`, `TWITTER_API_SECRET`, `TWITTER_ACCESS_TOKEN`, `TWITTER_ACCESS_TOKEN_SECRET`, `TWITTER_BEARER_TOKEN`.

---

## 🎵 TikTok (ждать approval 2-4 недели)

### Путь
1. https://developers.tiktok.com/ → Register → Create app
2. Products → Content Posting API → Submit for approval
3. Apple Review process: 2-4 недели
4. После approval — OAuth flow + токен

### Fallback на период ожидания
**Metricool** — €18/мес, подключается через OAuth к TikTok (как пользователь, не как approved app), постит автоматически.

- https://metricool.com/ → signup → TikTok connect
- API key в Settings → API
- Вписать `METRICOOL_API_KEY` и `METRICOOL_USER_ID` в `.env`

---

## 📊 GA4 (для Фазы 0 Analytics)

### Шаги

**1. Создать Service Account:**
- Google Cloud Console → IAM & Admin → Service Accounts → Create
- Имя: `visitime-analytics`
- Role: **Analytics Viewer**

**2. Скачать JSON:**
- Keys → Add Key → Create New Key → JSON
- Положи в `secrets/ga4-service-account.json`

**3. Добавить service account в GA4:**
- GA4 → Admin → Property Access Management → Add
- Email service account → Role: Viewer

**4. Узнать Property ID:**
- GA4 → Admin → Property Settings → Property ID

**В `.env`:**
```
GA4_PROPERTY_ID=<number>
GA4_SERVICE_ACCOUNT_JSON_PATH=./secrets/ga4-service-account.json
```

---

## ✅ Чек-лист после настройки

Проверка работоспособности каждого канала:

```bash
# Тестовые команды в Claude Code (я выполню)

# LinkedIn
протестируй LinkedIn API — опубликуй тестовый пост «Test from VisiTime Publisher» на моей странице

# Meta FB
протестируй Meta FB API — опубликуй тестовый пост на VisiTime Page

# Meta IG
протестируй Instagram API — опубликуй тестовое фото на @visitime

# YouTube
протестируй YouTube API — получи инфу о канале

# GA4
протестируй GA4 — покажи overview за вчера
```

Если всё работает — можно запускать полный pipeline.

---

## 🔒 Безопасность — правила

1. **НИКОГДА не коммить `.env` в git** — проверка: `git status` не должен показывать `.env`
2. **НИКОГДА не пересылать токены по email / messenger** — только через Bitwarden / 1Password
3. **Каждый токен имеет минимум-scope** — не давать permissions больше чем нужно
4. **Refresh токены хранить в Bitwarden** — как backup
5. **Ротация:** если токен утёк — сразу revoke в app settings и сгенерировать новый
6. **Separate apps для dev/prod** — на старте можно одну, но когда будет production-трафик разделить

---

## 🆘 Что делать если застрял

Для каждой платформы я могу:
- Написать Python-скрипт для OAuth-флоу
- Помочь настроить n8n workflow с OAuth node
- Разобрать конкретную ошибку API

Просто напиши:
```
помоги получить LinkedIn access token — застрял на шаге X
```

или

```
у меня есть client_id и client_secret Meta — проведи через получение Page Access Token
```
