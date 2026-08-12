---
id: notifications-flow
title: Notifications Flow / Push / Marketing
description: Marketing Flow для follow-up push'ей, три типа пушей, Push Distribution, Telegram-нотификации.
doc_type: mechanic
builds: [erp]
related: [marketing-flow, notification-center, visit-lifecycle, flow-model, sdk, conversion-model, distributions-model, mechanics-pwa, glossary, push-notifications]
language: ru
updated: 2026-08-11
---

# Notifications Flow / Push / Marketing

Push-уведомления и Marketing-флоу в AIO работают через **`Notification Flow`** (в UI так и называется; `Marketing Flow` — синоним, см. [models/marketing-flow.md](../models/marketing-flow.md)) и инфраструктуру `Marketing`. Из четырёх ноды-каналов (`Push` / `Telegram` / `Email` / `SMS`) в проде работают **Push и Telegram**; `Email` и `SMS` — заглушка (ноды и Sender Providers в UI есть, но доставка не работает, см. «Email/SMS-каналы — доставка не работает» ниже).

---

## Какие типы пушей существуют в AIO и что уже работает

Три типа пушей по способу отправки. Реализованы только Follow Up.

| Тип | Что это | Статус |
|---|---|---|
| **Manual** | Пуш руками по кнопке | Не реализован |
| **Scheduled** | Запланированные пуши по расписанию | Не реализован |
| **Follow Up** | Реализованы через Notifications Flow | **Реализованы** |

## Что такое Notifications Flow и его ноды

Notification Flow (синоним — `Marketing Flow`) — отдельный флоу рассылки, не Campaign Flow. Исполняется **параллельно** Campaign Flow. Запускается, когда Campaign Flow доходит до ноды `Notifications Flow node`.

### Шаги (ноды) Notifications Flow

| Нода | Тип во флоу | Что делает |
|---|---|---|
| **Start** | `Start` | Точка начала |
| **Finish** | `Finish` | Точка завершения |
| **Check** | `Plan Check` | Проверка условия (поле визита + оператор + значение) |
| **Check Conversion** | `Plan Conversion Exists Check` | Проверка наличия конверсии заданного типа — разветвление |
| **Wait** | `Plan Wait` | Задержка в секундах |
| **Push** | `Plan Push` | Отправка пуша. **Payload:** конкретное сообщение из Push Template **или** дистрибуция Remarketing Content (рекомендуется дистрибуция) |
| **Repeat** | `Plan Repeat` | Цикл, **макс. 10 повторов**; повтор идёт транзишеном `Repeat`, выход из цикла — транзишеном `End repeat` (обычно обратно в Check Conversion) |

### Как Notifications Flow node подключается к Campaign Flow

Нода в **Campaign Flow** для подключения Marketing Notifications Flow. Обычно ставится **после PWA-шага**. Открывается cogwheel-ом, в неё выбирается ранее созданный Notifications Flow.

Notifications Flow — прозрачный проход. Когда визит проходит ноду Notifications Flow в Campaign Flow — он не «застревает» в Marketing-флоу, а проходит дальше; клик по самому пушу возвращает визита в его **текущий** State (а не в начало флоу).

### Подписка на пуш проверяется только при отправке, не при старте флоу

`Notifications Flow` стартует, даже если у визита **нет подписки на пуши** — подписка проверяется только в момент попытки отправить push. Если визит подпишется по ходу — пуши пойдут.

### К одной точке Campaign Flow можно прицепить несколько Notifications Flow

К одной точке Campaign Flow можно прицепить несколько `Notifications Flow` — стартуют одновременно. Обычно проще один с развилками внутри.

### Подписаться на пуш можно только на лэнде

Подписаться на push визит может **только на лэнде** (через `Push Subscribe Script`). После ухода с лэнда подписаться нельзя.

### Подписка на пуш привязана к домену

Подписка на push привязана к **домену**, на котором юзер подписался. Подписаться на одном, слать с другого — **технически невозможно**.

### Аналитика по пушам — только Messages

Аналитика по пушам — только логи в `Marketing → Messages`. CTR, open-rate, конверсия после пуша **не трекаются**. Внутри Notifications Flow для retargeting-развилок можно использовать кастомное поле (напр. `offer_pushed`) в `Check Conversion`.

### Бан домена не блокирует пуши

Даже если домен забанен, пуши обычно **продолжают отправляться** (полная блокировка отправки на уровне домена — редкость). При клике браузер показывает warning-страницу.

### Минимальный Wait в Notifications Flow — 60 сек

В Notifications Flow UI не даёт `Wait` меньше **60 сек**. 600 (10 мин) — мало; для первого `Check Conversion` ставят 3600 (час)+.

### Как пушить тех, кто не нажал кнопку перехода с лэнда

Чтобы пушить тех, кто **не нажал** кнопку перехода с лэнда дальше (и при условии, что нужно посылать нотификации только тем визитам, которые ушли дальше) — ноду `Notifications Flow` ставят **до** лэнда + большой `Wait` в начале (среднее время на лэнде).

## Как устроена Push Distribution (Remarketing Content)

Push Distribution — применение **`Remarketing Content`** distribution для пушей. Структура дерева:

```
Root Folder (Strategy: First / Weights / Conversions AI / Metrics AI)
├── Folder country_code = US
│   ├── Push Template A (weight 1)
│   ├── Push Template B (weight 1)
├── Folder country_code = DE
│   └── Push Template C
└── Folder default
    └── Push Template fallback
```

Типовой паттерн — одна Remarketing-дистрибуция на все push-шаги, которая по полю `Flow State` внутри папок (а не отдельные дистрибуции под каждый шаг) маршрутизирует на нужный Push Template.

## Что такое Push Template и его поля

Шаблон push-уведомления в `Marketing → Message Templates`. Поля:

- **Name**, **Description**, **Tags**.
- **Push Title** — заголовок.
- **Push Message** — тело.
- **Push Icon** — картинка.

Используется в Notifications Flow и Remarketing Distribution.

## Как работает AIO Push Subscribe (макрос)

JS-макрос, выводящий браузерный prompt подписки на push.

- Включается через **Push Subscribe Toggle** на уровне кампании.
- Управление вызовом (по клику / автоматически) — правкой JS внутри макроса.
- После подписки факт фиксируется как конверсия типа `Push Subscribe`.

## Как устроены Telegram-уведомления

Telegram — отдельный канал отправки, существующий параллельно Push. Через него баер получает алерты о событиях кампании (например, о лидах) прямо в чат, не заходя в интерфейс.

### Компоненты Telegram-стека

| Компонент | Что делает |
|---|---|
| **`Telegram` нода-канал** (Marketing Flow, тип `Plan Telegram`) | Отправляет сообщение в TG. На входе — Push Template, сконфигурированный для Telegram |
| **Sender Provider** | Провайдер-отправитель типа `Telegram`. Настраивается через API-ключ бота + имя бота. Один и тот же провайдер обслуживает и рассылки, и `Telegram Destination` |
| **Messages (раздел)** | Логи отправки — кому, что, когда |
| **Buyer TG Chat ID** | Кастомное поле визита под TG chat_id владельца кампании. **Имя — пример; по дефолту в шаблонах тенантов поля нет, заводится вручную.** Заполняется через `Fill Fields` по правилу `Campaign Owner` |
| **Notification Flow** | Marketing Flow, запускается на каждую конверсию выбранного типа |

### Типичный сценарий Telegram-уведомления

Конверсия выбранного типа приходит → срабатывает Notification Flow → `Telegram` нода-канал с Push Template (например, «Проверь лид») → отправляется в TG-чат владельца кампании по полю `Buyer TG Chat ID`. Так статусы конверсий по кампаниям каждого владельца уходят его баеру в Telegram.

## Уведомление о бане рекламного аккаунта / новой конверсии — это НЕ рассылка

Если речь про алерт **тебе-медиабайеру** о твоём аккаунте — бан рекламного аккаунта Facebook (`Facebook ad account "..." disabled (...)`), новая конверсия, сбой пуллинга рекламного аккаунта, отчёт по расписанию — это **Notification Center** (колокольчик в UI, self-notification-алерты), а не Marketing/push-рассылка визитёрам из этого дока. Такой алерт наружу проксирует сам Notification Center, а не Sender Providers и ноды-каналы этого флоу; отчёт по расписанию, например, приходит в Telegram и Slack готовой картинкой-таблицей, а не текстом. Дословные тексты этих уведомлений, их триггеры и настройка — [mechanics/notification-center.md](notification-center.md). Здесь — только рассылки push/Telegram уже подписанным визитам.

## Не воспроизводится сессия / Replayer not enough events

Это ошибка Session Replay, не push-механики: визит длился меньше Session flush interval (10 сек), ивенты не успели сброситься. Разбор — [mechanics/visit-lifecycle.md](visit-lifecycle.md).

## Marketing-инфраструктура: Flows, Message Templates, Sender Providers, Messages

Раздел `Marketing` (`/app/remarketing`) — 4 подсекции: **Flows · Messages · Message templates · Sender providers**.

### Как устроен редактор Marketing Flows

`Marketing → Flows → + Flow → Notifications` («A separate step that schedules notifications for users») открывает нод-граф редактор (как `Tracker → Flows`, но для рассылок). Слева — Flow details (`Name` / `Description` / `Tags` / `Show archived states`) + палитра **Nodes to create**:

- **Каналы:** **SMS**, **Telegram**, **Email**, **Push** — 4 отдельные ноды-канала. Работают только **Push** и **Telegram**; **Email** и **SMS** — заглушка (нода есть, доставка не работает, см. «Email/SMS-каналы — доставка не работает» ниже). Тип каждой ноды-канала во флоу — `Plan Sms` / `Plan Telegram` / `Plan Email` / `Plan Push` (по одной ноде на канал).
- **Контроль:** **Wait** (задержка), **Check** (условие по полю визита), **Repeat** (цикл), **Check conversion** (развилка по факту конверсии), **Finish**. Типы контрольных нод — `Plan Wait` / `Plan Check` / `Plan Repeat` / `Plan Conversion Exists Check` (это и есть Check conversion).
- Канва стартует с ноды **Start** (Initial state).

### Что задаётся в конфиге ноды-канала

**Конфиг ноды-канала** (шестерёнка на ноде, пример Push): `Name` / `Description`; **`Settings availability`** (скоуп настройки, напр. `Campaign template` — но на нодах Notifications Flow он ни на что не влияет, см. «Settings availability на нодах Notifications Flow ни на что не влияет» ниже); стратегия выбора варианта **First / Weights**; строки вариантов (аудитория `All` + **`Select distribution`** + вес) + `+ Add another variant` + `Compact`/`Control`; **`Visual settings`** (`State color` / `Collapsed by default` / `Archive state` — только оформление ноды). **Тело сообщения в ноде не задаётся** — нода выбирает дистрибуцию/шаблон; контент живёт в Message Templates.

### Settings availability на нодах Notifications Flow ни на что не влияет

У нод маркетинг-каналов (Push / Telegram / Email / SMS) в конфиге есть скоуп `Settings availability` (`Campaign` / `Template`), как у шагов Campaign Flow (сам механизм — [models/flow-model.md](../models/flow-model.md) → State Settings). Но задать его тут бессмысленно: даже если у ноды выставлен `Settings availability`, в настройках кампании эта нода **всё равно не покажется** — это by design. Редактор Notifications Flow переиспользует Flow Builder, откуда скоупы и достались, но переопределения из кампании для маркетинг-нод не работают.

### Какие типы Message Templates существуют и их поля

`Marketing → Message Templates → + Message template` — выбор типа (список фильтруется по `Type` / `Owner` / `Team`):

| Тип | Поля тела |
|---|---|
| **Push** | `Push title` + `Push message` + `Push icon` (из image library) — подробнее см. «Что такое Push Template и его поля» выше |
| **Email** | `Email subject` + `Email text` (rich-text: **B / I / U** + нумерованный/маркированный список) |
| **Telegram** | `Telegram message` (текст) |
| **SMS** | `Sms text` (текст) |

У всех — `Name` / `Description` / `Tags`.

### Какие Sender Providers поддерживаются

`Marketing → Sender providers → + Sender provider`: `Name` / `Description` + **`Integration type`** (дропдаун). Провайдеры в дропдауне:

- **Telegram** — Telegram-бот. **Рабочий** (обслуживает и рассылки, и `Telegram Destination`).
- **Mailgun** — email. Заглушка — доставка email не работает (см. ниже).
- **Twilio** — SMS (поле учётки `Twilio Api Key`). Заглушка — доставка SMS не работает.
- **Prelude Bridge** — SMS / верификация. Заглушка — доставка SMS не работает.

Выбор типа раскрывает поле(я) учётки этого провайдера (напр. Twilio → `Twilio Api Key`). Это и есть «Sender Provider», на который опираются ноды-каналы в флоу.

### Email/SMS-каналы — доставка не работает

`Email` и `SMS` (ноды-каналы + провайдеры Mailgun / Twilio / Prelude Bridge) в UI уже есть, но **доставка не работает — это заглушка**. В проде из не-push каналов работает только `Telegram`; email/SMS-рассылка пока недоступна (даты запуска не анонсированы). Не настраивать email/SMS как рабочий канал: нода отработает, но сообщение не уйдёт.

> В списке Sender Providers **нет отдельного push-гейтвея** (Mailgun=email, Twilio/Prelude=SMS, Telegram=TG): браузерный/мобильный web-push отправляется AIO **нативно** через Service Worker (VAPID, `pushManager.subscribe`), а не через внешний провайдер — см. [reference/sdk.md](../reference/sdk.md) §II.15 (AIO Push Subscribe).

### Что такое Messages (лог отправки) и где смотреть аналитику

`Marketing → Messages` — **read-only** список фактически отправленных/запланированных сообщений (кнопки create нет — сообщения порождаются флоу). Фильтр **`Status`**: **Requested → Processing → Done / Failed**. Это единственная аналитика по рассылкам (CTR / open-rate не трекаются).

## Смежные темы

- [models/conversion-model.md](../models/conversion-model.md) — Conversion Types (Push Subscribe), Notification Flow.
- [models/distributions-model.md](../models/distributions-model.md) — Remarketing Content Distribution (тип 7).
- [mechanics/pwa.md](pwa.md) — PWA + Service Worker для push.
- [models/marketing-flow.md](../models/marketing-flow.md) — концепт-модель `Notification Flow` (`Marketing Flow` — синоним; ноды/транзишены, два триггера).
- [reference/glossary.md](../reference/glossary.md) — `Push Template`, `Notifications Flow`, `Remarketing Distribution`, `Telegram` (нода-канал), `Sender Provider`, `Buyer TG Chat ID`, `AIO Push Subscribe`.
- [reference/sdk.md](../reference/sdk.md) — AIO Push Subscribe макрос (§II.15).
- [how-to/push-notifications.md](../how-to/push-notifications.md) — пошаговые процедуры (сборка Follow Up Push, Telegram Sender Provider, Buyer TG Chat ID).
- *Пуш / PWA не работает* — диагностика «push не приходит».
