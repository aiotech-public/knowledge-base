---
id: conversion-model
title: Модель конверсий в AIO
description: Как устроены Conversion Types — single/multiple (дедупликация повторного события задаётся Uniqueness Strategy типа, а не источником события), три формата постбэк-URL (path, query, conversion-by-key), параметры delay / revenue / destination_uuid, поля визита через postback, Correction type, капы; какие Conversion Types засеяны шаблоном тенанта в каждой вертикали (Finance / iGaming / Nutra / Sweeps) и что их спавнит во флоу. Полный цикл длинной воронки Lead → статус-апдейты → финальная конверсия целевого действия.
doc_type: model
builds: [erp, mtk]
related: [flow-model, visit-lifecycle, glossary, destination, tracker, architecture, postback-generator, source-trackers, business-model, debug-with-logs, ui-common, analytics, distributions-model, notifications-flow, remarketing-campaigns, destinations, conversion-ai-testing, campaigns, meta-spend-allocation, sdk, api, metric, marketing-flow]
language: ru
updated: 2026-10-16
---

# Модель конверсий в AIO

Этот файл объясняет **как реально устроены конверсии** в AIO: атрибуты Conversion Type, разница single vs multiple, паттерн `Sale Status Update` для истории статусов, формат постбэков и обновление полей визита через них. Фундамент для понимания «почему лид / регистрация / покупка пишутся именно так» и для разбора любой проблемы с конверсиями.

## Что такое конверсия в AIO и как она создаётся

Визит и конверсия — это **разные объекты** в AIO. Конверсия — отдельная сущность, привязанная к визиту через `visit_uuid`. На один визит может быть **0, 1 или много** конверсий разных типов. Конверсия создаётся **тремя путями**:

1. **Внешний постбэк (основной)** — внешняя система (рекламодатель, партнёрка) присылает нам постбэк о том, что по этому визиту у неё что-то произошло (зарегистрировался, оплатил, статус изменился).
2. **Изнутри AIO во флоу** — шагом `Spawn conversion` во Flow либо JS-макросом с лэнда. Так фиксируют событие на стороне AIO без ожидания постбэка от Destination. Конверсия от шага `Spawn conversion` появляется не сразу, а примерно через минуту: шаг откладывает её создание. Задержка не проверяет, завершён ли Flow. Визит при этом идёт по флоу дальше сразу.
3. **Вручную из UI** — экшен `Trigger Conversion` по правому клику на визите (см. раздел «Trigger Conversion — ручное создание конверсии на визит из UI» ниже). Отдельный путь: не постбэк и не шаг флоу.

Конверсия фиксируется только если в тенанте есть `Conversion Type`, на UUID или slug которого пришёл постбэк. Иначе постбэк теряется (нет куда писать).

**У trash-визита** (`is_trash`) авто-конверсия ноды `Destination` (`Pushed conversion spawn` / `Reject conversion spawn`) **не создаётся** — лид при этом в Destination всё равно уходит. Отдельная нода `Spawn conversion` и входящий постбэк конверсию у trash-визита создают. Подробности — [models/flow-model.md](flow-model.md).

**Длинная воронка**: `Lead` (single) → серия статус-апдейтов `Sale Status Update` (multiple) → финальная конверсия целевого действия вертикали (single). Это **параллельные** конверсии, не альтернативные — на одном визите живут все три. Какие типы под это заведены в шаблоне тенанта — раздел «Какие Conversion Types заведены в тенанте из коробки».

*(Связано: [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) — путь визита; [reference/glossary.md](../reference/glossary.md) → `Conversion Type`, `Постбэк`.)*

## Trigger Conversion — ручное создание конверсии на визит из UI

`Trigger Conversion` — экшен по **правому клику на визите**, создаёт конверсию на этот визит вручную, без постбэка. Нужен, когда конверсию надо проставить руками: рекламодатель не прислал постбэк, тестовая отбивка, разовая ручная фиксация события.

Как выглядит: ПКМ на визите → `Trigger Conversion` → открывается диалог с одним обязательным полем `Conversion Type` (дропдаун типов тенанта) → `Confirm`. Диалог отправляет только два параметра — `visit_uuid` (выбранный визит) и `conversion_type_uuid` (выбранный тип); revenue / payout / прочие поля в этом окне не задаются (правятся потом через экшены `Update revenue` / `Update payout`, см. раздел «Conversions-таблица»). Конверсия создаётся по правилам `Uniqueness Strategy` выбранного типа, как и любая другая.

**Это отдельный путь**, не путать со смежными:

- **≠ постбэк** — ничего не приходит извне, оператор жмёт кнопку в UI.
- **≠ шаг флоу `Spawn conversion`** — тот срабатывает автоматически внутри Flow при прохождении визита; `Trigger Conversion` — ручное действие постфактум по конкретному визиту (`Spawn conversion` — раздел «Что такое конверсия в AIO и как она создаётся» + [models/flow-model.md](flow-model.md)).
- **≠ JS-макрос с лэнда** — тот фиксирует событие на самом лэнде в момент визита.
- **≠ `Re trigger trackers`** — тот **переотправляет уже существующую** конверсию в Source, а `Trigger Conversion` **создаёт новую** (`Re trigger trackers` — раздел «Retrigger конверсии» + [models/tracker.md](tracker.md)).

Термин и пометка конверсий как ручных — [reference/glossary.md](../reference/glossary.md) → `Trigger Conversion`.

### `Trigger Conversion` на свежем визите — ошибка `The selected visit uuid is invalid` больше не приходит

`Trigger Conversion` принимает и **только что пришедший визит** (`In Buffer = Yes`, ещё не осел в основное хранилище), и визит, лежащий в **trash**: проверка существования визита смотрит все слои хранения, а не только основное. Симптом «поставил конверсию руками на свежий лид — ошибка валидации» относится к прошлому поведению.

Как это выглядело раньше: нотификация `Validation error` / `Error while validating` с текстом `The selected visit uuid is invalid.` на визите, который в таблице виден и открывается. Если такая ошибка приходит сейчас — визита с переданным `visit_uuid` в AIO нет вовсе (опечатка в идентификаторе), а не «он ещё не сохранился».

Что такое `In Buffer` у визита и конверсии — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

## Что такое Conversion Type и зачем он нужен

Conversion Type — конфиг, определяющий, как записывать и считать событие-конверсию. Создаётся через `+Conversion Type` в `Settings → Conversion Types`.

### Основные поля Conversion Type (Name, Slugs, Strategy, Business Value)

| Атрибут | Что значит |
|---|---|
| **Name**, **Color**, **Description** | Отображение в UI. `Color` — цвет, которым тип отображается в `Tracker → Conversions`. |
| **Slugs** | Один или несколько slug'ов, перечисленных через запятую **без пробелов**: `purchase,sale`. В URL постбэка `conversion_type_uuid` принимает UUID либо точный slug этого типа: `?conversion_type_uuid=purchase`, `?conversion_type_uuid=sale`, `?conversion_type_uuid=<UUID>`. |
| **Uniqueness Strategy** | Сколько раз конверсия этого типа может быть на одном визите (`Single Conversion` / `Multiple Infinity` / `Multiple By Unique`). |
| **Business Value** | `Good` / `Neutral` / `Bad` (напр. `Lead` → `Good`, `Lead Rejected` → `Bad`, `Sale Status Update` → `Neutral`). Влияет на `Reward` в AI-оптимизации. |
| **Reward Weight** | Числовой вес для алгоритма, когда тип участвует в AI-сплитах Campaigns / Landings. В форме создания нужно указать значение; `1` — базовый вес. |

### Дополнительные поля Conversion Type (Correction type, Payout/Revenue, Notification Flow)

| Атрибут | Что значит |
|---|---|
| **Correction type** | Механизм корректировки revenue уже созданной конверсии через спавн коррекционной конверсии с разницей. Детально — раздел «Correction type — как поправить revenue уже созданной конверсии» ниже. |
| **Payout Settings** / **Revenue Settings** | Привязка к `Payout Distribution` / `Revenue Distribution`: в `Payout Settings` выбирается `Payout Tree`, в `Revenue Settings` — `Revenue Tree` (оба из дропдаунов). |
| **Notification Flow** | Поле `Notification flow` (тултип — `Notifications run when conversion spawn`): выбранное флоу типа `Notifications` сеется на визит при каждой конверсии этого типа. Кастомный путь для точечных алертов; штатный запуск рассылки по конверсии — триггер ремаркетинг-кампании, см. раздел «Как запустить рассылку при получении конверсии». |

### Correction type — как поправить revenue уже созданной конверсии

`Correction type` позволяет учесть новое revenue из повторного постбэка, не перезаписывая revenue исходной Single-конверсии: создаётся **коррекционная конверсия с дельтой**, которая в сумме доводит revenue до нужного значения. Отдельно revenue уже созданной конверсии можно задать вручную экшеном `Update revenue` в таблице конверсий.

Механизм включается при **двух** условиях одновременно:

- Оригинальная конверсия имеет стратегию `Single Conversion` — коррекция срабатывает ровно на **повторном** постбэке на уже созданную Single-конверсию (у `Multiple`-типов повторный постбэк и так создаёт новую конверсию, коррекция там не нужна).
- В качестве `Correction type` выбран Conversion Type со стратегией `Multiple Infinity`. Если у выбранного типа стратегия иная — коррекция не спавнится.

Дельта считается **накопительно**: `новый Revenue − (Revenue оригинала + сумма Revenue прошлых коррекций этого типа на визите)`. Нулевая дельта **не спавнит ничего**. Результат записывается в `Revenue` коррекционной конверсии — отдельного поля `Reward` у конверсии нет.

Пример: пришёл постбэк с `arrived_revenue=100`, затем переотбили постбэк на тот же визит с `arrived_revenue=120` → спавнится Correction-конверсия с `Revenue=20` (в сумме 100+20=120).

В шаблоне iGaming такая пара уже заведена: тип `Correction` (`Multiple Infinity`, `Business Value = Neutral`, описание `Revenue corrections from the partner`) назначен `Correction type` у типа первого целевого действия партнёра, а метрика `Corrections` считает число таких корректировок. В шаблонах Finance / Nutra / Sweeps коррекционного типа нет — при необходимости его заводят руками. Засеянные типы по вертикалям — раздел «Какие Conversion Types заведены в тенанте из коробки».

## Какие Conversion Types заведены в тенанте из коробки — по вертикали шаблона

**Набор Conversion Types тенант получает из шаблона своей вертикали, и он разный у Finance, iGaming, Nutra и Sweeps; MTK-шаблоны (Finance, iGaming, Sweeps) повторяют набор одноимённой ERP-вертикали.** Ниже — засеянные типы с их `Uniqueness Strategy`, слагами и описаниями из UI (описания на английском — так они видны в `Settings → Conversion Types`). У части типов слаг пустой: такой тип принимает постбэк только по UUID, пока слаг не добавлен. Набор дальше меняют под себя; как устроен шаблон тенанта и что ещё из него копируется — [context/architecture.md](../context/architecture.md).

### Finance — Lead, Sale Status Update, Push Subscribe, Lead Rejected, Fraud Lead + тип целевого действия

`Lead` (`Single Conversion`, slug `lead`, `Registered lead`); `Sale Status Update` (`Multiple Infinity`, `Neutral`, без слага — один тип на все статусы CRM, статус едет в поле конверсии); `Push Subscribe` (`Single Conversion`, без слага); `Lead Rejected` (`Multiple Infinity`, `Bad`, без слага — реджект пуша CRM, спавнит нода `Destination`); `Fraud Lead` (`Multiple Infinity`, `Bad`, без слага — лид, срезанный `AIO Antifraud` до пуша, спавнит нода `Spawn Fraud Lead` на ветке `rejected` шага `Check Antifraud Points`). Шестой тип — конверсия целевого действия вертикали (`Single Conversion`, со слагом); у него и у `Lead` в `Payout Settings` / `Revenue Settings` привязаны засеянные деревья, у остальных четырёх деревья не привязаны.

### iGaming — Install, Open PWA, Registration, Push Subscribe, Correction + два типа целевого действия

`Install` (`Single Conversion`, slug `install`, `Install Conversion for PWA / iOS` — в PWA-флоу `PWA / OneLink`, `Advanced - PWA / OneLink` и `Deeplink Android` его спавнит нода `Offer` при пуше, во флоу `PWA / OneLink w/ Landing` — отдельная нода `Spawn Install`); `Open PWA` (`Single Conversion`, slug `open`, `Opened PWA`); `Registration` (`Single Conversion`, slug `registration`, `Registered lead`); `Push Subscribe` (`Single Conversion`, слаги `push`, `subscribe`); `Correction` (`Multiple Infinity`, `Neutral`, `Revenue corrections from the partner`); плюс два типа целевого действия у партнёра — первое событие (`Single Conversion`, у него `Correction type = Correction`) и его повторы (`Multiple Infinity`, `every postback counts`). У всех семи привязаны засеянные `Payout Distribution` / `Revenue Distribution`. Тип реджекта в этом шаблоне не заведён.

### Nutra — Lead, Approved, Rejected, Trash, Push Subscribe

Четыре статусных типа, все `Single Conversion`, с длинными списками слагов под таксономии CPA-сетей: `Lead` (`lead`, `new`, `hold`, `pending`, `processing`, `wait`, `raw`… — `Registered lead`; его же спавнит нода `Destination` при успешном пуше), `Approved` (`approved`, `confirmed`, `sale`, `paid`, `shipped`… — `Sale/Approved Lead Status`), `Rejected` (`rejected`, `cancel`, `declined`, `fail`, `return`… — `Rejected Leads Status`, `Reward Weight = -1`), `Trash` (`trash`, `spam`, `invalid`… — `Trash Lead Status`, `Reward Weight = -1`; его спавнят нода `Destination` на реджекте пуша и нода `Spawn Trash Lead` на ветке `rejected` антифрода). Плюс `Push Subscribe` (`Single Conversion`, `push` / `subscribe`). Статус сети здесь маппится слагом на **разные типы**, а не на один тип с полем, как в паттерне `Sale Status Update`; полные списки слагов — в карточках типов.

### Sweeps — Lead, Registration, Fraud Lead, Push Subscribe, Lead Rejected

`Registration` (`Single Conversion`, слаги `registration`, `reg`, `signup` — `Visitor was pushed to the offer - the SOI event the buyer optimises on. Spawned by the Destination step, no postback needed`: создаёт нода `Destination` при успешном пуше); `Lead` (`Single Conversion`, `lead`, `soi`, `conversion` — `Lead accepted by the advertiser - the payable event. Arrives by postback with type=lead`); `Fraud Lead` (`Multiple Infinity`, `Bad`, `fraud`, `fraud_lead` — срезан антифродом до пуша, нода `Spawn Fraud Lead`); `Push Subscribe` (`Single Conversion`, `push`, `subscribe`); `Lead Rejected` (`Multiple Infinity`, `Bad`, `reject`, `rejected`, `lead_rejected` — отказ рекламодателя, спавнит нода `Destination`). Деревья Revenue / Payout привязаны у `Lead` и `Registration`.

## Uniqueness Strategy: чем Single отличается от Multiple

Самый важный атрибут Conversion Type. Определяет, может ли тип повторяться на одном визите.

### Single Conversion — одно событие на визит

Одна конверсия этого типа на визит. Повторное событие того же типа на том же визите **молча игнорируется** — новая конверсия не создаётся, существующая не перезаписывается, ошибка в логах AIO не записывается.

Применяется для событий, которые по природе случаются один раз — например, `Lead` (регистрация у рекламодателя), `Registration` (пуш в оффер в шаблоне Sweeps), `Approved` (подтверждённый заказ в шаблоне Nutra), `Install` (установка приложения или PWA), `Push Subscribe` (подписка на push-уведомления). Засеянный набор зависит от вертикали шаблона (раздел «Какие Conversion Types заведены в тенанте из коробки»), дальше его меняют под себя.

Частный случай того же правила — **повторный постбэк** с тем же `conversion_type_uuid` на визит, где конверсия этого типа уже есть: он отбрасывается так же молча. Исключение одно — у типа задан `Correction type`: тогда повторный постбэк вторую конверсию этого типа не создаёт, а спавнит коррекционную (см. «Correction type — как поправить revenue уже созданной конверсии»).

### Multiple Infinity — серия апдейтов без лимита

`Multiple Infinity` не ограничивает число конверсий этого типа на визите: при обработке нового события регистрация не отбрасывает его из-за уже существующих конверсий. Сам ответ приёмника постбэков ещё не подтверждает создание конверсии — см. диагностику HTTP `200` ниже.

**Применяется для серий апдейтов** и повторных событий (типично — повторные покупки/касания):

- `Sale Status Update` — апдейты статуса у рекламодателя. Статус сменился на `No Answer` → постбэк → новая конверсия `Sale Status Update` с `sale_status=No Answer`. Через час — `Callback` → ещё одна конверсия. И так далее.

### Multiple By Unique — серия с дедупликацией по полю `unique`

Несколько на визит, но с **дедупликацией по полю `unique`**. Постбэк должен передавать параметр `&unique=<значение>`. Конверсии с одинаковым `unique` для одного визита считаются дублями и повторно не создаются.

Use case: нужно ровно одну конверсию на одно бизнес-значение в рамках визита. Например:
- Один пуш на один `offer_id` — `&unique=<offer_id>`, повторные пуши того же оффера тому же визиту не плодят конверсии.
- Один Add to Cart на один `product_uuid` — `&unique=<product_uuid>`.

### Дедупликация — свойство Conversion Type, а не источника события

Повтор отбрасывается или создаёт вторую конверсию по `Uniqueness Strategy` того типа, на который пришло событие. Правило одинаково работает на всех трёх путях создания — входящий постбэк, шаг `Spawn conversion` или макрос с лэнда, ручной `Trigger Conversion` (см. «Что такое конверсия в AIO и как она создаётся»).

«Событие засчитается один раз» — это настройка типа, а не способ отправки события. При обработке повторного события `Multiple Infinity` допускает вторую конверсию на том же визите; `Multiple By Unique` допускает её при другом значении `unique`. Поэтому при жалобе «одно и то же событие посчиталось дважды» первым делом смотрят `Uniqueness Strategy` типа.

## Как работает Sale Status Update — паттерн истории статусов

Применяется в длинных воронках, где между регистрацией (`Lead`) и финальной конверсией целевого действия идёт долгий цикл обработки со множеством промежуточных статусов (несколько касаний, ручная обработка на стороне рекламодателя). Решает проблему «как одновременно хранить **последний статус** на визите И **полную историю** апдейтов».

### Из каких компонентов состоит Sale Status Update

Два компонента:

1. **Conversion Type `Sale Status Update`** (`Multiple Infinity`, `Business Value = Neutral`) — каждый постбэк создаёт новую конверсию с полем `sale_status=<значение>`. В шаблонах Finance (ERP и MTK) тип уже заведён с описанием `CRM status change sent by the advertiser via postback - approve, reject, hold, call later. One type for every status: the status text lives in the conversion fields`, **без слага** — постбэк адресуется по UUID типа, пока слаг (например `sale_status`) не добавлен руками; в шаблонах iGaming / Nutra / Sweeps типа нет.
2. **Поле визита `current_sale_status`** — обновляется постбэком через `visit[current_sale_status]=<значение>`. В нём остаётся значение последнего обработанного постбэка, записавшего это поле; AIO не сравнивает время смены статуса в CRM.

### Как выглядит постбэк рекламодателя на Sale Status Update

```
https://app.aio.tech/api/v1/trigger/conversion-request
  ?visit_uuid={CLICK_ID}
  &conversion_type_uuid=<UUID-Sale-Status-Update>
  &arrived_revenue={revenue}
  &visit[current_sale_status]={sale_status}
  &sale_status={sale_status}
```

При получении такого URL AIO:
- Создаёт новую конверсию типа `Sale Status Update`.
- В поле конверсии `sale_status` пишет значение из параметра `sale_status` (без префикса `visit[]`).
- В поле визита `current_sale_status` пишет значение из параметра `visit[current_sale_status]`.

### Что даёт паттерн Sale Status Update

- **Аналитика «текущий статус»** — фильтр визитов по `current_sale_status = "No Answer"` для retargeting / повторных касаний.
- **Аналитика «история»** — все конверсии `Sale Status Update` визита показывают полный путь (`No Answer` → `Callback` → `Recall` → `Approved`).
- **Не плодим Conversion Types** — один тип принимает все статусы, статус хранится в поле, а не в имени типа.

### Поле `current_sale_status` — как добавить

`current_sale_status` — поле визита, `Type = Visit`, формат `String`; `sale_status` — поле конверсии, `Type = Conversion`, формат `String`. Если нужного поля нет, создайте его через `Settings → Fields → +Field`.

Поле конверсии `sale_status` шаблоны не засевают; единственное засеянное поле конверсии во всех шаблонах — `arrived_revenue`.

## Постбэк-URL — общий формат и параметры

Любой постбэк, который создаёт конверсию (или обновляет поле визита):

```
https://<любой_домен_тенанта>/api/v1/trigger/conversion-request
  ?visit_uuid=<UUID визита>
  &conversion_type_uuid=<UUID Conversion Type>
  &<параметры>
```

**Тенант определяется** по `visit_uuid` + `conversion_type_uuid` (UUID глобально уникальны). Поэтому домен может быть **любой из вашего тенанта** — `app.aio.tech` или ваш брендовый домен — без разницы: конверсия создаётся одинаково. Отличается только HTTP-ответ, который увидит рекламодатель: на домене панели он читаемый (`200`/`400`), на трекерном домене статус результат не показывает — секция «На каком домене смотреть ответ постбэка».

### Три формата постбэк-URL — path-style, query-style и свои имена параметров

Ссылку постбэка рекламодателю отдают в одном из **трёх форматов**. Приёмник обрабатывает их одинаково — формат выбирается под платформу, на которой постбэк прописывается.

- **Path-style** (в UI — вкладка `Path`): визит и тип конверсии стоят прямо в пути URL.
  ```
  https://<домен>/api/v1/trigger/conversion/{CLICK_ID}/{conversion_type_uuid}?arrived_revenue={revenue}
  ```
- **Query-style** (вкладка `Query`): оба идентификатора едут query-параметрами с именами AIO.
  ```
  https://<домен>/api/v1/trigger/conversion-request?visit_uuid={CLICK_ID}&conversion_type_uuid={conversion_type_uuid}&arrived_revenue={revenue}
  ```
- **`conversion-by-key`** (`Query with custom parameter names`): имена параметров задаёт рекламодатель — разбор в секции про формат `conversion-by-key`.

Slug типа конверсии работает вместо UUID во всех трёх форматах, в том числе в path-style: `.../conversion/{CLICK_ID}/purchase`.

`{CLICK_ID}` = UUID визита AIO. Если в партнёрку слался `{{aio.visit.uuid}}` в `sub1`, то на их стороне `{CLICK_ID}` подставляется как `{sub1}`. В query-style `{CLICK_ID}` — это значение `visit_uuid`.

В `arrived_revenue={revenue}` плейсхолдер `{revenue}` заменяется соответствующим плейсхолдером на стороне Destination (например, `{payout}`) или статическим значением. Собрать такую ссылку в UI, а не набирать руками, — [how-to/postback-generator.md](../how-to/postback-generator.md).

### Партнёрка умеет слать постбэк только `POST`-ом — метод не важен

Приёмник конверсий принимает постбэк и `GET`-ом, и `POST`-ом: у всех четырёх эндпоинтов приёма метод не ограничен. Параметры читаются одинаково из query-строки и из тела запроса, поэтому рекламодатель, который умеет отбивать только `POST` с параметрами в теле, настраивается без обходных путей — ссылка та же, менять в ней ничего не нужно.

Это верно для всех трёх форматов постбэк-URL: path-style, query-style и `conversion-by-key`. То же и у постбэка на счётчик Destination (`/api/v1/trigger/conversion-by-destination-counter/...`).

### Рекламодатель не может переименовать свои параметры — формат `conversion-by-key`

`conversion-by-key` — формат для случая, когда рекламодатель шлёт постбэк **своими** именами параметров и переименовать их не может. В ссылке отдельно указывается, как эти параметры называются, и приёмник читает идентификаторы из них:

```
https://<домен>/api/v1/trigger/conversion-by-key
  ?visit_uuid_key=clickid
  &conversion_type_uuid_key=goal
  &clickid=<UUID визита>
  &goal=<UUID или slug типа конверсии>
```

`visit_uuid_key` называет параметр с UUID визита, `conversion_type_uuid_key` — параметр с типом конверсии. `clickid` и `goal` в примере — выбранные имена; приёмник не подставляет их автоматически. Можно задать другую пару, например `visit_uuid_key=cid&conversion_type_uuid_key=event&cid=<UUID визита>&event=lead`.

Оба `*_key`-параметра обязательны, и названный ими параметр обязан присутствовать в ссылке. Иначе приёмник синхронно отвечает `400` с текстом `visit uuid key not found` / `conversion type uuid key not found` / `visit uuid not found` / `conversion type uuid not found`.

Ссылку этого формата собирают руками: [how-to/postback-generator.md](../how-to/postback-generator.md) его не показывает, там только `Path` и `Query`.

### Дополнительные параметры постбэк-URL

- **`currency`** — валюта суммы в постбэке (`&currency=usd` / `eur` / `gbp` / …); записывается в колонку `Currency` конверсии, а кто и когда конвертирует сумму в USD — секция «Параметр `currency` — сумму конвертирует формула Business Model, а не приёмник» ниже.
- **`delay`** — откладывает **обработку** постбэка на стороне AIO на N секунд (`&delay=10`): запрос принимается сразу, а создание конверсии откладывается. Отправку постбэка рекламодателем параметр не двигает, и на ответ приёмника не влияет — ответ уходит сразу. Границы — **0–300 секунд**; значение вне диапазона (больше 300 или отрицательное) молча заменяется на **10 секунд**, запрос при этом не отклоняется и предупреждения не будет. Пусто или `0` — без дополнительной задержки; конверсия при этом появляется после обычной обработки постбэка, не мгновенно. Это параметр **входящего** URL, он отдельно от поля `Delay in seconds` исходящего трекера (см. [how-to/source-trackers.md](../how-to/source-trackers.md)).
- **`destination_uuid`** — привязать конверсию к конкретному Destination вместо того, куда фактически ушёл визит. Разбор — секция «Конверсия привязалась не к тому Destination — параметр `destination_uuid`».

### Параметр `currency` — сумму конвертирует формула Business Model, а не приёмник

Сумму постбэка в USD конвертирует `usd(...)` в формуле Business Model: засеянные `Arrived Revenue` / `Arrived Payout` используют `usd({{aio.conversion.fields.arrived_revenue}})` ([models/business-model.md](business-model.md)), валюта берётся из `currency`. Формула без `usd()` и готовое `revenue=` сами сумму не конвертируют. При обработке используется курс за текущий час. Если актуальный курс получить не удалось, берётся последний сохранённый; если курса нет вовсе, формула даёт `0`.

### Конверсия «не записалась» после завершения флоу — отложи обработку через `&delay=N`

Флоу уже завершён, а конверсия по постбэку **не записалась** — типичная причина это гонка по времени: постбэк пришёл раньше, чем Destination успел зафиксировать событие на своей стороне. **Основной фикс со стороны рекламодателя** — добавить `&delay=N` (напр. `&delay=10`) в свой постбэк-URL, чтобы AIO взял постбэк в работу на N секунд позже. Это **штатный** приём тайминга постбэка. Больше 300 секунд ставить бессмысленно: приёмник превратит такое значение в 10 секунд и отложит меньше, чем было задумано.

Диагностика самой пропажи конверсии (лид ушёл, подтверждения нет); разбор прихода постбэка по логам — [how-to/debug-with-logs.md](../how-to/debug-with-logs.md).

### Таблица параметров постбэка

| Тип параметра | Префикс | Куда пишется | Пример |
|---|---|---|---|
| **Поле конверсии** | без префикса | в саму конверсию | `arrived_revenue=10`, `sale_status=Approved` |
| **Поле визита** | `visit[]` | в поле визита: обычное значение заменяется, у `Registry` добавляется запись | `visit[current_sale_status]=Approved`, `visit[phone]=%2B79991234567` |
| **Дедупликация (только для `Multiple By Unique`)** | без префикса, ключ `unique` | используется для проверки «уже было такое или нет» | `unique=<offer_id>` или `unique=<product_uuid>` |
| **Готовая сумма Revenue** | без префикса, ключ `revenue` | прямо в `Revenue` конверсии, **мимо** бизнес-модели | `revenue=12.5` |
| **Привязка к Destination** | без префикса, ключ `destination_uuid` | в конверсию вместо Destination визита | `destination_uuid=<UUID>` |

`conversion_type_uuid` принимает **либо UUID, либо slug** конверсии. Если у типа несколько slug'ов — любой работает. Это позволяет принимать постбэки от разных рекламодателей с разной таксономией без нормализации на их стороне.

### `revenue=` в постбэке перебивает бизнес-модель, `payout=` — нет

`revenue=<сумма>` используется как готовый Revenue и отключает расчёт по `Revenue Distribution` и Business Model. Исключение — пустое значение и буквальное `revenue=0`: они расчёт не отключают. `revenue=0.00` считается непустой готовой суммой и отключает модель. Если revenue должна считать модель, параметр `revenue` лучше убрать и передать сумму полем конверсии.

`payout=<сумма>` из постбэка расчёт не отключает: `Payout` заново считается по `Payout Distribution` и Business Model. Без привязанной дистрибуции берётся первая Payout-модель тенанта; если подходящей модели нет, результат остаётся `0`. Передавать готовое `payout=` для обхода этого расчёта бессмысленно.

Отсюда ловушка: рекламодатель, который «на всякий случай» шлёт `&revenue=` рядом с настроенной бизнес-моделью, молча выключает её для своих конверсий — суммы в отчётах перестают сходиться с формулой, хотя настройки в UI выглядят рабочими.

Штатный способ принять сумму из постбэка — **поле конверсии** (например `arrived_revenue=<сумма>`), на которое ссылается формула Business Model: тогда работают и распределение, и коэффициенты. Полная связка «поле → Payout» — секция «Кастомная финансовая конверсия с суммой в постбэке — как настроить»; сами формулы — [models/business-model.md](business-model.md).

### Конверсия привязалась не к тому Destination — параметр `destination_uuid`

По умолчанию конверсия наследует Destination от своего визита — тот, куда визит фактически ушёл. Параметр `destination_uuid=<UUID>` в постбэке это переопределяет: в конверсию пишется указанный Destination, а не Destination визита. Тем же способом переопределяется рекламодатель — `advertiser_uuid=<UUID>`. Оба параметра опциональные: нет в ссылке — привязка идёт по визиту.

Отсюда симптом «конверсии сели не на тот Destination»: смотреть надо постбэк-URL, прописанный у рекламодателя. Если `destination_uuid` зашит в ссылку, все конверсии по ней будут висеть на этом Destination независимо от реального маршрута визита.

При сборке ссылки в UI дестинейшн дополнительно проверяется на принадлежность тенанту — [how-to/postback-generator.md](../how-to/postback-generator.md).

### `Last Advertiser` и `Last Destination` в таблице `Conversions` — что показывают

**`Last Destination` — колонка таблицы `Tracker → Conversions`, видимая по умолчанию; `Last Advertiser` — скрытая колонка категории `Latest`.** Они показывают Destination и Advertiser, записанные в самой конверсии: по умолчанию с визита при её создании, а `destination_uuid` / `advertiser_uuid` переопределяют эти значения независимо. Рядом скрытые `Destinations` и `Advertisers` показывают записанные в конверсии списки из визита; постбэк может переопределить и эти списки. Колонки включаются через `Settings → Table settings` ([reference/ui-common.md](../reference/ui-common.md)); в CSV значения идут UUID-ами ([how-to/analytics.md](../how-to/analytics.md)).

Отдельного группера `Last Advertiser` в отчётах нет: разрез по рекламодателю даёт позиционный группер `Advertiser` с позициями `First` / `Second` / `Third` ([how-to/analytics.md](../how-to/analytics.md)).

### Что значит `visit[<field>]=` — обновление поля визита через постбэк

`visit[<slug>]=<значение>` записывает **непустое скалярное значение** в существующее поле визита. Подходит не только для `current_sale_status`, но и для `email`, `fraud_score` или собственного поля. У обычного поля значение заменяется, у поля `Registry` — добавляется в историю. Пустое значение и `0` этим путём не записываются.

Поле визита **должно существовать** в тенанте (`Settings → Fields`). Если в URL передан `visit[nonexistent_field]=...` — само поле не обновится (его нет), но **конверсия всё равно создастся**. Ошибка в логах AIO **не записывается** — несоответствие имени поля молча игнорируется.

## Полный конверсионный цикл длинной воронки — как идут постбэки

Сценарий end-to-end для вертикали с длинной воронкой (регистрация → серия статусов → финальная оплата) идёт в три фазы: регистрация и `Lead`-постбэк → цикл статус-апдейтов → финальная конверсия.

### Регистрация и Lead-постбэк

1. **Визит на лэнде.** Пользователь пришёл на Preland → клик `{{link}}` → Offer с `{{form}}` → submit формы.
2. **AIO Antifraud.** Если включён во флоу — лид проходит встроенные проверки. Баллы и сработавшие правила записываются в настроенные поля визита, например `fraud_score` и `triggered_rules` (настройка).
3. **Push в Destination.** AIO шлёт лид в API рекламодателя. Получает ответ — auto-login URL + другие поля.
4. **Auto-login redirect.** Пользователь автоматически попадает в кабинет рекламодателя через возвращённую ссылку.
5. **Рекламодатель регистрирует лида** на своей стороне — присваивает свой ID, кладёт в очередь обработки.
6. **Постбэк на `Lead`.** Рекламодатель шлёт нам:
   ```
   .../conversion-request?visit_uuid=<UUID>&conversion_type_uuid=<UUID-Lead>
   ```
   AIO создаёт конверсию `Lead` (Single). Это единственный раз — повторно `Lead` не возникнет.

### Цикл статус-апдейтов

**Цикл обработки у рекламодателя.** Лид проходит серию статусов дни/недели:

- Не удалось связаться → `sale_status=No Answer` → постбэк на `Sale Status Update`.
- Попросил связаться позже → `sale_status=Callback` → постбэк.
- Заинтересовался → `sale_status=Hot Lead` → постбэк.
- … и так далее.

Каждый постбэк:
```
.../conversion-request
  ?visit_uuid=<UUID>
  &conversion_type_uuid=<UUID-Sale-Status-Update>
  &visit[current_sale_status]=<статус>
  &sale_status=<статус>
```
создаёт новую конверсию `Sale Status Update` И обновляет поле визита `current_sale_status`.

### Финальная конверсия

**Финальная конверсия.** Лид совершил целевое действие (первая оплата). Рекламодатель обычно шлёт два постбэка почти одновременно:

- На `Sale Status Update` с `sale_status=Approved/Sale` — фиксирует факт в истории.
- На тип целевого действия (`Single Conversion`; в шаблоне Finance он заведён рядом с `Lead`, со своим слагом и привязанными деревьями Revenue / Payout) — фиксирует само событие. Часто с `arrived_revenue=<сумма>`.

**Дальнейшие апдейты** (повторные покупки, апсейлы и т.д.) — продолжают идти через `Sale Status Update`.

### Визуализация цикла

```
Визит ──→ Lead (single)
         ├──→ Sale Status Update (No Answer)
         ├──→ Sale Status Update (Callback)
         ├──→ Sale Status Update (Hot Lead)
         ├──→ Sale Status Update (Approved)
         └──→ <тип целевого действия> (single, +revenue)
```

На визите `current_sale_status` остаётся статус из последнего обработанного постбэка; в примере это `Approved`.

## Конверсий нет, хотя рекламодатель шлёт постбэки — диагностика

### На каком домене смотреть ответ постбэка — домен панели vs трекерный домен

**HTTP-ответ, который видит рекламодатель, зависит от того, на какой домен он шлёт постбэк:**

- **Домен панели (`app.aio.tech`)** — приёмник отвечает по-настоящему: `200 {"status":"catch"}` на приём, синхронный `400 {"status":"error","message":"…"}` при пустом или отсутствующем параметре. **Для проверки постбэка и чтения текста ошибки шлите сюда.**
- **Трекерный / контентный домен тенанта** — HTTP-статус ответа **результат постбэка не показывает**: код не зависит от того, принят запрос или отклонён, и синхронной ошибки (`visit uuid not found` и т.п.) в ответе не видно. Успех определяйте по тому, **появилась ли конверсия в AIO** (`Tracker → Visits` / Conversions, логи), а не по коду ответа. Сам постбэк при этом обрабатывается так же, как на домене панели, — отличается только ответ.

### В логах рекламодателя `200`, а конверсии нет — `200` не значит «конверсия создана»

Ответ `200 {"status":"catch"}` на **домене панели** означает приём запроса: конверсия создаётся уже после его обработки, поэтому `200` не подтверждает её создание. При ошибке в идентификаторе визита или типа ответ может быть и не `200`. (На трекерном домене статус ответа результат не показывает вовсе — см. секцию выше.)

Что проверить, если конверсия не появилась:

- `Single Conversion` и такой тип уже есть на визите — повторный постбэк игнорируется;
- `Multiple By Unique` и такой `unique` уже есть на визите — дубль;
- существует ли визит и тип в его тенанте;
- добавлена ли задержка `&delay=N`: конверсия появится только после отложенной обработки постбэка.

Для `conversion-request` ошибки отсутствующих или пустых параметров возвращаются синхронно **на домене панели** (`app.aio.tech`): `400 {"status":"error","message":"…"}`. Тексты: `visit uuid not found` / `conversion type uuid not found` — параметра нет; `empty visit uuid` / `empty conversion type uuid` — параметр пуст. На трекерном домене тенанта этих синхронных ошибок в ответе не видно (секция «На каком домене смотреть ответ постбэка»), поэтому отлаживайте параметры на домене панели. `200` (на домене панели) не исключает дублирование или ошибку при последующей обработке.

Где посмотреть, дошёл ли постбэк и что с ним стало, — [how-to/debug-with-logs.md](../how-to/debug-with-logs.md).

### Конверсий нет — проверь slug и Conversion Types тенанта

Первым делом проверь, какие slug'и шлёт рекламодатель и есть ли они в Conversion Types тенанта. Если нет — добавить slug к существующему типу или создать новый под событие (если событие новое, типа Install).

### Не вижу последний статус на визите — рекламодатель не передаёт `visit[]`

Проверь, что в постбэке рекламодатель передаёт `visit[current_sale_status]=...`, а не только `sale_status=...` без префикса. Без `visit[]` обновляется только сама конверсия, поле визита остаётся как было.

### Не создавать `+Conversion Type` под каждый статус рекламодателя

Если статус — это апдейт в цикле обработки (No Answer, Recall, Hot Lead, и т.п.) — он идёт через **один** Conversion Type `Sale Status Update` с разными значениями `sale_status`.

### Один Sale Status Update под несколько рекламодателей с разными именами

Один тип `Sale Status Update` может иметь несколько slug'ов — если разные рекламодатели шлют разные имена (`sale_status` / `status` / `status_update`). Все попадают в один тип.

### Не видно конверсию, которую видит коллега — проверь доступ к Campaign

Список конверсий ограничен тенантом. Для пользователя без полного доступа таблица дополнительно отбирает конверсии по доступным `Campaigns` **и** `Sources`. Если «не вижу конверсию, которую видит коллега» — проверьте доступ к обеим сущностям.

### Conversions-таблица (`Tracker → Conversions`) — колонки и экшены

Зелёная кнопка в шапке таблицы — `Postback generator` (собирает ссылку постбэка для рекламодателя, ничего не сохраняет — [how-to/postback-generator.md](../how-to/postback-generator.md)). Отдельной кнопки `+ Conversion type` на странице больше нет: создание типов и бизнес-моделей уехало в шеврон-дропдаун рядом с кнопкой, пункты `Business models` и `Manage types`. В MTK кнопка та же, а дропдауна на этой странице нет вовсе. Сама таблица — список конверсий. Колонки: **Type** (configured — напр. `Lead` / `Registration` / `Sale Status Update`), `Geo` (by IP), `Visit Date` / `Conversion Date`, **`Delta`** (время визит → конверсия), `Revenue`, `Campaign`, `Source`, `Last destination`, `Currency`, `Offer name` (+ скролл вправо — поля визита/конверсии). `Offer Visit` и `Autologin` — не системные типы, а вручную созданные Conversion Types (частный кейс).

Флаги визита, с которого пришла конверсия, видны колонками `Trash` и `Bot`, а в ERP ещё и `Back Fix` (категория `Forced steps`): `Yes` — визит перебросило вперёд по флоу через Backfix. Колонка `Back Fix` попадает и в CSV-выгрузку — так видно, сколько конверсий дал трафик, вернувшийся через «Назад».

**Экшены (правый клик по конверсии):**
- **Show visit** — перейти к визиту, породившему конверсию.
- **Show nearby visits** — показать другие конверсии того же визита.
- **Re trigger trackers** — переотправить конверсию в Source с актуальными значениями (механика — [models/tracker.md](tracker.md)).
- **Update revenue** / **Update payout** — вручную задать revenue / payout конверсии.

## Смежные темы

**Концепты:**
- [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) — где в жизни визита возникают конверсии.
- *Вертикали в арбитраже трафика* — почему `Sale Status Update` критичен в вертикалях с длинной воронкой.
- [models/distributions-model.md](distributions-model.md) — Payout/Revenue Distribution, как BM связана с Conversion Type.
- [models/business-model.md](business-model.md) — модель Business Model (Type + Format + Formula, восемь засеянных моделей, дефолты Arrived Revenue / Zero Payout).
- [mechanics/notifications-flow.md](../mechanics/notifications-flow.md) — ноды флоу рассылки; поле `Notification flow` на Conversion Type как точечный алерт.

**Playbooks:**

**How-to:**
- [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md) — рассылка по конверсии: ремаркетинг-кампания с триггером `Conversion`.
- [how-to/destinations.md](../how-to/destinations.md) — настройка Destination + `Conversion Cap`.
- [how-to/source-trackers.md](../how-to/source-trackers.md) — постбэк-трекеры, FB CAPI Custom Key/Value, формат постбэк-URL.

**Эвристики:**
- [heuristics/conversion-ai-testing.md](../heuristics/conversion-ai-testing.md) — мягкие Conversion Type под низковолюмные вертикали; Attribution toggle команды.

**Реф:**
- [reference/glossary.md](../reference/glossary.md) — `Conversion Type`, `Uniqueness Strategy`, `Sale Status Update`, `current_sale_status`, `Conversion Postback URL`, `Sale Status Mapping` (что не в AIO).

---

## Типы конверсий в MTK: использование готовых, не создание

В MTK `Settings → Conversion Types` недоступен для создания — управление типами скрыто by-design. Нужные типы уже в тенанте: скопированы из шаблона вертикали при создании (`AIO → Templates`; засеянный набор MTK-шаблонов Finance / iGaming / Sweeps — раздел «Какие Conversion Types заведены в тенанте из коробки») или заведены командой AIO разово.

Что в MTK делается самостоятельно:

- Просматривает лог конверсий (`Tracker → Conversions`) по доступным Campaigns и Sources.
- Принимает постбэки рекламодателя на имеющиеся типы — по UUID или slug, формат стандартный (раздел «Постбэк-URL — общий формат и параметры»).
- Строит аналитику по конверсиям и фильтры по полям визита, например `current_sale_status`.

В интерфейсе MTK режимы `Conversions AI` и `Metrics AI` не предлагаются.

**Если нужного типа нет** — запросите проверку и настройку у команды AIO. Для постбэка должны совпадать тип в тенанте визита и переданный UUID или slug; создание также зависит от `Uniqueness Strategy`.

В MTK используются уже заведённые Conversion Types, а раздел их создания и редактирования скрыт. Разбор остальных возможностей билда — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

## Граничные случаи

- Если один slug добавлен к нескольким `Conversion Types` тенанта, обработчик выбирает тип, созданный последним. Чтобы постбэк попадал в нужный тип, используйте разные слаги или точный UUID.

## Конверсия только при известном `visit_uuid`

Конверсия создаётся **только для существующего визита с `visit_uuid`** — по одним хедерам server-to-server постбэка визит не создаётся. Не все CRM умеют отправлять `Sale Status Update` при каждой смене своего статуса; возможность нужно согласовать с рекламодателем.

### Постбэк с неизвестным `visit_uuid` — конверсия не создастся

Для создания нужен существующий визит и тип конверсии в его тенанте. Если их не удалось найти, возникает ошибка обработки. Проверяйте ответ приёмника и логи ([how-to/debug-with-logs.md](../how-to/debug-with-logs.md)): отсутствие визита или типа не гарантирует штатный `200`.

### Истекает ли `visit_uuid` — возраст визита не ограничивает постбэк

Обычный `visit_uuid` не истекает: визит сохраняется в базе. Возраст визита сам по себе не запрещает постбэк — через минуту или через год. Создание конверсии зависит от существующего визита, типа и `Uniqueness Strategy`.

Исключение — визит, отсеянный в trash ещё на входе: если постбэк по нему пришёл сильно позже самого визита, AIO может уже не найти визит, и конверсия не создастся.

Скользящее окно 30 дней (→ [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md)) — это про **сессию визита** (когда возврат того же браузера считается тем же визитом, а не новым), **не** про срок жизни `visit_uuid`. Идентификатор визита не «истекает» через 30 дней.

## Reject-tracking как отдельный Conversion Type

Реджект push можно считать отдельным `Conversion Type`: выберите его в `Reject conversion spawn` ноды `Destination`. Конверсию запускает неуспешный результат push; наличие перехода `destination-rejected` для этого не требуется ([models/flow-model.md](flow-model.md)). В шаблонах Finance и Sweeps такой тип уже заведён и прописан в ноде: `Lead Rejected` (`Multiple Infinity`, `Business Value = Bad`, без revenue; в Sweeps слаги `reject`, `rejected`, `lead_rejected`). Стратегия именно `Multiple Infinity`, а не `Single`: лид может быть отклонён, а на повторном пуше принят, поэтому реджекты считаются событиями, а не признаком визита (описание типа в шаблоне Finance: `A lead can be rejected first and pushed successfully on a retry, so rejects are counted as events`; в Sweeps — `One lead can collect several rejects and still be pushed by a retry`). В шаблоне Nutra в этой роли стоит тип `Trash` (`Single Conversion`); в iGaming нода `Offer` реджект-тип не спавнит. Метрика `Rejected Leads` в шаблонах Finance и Sweeps считает этот тип через `Once Conversions By Type Count` — один реджект на визит, сколько бы их ни пришло.

## Что происходит, если визит зареджектился в Destination

После отказа Destination визит идёт по настроенному переходу `Destination Rejected` или `Destination Interacted`. Если этих переходов нет, возвращается к Content текущего шага; при отказе из-за капы сначала проверяется `Destination Full`. Thank-you-страница зависит от выбранной ветки: на неё может вести и reject. Антифрод перед push также направляет визит по настроенной ветке; чтобы показывать thank-you, свяжите её с соответствующим шагом.

## Кастомная финансовая конверсия с суммой в постбэке — как настроить

Когда рекламодатель шлёт в постбэке денежную сумму, которую нужно завести в Payout (например, `OrderValue` — размер целевого действия), собирается связка из **4 шагов**. Слаг типа вы называете сами — здесь `OrderValue` для примера:

1. Поле конверсии `OrderValue`.
2. `Conversion Type OrderValue` с этим полем, обязательно **Multiple** (не Single — это серия событий).
3. `Business Model OrderValue`, мапящая «поле постбэка → Payout».
4. В `Payout Distribution` папка с правилом `Conversion Type = OrderValue → Business Model OrderValue`.

Постбэк: `OrderValue=20` (вместе с визитом и типом). Проверьте, что формула выбранной Payout-модели читает это поле. Одно поле постбэка само по себе не задаёт Payout; при отсутствии Distribution может примениться первая Payout Business Model тенанта.

## Подписочная модель (mobile apps) — три Conversion Type под Install / Trial / Subscription

Для раздельного учёта можно завести `Install` / `Trial` / `Subscription` как три `Conversion Types` либо сопоставить события подходящим существующим типам. Трекер настроен на один тип: под три разных типа нужны отдельные FB CAPI трекеры. Кнопки копирования трекера в UI нет — настройки дублируют вручную ([models/tracker.md](tracker.md)).

## Постбэк-трекер — обязательные поля при создании

При создании исходящего постбэк-трекера (`Tracker → Trackers` в ERP; `Settings → Trackers` в MTK) указывают `Source`, тип конверсии и URL. В поле `Delay` выберите задержку или `No delay`.

## FB CAPI и FB Catalogs — как разделить

Имя FB CAPI события задаётся в трекере, например `Lead` или `Purchase`. Чтобы разделить «обычный FB» и «FB Catalogs», можно продублировать `Source` и привязать к нему отдельные трекеры.

Если за аккаунт платят процент со спенда, комиссия задаётся у привязанного `Facebook Supplier`; без поставщика — полем `Commission` соц-профиля в `Meta`. AIO прибавляет действующую комиссию к спенду при пуллинге (`Total Spend` = `Raw Spend + Commission`). Настройка — [how-to/campaigns.md](../how-to/campaigns.md), аллокация — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

## FB CAPI source — Dynamic Actions

Для source `FB CAPI` при генерации ссылки указывают пиксель, токен и **Dynamic Actions** — выбор, какое FB-событие на каждый тип конверсии (`Lead`, `Purchase`).

## Destination External ID — что это за поле визита

`destination_external_id` (в UI — `Destination External ID`) — поле визита с внешним идентификатором, который вернула интеграция при push. В API-интеграции это может быть ID лида рекламодателя; `Simple Redirect` генерирует UUID. Поле видно в `Tracker → Visits` и используется для поиска по этому идентификатору. Имена `external_id` / `Destination External UUID` не являются отдельными полями визита.

## 3rd party PWA — Sub-параметр для конверсий

Для возврата из 3rd party PWA передают сессию визита в sub; для постбэков сервису может понадобиться также `visit_uuid`. Параметры задаются интеграцией. FB-параметры сохраняются в AIO до перехода в PWA.

## AIO Meta и потерянные косты — две причины

`Meta Spend` сопоставляется с трафиком по Meta-идентификаторам визита. В меню инструментов таблицы с Meta-колонками есть `Attribution level` (`Campaign` / `Adset` / `Ad`); дефолт — `campaign`, выбор хранится в браузере. Meta-группер или фильтр может определить уровень сопоставления независимо от выбора. Разбор — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

Проверьте причины отсутствующего соответствия:
1. В визитах не записаны нужные Meta-идентификаторы, например из-за ссылки без трекинговых параметров.
2. На FB-аккаунте есть кампании без соответствующего трафика в AIO.

## FB CAPI — что AIO отправляет в событии

AIO собирает server-side CAPI-событие. Из визита заполняются `country_code`, `external_id` (= `visit_uuid`), User-Agent и IP. `email`, `phone`, `first_name`, `last_name`, `city`, `zip`, `fbc` и `fbp` берутся из настроек трекера после подстановки плейсхолдеров и добавляются, если непустые. В `CustomData` уходит `value` из настройки `Revenue` в валюте `USD`; чтобы передавать сумму конверсии, настройте соответствующий плейсхолдер.

Если в настройках после подстановки есть `fbclid`, а `fbc` пуст, он генерируется в формате `fb.1.<ts>000.<fbclid>`. При том же условии и пустом `fbp` генерируется `fbp` со случайным числовым суффиксом. Матчинг события с рекламой выполняет Meta.

## fbclid и атрибуция Adset — как это работает

AIO сохраняет `fbclid` и рекламные идентификаторы в полях визита. В CAPI `fbclid` используется для генерации `fbc`, если готового значения нет; само событие не задаёт целевой Adset/Ad. Атрибуцию выполняет Meta. При жалобе «лид упал не в тот Adset» проверьте переданные `fbc`/`fbp` и настройки трекера, затем атрибуцию в Meta.

## Дубликаты FB-лидов — диагностика

`event_id` AIO проставляет только серверному CAPI-событию (`conversion.<visit_uuid>.<conversion_type_uuid>`); у браузерного события пикселя его нет, поэтому пара «пиксель + CAPI» по идентификатору не дедуплицируется. Типовые источники дублей:
1. Третья сторона (рекламодатель) дополнительно отбивает Pixel.
2. На лэнде остался свой Pixel-скрипт, отправляющий то же событие параллельно SDK.
3. Meta не схлопнула события — проверьте их идентификаторы и источники в Events Manager.

Диагностика — `FB Events Manager`.

## Google Ads — Account ID при offline-конверсиях и разбивке костов

**Основной путь:** при генерации Google-ссылки в `Link Generator` указать **отдельно `Account ID`** — он вписывается в URL и пишется в визит автоматически. Это нужно при загрузке offline-конверсий (CSV — External Report `Google Conversions CSV`), чтобы разделять конверсии по аккаунту через фильтр `Google Account ID`.

Account ID нужен не только для разделения конверсий, но и для **разбивки ручных костов по аккаунтам**: если на одну кампанию запускается несколько Google-аккаунтов, косты вводятся по сумме каждого аккаунта и разбиваются по визитам с соответствующим `Google Account ID` — см. [how-to/campaigns.md](../how-to/campaigns.md) → Update Costs.

### Альтернатива через флоу — если ссылка уже разлита

**Альтернатива** (если ссылка уже разлита или айди не известен при генерации): маппинг `domain → Account ID` через `Fill Field` ноды во флоу — см. [how-to/campaigns.md](../how-to/campaigns.md) секцию «Google Ads — Account ID и Conversion ID для атрибуции».

## Google: два пути конверсий — gtag и Google Conversions CSV

В Google Ads конверсии передаются двумя путями. Рекомендуется слать **оба параллельно**:

- **gtag** — макрос `AIO GTAG`: в шаблонах Finance / Nutra / Sweeps (ERP и MTK) он засеян активным, но не глобальным — на лэнд его ставят тегом `{{aio:macros:aio_gtag}}`; в шаблонах iGaming его нет. На трафик других площадок не влияет: без Conversion ID Google у визита тег не грузится. Конверсию шлёт на успешной отправке формы, итог по визиту пишет в поле `gl_gtag_status`. Работает только на лэнде, поэтому **офлайн-конверсии (например, покупку) не покрывает**. Настройка макроса — [reference/sdk.md](../reference/sdk.md).
- **External Report `Google Conversions CSV`** — офлайн-конверсии и прочие события вне лэнда. Настройка — [how-to/api.md](../how-to/api.md) → Google Conversions CSV.

Пути **не конфликтуют**: оба передают `gclid`, и Google дедуплицирует конверсии по нему. gtag даёт оперативность (лид отбивается сразу с сабмита), CSV — полноту (офлайн-конверсию иначе не передать — для неё используйте `Google Conversions CSV`, External Report; этот путь живёт в ERP-виде).

### Офлайн-конверсия в Google — не имитировать «фантомным» заходом

**Анти-паттерн:** имитировать офлайн-конверсию «фантомным» повторным заходом на thank-you-страницу с тем же `gclid` — работает очень нестабильно, жёсткий костыль. Не использовать. Для офлайн-конверсий — **только** External Report `Google Conversions CSV`.

## Маршрут визита не детерминирован заранее — метрики Destination

Маршрут визита в AIO выбирается по мере переходов Flow: Destination определяется в момент перехода. В разборе затрат полезен контекст `Destination × Campaign` / `Destination × Source`. Входящие конверсии учитываются по своему Destination-снимку. `Meta Spend` распределяется по посещениям и Meta-идентификаторам; механизм — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

## Капа на конверсии ставится только на Destination — режимы Infinity, Daily, Monthly и Lifetime

Капа **на конверсии** в AIO ставится только на Destination/оффер, и это `Conversion Cap`. На **бренд** капу поставить некуда. Под-капа на один оффер под разных байеров — нет.

Режимов `Cap Type` четыре: `Infinity` (без лимита; в диалоге `Unlimited`), `Daily`, `Monthly` и `Lifetime`. Для Daily/Monthly граница периода — полночь дня / первого числа месяца в `Reset timezone` (по умолчанию UTC); счётчик обнуляется вскоре после этой границы. `Lifetime` автоматически не обнуляется: лимит увеличивают или сбрасывают счётчик вручную. Настройка — [how-to/destinations.md](../how-to/destinations.md).

В `When the cap is full` выберите `Reject traffic` или `Do nothing`. По умолчанию `Reject traffic`: заполненная капа блокирует исходящий push; при наличии `Destination Full` визит идёт по нему, иначе — по reject/interacted или назад к Content. При `Do nothing` трафик продолжается, капа только считает; для алертов настройте правило уведомлений.

При блокирующей капе уведомление отправляется настроенному активному `Monitoring User` с Telegram chat ID. Приём входящих постбэков капа не блокирует.

Слово «cap» встречается в интерфейсе ещё в одном месте: в статусе remarketing-кампании (`Seeding paused (hourly cap)`) так подписан её часовой лимит отправок — к `Conversion Cap` он отношения не имеет ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)).

---

## Эвристики — Conversion Types и Business Model

### Uniqueness Strategy — выбирай по семантике события, не «как у соседа»

**Когда:** заводишь новый `Conversion Type`.

**Делай:** выбирай `Uniqueness Strategy` **по семантике события**, не «как у соседа»:

- **`Single Conversion`** — событие может произойти **ровно один раз** на визит. `Lead`, `Registration`, `Install`, `Push Subscribe`.
- **`Multiple Infinity`** — серия событий с одинаковой природой, дедупликация **не нужна** (или невозможна). `Sale Status Update`, `Push Click`, любые «обновления статуса», а также `Lead Rejected` и `Fraud Lead` — в шаблонах Finance / Sweeps оба заведены как `Multiple Infinity`: лид может быть отклонён, а на повторном пуше принят, поэтому реджект считается событием, а не признаком визита.
- **`Multiple By Unique`** — серия событий с дедупликацией **по бизнес-значению**. Повторные покупки (одна на `transaction_id`), `Add to Cart` (одна на `product_uuid`), `Payout` (один на `payment_id`).

Для `Multiple By Unique` — обязательно передавать в постбэке `&unique=<hash>` или `&unique=<id>`. **Если параметра нет, все постбэки получают пустое значение `unique` и считаются дублями друг друга**: создастся только первая конверсия, остальные молча отбросятся, то есть тип поведёт себя как `Single Conversion`. Симптом — «настроил Multiple By Unique, а конверсия одна на визит».

**Особый случай:** партнёрки часто шлют **дубли постбэков**. Для финансовой конверсии используй `Multiple By Unique` + `Transaction Hash` от партнёрки — защита от двойного учёта.

**Почему:** ошибка в Uniqueness Strategy не ловится сразу — проявляется через недели, когда обнаруживается «у визита 15 покупок» (если поставили Multiple Infinity для разового события) или «не вижу повторные покупки» (если поставили Single для серии).

### Финансовые поля — не смешивай Revenue и Payout

**Когда:** проектируешь Conversion Type + Business Model для финансовых событий (покупка, выплата, целевое действие с суммой).

**Делай:** **не смешивай семантику** финансовых полей (определения Revenue / Payout — [models/business-model.md](business-model.md)). Заводи **отдельные поля** под каждое значение:

- `arrived_revenue` — доход, который платит рекламодатель.
- `order_value` (или `payout`) — сумма целевого действия на стороне рекламодателя.

Передавайте значения в разные поля и используйте их в соответствующих формулах Business Model / Revenue и Payout Distribution. Само имя custom field не назначает ему Revenue или Payout: в аналитике отображаются рассчитанные значения конверсии.

### Новый поток — заводи отдельный Source + отдельные трекеры

**Когда:** добавление принципиально нового потока конверсий — мобильная связка (Install / Trial / Subscription), новая вертикаль с другой воронкой, отдельная партнёрка с другой таксономией.

**Делай:** заводи **отдельный `Source`** и **отдельные постбэк-трекеры** под новый поток. **Не привязывай** к существующему Source — продублируй (Source копируется через duplicate; трекеры — руками, кнопки «копировать трекер» в UI нет).

**Почему:**

- Source — это разделение в аналитике. Если новый поток поедет через старый Source, его метрики смешаются со старыми → не отличить «было 10 покупок на FB CAPI» от «10 на новой мобильной связке».
- Трекер настроен под конкретный тип конверсии + конкретный Source. Один трекер не покрывает «и Install, и Purchase» — это разные ивенты, нужны разные трекеры на каждое событие.
- Параметры FB CAPI / Google Ads / постбэка партнёрки могут не совпадать со старыми → один Source не подойдёт.

Также см. раздел «Подписочная модель (mobile apps) — три Conversion Type под Install / Trial / Subscription» и [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

### Destination: штатно — By Advertiser (ссылки) или By Integration (интеграции)

**Когда:** создание нового оффера через `Tracker → Destinations`.

**Делай:** для ссылочного оффера удобно использовать `By Advertiser`: Destination получает интеграцию рекламодателя и участвует в аналитике по нему. `By Integration` создаёт самостоятельный Destination выбранной интеграции; поля `Advertiser` в этой форме нет.

`Simple Redirect` без Advertiser подходит и для офферной ссылки. Без привязки нельзя собрать её под конкретным Advertiser в отчёте. Счётчик посещений группы `Offer` зависит от поля `Metrics event group` типа Advertiser; обычный редирект без него не попадает в эту группу. Это причина рекомендовать Advertiser для офферов, а не запрет самостоятельных редиректов. Настройка и замена Destinations — [how-to/destinations.md](../how-to/destinations.md).

Также см. [reference/glossary.md](../reference/glossary.md) → `Destination Integrations`.

---

## Процедуры

### Создать Conversion Type

**Путь:** `Settings → Conversion Types → +Conversion Type`.

**Поля:**

- **Name** (обязательно).
- **Color** (обязательно); **Slugs** / **Description** (опционально).
- **Uniqueness Strategy** — `Single Conversion` / `Multiple Infinity` / `Multiple By Unique` (см. раздел о стратегии выше).
- **Business Value** — `Good` / `Neutral` / `Bad` (напр. `Lead` → `Good`, `Fraud Lead` / `Lead Rejected` → `Bad`).
- **Reward Weight** — обязательное числовое значение для веса в AI-сплитах Campaigns / Landings; для базового веса укажите `1`.
- **Payout Settings / Revenue Settings** — выбор `Payout Tree` / `Revenue Tree` из дропдаунов (привязка к Payout / Revenue Distribution).
- Для отдельного счётчика типа создайте `Conversions Count`: `Settings → Metrics → +Metric → Conversions Count` → выберите Conversion Type. Сам лог `Tracker → Conversions` доступен независимо от отдельного счётчика. См. [models/metric.md](metric.md).
- **Notification Flow** — поле для точечного алерта на конверсию этого типа; рассылку им не настраивают (см. раздел «Как запустить рассылку при получении конверсии» ниже).

### Папка с Business Model «если Conversion Type = X»

Когда: для определённого Conversion Type (например, кастомной финансовой конверсии с суммой) нужно применить специфичный Payout-расчёт.

**Путь:** `Payout Distribution` → `+Folder` → условие `Conversion Type = <type>` → внутри Folder → Business Model.

**Шаги:**

1. Открыть Payout Distribution.
2. `+Folder` — создать папку.
3. На папке задать `Rule to pass`: `Conversion Type = <UUID конкретного типа>`.
4. Внутри папки добавить `Add Payout` + Business Model.
5. Сохранить.

Также см. раздел «Кастомная финансовая конверсия с суммой в постбэке — как настроить» выше — это полный паттерн из 4 шагов.

### Как запустить рассылку при получении конверсии

**Штатный способ — ремаркетинг-кампания с триггером `Conversion`** и выбранным типом конверсии. Визит должен уже входить в аудиторию кампании; без аудиторных привязок никто не подходит. Для запуска также нужны неархивная кампания, Flow с начальным состоянием и отсутствие активного Message этой кампании у визита; проверяется удаление визита из аудитории. Настройка — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md), концепт — [models/marketing-flow.md](marketing-flow.md).

Триггер срабатывает и на коррекционной конверсии, а не только на первичном приходе конверсии этого типа.

Поле `Notification flow` на самом Conversion Type (`Settings → Conversion Types → <type>`) — второй, кастомный путь. Он жив и работает, но аудиторного гейта у него нет: визит сеется в выбранное `Notifications`-флоу при каждой конверсии этого типа. Годится для точечного алерта (например, сообщение в Telegram на невалидную конверсию), а не как способ настроить рассылку.

Оба пути могут работать параллельно: `Notification flow` на типе не отменяет триггер ремаркетинг-кампании. Ремаркетинг запускается при выполнении своих условий; наличие двух настроек не гарантирует два запуска на каждую конверсию. Ноды рассылки — [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

### Retrigger конверсии

Когда: пиксель / токен на момент создания конверсии были инвалидны — нужно переотправить конверсию в Source; механика `Re trigger trackers` — [models/tracker.md](tracker.md).

**Путь:** `Tracker → Conversions` → ПКМ на конверсии → `Re trigger trackers`. Массовый — выделить конверсии чекбоксами в том же списке → ПКМ → `Re trigger trackers` → `Confirm`. Нужны права `tracker.conversions.edit.re-trigger`. Программный вызов через API — [how-to/campaigns.md](../how-to/campaigns.md).
