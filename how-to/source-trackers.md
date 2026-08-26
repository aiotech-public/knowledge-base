---
id: source-trackers
title: Sources / Trackers / Postbacks (How-to)
description: Source Rewrites (UTM → slug), создание постбэк-трекеров, FB CAPI кастомные события, чтение логов трекеров.
doc_type: how-to
builds: [erp, mtk]
related: [source, tracker, glossary, conversion-model, custom-fields, campaigns, sdk, placeholders, visit-lifecycle, debug-with-logs]
language: ru
updated: 2026-08-11
---

# Sources / Trackers / Postbacks (How-to)

Процедуры по Source-уровню (rewrites / mapping) и Trackers (постбэк-трекеры, FB CAPI). Концепт-модель Source — [models/source.md](../models/source.md), Tracker — [models/tracker.md](../models/tracker.md) (общий контекст также в [reference/glossary.md](../reference/glossary.md) и [models/conversion-model.md](../models/conversion-model.md)).

> Навигация ниже дана для ERP. В MTK группы `Tracker` в топ-нав нет — Trackers лежат в `Settings → Trackers` (Sources — отдельная топ-таба `Sources`). Сами процедуры идентичны.

---

## TL;DR

- **Создать Source** — `Tracker → Sources → + Source` (зелёная кнопка). Сначала выбираете маркетинг-платформу (FB CAPI / Google Ads / TikTok …) → подгружается шаблон → правите `Name` и код источника в JSON.
- **Source Rewrites** = маппинг входящего параметра ссылки на slug поля визита. `Tracker → Sources → <Source> → правый клик → Edit source` → ключ `rewrites` в JSON.
- **Постбэк-трекер** создаётся в `Tracker → Trackers → +Add`. Обязательно указать Source-фильтр, тип конверсии, URL.
- **FB CAPI кастомное событие** — в настройках FB CAPI трекера, поля `Custom Key` / `Custom Value`.
- Логи отправки в FB CAPI — `Tracker → Trackers → ПКМ → Tracker logs`.

---

## Создать Source

Когда: завести новый источник трафика (новый рекламный канал / новый пиксель / новая таксономия событий).

**Путь:** `Tracker → Sources` → зелёная кнопка **`+ Source`** (верхний-левый угол).

**Шаги:**

1. Открывается поп-ап **Create Source** — сначала выбираете **маркетинг-платформу**, под которую делается Source (FB CAPI, Google Ads, TikTok и т.д.). Выбор платформы **подгружает преднастроенный Source** — ссылки и параметры уже разложены, с нуля ничего писать не нужно.
2. Открывается окно редактора Source с полями:
   - **`Name`** — имя источника.
   - **Структурированный редактор** с вкладками: **`Links`** — готовые ссылки для рекламного кабинета и их `Parameters` (маппинг URL-параметров на поля визита через плейсхолдеры), **`Rewrites`** — правила «ключ URL → slug поля», **`Replaces`**, **`Raw`** — сырой JSON источника целиком.
3. `Confirm` — Source готов и доступен в `Campaigns → Link Generator`.

> Тип источника при создании (`Facebook` / `Google UAC` / `Unknown`) **не несёт функциональной разницы** — разбивка только для удобства навигации и поиска. Также Source-ы можно создавать из быстрого меню на Dashboard (верхний-левый угол).

### После создания Source — поля, расходы, постбэки

Код источника **обязан содержать поля, заведённые в AIO под этот Source** — иначе параметры из tracking-ссылки не захватятся. Поля заводятся заранее в `Settings → Fields` (см. [how-to/custom-fields.md](custom-fields.md)).

> Source может принимать **URL-параметры** (напр. CPC, tracking-плейсхолдеры рекламных платформ) через `rewrites` / JSON-конфиг и маппить их в поле `Cost` визита. Это **не** автоматический pullback расходов через Meta-интеграцию — тот работает на **уровне кампании** через `Cost Update Strategy` (см. [how-to/campaigns.md](campaigns.md)).

> После настройки Source ему почти всегда нужны **свои постбэки** — это **ключевой шаг при создании нового Source** (отбивка конверсий наружу). См. секцию «Создать постбэк-трекер» ниже.

### Структура JSON источника

Сырой JSON виден во вкладке **`Raw`** редактора (обычная настройка делается в структурированных вкладках `Links` / `Rewrites` / `Replaces`). JSON источника — это не только `rewrites`. На примере дефолтного **FB CAPI** источника блоки такие:

- **`replace`** — массив определений полей, которые юзер заполняет в Link Generator. Каждый элемент: `required`, `key`, `label`, `placeholder`, `description`, `tooltip`, `values`, `dynamic-type`, `static-data-key`, `dynamic-data-type`, `value`.
  - **`dynamic-type`**: `text` (свободный ввод) или `select` (дропдаун — тогда `values` = map опций `ключ→label`, а `value` задаёт дефолт).
  - Дефолты событий FB: `fb_lead_action` → `value: "Lead"`, `fb_ftd_action` → `value: "Purchase"`.
  - Примеры полей в редакторе: `creative` (Creative, использует launcher), `fb_account_id` (Account ID, напр. `200000000002`), `fb_capi_token` (FB CAPI Token, `EAAd…`-токен из BM), `fb_pixel` (FB Pixel, напр. `1000000000000000`), плюс `fb_pixel_domain` как replace-параметр.
- **`links`** — три варианта генерируемой ссылки: `FB Ad Short Link` (type `link`), `FB AD Parameters` (type `url_parameters` — добавляется к FB-объявлению как appendix с FB-макросами `{{ad.id}}`, `{{adset.id}}` и т.д.), `FB Ad Full Link` (type `link`, для теста).
- **`settings`** / **`placeholders`** / **`rewrites`** — доп. настройки источника и маппинг URL→поля (см. ниже).

---

## Source Mapping / Rewrites

Когда: внешний источник передаёт параметр с одним именем (`utm_buyer`), а в AIO хочется писать в поле визита с другим slug (`buyer_utm`). Маппинг — на стороне Source.

**Путь:** `Tracker → Sources` → правый клик по источнику → **Edit source**. *(Source'ы живут в `Tracker → Sources`, не в Settings.)*

Source настраивается **JSON-конфигом** (поле `Source code`):

```json
{
  "settings": {},
  "placeholders": [],
  "rewrites": {
    "<входящий_параметр>": "<slug_поля_визита>"
  }
}
```

- **`rewrites`** — это и есть Source Rewrites: ключ = имя параметра в URL ссылки, значение = slug поля визита.
- **`settings`** / **`placeholders`** — доп. настройки источника. Полная структура JSON (блоки `replace` / `links`) — см. секцию «Создать Source → Структура JSON источника».

После `Confirm`: каждый визит, пришедший на этот Source с заданным параметром, пишет значение в указанное поле визита.

### Пример rewrites — дефолтный источник «FB w/ CAPI»

**Пример — дефолтный источник «FB w/ CAPI»** (его `rewrites`): `pixel→fb_pixel`, `token→fb_capi_token`, `fb_action→fb_lead_action`, `fb_action_f→fb_ftd_action`, `adset_name→fb_ad_set_name`, `adset_id→fb_ad_set_id`, `placement→fb_placement`.

> Эти поля (`fb_pixel`, `fb_lead_action`, `fb_capi_token`) затем **читает SDK** на лэнде (`fbPixel` / `fbCapi`, см. [reference/sdk.md](../reference/sdk.md)). Цепочка: параметр в ссылке → Source rewrite → поле визита → SDK.

Это **Source-level rewrites** — действует только для визитов с этим Source. Каждый Source — свой набор rewrites. Если нужно маппить параметр **для всех source-ов** — заводи поле с тем же slug, что и URL-параметр (см. [reference/placeholders.md](../reference/placeholders.md)).

### Экшены источника (правый клик)

**Экшены источника (правый клик):** `Edit source`, `Copy source`, `Source logs`, `Assign to folder`, `Source visits`, `Source conversions`, `Show sessions`, `Build Roll Up report`, `Share`, `Change ownership`, `Archive`.

Также см. [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) → как заполняются поля визита.

## Создать постбэк-трекер

Постбэк-трекер — это **исходящая** отбивка: когда нужно из AIO отправлять постбэки наружу (в рекламную платформу / партнёрку / Source), напр. трекер типа `HTTP Get`. Не путать со входящим постбэком рекламодателя, который создаёт конверсию (это Destination-постбэк, см. [models/conversion-model.md](../models/conversion-model.md)). Трекер всегда исходящий — направление и модель в [models/tracker.md](../models/tracker.md).

**Путь:** `Tracker → Trackers → + Tracker` (или правый клик по трекеру → `Edit tracker`).

**Поля редактора:**

1. **Name** — имя трекера.
2. **Sources** — для каких Source-ов трекер срабатывает (мульти-селект). **Обязательно**.
3. **Conversion type** — тип конверсии (`Lead`, `Purchase`, `Push Subscribe`, кастомные из `Settings → Conversion types`).
4. **Tracker type** — механизм отправки (полный список 9 типов — [models/tracker.md](../models/tracker.md)). **От типа зависят поля payload ниже.** Если у Source нет conversion API (как Facebook CAPI / TikTok Events API) — выбирают `HTTP Get`.
5. **Delay in seconds** — задержка перед отправкой (`No delay` / N секунд).
6. `Confirm`. Вкл/выкл трекера — тоггл `Switch Activity` в редакторе (или экшен `Switch activity` правым кликом).

### Поля payload трекера — что ставить для FB CAPI и HTTP Get

Поля payload зависят от выбранного Tracker type.

Для `Facebook Conversion API`: `Placement` (`{{aio.visit.fields.fb_placement}}`), `Exclude Placement`, `City` (`{{aio.visit.location_city}}`), `Zip Code` (`{{aio.visit.location_postal_code}}`), `Revenue` (`{{aio.conversion.revenue}}`), и пары **`Custom Key N` / `Custom Value N`** под кастомные события.

Для `HTTP Get` / постбэк-URL — поле **`URL`** с эндпоинтом партнёрки. Макросы партнёрки в URL заменяют на AIO-плейсхолдеры, напр. `click_id={{aio.visit.fields.<source>_click_id}}` — поле визита, куда для данного Source пишется click ID. Список доступных плейсхолдеров открывается кликом по иконке-шестиграннику (hexagon) в поле `URL`.

> В UI **два разных «type»**: `Conversion type` (что за конверсия — Lead / Purchase) и `Tracker type` (механизм отправки — FB CAPI / постбэк). Не путать.

В списке трекеров есть колонка **`Quality`** — ранний индикатор поломки отбивки без захода в логи; метрика применима ко всем типам трекеров (`HTTP Get`, CAPI, TikTok и т.д.). Механика и определение — [models/tracker.md](../models/tracker.md).

Также см. [models/conversion-model.md](../models/conversion-model.md) → Conversion Types и постбэк-URL формат.

### Когда нужен отдельный трекер на каждое событие

Один трекер — один Conversion Type (правило и детали — [models/tracker.md](../models/tracker.md)). Если у тебя `Install / Trial / Subscription` — нужно **три** трекера, дублируют руками. См. также [models/conversion-model.md](../models/conversion-model.md) → секция «Подписочная модель».

### Подключить ещё один Source к существующему CAPI-трекеру

Если завёл новый FB-источник и хочешь, чтобы по нему тоже уходили FB CAPI отбивки, не обязательно создавать трекер с нуля — можно добавить Source в уже существующий: `Tracker → Trackers → ПКМ по трекеру → Edit Tracker` → добавить FB-источник в поле `Sources` → `Confirm`. По умолчанию `Lead` и `Purchase` конверсии триггерят трекеры дефолтного `FB w/ CAPI` источника.

## FB CAPI трекер с кастомным событием

Когда: дефолтное событие FB CAPI (например, `Lead`) не подходит — нужно отправить кастомное событие или из плейсхолдера поля визита.

**Путь:** открыть FB CAPI трекер → `Settings → Custom Key / Custom Value`.

**Шаги:**

1. В настройках трекера найти `Custom Key` и `Custom Value`.
2. **Custom Key** — имя события (`Lead`, `Purchase`, кастомное).
3. **Custom Value**:
   - Static — фиксированное значение.
   - Dynamic — плейсхолдер (`{{aio.visit.fields.offer_name}}`) — событие будет зависеть от значения поля визита.
4. `Save`.

### Поле `Event` в FB CAPI трекере

В FB CAPI трекере поле **`Event`** обычно ссылается на плейсхолдер по типу конверсии:

- `{{aio.visit.fields.fb_lead_action}}` — событие лида, выбранное как **Lead Action** в Link Generator;
- `{{aio.visit.fields.fb_ftd_action}}` — парное событие целевого действия / покупки, выбранное в Link Generator.

Так событие определяется тем, что выбрали при генерации ссылки (см. [how-to/campaigns.md](campaigns.md) → Lead Action и событие покупки). **Альтернатива** — захардкодить `Event` статикой прямо в трекере; тогда для этого Conversion Type всегда отправляется одно и то же событие, независимо от выбора в Link Generator.

Большинство плейсхолдеров — общие для всех FB CAPI трекеров, в т.ч. `{{aio.visit.fields.fb_capi_token}}` и `{{aio.visit.fields.fb_pixel}}`.

## Посмотреть отправку в FB CAPI (логи трекера)

Когда: «AIO отправляет ли реально Lead в FB?» / «FB говорит, что не получает события».

**Путь:** `Tracker → Trackers → ПКМ → Tracker logs` (в UI экшен — `Show Logs`).

Полная механика логов — [models/tracker.md](../models/tracker.md). Ключевые UI-детали: полная запись открывается кликом по полю **`Message`** (защищено 2FA — открыть могут только юзеры с включённой 2FA); успешные ответы помечаются `Info`, неуспешные — `Error`. Работает и для постбэк-трекеров (`HTTP Get` и др.), не только FB CAPI.

Также см. [how-to/debug-with-logs.md](debug-with-logs.md) → Trackers → Tracker logs.

## Типичные ошибки при настройке трекеров и Source Rewrites

### Source Rewrites не работает — rewrite стоит не на том Source

Проверить, что rewrite **на нужном Source** (а не на дефолтном `FB`). Каждый Source — свой набор rewrites.

### «Платформа не видит лиды» — смотреть Tracker logs

`Trackers → Tracker logs` → проверь outgoing-запрос. Возможные причины: (1) токен рекламной платформы инвалидирован — нужно перегенерить и пересоздать ссылку; (2) трекер не для нужного Source; (3) не тот тип конверсии.

### `Install` и `Purchase` в одном трекере не работают — нужно два трекера

Один трекер на один Conversion Type. Создать два трекера.

### `Tracker logs` пуст — конверсия не сматчилась с трекером

Нет визита по `visit_uuid` или Source-фильтр не совпал.

## Смежные темы

- [models/conversion-model.md](../models/conversion-model.md) — Conversion Types, формат постбэк-URL, обновление полей визита через `visit[<field>]=`.
- [models/source.md](../models/source.md) — концепт-модель Source (suuid, rewrites, двунаправленность).
- [models/tracker.md](../models/tracker.md) — концепт-модель Tracker (исходящая отбивка, 9 Tracker type, Retrigger trackers).
- [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) — как Source формирует визит, какие поля он пишет.
- [how-to/campaigns.md](campaigns.md) — генерация ссылки, Lead Action и событие покупки.
- [how-to/debug-with-logs.md](debug-with-logs.md) — чтение логов FB CAPI и Destination Handler.
- [reference/glossary.md](../reference/glossary.md) — Trackers, Retrigger trackers, Sale Status Mapping, Source-level rewrites.
