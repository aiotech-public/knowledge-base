---
id: advertiser
title: Advertiser — концепт (модель)
description: Что такое Advertiser в AIO — компания/сеть, которой вы отдаёте лиды и которая платит вам Revenue; контейнер для Destination-ов. Чем отличается от Destination/Source/Conversion. Концепт; процедуры — how-to/destinations.md.
doc_type: model
builds: [erp, mtk]
related: [destination, destinations, distributions-model, landing, placeholders, user-fields, conversion-model, business-model, source, ui-map]
language: ru
updated: 2026-08-11
---

# Advertiser — концепт (модель)

> Модель сущности **Advertiser**. Конкретный канал пуша — [models/destination.md](destination.md). Процедуры (завести Advertiser, создать Destination By Advertiser) — [how-to/destinations.md](../how-to/destinations.md). UI — `Settings → Advertisers`.

## Суть в одном абзаце

**Advertiser — это компания/сеть, которой вы отдаёте лиды и которая платит вам Revenue.** Это организационная сущность «кому льём» (рекламодатель / партнёрка / CRM как контрагент), под которой живут конкретные **Destination**-ы (каналы пуша). Advertiser задаёт *отношение* (с кем работаете, под каким оффер-неймом), а как именно туда уходит лид и сколько за него платят — в его Destination-ах и Revenue-дистрибуциях.

## Как это работает на самом деле

### 1. Advertiser → Destination: компания и её каналы

- **Advertiser** — *кому* (компания). **Destination** — *конкретный канал пуша* под ним (URL/интеграция + маппинг полей визита в параметры получателя). У одного Advertiser — **много** Destination.
- Рекомендуемый способ создать дестинейшн — **By Advertiser**: выбираете сервис-адвертайзера, AIO подтягивает преднастройки (эндпоинты, формат). Поэтому Advertiser — это «папка» с готовой интеграцией, а не просто метка.
- Advertiser-ы заводятся в `Settings → Advertisers`; сами Destination — в `Tracker → Destinations`. У Advertiser есть **Advertiser Type** (классификация).

### 2. Сторона денег: Advertiser платит вам Revenue

В цепочке трафика Advertiser — это **доходная** сторона: вы поставляете ему лиды, он платит **Revenue** за конверсию. (Выплата вашему трафику — **Payout** — это другая сторона, см. [models/distributions-model.md](distributions-model.md) → Payout.) Сколько начислять за конверсию с этого направления — задаёт **Revenue-дистрибуция** + **Business Model**. То есть Advertiser сам денег не считает — он «адресат», а суммы живут в Finance-дереве.

### 3. Offer Name «для рекламодателя»

Один и тот же оффер имеет два имени: **`offer_name.for_visitor`** (что видит визит на лэнде) и **`offer_name.for_advertiser`** (что уходит адвертайзеру в пуше). Это позволяет показывать визиту привлекательное название, а получателю — то, что он ожидает. См. [models/landing.md](landing.md), [reference/placeholders.md](../reference/placeholders.md).

### 4. Поля визита прямо в карточке Advertiser — секция `User fields`

В форме адвертайзера (`Settings → Advertisers`) есть секция `User fields`: кнопкой `Add field` в неё добавляются поля визита с `Type = Destination` — тот же пул, что и у самого `Destination`. Вписанное значение доезжает до визита, который ушёл на дестинейшн этого адвертайзера.

Поля адвертайзера «родительские»: у каждого есть режим — `Advertiser only`, `Destination template` или `Destination only`, — который решает, кто вписывает значение, карточка адвертайзера или сам `Destination`. Что делает каждый режим, когда подставляется шаблон и почему нужного поля нет в списке — [how-to/user-fields.md](../how-to/user-fields.md).

## Как Advertiser связан с остальными сущностями

- **Destination → Advertiser** — каждый дестинейшн принадлежит адвертайзеру.
- **Conversion → Revenue** приходит с направления адвертайзера (через его Destination + Revenue-дистрибуцию).
- **Offer Name (`for_advertiser`)** — что уходит этому адвертайзеру.
- Концептуально Advertiser — **зеркало Source**: Source = вход (откуда трафик), Advertiser = адресат лида (куда и кому).

## Чем Advertiser отличается от смежных понятий

| Не путать | Разница |
|---|---|
| **Advertiser vs Destination** | Advertiser — **компания** («кому»). Destination — **канал пуша** под ней («куда конкретно» + маппинг). Один Advertiser → много Destination. |
| **Advertiser vs Source** | Source — вход трафика (откуда). Advertiser — адресат лида (кому отдаём). Зеркальные концы воронки по «контрагентам». |
| **Advertiser vs Conversion** | Advertiser — направление. Conversion — событие, пришедшее **обратно** с этого направления (постбэком). |

## Подводные камни Advertiser — частые ошибки настройки

- **Штатно оффер создаётся через `By Advertiser`** (редирект/ссылка к рекламодателю — подтягиваются преднастройки интеграции + аналитика по адвертайзеру) или `By Integration` (готовая API-интеграция, без адвертайзера). Голый `Simple Redirect` — не для реальных офферов (нет Advertiser → теряется аналитика по нему); см. [models/conversion-model.md](conversion-model.md).
- **Два оффер-нейма** (`for_visitor` ≠ `for_advertiser`) — частая путаница; визиту одно, получателю другое.
- **Advertiser ≠ деньги** — суммы Revenue задаются не на адвертайзере, а в Revenue-дистрибуции ([models/distributions-model.md](distributions-model.md)).

## Куда идти за деталями — смежные темы

- **Канал пуша** → [models/destination.md](destination.md).
- **Процедуры** (Advertiser, Destination By Advertiser, капы) → [how-to/destinations.md](../how-to/destinations.md).
- **Поля визита в карточке Advertiser** (секция `User fields`, режимы раздачи на `Destination`) → [how-to/user-fields.md](../how-to/user-fields.md).
- **Деньги за конверсии** → [models/distributions-model.md](distributions-model.md) (Revenue/Payout), [models/business-model.md](business-model.md) (формула Revenue/Payout).
- **Зеркальный вход воронки** → [models/source.md](source.md) (Source — вход трафика).
- **Конверсии с направления** → [models/conversion-model.md](conversion-model.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (`Settings → Advertisers`; `Tracker → Destinations`).
