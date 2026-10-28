---
id: tracker
title: Tracker — исходящая отбивка событий/конверсий — концепт (модель)
description: Что такое Tracker в AIO — конфиг отправки события/конверсии ИЗ AIO наружу (FB CAPI / TikTok / Reddit CAPI / HTTP Get…); связан с Source, один трекер = один Conversion Type, типы Tracker type, Tracker logs vs Re trigger trackers. Концепт; процедуры — how-to/source-trackers.md.
doc_type: model
builds: [erp, mtk]
related: [source-trackers, source, conversion-model, ui-map, destination, campaigns, architecture, ui-common, placeholders, distributions-model, sdk, glossary]
language: ru
updated: 2026-10-28
---

# Tracker — исходящая отбивка событий/конверсий — концепт (модель)

> Модель сущности **Tracker** — исходящая отбивка событий/конверсий наружу. ERP: `Tracker → Trackers`; MTK: `Settings → Trackers`. Процедуры (создать трекер, FB CAPI Custom Key/Value, чтение Tracker logs) — [how-to/source-trackers.md](../how-to/source-trackers.md). Связь с Source — [models/source.md](source.md). Conversion Types, входящий постбэк-URL, Dynamic Actions, дубликаты FB — [models/conversion-model.md](conversion-model.md). UI — [reference/ui-map.md](../reference/ui-map.md): ERP `Tracker → Trackers`, MTK `Settings → Trackers`.

## Что такое Tracker и зачем он нужен

**Tracker — это конфиг отправки события/конверсии ИЗ AIO НАРУЖУ в стороннюю систему (FB CAPI, Taboola, другой Source, партнёрку).** Трекеры «видят», что в системе AIO появилась конверсия, и передают её по прописанному в трекере пути — для оптимизации рекламной платформы или для отчёта партнёрке. Чаще всего это постбэк, но уже постбэк **Source** (его вам выдаёт сторона Source, вы ставите его у себя) — в отличие от постбэков Destination, которые ваши и которые ставит у себя получатель. Поэтому ключевая ось понимания трекера — **направление потока: трекер всегда исходящий**, это «отбивка от вас — им».

## Как это работает на самом деле

### Где трекер стоит в цепочке клик→конверсия — это всегда ВЫХОД

Типовой сценарий с внешним получателем конверсии:
1. визит приходит по ссылке из Source → пролетает через AIO → уходит в Destination;
2. на стороне Destination/получателя происходит конверсия → получатель вызывает **постбэк-URL AIO** (это **вход**, см. [models/destination.md](destination.md), [models/conversion-model.md](conversion-model.md));
3. у нас появляется **Conversion**, привязанная к визиту через `visit_uuid`;
4. **Trackers отбивают эту инфу ОБРАТНО в Source** — в FB/Snapchat через API (не постбэк), в Taboola иначе; суть одна — «от нас отдаём им».

То есть трекер реагирует не на трафик, а на **уже созданную конверсию** заданного типа и шлёт сигнал наружу. Конверсия фиксируется только при известном `visit_uuid` — без визита трекеру нечего отбивать.

### Как Tracker связан с Source: обязательное поле Sources

Один Tracker связывается с одним или несколькими Source; к одному Source может быть прикреплено много Trackers. Source двунаправлен (ловит на входе через `suuid` + rewrites, рапортует на выходе через трекеры) — а сам Tracker только исходящий. В редакторе трекера поле **Sources** (мульти-селект) — **обязательное**: оно определяет, для каких Source трекер срабатывает. Если заданный Source-фильтр не включает источник визита, конверсия не сматчится с трекером. Иерархия и двунаправленность Source — [models/source.md](source.md).

### Какие поля в редакторе трекера (Edit tracker)

Редактор (`Edit tracker`) содержит:
- **Name** — имя трекера;
- **Sources** — мульти-селект, для каких Source срабатывает (обязательно);
- **Conversion type** — тип конверсии, на который реагирует (в ERP набор редактируется в `Settings → Conversion types`; засеянный набор зависит от вертикали шаблона — `Lead`, `Registration`, `Approved`, `Install`, `Push Subscribe` …);
- **Tracker type** — механизм отправки (карточки в `Create tracker`);
- **Delay in seconds** — задержка перед отправкой (`No delay` / N секунд);
- **поля payload** — зависят от выбранного Tracker type;
- **Confirm**.

Conversion Types и их семантика — [models/conversion-model.md](conversion-model.md).

### Какие бывают Tracker type — механизмы отправки

При создании (`+ Tracker` → модалка `Create tracker` с подсказкой `Select or search an integration by name.`) выбирается механизм отправки — сетка карточек типов с поиском по имени: **Facebook Conversion API, TikTok Events API, SnapChat Conversion API, Quora Conversion API, ChatGPT Conversion API, Bing Conversion API, Reddit Conversion API, X Conversion API, Seznam Conversion API, AppsFlyer, Affise, RoopyApps, AIO Push, HTTP Get** (generic постбэк-URL). Клик по карточке открывает форму трекера (заголовок тот же — `Create tracker`), где выбранный тип уже стоит в поле `Tracker type`: **набор обязательных полей зависит от выбранного типа**. От типа зависят и поля payload — они разобраны для **Facebook Conversion API** (секция «Что в payload FB CAPI-трекера» в этом доке), а для `ChatGPT Conversion API`, `Bing Conversion API`, `Reddit Conversion API`, `X Conversion API`, `Seznam Conversion API`, `RoopyApps`, `TikTok Events API` и `HTTP Get` — в [how-to/source-trackers.md](../how-to/source-trackers.md).

### `ChatGPT Conversion API`, `Bing Conversion API`, `Reddit Conversion API` и `X Conversion API` — серверные CAPI, а не постбэк-URL

Типы `ChatGPT Conversion API`, `Bing Conversion API`, `Reddit Conversion API` и `X Conversion API` шлют платформе **серверное событие собственным запросом AIO** — постбэк-ссылку никуда вставлять не нужно. В форме трекера настраиваются идентификатор пикселя площадки, ключ доступа и сам состав события: у `ChatGPT Conversion API` это `Pixel ID` + `API Key` + `Event Type` (закрытый список событий), у `Bing Conversion API` — `UET Tag ID` + `CAPI Token` + `Event Name` (произвольная строка), у `Reddit Conversion API` (Reddit CAPI, конверсии в Reddit Pixel) — `Pixel ID` + `Conversion Access Token` + `Reddit Event` (одно из девяти событий Reddit, можно плейсхолдером из поля визита), у `X Conversion API` (X / Twitter) — `Pixel ID` + `Event ID` + ключ и секрет приложения X (`Consumer Key`, `Consumer Secret`) + токен пользователя рекламного аккаунта (`Access Token`, `Access Token Secret`). Полный состав полей payload всех четырёх типов — [how-to/source-trackers.md](../how-to/source-trackers.md).

### 4a. Какие трекеры заведены из коробки — FB CAPI под основные конверсии вертикали

Из коробки в тенанте заведены трекеры типа **`Facebook Conversion API`**: все привязаны к источнику **`FB w/ CAPI`**, включены, по одному трекеру на тип конверсии (один трекер = один Conversion Type); покрыты только основные шаги воронки вертикали, а не все её типы конверсий (например, `Push Subscribe` трекера не имеет). Кроме FB CAPI, шаблоны включают **ChatGPT Conversion API** под те же основные типы конверсий. В iGaming также есть выключенный `HTTP Get`-пример. Набор FB CAPI зависит от вертикали шаблона (в ERP и MTK одной вертикали он одинаков):

- **Finance** — `FB Lead` (`Lead`) и трекер под целевое действие вертикали.
- **Nutra** — `FB Lead` (`Lead`) и `FB Approved` (`Approved`).
- **Sweeps** — `FB Registration` (`Registration`) и `FB Lead` (`Lead`).
- **iGaming** — `FB AIO Install` (`Install`), `FB AIO Registration` (`Registration`) и трекер под целевое действие; плюс выключенный пример постбэка `Example - 3rd Party PWA Postback` (`HTTP Get`, `Registration`, URL-заготовка с `{{aio.visit.fields.pwa_click_id}}` — подставить эндпоинт сервиса и включить).

Событие FB каждый трекер берёт плейсхолдером из ссылки: `{{aio.visit.fields.fb_lead_action}}` у трекера первого события (`Lead` / `Registration`), `{{aio.visit.fields.fb_ftd_action}}` у трекера целевого действия (`Approved` в Nutra, `Lead` в Sweeps), `{{aio.visit.fields.fb_install_action}}` у `FB AIO Install`. Что выбрано в этих селектах Link Generator по умолчанию — [how-to/campaigns.md](../how-to/campaigns.md) («Lead Action и событие покупки»). Под другие Conversion Types или другой FB-Source существующие трекеры редактируют (добавить Source в поле `Sources`) или создают новые. Что ещё клонируется из шаблона — [context/architecture.md](../context/architecture.md).

### 4b. Что стоит в payload засеянных FB CAPI-трекеров

Поля payload у засеянных трекеров уже заполнены плейсхолдерами: `Event` — селект события из ссылки (см. 4a), `Action Source` = `website`, `Token` = `{{aio.visit.fields.fb_capi_token}}`, `Pixel` = `{{aio.visit.fields.fb_pixel}}`, `FBP Field` / `FBC Field` / `FBClic Field` = `{{aio.visit.fields.fbp}}` / `{{aio.visit.fields.fbc}}` / `{{aio.visit.fields.fbclid}}`, `City` = `{{aio.visit.location_city}}`, `Zip Code` = `{{aio.visit.location_postal_code}}`, `Revenue` = `{{aio.conversion.revenue}}`. `Email Field` / `Phone Field` в Finance, Nutra и Sweeps = `{{aio.visit.fields.email}}` / `{{aio.visit.fields.phone}}` (в Finance заполнены ещё `First Name` / `Last Name`); у iGaming-трекеров они пустые, а у `FB AIO Install` пуст и `Revenue`. `Placement`, `Exclude Placement`, `Event Source Domain` и пары `Custom Key N` / `Custom Value N` из коробки не заданы.

### Что в таблице Tracker → Trackers

Раздел `Tracker → Trackers` показывает трекеры тенанта, по умолчанию без архивных. Тип трекера виден в подсказке у имени. Отдельные колонки: Conversion type, привязанные Sources, Description, задержка, статус активности. Состав и порядок колонок настраиваются кнопкой `Settings` над таблицей → `Table settings` (включить/выключить колонку, переставить — [reference/ui-common.md](../reference/ui-common.md)). Чтобы проверить, доходят ли отбивки наружу, смотри `Tracker logs` (см. секцию про Tracker logs ниже): там видна запись об отправке и ответ платформы.

### 4c. Список трекеров в MTK — где найти (Settings → Trackers)

В MTK трекеры находятся не в отдельной группе верхнего меню, а в `Settings → Trackers` — см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

### Что в payload FB CAPI-трекера: `Event`, `custom_data`, Custom Key/Value

Для трекера типа **Facebook Conversion API** имя события задаёт поле `Event` (статикой или плейсхолдером), остальные поля payload: `Placement`, `Exclude Placement`, `City`, `Zip Code`, `Revenue` (заполняются плейсхолдерами полей визита/конверсии — точный синтаксис см. [how-to/source-trackers.md](../how-to/source-trackers.md) и [reference/placeholders.md](../reference/placeholders.md)), необязательные стандартные поля `custom_data` (`Content category`, `Content Ids`, `Content Name`, `Content Type`, `Contents`, `Delivery category`, `Num Items`, `Predicted LTV`, `Search string`, `Status`) и пары `Custom Key N` / `Custom Value N` — свои свойства внутри `custom_data`. Некорректный JSON в `Contents`, нечисловые `Num Items` или `Predicted LTV` пропускаются с причиной в `warnings` записи `Tracker logs`. Это правило относится к этим проверкам, а не к любому неверному значению `custom_data`. Для постбэк-URL типов — поле URL эндпоинта.

Пошагово и форматы полей — [how-to/source-trackers.md](../how-to/source-trackers.md). В FB CAPI имеет смысл отдавать максимум данных: чем полнее payload (FBC/FBP/FB Click ID плюс заполненные плейсхолдерами `City`/`Zip Code`), тем лучше match-quality на стороне FB.

### Откуда берутся city / postal code в FB CAPI-payload

Гео-поля `City` и `Zip Code` — это поля payload трекера, которые заполняются **статикой или плейсхолдерами** полей визита/конверсии (ручной маппинг в редакторе трекера, как и `Placement`/`Revenue`), а не подставляются автоматически из IP визита. Чтобы `City`/`Zip Code` уходили в CAPI-payload, для гео текущего визита в эти поля надо прописать соответствующий плейсхолдер (синтаксис — [reference/placeholders.md](../reference/placeholders.md), пошагово — [how-to/source-trackers.md](../how-to/source-trackers.md)). У трекеров, засеянных из шаблона, это уже сделано: `City` = `{{aio.visit.location_city}}`, `Zip Code` = `{{aio.visit.location_postal_code}}` (секция 4b); если в сохранённых настройках `City`/`Zip Code` пустые, их нужно заполнить — в том числе после изменения засеянного трекера. Смысл — отдать в Conversion API больше сигналов: чем полнее payload, тем выше match-quality и атрибуция на стороне FB.

### Почему один трекер = один Conversion Type

Трекер работает **только на один** Conversion Type. Подписочная модель (Install / Trial / Subscription) → нужны **три** FB CAPI трекера. Кнопки «копировать трекер» в UI **нет** — дублируют руками. Подробнее про мультисобытийные модели — [models/conversion-model.md](conversion-model.md).

### Где смотреть Tracker logs для аудита и дебага (ERP)

Логи отправки: `Tracker → Trackers → ПКМ по трекеру → Tracker logs`. Состав записи по типам трекера — секция «Что пишет Tracker logs — состав записи по типу трекера» в этом доке. Это одновременно журнал аудита и инструмент дебага. Пустые `Tracker logs` сами по себе не доказывают несовпадение конверсии с трекером. Проверь Conversion type, Sources, активность трекера, `Delay in seconds` и выбранный период журнала. Пустой журнал не всегда означает баг самого трекера. Включение/выключение трекера — экшен `Switch activity` (ПКМ).

### Где смотреть Tracker logs в MTK (Settings → Trackers)

Логи отправки в MTK: `Settings → Trackers → ПКМ по трекеру → Tracker logs`. Семантика та же: запись об отправке (состав по типам — секция «Что пишет Tracker logs — состав записи по типу трекера» в этом доке), пустые логи сами по себе не доказывают несовпадение конверсии с трекером. Проверь Conversion type, Sources, активность, задержку и период журнала. Подробнее о навигации Settings в MTK — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

### Что пишет Tracker logs — состав записи по типу трекера

Состав записи `Tracker logs` зависит от типа трекера и одинаков в ERP и MTK. При успешном выполнении отправки FB Conversions API пишет параметры, событие и ответ платформы; HTTP Get — URL запроса и ответ; TikTok Events API — только ответ платформы. При ошибке состав отличается: например, TikTok может записать только текст ошибки.

`Reddit Conversion API` пишет отправленное тело события и ответ площадки (`<код> : <тело>`); при отказе площадки — `Error - <код> , Response - <ответ> , Payload - <тело>`. Если у Reddit-трекера пусты `Pixel ID`, `Conversion Access Token` или `Reddit Event` (или плейсхолдер в поле не подставился), в записи только текст вида `There is no Pixel ID at settings`, без тела. Токен доступа в запись не попадает; email и телефон в теле уже в виде хеша SHA-256.

`X Conversion API` пишет отправленное тело события и ответ X (`<код> : <тело>`). При отказе X — `Error - <текст ошибки> , Response - <ответ X> , Payload - <тело>`; если X ответил `200`, но событие не засчитал, — `X did not process the conversion , Response - <ответ> , Payload - <тело>`. Пустое обязательное поле даёт запись вида `There is no Pixel ID at settings`; нет ни `twclid`, ни email, ни телефона — `There is no twclid, email or phone for the conversion - X can not match it`. Ключи приложения X в запись не попадают, email и телефон в теле — хешами SHA-256.

`Seznam Conversion API` пишет отправленное тело события и ответ Sklik (`<код> : <тело>`); любой ответ, кроме `200`, — ошибка `Error - <код> , Response - <ответ> , Payload - <тело>`. `RoopyApps` пишет URL постбэка и ответ (`<код> : <тело>`); при ошибке — `Error - <ошибка> , Response - <ответ> , Url - <URL>`. Ошибки настроек обоих трекеров — [how-to/source-trackers.md](../how-to/source-trackers.md).

### Как переотбить уже пришедшие конверсии — Re trigger trackers (retrigger)

`Re trigger trackers` — это экшен на **конверсии**, не на трекере. ERP: `Tracker → Conversions`; MTK: верхняя вкладка `Conversions` → ПКМ по конверсии → `Re trigger trackers` (открывает pop-up). Он заново выбирает подходящие **активные** трекеры и запускает повторную отправку с их задержкой (`Delay in seconds`). При выполнении используются актуальные настройки трекера и значения плейсхолдеров визита/конверсии.

Когда нужно: пиксель/токен на момент создания конверсии были невалидны, первая отправка не дошла, или настройка отбивки была сломана. `Re trigger trackers` повторно отправляет события наружу, но **НЕ меняет** payout/revenue в AIO — это отдельное от Distribution действие ([models/distributions-model.md](distributions-model.md)).

### Какую дату AIO отправляет при Re trigger trackers

У типов `ChatGPT Conversion API`, `Bing Conversion API` и `Reddit Conversion API` есть нюанс с датой: площадки не принимают события старше 7 суток. AIO ограничивает отправляемую метку: у ChatGPT и Reddit — не ранее 7 суток минус 60 секунд до отправки, у Bing — не ранее 6 суток. Метки из будущего ограничиваются текущим временем.

У старой конверсии метка может измениться; повторная отправка сама по себе не гарантирует приём события площадкой. Результат проверяется в `Tracker logs`. `X Conversion API` и `Seznam Conversion API` сохраняют исходное время конверсии из прошлого, а метку из будущего ограничивают текущим временем.

### Где найти Re trigger trackers в MTK

В MTK групп `Tracker` и `Analytics` в верхнем меню нет: вкладка `Conversions` — плоский таб верхнего меню. Путь к экшену: `Conversions` (топ-нав) → ПКМ по конверсии → `Re trigger trackers`. Сам экшен и его семантика те же. Навигация MTK — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

## Как Tracker связан с остальными сущностями

### Связь и триггер: Source и Conversion

- **SOURCES — Source.** Tracker связан с одним или несколькими Source; обязательное поле `Sources` определяет, для каких Source он срабатывает. У одного Source — много Trackers. → [models/source.md](source.md)
- **TRIGGER — Conversion.** Трекер реагирует на появление конверсии заданного `Conversion type` (один трекер = один тип). `Re trigger trackers` — экшен на конверсии. → [models/conversion-model.md](conversion-model.md)

### Соседи: Destination, поля визита, FB-оптимизация

- **COUNTERPART — Destination.** Destination = **вход** конверсий (ваши постбэки, их ставит у себя получатель; визит уходит туда). Tracker = **выход** (постбэки/API стороны Source, их вам выдают, вы ставите у себя). Зеркальные направления постбэков. → [models/destination.md](destination.md)
- **READS поля визита/конверсии.** Payload трекера берёт значения через плейсхолдеры полей визита/конверсии (`{{aio.visit.fields.*}}`; точный синтаксис — [reference/placeholders.md](../reference/placeholders.md)). На входе FB-поля могут заполняться из Source и URL: существующие поля захватываются по совпадению slug, а rewrites переименовывает входные параметры; FBP/FBC может записывать SDK. → [models/source.md](source.md), [reference/placeholders.md](../reference/placeholders.md)
- **FEEDS — FB Meta optimization.** FB CAPI трекер шлёт event-сигнал (`Lead` / `Purchase` / …) обратно в FB pixel для оптимизации. Имя события берётся из поля `Event` трекера — это статика или плейсхолдер. При настройке через поля Source выбор события задают Dynamic Actions ([models/conversion-model.md](conversion-model.md)) и селекты в Link Generator (`Lead Action` и парное событие целевого действия, [how-to/campaigns.md](../how-to/campaigns.md)).
- **FLOW и SDK тоже могут создать Conversion.** Помимо внешнего постбэка конверсию вызывают шаг `Spawn Conversion` во Flow или SDK. Созданная конверсия попадает в ту же цепочку подбора Trackers.

### Соседи Tracker в топ-меню (ERP)

Sibling в группе `Tracker` верхнего меню — Campaigns, Sources, Destinations, Flows, Visits, Conversions, Trash ([reference/ui-map.md](../reference/ui-map.md)).

### Соседи Tracker в топ-меню (MTK)

В MTK группы `Tracker` нет — её разделы плоские табы верхнего меню: Campaigns, Sources, Destinations, Visits, Conversions (Flows и Trash скрыты; Trackers/Trash — в `Settings`). См. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

## Чем Tracker отличается от смежных понятий

### Conversion type vs Tracker type — два поля «type» в редакторе трекера

В редакторе трекера два поля «type». **Conversion type** = *ЧТО* за конверсия (`Lead` / `Registration` / `Push Subscribe` …). **Tracker type** = *МЕХАНИЗМ* отправки (FB CAPI / TikTok / AppsFlyer / HTTP Get…).

## Подводные камни Tracker — частые ошибки настройки

### «FB не видит лиды» — смотри Tracker logs

`Trackers → Tracker logs` → проверить outgoing-запрос. Частые причины: заданный фильтр `Sources` не включает нужный Source; либо выбран не тот `Conversion type`.

### Массовый Re trigger trackers — через чекбоксы или API

Отметить конверсии чекбоксами — в ERP `Tracker → Conversions`, в MTK верхняя вкладка `Conversions` → ПКМ → `Re trigger trackers` → `Confirm` (или по одной через ПКМ); для больших объёмов — API.

## Куда идти за деталями — смежные темы Tracker

- **Процедуры** (создать трекер, FB CAPI `Event` и `custom_data`, Tracker logs) → [how-to/source-trackers.md](../how-to/source-trackers.md).
- **Связь с Source** (suuid, rewrites, двунаправленность) → [models/source.md](source.md).
- **Conversion Types, входящий постбэк-URL, Dynamic Actions, дубликаты FB** → [models/conversion-model.md](conversion-model.md).
- **Диагностика FB CAPI** (токен / атрибуция / двойной пиксель) → *уточните у поддержки*.
- **SDK-фичи `fbCapi` / `fbPixel` на лэнде** → [reference/sdk.md](../reference/sdk.md).
- **Re trigger trackers ≠ Distribution** (не меняет payout/revenue) → [models/distributions-model.md](distributions-model.md).
- **Глоссарий** (Tracker, Re trigger trackers, Custom Key/Value) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md). ERP: `Tracker → Trackers` (`/app/tracker/trackers`); MTK: `Settings → Trackers` (`/app/mtk/settings/trackers`). Создание: `+ Tracker` → `Create tracker`; ПКМ: Edit tracker / Switch activity / Tracker logs. Re trigger trackers — ПКМ на конверсии: ERP `Tracker → Conversions`, MTK верхняя вкладка `Conversions`.
