---
id: advertiser
title: Advertiser — концепт (модель)
description: Что такое Advertiser в AIO — компания/сеть, которой вы отдаёте лиды и которая платит вам Revenue; контейнер для Destination-ов. Чем отличается от Destination/Source/Conversion. Концепт; процедуры — how-to/destinations.md.
doc_type: model
builds: [erp, mtk]
related: [destination, destinations, distributions-model, business-model, landing, placeholders, user-fields, source, conversion-model, ui-map]
language: ru
updated: 2026-10-09
---

# Advertiser — концепт (модель)

> Модель сущности **Advertiser**. Конкретный канал пуша — [models/destination.md](destination.md). Процедуры (завести Advertiser, создать Destination By Advertiser) — [how-to/destinations.md](../how-to/destinations.md). UI — `Settings → Advertisers`.

## Суть в одном абзаце

**Advertiser — это компания/сеть, которой вы отдаёте лиды и которая платит вам Revenue.** Это организационная сущность «кому льём» (рекламодатель / партнёрка / CRM как контрагент), под которой живут конкретные **Destination**-ы (каналы пуша). Advertiser задаёт *отношение* (с кем работаете, под каким оффер-неймом), а как именно туда уходит лид — задаёт Destination. Учёт Revenue связан с конверсиями, а не с расчётом в карточке Advertiser.

## Как это работает на самом деле

### 1. Advertiser → Destination: компания и её каналы

- **Advertiser** — *кому* (компания). **Destination** — *конкретный канал пуша* (URL/интеграция + маппинг полей визита в параметры получателя). В пути `By Advertiser` он связан с Advertiser. У одного Advertiser — **много** Destination.
- Рекомендуемый способ создать дестинейшн — **By Advertiser**: выбираете сервис-адвертайзера, AIO берёт тип интеграции и общие настройки из Advertiser. Поэтому Advertiser — это «папка» с готовой интеграцией, а не просто метка.
- Advertiser-ы заводятся в `Settings → Advertisers`. В ERP Destination находятся в `Tracker → Destinations`, в MTK — в отдельной верхней вкладке `Destinations`. У Advertiser есть **Advertiser Type** (классификация): из коробки в каждом шаблоне тенанта два типа — `Offer` и `PWA`.

В ERP свои типы добавляются через `Manage types` в меню рядом с кнопкой создания Advertiser; нужны права на просмотр и изменение типов. В MTK в этом меню управление типами не представлено. Подробнее — [how-to/destinations.md](../how-to/destinations.md).

### 2. Сторона денег: Advertiser платит вам Revenue

В цепочке трафика Advertiser — это **доходная** сторона: вы поставляете ему лиды, он платит **Revenue** за конверсию. (Выплата вашему трафику — **Payout** — это другая сторона, см. [models/distributions-model.md](distributions-model.md) → Payout.) Advertiser сам Revenue не рассчитывает: сумма хранится у **Conversion**. В ERP условия расчёта можно задать через **Revenue-дистрибуцию** и **Business Model**. Ненулевое Revenue, переданное при создании конверсии, принимается напрямую; без Revenue-дистрибуции расчёт возможен по формуле Business Model типа `Revenue`. Подробнее — [models/distributions-model.md](distributions-model.md) и [models/business-model.md](business-model.md).

### 3. Offer Name «для рекламодателя»

При работе с **Offer Name** можно хранить два названия одного оффера: `for_visitor` для посетителя и `for_advertiser` для получателя лида. Для вывода выбранного названия на лэнде используют `{{aio.visit.fields.offer_name.for_visitor}}`, для передачи в Destination — `{{aio.visit.fields.offer_name.for_advertiser}}` в его настройках. Передача зависит от настройки Destination. Это позволяет показывать визиту привлекательное название, а получателю — то, что он ожидает. См. [models/landing.md](landing.md), [reference/placeholders.md](../reference/placeholders.md).

В шаблоне ERP Finance эта пара заведена из коробки: поле визита `offer_name` и группа Content Library `Offer Names` с ключами `for_visitor` (`For Visitor` — «what we show to visit») и `for_advertiser` (`For Advertiser` — «what we send to advertiser»); флоу `1/2Step | Offer Name Distribution` заполняет поле дистрибуцией `Offer Names Distribution` ([models/distributions-model.md](distributions-model.md)), флоу `1/2Step | Manual Offer Name` (и его варианты `+ TY` / `+ TY + Domain Change`) — шагом `Fill Offer Name`, значение которого выбирается в настройках кампании.

### 4. Поля визита прямо в карточке Advertiser — секция `User fields`

В форме адвертайзера (`Settings → Advertisers`) есть секция `User fields`: кнопкой `Add field` в неё добавляются поля визита с `Type = Destination` — тот же пул, что и у самого `Destination`. Перед новой попыткой пуша Destination поля добавляются к визиту с учётом режима: `Advertiser only` берётся из Advertiser, а для остальных режимов используются значения самого Destination. `Destination template` подставляет родительский шаблон при создании Destination; его можно изменить. Для Trash-визита эта передача полей не выполняется.

Поля адвертайзера «родительские»: у каждого есть режим — `Advertiser only`, `Destination template` или `Destination only`, — который решает, кто вписывает значение, карточка адвертайзера или сам `Destination`. Что делает каждый режим, когда подставляется шаблон и почему нужного поля нет в списке — [how-to/user-fields.md](../how-to/user-fields.md).

## Как Advertiser связан с остальными сущностями

- **Destination → Advertiser** — связь задаётся в пути `By Advertiser`. В `By Integration` и пресете `Simple Redirect` Destination создаётся без Advertiser.
- **Conversion → Revenue** — конверсия может быть связана с Advertiser через Destination. Сумма Revenue относится к конверсии; Revenue-дистрибуция — один из способов её расчёта.
- **Offer Name (`for_advertiser`)** — название для получателя лида; передаётся, если соответствующий placeholder используется в настройках Destination.
- Концептуально Advertiser — **зеркало Source**: Source = вход (откуда трафик), Advertiser = адресат лида (куда и кому).

## Чем Advertiser отличается от смежных понятий

| Не путать | Разница |
|---|---|
| **Advertiser vs Destination** | Advertiser — **компания** («кому»). Destination — **канал пуша** («куда конкретно» + маппинг), который может быть связан с Advertiser. Один Advertiser → много Destination. |
| **Advertiser vs Source** | Source — вход трафика (откуда). Advertiser — адресат лида (кому отдаём). Зеркальные концы воронки по «контрагентам». |
| **Advertiser vs Conversion** | Advertiser — направление. Conversion — событие визита: например, из постбэка получателя лида или шага `Spawn Conversion` во Flow. |

## Подводные камни Advertiser — частые ошибки настройки

- **Для офферов рекомендуется `By Advertiser`**: Destination получает тип интеграции и общие настройки Advertiser и участвует в аналитике по нему. `By Integration` создаёт Destination без Advertiser; пресет `Simple Redirect` — тот же путь с заранее выбранной redirect-интеграцией. `Simple Redirect` подходит и для офферной ссылки, но не даёт разбивку по Advertiser. См. [how-to/destinations.md](../how-to/destinations.md).
- **Два оффер-нейма** (`for_visitor` ≠ `for_advertiser`) — частая путаница; визиту одно, получателю другое.
- **Advertiser ≠ деньги** — сумма Revenue хранится у конверсии, а не у Advertiser; она может прийти напрямую или рассчитываться по Business Model, в том числе с Revenue-дистрибуцией ([models/distributions-model.md](distributions-model.md)).

## Куда идти за деталями — смежные темы

- **Канал пуша** → [models/destination.md](destination.md).
- **Процедуры** (Advertiser, Destination By Advertiser, капы) → [how-to/destinations.md](../how-to/destinations.md).
- **Поля визита в карточке Advertiser** (секция `User fields`, режимы раздачи на `Destination`) → [how-to/user-fields.md](../how-to/user-fields.md).
- **Деньги за конверсии** → [models/distributions-model.md](distributions-model.md) (Revenue/Payout), [models/business-model.md](business-model.md) (формула Revenue/Payout).
- **Зеркальный вход воронки** → [models/source.md](source.md) (Source — вход трафика).
- **Конверсии с направления** → [models/conversion-model.md](conversion-model.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (`Settings → Advertisers`; ERP — `Tracker → Destinations`, MTK — верхняя вкладка `Destinations`).
