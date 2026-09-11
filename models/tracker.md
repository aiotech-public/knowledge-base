---
id: tracker
title: Tracker (Tracker → Trackers — исходящая отбивка событий/конверсий) — концепт (модель)
description: Что такое Tracker в AIO — конфиг отправки события/конверсии ИЗ AIO наружу (FB CAPI / TikTok / HTTP Get…); ребёнок Source, один трекер = один Conversion Type, 11 Tracker type, Tracker logs vs Retrigger trackers. Концепт; процедуры — how-to/source-trackers.md.
doc_type: model
builds: [erp, mtk]
related: [source-trackers, source, conversion-model, ui-map, destination, ui-common, placeholders, distributions-model, sdk, glossary]
language: ru
updated: 2026-09-11
---

# Tracker (Tracker → Trackers — исходящая отбивка событий/конверсий) — концепт (модель)

> Модель сущности **Tracker (Tracker → Trackers)** — исходящая отбивка событий/конверсий наружу. Процедуры (создать трекер, FB CAPI Custom Key/Value, чтение Tracker logs) — [how-to/source-trackers.md](../how-to/source-trackers.md). Родитель Source — [models/source.md](source.md). Conversion Types, входящий постбэк-URL, Dynamic Actions, дубликаты FB — [models/conversion-model.md](conversion-model.md). UI — [reference/ui-map.md](../reference/ui-map.md) → Tracker → Trackers.

## Что такое Tracker и зачем он нужен

**Tracker — это конфиг отправки события/конверсии ИЗ AIO НАРУЖУ в стороннюю систему (FB CAPI, Taboola, другой Source, партнёрку).** Трекеры «видят», что в системе AIO появилась конверсия, и передают её по прописанному в трекере пути — для оптимизации рекламной платформы или для отчёта партнёрке. Чаще всего это постбэк, но уже постбэк **Source** (его вам выдаёт сторона Source, вы ставите его у себя) — в отличие от постбэков Destination, которые ваши и которые ставит у себя получатель. Поэтому ключевая ось понимания трекера — **направление потока: трекер всегда исходящий**, это «отбивка от вас — им».

## Как это работает на самом деле

### Где трекер стоит в цепочке клик→конверсия — это всегда ВЫХОД

Полная петля от клика до отбивки:
1. визит приходит по ссылке из Source → пролетает через AIO → уходит в Destination;
2. на стороне Destination/получателя происходит конверсия → получатель шлёт **свой** постбэк (это **вход**, см. [models/destination.md](destination.md), [models/conversion-model.md](conversion-model.md));
3. у нас появляется **Conversion**, привязанная к визиту через `visit_uuid`;
4. **Trackers отбивают эту инфу ОБРАТНО в Source** — в FB/Snapchat через API (не постбэк), в Taboola иначе; суть одна — «от нас отдаём им».

То есть трекер реагирует не на трафик, а на **уже созданную конверсию** заданного типа и шлёт сигнал наружу. Конверсия фиксируется только при известном `visit_uuid` — без визита трекеру нечего отбивать.

### Tracker — ребёнок Source: обязательное поле Sources

Трекеры живут **под Source** как дети: к одному Source прикрепляется много Trackers. Source двунаправлен (ловит на входе через `suuid` + rewrites, рапортует на выходе через трекеры) — а сам Tracker только исходящий. В редакторе трекера поле **Sources** (мульти-селект) — **обязательное**: оно определяет, для каких Source трекер срабатывает. Не привязал к нужному Source → конверсия не сматчится, наружу ничего не уйдёт. Иерархия и двунаправленность Source — [models/source.md](source.md).

### Какие поля в редакторе трекера (Edit tracker)

Редактор (`Edit tracker`) содержит:
- **Name** — имя трекера;
- **Sources** — мульти-селект, для каких Source срабатывает (обязательно);
- **Conversion type** — тип конверсии, на который реагирует (Lead / Purchase / Push Subscribe / кастомные из `Settings → Conversion types`);
- **Tracker type** — механизм отправки (карточки в `Create tracker`);
- **Delay in seconds** — задержка перед отправкой (`No delay` / N секунд);
- **поля payload** — зависят от выбранного Tracker type;
- **Confirm**.

Conversion Types и их семантика — [models/conversion-model.md](conversion-model.md).

### Какие бывают Tracker type — механизмы отправки

При создании (`+ Tracker` → модалка `Create tracker`) выбирается механизм отправки (карточки `Integration Type`): **Facebook Conversion API, TikTok Events API, SnapChat Conversion API, Quora Conversion API, ChatGPT Conversion API, Bing Conversion API, AppsFlyer, Affise, AIO Push, HTTP Get** (generic постбэк-URL). После выбора карточки открывается форма трекера (заголовок тот же — `Create tracker`): **набор обязательных полей зависит от выбранного типа**. От типа зависят и поля payload — они разобраны для **Facebook Conversion API** (см. ниже), а для `ChatGPT Conversion API`, `Bing Conversion API`, `TikTok Events API` и `HTTP Get` — в [how-to/source-trackers.md](../how-to/source-trackers.md).

### `ChatGPT Conversion API` и `Bing Conversion API` — серверные CAPI, а не постбэк-URL

Типы `ChatGPT Conversion API` и `Bing Conversion API` шлют платформе **серверное событие собственным запросом AIO** — постбэк-ссылку никуда вставлять не нужно. В форме трекера настраиваются идентификатор пикселя площадки, ключ доступа и сам состав события: у `ChatGPT Conversion API` это `Pixel ID` + `API Key` + `Event Type` (закрытый список событий), у `Bing Conversion API` — `UET Tag ID` + `CAPI Token` + `Event Name` (произвольная строка). Полный состав полей payload обоих типов — [how-to/source-trackers.md](../how-to/source-trackers.md).

### 4a. Какие трекеры заведены по дефолту в шаблоне тенанта

По дефолту в шаблоне тенанта заведены ТОЛЬКО CAPI-постбэки (CAPI работает из коробки): преднастроенные CAPI-трекеры привязаны к Source **`FB w/ CAPI`**, и по дефолту их триггерят конверсии типов **Lead** и **Purchase**. Под дополнительные Conversion Types или другие FB-Source существующие трекеры редактируют (добавить Source в поле `Sources`) или создают новые.

### Что в таблице Tracker → Trackers

Раздел `Tracker → Trackers` показывает все трекеры тенанта в таблице. Колонки: Tracker type, Conversion type, привязанные Sources, Description, delay details, статус активности. Состав и порядок колонок настраиваются кнопкой `Settings` над таблицей → `Table settings` (включить/выключить колонку, переставить — [reference/ui-common.md](../reference/ui-common.md)). Чтобы проверить, доходят ли отбивки наружу, смотри `Tracker logs` (см. секцию про Tracker logs ниже): там виден полный outgoing-запрос и ответ платформы.

### 4c. Список трекеров в MTK — где найти (Settings → Trackers)

В MTK трекеры находятся не в отдельной группе верхнего меню, а в `Settings → Trackers` — см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

### Что в payload FB CAPI-трекера и как задать Custom Key/Value

Для трекера типа **Facebook Conversion API** поля payload: `Placement`, `Exclude Placement`, `City`, `Zip Code`, `Revenue` (заполняются плейсхолдерами полей визита/конверсии — точный синтаксис см. [how-to/source-trackers.md](../how-to/source-trackers.md) и [reference/placeholders.md](../reference/placeholders.md)), и пары `Custom Key N` / `Custom Value N` под кастомные события. Для постбэк-URL типов — поле URL эндпоинта.

Кастомное FB-событие задаётся парой:
- **Custom Key** = имя события (`Lead` / `Purchase` / кастомное custom event, созданное руками в пикселе);
- **Custom Value** = `Static` (фиксированное) или `Dynamic` (плейсхолдер, напр. `{{aio.visit.fields.offer_name}}` — событие зависит от значения поля визита) → `Save`.

Пошагово — [how-to/source-trackers.md](../how-to/source-trackers.md). В FB CAPI имеет смысл отдавать максимум данных: чем полнее payload (FBC/FBP/FB Click ID плюс заполненные плейсхолдерами `City`/`Zip Code`), тем лучше match-quality на стороне FB.

### Откуда берутся city / postal code в FB CAPI-payload

Гео-поля `City` и `Zip Code` — это поля payload трекера, которые заполняются **плейсхолдерами** полей визита/конверсии (ручной маппинг в редакторе трекера, как и `Placement`/`Revenue`), а не подставляются автоматически из IP визита. Чтобы `City`/`Zip Code` уходили в CAPI-payload, в эти поля надо прописать соответствующий плейсхолдер (синтаксис — [reference/placeholders.md](../reference/placeholders.md), пошагово — [how-to/source-trackers.md](../how-to/source-trackers.md)). Смысл — отдать в Conversion API больше сигналов: чем полнее payload, тем выше match-quality и атрибуция на стороне FB.

### Почему один трекер = один Conversion Type

Трекер работает **только на один** Conversion Type. Подписочная модель (Install / Trial / Subscription) → нужны **три** FB CAPI трекера. Кнопки «копировать трекер» в UI **нет** — дублируют руками. Подробнее про мультисобытийные модели — [models/conversion-model.md](conversion-model.md).

### Где смотреть Tracker logs для аудита и дебага (ERP)

Логи отправки: `Tracker → Trackers → ПКМ по трекеру → Tracker logs`. Видно полный outgoing-запрос (например, в FB Conversions API) + ответ платформы. Это одновременно журнал аудита и инструмент дебага. Пустые `Tracker logs` = конверсия не сматчилась с трекером (нет визита по `visit_uuid` либо Source-фильтр не совпал) — не всегда баг самого трекера. Включение/выключение трекера — экшен `Switch activity` (ПКМ).

### Где смотреть Tracker logs в MTK (Settings → Trackers)

Логи отправки в MTK: `Settings → Trackers → ПКМ по трекеру → Tracker logs`. Семантика та же: полный outgoing-запрос + ответ платформы, пустые логи = конверсия не сматчилась с трекером. Подробнее о навигации Settings в MTK — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

### Как переотбить уже пришедшие конверсии — Retrigger trackers

`Retrigger trackers` — это экшен на **конверсии**, не на трекере: `Tracker → Conversions` (или Analytics → Conversions) → ПКМ по конверсии → `Retrigger trackers` (открывает pop-up). Он перезапускает все трекеры, под условие которых попадает конверсия, и повторно шлёт событие наружу (FB CAPI / Google / партнёрка) с **актуальными** значениями.

Когда нужно: пиксель/токен на момент создания конверсии были невалидны, первый отстрел не дошёл, или настройка отбивки была сломана — надо переотбить уже пришедшие конверсии. У типов `ChatGPT Conversion API` и `Bing Conversion API` есть нюанс с датой: площадки не принимают события старше 7 суток, поэтому AIO подрезает метку времени события под это окно — переотбитая старая конверсия дойдёт, но с подрезанной датой, а не с исходной. Важно: `Retrigger trackers` перепушивает постбэки в Source, но **НЕ меняет** payout/revenue в AIO — это отдельное от Distribution действие ([models/distributions-model.md](distributions-model.md)).

### Где найти Retrigger trackers в MTK

В MTK групп `Tracker` и `Analytics` в верхнем меню нет: вкладка `Conversions` — плоский таб верхнего меню. Путь к экшену: `Conversions` (топ-нав) → ПКМ по конверсии → `Retrigger trackers`. Сам экшен и его семантика те же. Навигация MTK — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

## Как Tracker связан с остальными сущностями

### Родитель и триггер: Source и Conversion

- **PARENT — Source.** Tracker живёт под Source как ребёнок; обязательное поле `Sources` определяет, для каких Source он срабатывает. У одного Source — много Trackers. → [models/source.md](source.md)
- **TRIGGER — Conversion.** Трекер реагирует на появление конверсии заданного `Conversion type` (один трекер = один тип). `Retrigger trackers` — экшен на конверсии. → [models/conversion-model.md](conversion-model.md)

### Соседи: Destination, поля визита, FB-оптимизация

- **COUNTERPART — Destination.** Destination = **вход** конверсий (ваши постбэки, их ставит у себя получатель; визит уходит туда). Tracker = **выход** (постбэки/API стороны Source, их вам выдают, вы ставите у себя). Зеркальные направления постбэков. → [models/destination.md](destination.md)
- **READS поля визита/конверсии.** Payload трекера берёт значения через плейсхолдеры полей визита/конверсии (`{{aio.visit.fields.*}}`; точный синтаксис — [reference/placeholders.md](../reference/placeholders.md)); FB-поля заполняет Source rewrites на **входе**. → [models/source.md](source.md), [reference/placeholders.md](../reference/placeholders.md)
- **FEEDS — FB Meta optimization.** FB CAPI трекер шлёт event-сигнал (Lead/Purchase/…) обратно в FB pixel для оптимизации; какое именно FB-событие — задают Dynamic Actions / Lead Action / Purchase Action ([models/conversion-model.md](conversion-model.md)).
- **NOT-related-to-Flow.** Конверсии (а значит и отбивка трекеров) с Flow не связаны — конверсия происходит на стороне Destination после финала флоу.

### Соседи Tracker в топ-меню (ERP)

Sibling в группе `Tracker` верхнего меню — Campaigns, Sources, Destinations, Flows, Visits, Conversions, Trash ([reference/ui-map.md](../reference/ui-map.md)).

### Соседи Tracker в топ-меню (MTK)

В MTK группы `Tracker` нет — её разделы плоские табы верхнего меню: Campaigns, Sources, Destinations, Visits, Conversions (Flows и Trash скрыты; Trackers/Trash — в `Settings`). См. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

## Чем Tracker отличается от смежных понятий

### Conversion type vs Tracker type — два поля «type» в редакторе трекера

В редакторе трекера два поля «type». **Conversion type** = *ЧТО* за конверсия (Lead / Purchase / Push Subscribe). **Tracker type** = *МЕХАНИЗМ* отправки (FB CAPI / TikTok / AppsFlyer / HTTP Get…).

## Подводные камни Tracker — частые ошибки настройки

### «FB не видит лиды» — смотри Tracker logs

`Trackers → Tracker logs` → проверить outgoing-запрос. Частые причины: трекер не привязан к нужному Source (поле `Sources`); либо выбран не тот `Conversion type`.

### Массовый Retrigger trackers — через чекбоксы или API

Отметить конверсии чекбоксами в `Tracker → Conversions` → ПКМ → `Retrigger trackers` → `Confirm` (или по одной через ПКМ); для больших объёмов — API.

## Куда идти за деталями — смежные темы Tracker

- **Процедуры** (создать трекер, FB CAPI Custom Key/Value, Tracker logs) → [how-to/source-trackers.md](../how-to/source-trackers.md).
- **Родитель Source** (suuid, rewrites, двунаправленность) → [models/source.md](source.md).
- **Conversion Types, входящий постбэк-URL, Dynamic Actions, дубликаты FB** → [models/conversion-model.md](conversion-model.md).
- **Диагностика FB CAPI** (токен / атрибуция / двойной пиксель) → *уточните у поддержки*.
- **SDK-фичи `fbCapi` / `fbPixel` на лэнде** → [reference/sdk.md](../reference/sdk.md).
- **Retrigger trackers ≠ Distribution** (не меняет payout/revenue) → [models/distributions-model.md](distributions-model.md).
- **Глоссарий** (Tracker → Trackers вкладка, Retrigger trackers, Custom Key/Value) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) → Tracker → Trackers (`/app/tracker/trackers`; создание `+ Tracker` → `Create tracker`; ПКМ: Edit tracker / Switch activity / Tracker logs; Retrigger trackers — ПКМ на конверсии в Tracker → Conversions).
