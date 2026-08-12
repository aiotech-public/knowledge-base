---
id: push-notifications
title: Push / Notifications Flow (How-to)
description: Создание Push Template, Notifications Flow и Push Distribution. Telegram-уведомления — Sender Provider + Buyer TG Chat ID.
doc_type: how-to
builds: [erp]
related: [notifications-flow, distributions-model, postback-generator, conversion-model, destinations, distributions, mechanics-pwa]
language: ru
updated: 2026-08-11
---

# Push / Notifications Flow (How-to)

Процедуры по push-уведомлениям и Marketing Flows: как собрать follow-up пуш, подключить его к кампании, настроить Telegram-уведомления. Концепт (типы пушей, ноды Notifications Flow, Push Distribution = Remarketing Content) — в [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

---

## TL;DR

- Полный путь Follow Up Push: `Push Template` → `Notifications Flow` → `Push Distribution (Remarketing Content)`.
- **Notification Flow на Conversion Type** — Telegram-нотификация при конверсии заданного типа (Invalid / Reject Lead и т.д.).
- **Telegram Sender Provider** — `Marketing → Sender providers → + Sender provider` (Integration type: Telegram; API key + Bot Name).
- **Buyer TG Chat ID** — **кастомное** поле визита (имя — пример, по дефолту в шаблонах тенантов его нет), заполняется через `Fill Field` distribution по `Campaign Owner`.

---

## Как собрать Follow Up Push: Push Template → Notifications Flow → Distribution

### Шаг 1 — создать Push Template

**Путь:** `Marketing → Message templates` → создать `Push Template`.

**Поля:**

- **Name** — имя.
- **Push Title** — заголовок (короткий).
- **Push Message** — тело.
- **Push Icon** — картинка (из CDN).

### Шаг 2 — настроить Notifications Flow

**Путь:** `Marketing → Flows → создать новый Notifications Flow`.

**Шаги:**

1. Добавить ноды: `Start → Wait → Check Conversion → Push → (Repeat or Finish)`.
2. На ноде `Push` → cogwheel → выбрать **Push Template** (одиночный) **или** дистрибуцию (рекомендуется).
3. На ноде `Wait` — задать задержку (минимум **60 секунд**; для первого Check Conversion обычно 3600+ сек).
4. Сохранить.

### Шаг 3 — создать Push Distribution (Remarketing Content)

**Путь:** `Settings → Distributions → +Add → Remarketing Content` (или `Flow Content`).

**Шаги:**

1. Выбрать **Notifications Flow**, который дистрибуция обслуживает.
2. **Папка на каждый Push-шаг** в флоу (Reg 1, Reg 2, …).
3. На папке — `Flow State` (соответствует ноде в флоу).
4. Внутри папки — `Strategy: Weights` или `First`.
5. Добавить `Add Push Message` ноды — выбрать Push Templates.

См. эвристику в [models/distributions-model.md](../models/distributions-model.md) → «Notifications Flow — сначала флоу, потом одну дистрибуцию».

Также см. [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

## Подключение Notifications Flow к Campaign Flow

Когда: Notifications Flow построен, его нужно «вшить» в Campaign Flow (запускать на визитах кампании).

**Путь:** `Edit Flow (Campaign Flow) → +Step → Notifications Flow node → выбрать NF`. Обычно ставится **после PWA-шага** или после `Destination`.

Notifications Flow стартует, **даже если у визита нет подписки на push** — подписка проверяется только в момент попытки отправить push. Если подпишется по ходу — пуши пойдут.

Также см. [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

## Notification Flow на Conversion Type (Telegram-уведомления)

Когда: при определённом типе конверсии (Invalid / Reject / Sale) нужно отправить Telegram-нотификацию владельцу кампании (баеру).

**Путь:** `Settings → Conversion types → <type> → Notification Flow → выбрать` (в ERP типы конверсий открываются и со страницы конверсий — шеврон у `Postback generator` → `Manage types`, см. [how-to/postback-generator.md](postback-generator.md)).

**Шаги:**

1. Открыть Conversion Type (например, `Invalid Lead`, `Reject Lead`).
2. Выбрать в поле `Notification Flow` ранее созданный Marketing Flow с нодой-каналом `Telegram`.
3. `Save`.

Каждый раз при создании конверсии этого типа — флоу запустится и пошлёт сообщение.

Также см. [models/conversion-model.md](../models/conversion-model.md) → Notification Flow атрибут.

### Telegram Sender Provider

Когда: первое подключение Telegram-канала. Без Sender Provider нода `Telegram` (в флоу-редакторе) не сможет отправить.

Тот же Telegram Sender Provider используется и для `Telegram Destination` (трафик в TG-бот/канал, `Behaviour` `Bot -> Channel` / `Channel`) — см. [how-to/destinations.md](destinations.md).

**Путь (UI 2026-06-06):** `Marketing → Sender providers → + Sender provider → Integration type: Telegram`. В дропдауне `Integration type` также: **Mailgun** (email), **Twilio** (SMS), **Prelude Bridge** (SMS) — но email/SMS-доставка не работает (заглушка), рабочий из этих провайдеров только Telegram (см. [mechanics/notifications-flow.md](../mechanics/notifications-flow.md) → «Email/SMS-каналы — доставка не работает»). Выбор типа раскрывает поля учётки именно этого провайдера.

**Поля провайдера (Telegram):**

| Поле | Что делает |
|---|---|
| **Name** | Имя провайдера (обяз.) |
| **Description** | Описание (опц.) |
| **Bot API Key** | Токен бота от `@BotFather` (обяз.) |
| **Bot Name** | Username/имя бота для отображения (обяз.) |
| **Start Message** | Сообщение, которое бот шлёт юзеру при первом заходе (`/start`) |
| **Joined Message** | Сообщение после вступления юзера в канал |
| **Start conversion type** | Тип конверсии, который фается, когда юзер запустил бота |
| **Join conversion type** | Тип конверсии при вступлении в канал — это **«конверсия подписки»** |
| **Join Channel** | Целевой канал (ссылка/ID), куда зовём вступить |
| **Join Channel Header Text** | Заголовок в приглашении вступить (режим `Bot -> Channel`) |
| **Join Channel Button Text** | Текст кнопки-приглашения вступить |
| **Chat ID Field** | Поле визита, куда бот пишет `chat_id` юзера |
| **Username Field** | Поле визита, куда бот пишет `username` юзера |

Для простой Telegram-нотификации (нода `Telegram` шлёт шаблон) достаточно **Bot API Key + Bot Name**. Остальные поля — про acquisition-флоу `Telegram Destination`: приветствие → приглашение в канал → фиксация подписки.

`Chat ID Field` / `Username Field` указывают на **кастомные** поля визита — по дефолту в шаблонах тенантов их нет, заведи заранее. `Join conversion type` сработает, только если **бот — админ канала** (иначе Telegram не отдаёт событие вступления).

### Заполнить `Buyer TG Chat ID` через Fill Field Distribution

Когда: разные Telegram-чаты для разных байеров — чтобы статусы по кампаниям каждого владельца уходили именно его баеру, в его чат.

*(Поле `Buyer TG Chat ID` — кастомное; имя приведено как пример, по дефолту в шаблонах тенантов его нет. Заведи его в кастомных полях визита.)*

**Путь:** `Settings → Distributions → <Fill Field distribution> → Add Fill Field`.

**Шаги:**

1. Создать (или открыть) Fill Field Distribution.
2. Добавить правило: `Campaign Owner == <Buyer X>` → `Buyer TG Chat ID = <chat_id>`.
3. Повторить для каждого байера.
4. Сохранить и убедиться, что в Campaign Flow есть шаг `Fill Fields` или `Fields by Distribution`, который применяет эту дистрибуцию.

Также см. [how-to/distributions.md](distributions.md) → Fill Field Distribution, эвристика «сначала по байеру».

## Частые ошибки при настройке пушей и Notifications Flow

- **Push не приходит — а Flow стартует.** Это норма: NF стартует независимо от подписки. Подписка проверяется только при отправке. Проверь, что визит подписан (Push Subscribe Toggle на лэнде + подписался в реале).
- **Подпись на одном домене, отправка с другого.** **Технически невозможно** — подписка привязана к домену.
- **Wait < 60 секунд.** UI не даёт. Минимум 60 сек.
- **«Хочу видеть CTR / open-rate пушей».** Аналитика по пушам сейчас — только логи в `Marketing → Messages`. CTR / open-rate / конверсия после пуша **не трекаются**.
- **Несколько Notifications Flow на одну точку Campaign Flow.** Можно — стартуют одновременно. Но проще один с развилками внутри.

## Смежные темы

- [mechanics/notifications-flow.md](../mechanics/notifications-flow.md) — концепт: типы пушей (Manual / Scheduled / Follow Up), Notifications Flow ноды, каналы Push / Telegram / Email / SMS (**работают Push и Telegram; Email/SMS-доставка не работает**), Sender Providers (Mailgun / Twilio / Prelude Bridge / Telegram), Message-статусы (Requested/Processing/Done/Failed), AIO Push Subscribe макрос, Telegram-стек.
- [models/distributions-model.md](../models/distributions-model.md) — Remarketing Content Distribution + эвристика «одна дистрибуция на флоу».
- [mechanics/pwa.md](../mechanics/pwa.md) — PWA + Service Worker.
- [models/conversion-model.md](../models/conversion-model.md) — Notification Flow атрибут на Conversion Type.
