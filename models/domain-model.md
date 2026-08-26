---
id: domain-model
title: Доменная модель AIO — карта сущностей и связей
description: Корневой спайн доменной модели. Что за сущности есть в AIO (Tenant, Campaign, Flow, Visit, Conversion, Destination, Distribution…) и как они сцеплены. Точка входа во все concept-слои.
doc_type: model
builds: [erp, mtk]
related: [tenant, user, permissions-model, source, campaign, flow-model, visit, visit-lifecycle, visit-field, landing, sdk, form, content-library, landings, destination, advertiser, conversion-model, tracker, distributions-model, business-model, domain, server, architecture, marketing-flow, notifications-flow, metric, analytics, domains, glossary, placeholders, ui-map]
language: ru
updated: 2026-08-12
---

# Доменная модель AIO — карта сущностей и связей

Это **корень** доменной модели: одна карта, которая называет ядро сущностей AIO и показывает, **как они связаны**. Отсюда расходятся развилки — вглубь к детальным моделям, вбок к how-to и разбору частых кейсов. Читать первым, если хочешь понять «как AIO устроен в целом», а не отдельную фичу.

> Принцип: **детали — в профильных файлах**, здесь только определения в одну строку + связи. Если факт раскрыт глубже — даётся ссылка, не дублируется.

## Как сущности AIO связаны — краткая схема за 30 секунд

- **Tenant** — рабочее пространство клиента; внутри живёт всё остальное.
- **Campaign** связывает **Flow** (путь трафика) + домен + **Source** (откуда трафик).
- Реклама приводит **Visit** — одну браузерную сессию. Визит — «живой токен», который **проходит по шагам (Flow States) флоу** через **переходы (Transitions)**.
- На шагах визиту показывают **Landing** (Content), фильтруют **Filter**, заполняют **поля визита**, и в конце пушат лид в **Destination**.
- Рекламодатель присылает **постбэк** → создаётся **Conversion** (Lead/Registration/Purchase…), привязанная к визиту через `visit_uuid`.
- Деньги (Revenue/Payout) и динамический контент назначаются **Distribution**-деревьями автоматически.

```
Tenant
  └─ Campaign ──binds──> Flow ──made of──> [Flow States] ──linked by──> [Transitions]
        │                                        ▲
        │                                        │ a Visit occupies one State at a time
   Source ──entry──> Visit ─────travels─────────┘
                       │
                       ├─ shown Landing (Content step) ─ carries ─ SDK / Form
                       ├─ filtered by Filter (Passed/Rejected)
                       ├─ fields written by Fill fields / Distribution
                       └─ pushed to Destination ──> postback ──> Conversion (visit_uuid)
```

---

## Какие сущности есть в AIO — полная карта

Каждая сущность — одна строка + ссылка на её **модель** или ближайший детальный файл. Строки с префиксом `·` — под-сущности без отдельной модели (ссылка на ближайший источник); строки без префикса — у сущности есть собственный файл-модель.

### Аккаунт и организация: Tenant, User, Permissions

- **Tenant** — воркспейс клиента (он же *build*/тариф: MTK/ERP). Контейнер всего. → [models/tenant.md](tenant.md)
- **User** — аккаунт-actor; **один аккаунт может состоять сразу в нескольких тенантах**. Владелец сущностей (Owner), Launcher. → [models/user.md](user.md)
- **Permissions** (Role / Position / Team / Imperative / Sharing) — **модель доступа**: что у юзера есть в данном тенанте. → [models/permissions-model.md](permissions-model.md)

### Маршрутизация трафика (runtime): Source, Campaign, Flow, Visit

- **Source** — пресет входа: `suuid` + маппинг URL-параметров в поля визита + постбэк-трекеры. → [models/source.md](source.md)
- **Campaign** — связывает один Flow + Link-домен + Source; запускаемая единица (инстанс Flow). → [models/campaign.md](campaign.md)
- **Flow** — конструктор пути визита Source→Destination: граф **Flow States** + **Transitions**. Один Flow → много Campaign. → [models/flow-model.md](flow-model.md)
- **Visit** — одна браузерная сессия в кампании (`cuuid`+`suuid`+cookie); «живой токен», идущий по флоу. Две оси: **Visit Status** (Wait/Live/Left) и **текущий Flow State**. → [models/visit.md](visit.md) *(процесс — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md))*
- **Visit Field** — именованный слот данных на визите; субстрат всей системы. → [models/visit-field.md](visit-field.md)

### Из чего состоит Flow — Flow States и Transitions

Flow — это граф: узлы (**Flow State**, «шаг») + рёбра (**Transition**, «переход»). Полное описание каждого — [models/flow-model.md](flow-model.md).

- · **Flow State (шаг)** — узел графа: Source / Content / Filter / Temp Destination / Domain change / Fill Fields / Fields by distribution / Condition / Fill form / Spawn conversion / Field verification / SubFlow / Notifications flow / Finish.
- · **Transition (переход)** — ребро графа: Arrived / Handle / Handle-form / Handle-link / Passed / Rejected / Destination-pushed / Destination-rejected / Destination-interacted / No payload.

### Контент: Landing, Form, SDK, Content Library

- **Landing (Lander)** + **Lander Type** (Preland/Offer/White/…) — что показывают визиту на Content-шаге. → [models/landing.md](landing.md)
- · **SDK / Macros** — что исполняется на лэнде (формы, трекинг, fbCapi, push…). → [reference/sdk.md](../reference/sdk.md)
- **Form** — захват лида на лэнде (SDK-форма / Non-SDK Handler); Form Builder, маппинг контрол↔поле визита. → [models/form.md](form.md)
- **Content Library** (Content Type → Content Item) · **Splits** — динамический контент (офферы, A/B-сплиты лэнда). → [models/content-library.md](content-library.md), [how-to/landings.md](../how-to/landings.md)

### Деньги и результат: Destination, Conversion, Distribution

- **Destination** — канал пуша лида (выход флоу/монетизация). → [models/destination.md](destination.md)
- **Advertiser** — компания/сеть, кому отдаём лиды (платит Revenue); контейнер Destination-ов. → [models/advertiser.md](advertiser.md)
- **Conversion** · **Conversion Type** — событие(я) по визиту из постбэка (Lead/Registration/Purchase…); привязка через `visit_uuid`. → [models/conversion-model.md](conversion-model.md)
- **Tracker** (`Tracker → Trackers`) — исходящая отбивка конверсии наружу (FB CAPI / TikTok / HTTP Get…); ребёнок Source, реагирует на Conversion Type. → [models/tracker.md](tracker.md)
- **Distribution** (8 типов) — rule-деревья: Payout/Revenue settings (деньги), Campaign/Flow content, Fill field, Direct traffic, Remarketing, `Message Templates` (шаблоны сообщений для ремаркетинг-рассылок). → [models/distributions-model.md](distributions-model.md)
- **Business Model** — формула расчёта Revenue/Payout (лист Finance-дерева). → [models/business-model.md](business-model.md) *(движок дерева — [models/distributions-model.md](distributions-model.md))*

### Фильтрация и антифрод: Filter, Traffic Filter, AIO Antifraud

- **Filter** — шаг флоу, на котором визит уходит по транзишену `Passed` или `Rejected`. → *уточните у поддержки*
- **Traffic Filter** — сущность-интеграция фильтрации/обогащения трафика; нода Filter выбирает её как `Filter Type`. → *уточните у поддержки*
- · **AIO Antifraud** — встроенные проверки лида перед пушем в Destination. → *уточните у поддержки*

### Инфраструктура: Domain, Server, Agent

- **Domain** (трекинговый ↔ публичный) · **Repoint Group** · **CDN**. → [models/domain.md](domain.md)
- **Server / Agent** — машина с AIO-агентом (FSM, работающий по командам AIO: принимает запрос визитёра и исполняет решения флоу, которые вычисляет AIO); Deploy, Repoint Group, мониторинг, агностичность (лэнды на CDN). → [models/server.md](server.md), [context/architecture.md](../context/architecture.md)

### Маркетинг и ретеншн: Notification Flow, Message Template

- **Notification Flow** (синоним — `Marketing Flow`) — отдельный флоу рассылки: ноды Check / Wait / Repeat / Push, вешается на Campaign Flow / Conversion Type. → [models/marketing-flow.md](marketing-flow.md)
- · **Message Template** (Push/Email/Telegram/SMS) · **Sender Provider** · **Remarketing Distribution** (контент рассылки). → [mechanics/notifications-flow.md](../mechanics/notifications-flow.md), [models/distributions-model.md](distributions-model.md)

### Аналитика и измерения: Metric, Grouper

- **Metric** (Conversions count = штуки, Revenue/Payout count = деньги, Computable, Data feed) — числовой показатель-колонка в отчётах; ось всей аналитики, адресуется по UUID. → [models/metric.md](metric.md)
- · **Grouper** — измерение для разбивки строк отчёта (поле визита с `Availability as grouper`). → [models/visit-field.md](visit-field.md), [how-to/analytics.md](../how-to/analytics.md)

---

## Как сущности AIO связаны между собой — ключевые зависимости

Это и есть «как связаны» — то, что в профильных файлах раскрыто с одной стороны, а здесь собрано как связи.

- **Tenant ⊃ всё.** Кампании, флоу, сорсы, домены, лэнды, дестинейшны, дистрибуции, юзеры — всё внутри одного тенанта; между тенантами не шарится ([models/permissions-model.md](permissions-model.md)).
- **Campaign → ровно один Flow.** Кампания наследует структуру флоу; часть настроек шагов переопределяется на уровне кампании (`Settings Availability`: Campaign Template / Flow Only / Campaign Only). → [models/campaign.md](campaign.md), [models/flow-model.md](flow-model.md)
- **Visit входит через `cuuid`(Campaign) + `suuid`(Source).** Нет их → Direct Traffic (`Default Query` / `Direct Traffic Distribution`). → [models/visit.md](visit.md)
- **Conversion → Visit через `visit_uuid`.** На один визит — много конверсий (single: Lead/Registration; multiple: `Sale Status Update`). Конверсия ≠ визит. → [models/conversion-model.md](conversion-model.md)
- **Distribution → шаги флоу и деньги.** `Fields by distribution`/`Fill fields` пишут поля визита; Revenue/Payout-деревья назначают деньги конверсии; Content-дистрибуция отдаёт лэнд/оффер. → [models/distributions-model.md](distributions-model.md)
- **Landing живёт на Content-шаге; Form живёт на Landing; SDK исполняется на Landing.** Lander Type управляет, грузятся ли макросы/события (White = нет). → [models/landing.md](landing.md)
- **Domain-Change-шаг переключает трекинговый ↔ публичный домен** внутри пути визита. → [how-to/domains.md](../how-to/domains.md)

## Как связаны Flow States и текущий стейт Visit — самая частая путаница

**Flow — это статичный чертёж (граф States+Transitions). Visit — живой токен, который по этому чертежу едет.**

- В каждый момент визит **«стоит» на одном Flow State** — это его *текущий стейт* (позиция в графе).
- **Transition** перекидывает визит на следующий State. Какой переход сработает — зависит от того, что произошло:
  - кликнул `{{link}}` → **Handle-link**; засабмитил форму → **Handle-form**; прошёл фильтр → **Passed**; зарубил → **Rejected**; дестинейшн принял/отверг → **Destination-pushed/rejected**; ни один вариант шага не подошёл по правилам → **No payload**.
  - плюс **State Rules** и цвет **Split Group** уточняют, в какой *вариант* шага визит попадёт.

### Текущий Flow State и Visit Status — это две разные оси

- **Visit Status (Wait / Live / Left) — это другое, ортогональное измерение.** Оно про «живость» сессии, не про позицию в графе: визит может быть `Live` стоя на Offer, или `Left`, бросив на Preland. (`Wait` = визит ждёт первые ивенты от SDK / не догрузился JS и т.п.) → *уточните у поддержки*
- **Практический смысл (главный дебаг-объектив):** «куда уехал визит / где застрял» = найти, **на каком State** он сейчас и **каким Transition** (или `No payload`) туда попал. Это первое, что смотрят при разборе «лиды не доходят» / «лэнд не тот».

Модель Visit целиком — [models/visit.md](visit.md); ноды и переходы — [models/flow-model.md](flow-model.md); путь визита по шагам — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).
