---
id: marketing-flow
title: Marketing / Notifications Flow (отдельный тип флоу) — концепт (модель)
description: Что такое Notification Flow как сущность (Marketing Flow — синоним; в UI — Notification Flow) — отдельный тип флоу, параллельный Campaign Flow, со своими нодами (Start/Check/Check Conversion/Wait/Repeat/Push + каналы) и транзишенами (Passed/Rejected/Repeat); как вешается (нода в Campaign Flow / поле на Conversion Type); чем отличается от Campaign Flow и Remarketing Distribution. Концепт; механика рассылки — mechanics/notifications-flow.md, процедуры — how-to/push-notifications.md.
doc_type: model
builds: [erp]
related: [flow-model, distributions-model, conversion-model, visit-field]
language: ru
updated: 2026-08-11
---

# Marketing / Notifications Flow (отдельный тип флоу) — концепт (модель)

> Модель сущности **`Notification Flow`** (`Marketing Flow` — синоним, тоже применяется; в UI — `Notification Flow`). Нода `Notifications flow` в Campaign Flow — [models/flow-model.md](flow-model.md). Дерево контента рассылки — [models/distributions-model.md](distributions-model.md).

## Как называется этот флоу — `Notification Flow` (канон), `Marketing Flow` — синоним

Каноничное имя сущности — **`Notification Flow`**; так она называется в UI. **`Marketing Flow` — синоним того же самого**, тоже употребляется (напр. в разделе `Marketing → Flows`, где флоу создаётся и живёт). Это не тип-и-подтип — одна и та же сущность под двумя названиями. Дальше в тексте оба имени используются взаимозаменяемо.

## Что такое Notification Flow и зачем он нужен

**`Notification Flow` — это отдельный тип флоу, который отвечает за РАССЫЛКУ уведомлений (follow-up пуши / сообщения) и работает параллельно Campaign Flow.** Это не часть пути визита Source → Destination: у него собственный нод-граф (`Start` / `Finish` / `Check` / `Check Conversion` / `Wait` / `Repeat` / `Push` + ноды-каналы) и собственные транзишены (`Passed` / `Rejected` / `Repeat`). Сам по себе он ничего не запускает — его **вешают** на основной флоу (нода `Notifications flow` в Campaign Flow) или на тип конверсии (поле `Notification Flow` у Conversion Type), и тогда он стартует и крутится для визита сам по себе, отбивая follow-up пуши/сообщения по своему расписанию.

## Notifications Flow — это ТРЕТИЙ тип флоу, не Campaign Flow

В AIO есть Campaign Flow (полный путь визита, см. [models/flow-model.md](flow-model.md)), внутри него SubFlow (вставной кусок без Source/Destination), и **отдельно** — Notifications Flow. Это разные конструкторы: у Notifications Flow **другие ноды и другие транзишены**, и он **по-другому работает**. Создаётся как самостоятельный объект: `Marketing → Flows → + Flow → Notifications` (подзаголовок типа в UI — «A separate step that schedules notifications for users»). Открывается нод-граф редактор как у `Tracker → Flows`, но для рассылок.

## Какие ноды есть в Notifications Flow — собственный набор

Ноды (steps): `Start`, `Finish`, `Check`, `Check Conversion`, `Wait`, `Repeat`, `Push`, плюс ноды-каналы. Каждую настраивают через cogwheel (шестерёнку). Транзишены — `Passed` / `Rejected` (после `Check` / `Check Conversion`) и `Repeat` (цикл назад). Это **не** транзишены Campaign Flow (там Arrived / Handle / Passed / Rejected / Destination-pushed и т.д., 10 типов) — набор свой.

### Нода `Start` — точка входа флоу, не момент подписки

`Start` — дефолтная начальная нода, создаётся автоматически. Это момент, **когда флоу начинает исполняться** (визит дошёл до точки подключения), **а не** момент подписки на push.

### Нода `Finish` — конечная нода

`Finish` — конечная нода.

### Нода `Check` — условие по полю визита

`Check` — условие по полю визита (одно или несколько allowance rules: поле + оператор + значение). Транзишены `Passed` / `Rejected`. Напр. слать только если `country_code = Argentina`.

### Нода `Check Conversion` — условие по конверсии (ключевая развилка)

`Check Conversion` — условие, но **только для конверсий**: проверяет, стал ли визит конверсией выбранного типа (тип берётся из `Settings → Conversion Types`). Транзишены `Passed` / `Rejected`. Это ключевая нода-развилка follow-up логики.

### Нода `Wait` — задержка перед следующим шагом

`Wait` — задержка перед следующим шагом, в **секундах** (Delay progress bar). Минимум в UI — **60 сек**.

### Нода `Repeat` — цикл повторов (максимум 10 раз)

`Repeat` — цикл: повторяет push-циклы **максимум 10 раз**, отправляя визит транзишеном `Repeat` обратно на предыдущий шаг (обычно в `Check Conversion`). Два пути: повтор и «повторы кончились» (выход дальше).

### Нода `Push` — отправка сообщения

`Push` — отправка. Payload: либо одиночный Push Template, либо целая Remarketing Distribution (рекомендуется дистрибуция).

## Как запустить Notifications Flow — два способа подключения (триггеры)

Сам флоу пассивен; его подключают одним из двух способов — и один и тот же флоу может использоваться обоими:

- **Триггер #1 — нода в Campaign Flow.** В `Tracker → Flows → Edit Flow` ставят ноду `Notifications flow`, в её cogwheel выбирают готовый Notifications Flow. Флоу стартует, когда визит **доходит до этой ноды** (follow-up на пути). См. [models/flow-model.md](flow-model.md) (нода `Notifications flow` → поле `Notification flows`).
- **Триггер #2 — поле на Conversion Type.** В `Settings → Conversion Types → <type> → Notification Flow` вешают Marketing Flow. Тогда флоу запускается **на каждую конверсию этого типа** — типичный кейс: слать статусы конверсий в Telegram (напр. на типах `Lead` / `Registration`), а не только в интерфейсе. См. [models/conversion-model.md](conversion-model.md).

### Подключение к Campaign Flow — прозрачный проход для визита

Когда визит в Campaign Flow проходит ноду `Notifications flow`, он **не застревает** в marketing-флоу: для визита это обычный редирект дальше (на оффер), а параллельно стартует отдельный marketing-флоу, который крутится сам. Ноду рекомендуют ставить **в конце** (после PWA / Destination-шага), чтобы пуши шли только редиректнутым визитам — но локация по сути не критична. К одной точке можно прицепить **несколько** Notifications Flow (стартуют одновременно); обычно проще один флоу с внутренними развилками (`Check` / `Check Conversion`).

### Подписка не нужна для старта Notifications Flow — проверяется только при отправке

Частый нюанс: Notifications Flow **стартует даже если у визита нет подписки на push**. Подписка не является требованием для запуска — она проверяется **только в момент попытки отправить push** (нода `Push`). Если визит подпишется по ходу (за время `Wait`) — пуши пойдут. Флоу исполняется **для каждого визита**. Отсюда паттерн «пушить тех, кто ещё не нажал кнопку»: ноду ставят до лэнда + большой `Wait` в начале.

## Где задаётся тело сообщения рассылки — в Remarketing Distribution, не в ноде `Push`

Нода `Push` (и ноды-каналы) **тело сообщения не задают** — они выбирают, что отдать: одиночный Message Template или целую Remarketing Content Distribution (рекомендуется). Дистрибуция при создании привязывается к конкретному флоу (поле `Select Flow`), а внутри — дерево папок (по `Flow State` / `country_code`) и стратегия (`First` / `Weights` / `ML`) на множестве шаблонов. Сам контент сообщений живёт в `Marketing → Message Templates` (4 типа: Push / Email / Telegram / SMS). Это отдельная сущность, работающая в паре с флоу, — [models/distributions-model.md](distributions-model.md) (тип 7).

## Какие каналы рассылки есть в Notifications Flow — SMS / Telegram / Email / Push

В флоу-редакторе четыре отдельные ноды-канала: **SMS / Telegram / Email / Push**. Из них в проде доступны **Push и Telegram** — **Email и SMS пока недоступны** (ноды в редакторе есть, но доставка по этим каналам не работает; закладывать рассылку на Email/SMS сейчас нельзя). Для не-push каналов нода опирается на Sender Provider (Telegram работает через бота). Аналитика по рассылке — только лог `Marketing → Messages` (outbox, статусы `Requested → Processing → Done / Failed`); CTR и open-rate не трекаются.

## Как Notifications Flow связан с остальными сущностями системы

### Куда вешается Notifications Flow — на Campaign Flow и на Conversion Type

- **Вешается на Campaign Flow** через ноду `Notifications flow` (поле `Notification flows`). Триггерится, когда визит доходит до ноды. → [models/flow-model.md](flow-model.md)
- **Вешается на Conversion Type** через поле `Notification Flow` — стартует на каждую конверсию типа. → [models/conversion-model.md](conversion-model.md)

### Что использует и порождает Notifications Flow — контент, провайдеры, аналитика

- **Использует Remarketing Content Distribution** (тип 7): ноды выбирают дистрибуцию; дистрибуция привязана к флоу (`Select Flow`). → [models/distributions-model.md](distributions-model.md)
- **Использует Message Templates** (Push / Email / Telegram / SMS) — контент сообщений.
- **Использует Sender Providers** — для каналов кроме push (Telegram работает через бота).
- **Проверяет Conversion Types** (`Check Conversion`) и **поля визита** (`Check` / allowance rules). → [models/conversion-model.md](conversion-model.md), [models/visit-field.md](visit-field.md)
- **Порождает записи в `Marketing → Messages`** (outbox) — единственная аналитика по рассылкам.
- **Связан с PWA**: follow-up push важен для PWA-сетапов, ноду обычно ставят после PWA-шага.

## Как слать статусы конверсий в Telegram — Notification Flow на Conversion Type

**Чтобы статусы конверсий приходили в Telegram, а не только в интерфейсе, на тип конверсии вешают `Notification Flow` (синоним — `Marketing Flow`): `Settings → Conversion Types → <тип> → Notification Flow`.** Тогда на каждую конверсию этого типа стартует отдельный marketing-флоу и отбивает сообщение в TG.

Сборка из двух частей:

- **Куда слать текст** — `Notification Flow` с нодой-каналом `Telegram`. Текст — шаблон с плейсхолдерами `{{aio.visit.fields.*}}`: в сообщение подставляются поля визита и данные конверсии.
- **Кому слать** — Telegram-контакт получателя лежит в поле визита `Buyer TG Chat ID`. Оно заполняется дистрибуцией `Fill Fields Distribution` по правилу `Campaign Owner` — так статусы по кампаниям каждого владельца уходят его баеру. Как строится Fill Fields-дистрибуция по `Campaign Owner` — [models/distributions-model.md](distributions-model.md).

## Чем Notifications Flow отличается от Campaign Flow и соседних сущностей

### Notifications Flow vs Campaign Flow — разные конструкторы

Campaign Flow — полный путь визита Source → Destination (ноды Source / Content / Filter / Destination…, 10 транзишенов). Notifications Flow — отдельный marketing-флоу рассылки (ноды Start / Check / Check Conversion / Wait / Repeat / Push + каналы, транзишены Passed / Rejected / Repeat). Notifications Flow **вешается** на Campaign Flow и крутится параллельно.

### Нода `Notifications flow` vs сама сущность Notifications Flow — не одно и то же

Нода `Notifications flow` — это **шаг внутри Campaign Flow**, который ПОДКЛЮЧАЕТ готовый marketing-флоу (gear → поле `Notification flows`). Сама сущность — отдельный объект в `Marketing → Flows`. Одно — точка подключения, другое — подключаемый флоу.

### Notifications Flow vs Remarketing Distribution — логика vs контент

Notifications Flow = **логика/расписание** рассылки (когда, кому, с каким повтором — `Wait` / `Check Conversion` / `Repeat`). Remarketing Distribution = **дерево выбора контента** (какой шаблон отдать по `country_code` / `Flow State` / стратегии). Нода ссылается на дистрибуцию; дистрибуция привязана к флоу. Две разные сущности в паре.

### «Notifications Subflow» vs SubFlow Campaign Flow — разные вещи

«Notifications Subflow» = то же самое, что сам marketing-флоу (`Notification Flow`, он же `Marketing Flow` — синонимы). SubFlow из Campaign Flow ([models/flow-model.md](flow-model.md)) — вставной кусок пути без Source / Destination. Разные вещи, общее только слово «subflow».

### Триггер #1 (нода) vs триггер #2 (Conversion Type) — два способа запустить

Два способа запустить один и тот же флоу. Нода в Campaign Flow → стартует на пути визита (follow-up). Поле на Conversion Type → стартует на каждую конверсию типа (например, слать статусы конверсий в Telegram).

### `Start` vs момент подписки на push — разные события

`Start` = когда флоу НАЧИНАЕТ исполняться (визит дошёл до точки подключения), не момент подписки. Подписка для старта не обязательна — проверяется только при отправке (подробнее — в разделе «Подписка не нужна для старта» выше).

### Push-нода vs «три типа пушей» — шаг флоу vs классификация явления

Push-нода — конкретный шаг отправки во флоу. «Три типа пушей» (Manual / Scheduled / Follow Up) — классификация пушей как явления; на практике доступен **только Follow Up** (через Notifications Flow). Manual и Scheduled как отдельные режимы сейчас недоступны.

## Частые ошибки и подводные камни Notifications Flow

### Лимиты и единицы — Repeat 10 раз, Wait в секундах, свои транзишены

- **`Repeat` — максимум 10 раз** (жёсткий лимит). Не «неограниченно».
- **`Wait` — минимум 60 сек, в СЕКУНДАХ** (не в минутах); `Repeat Pattern` тоже в секундах через запятую. Для первого `Check Conversion` обычно ставят 3600 (час)+.
- **Транзишены свои** — `Passed` / `Rejected` / `Repeat`, а не 10 транзишенов Campaign Flow. Не копировать набор из [models/flow-model.md](flow-model.md).

### Поведение и контент — подписка, аналитика, клик по пушу, где задаётся тело

- **Подписка не требуется для старта** и проверяется только при отправке — частая путаница. Если визит подпишется за время `Wait` — пуши пойдут.
- **Аналитики по пушам (CTR / open-rate / конверсия после пуша) НЕТ** — только лог `Marketing → Messages`. Трекинга открытий сообщений в продукте нет — не рассчитывайте на него.
- **Клик по пушу возвращает визита в его ТЕКУЩИЙ State** (не в начало флоу), по дефолту на тот же домен, на котором он подписался → визит попадает в ту же кампанию.
- **Контент задаётся не в ноде**, а в Message Templates / Remarketing Distribution; нода только выбирает, что отдать.

## Куда смотреть дальше по Notifications Flow — связанные доки

- **Нода `Notifications flow` в Campaign Flow** (где в палитре, поле `Notification flows`) → [models/flow-model.md](flow-model.md).
- **Дерево контента рассылки** (Remarketing Content, `Select Flow`, стратегии, папки по Flow State) → [models/distributions-model.md](distributions-model.md).
- **Триггер по конверсии** (поле `Notification Flow` на Conversion Type) → [models/conversion-model.md](conversion-model.md).
