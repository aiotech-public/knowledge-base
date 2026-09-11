---
id: api
title: API / External Integrations (How-to)
description: Data API (/api/v1/data/* — задокументированные справочники сущностей, спека в интерфейсе), MCP-сервер для внешних AI-агентов (POST /api/v1/mcp), получение payload через F12 (Tables Data / Pivot Report), Actions API (домены, косты), External Report CSV (Visits / Google Conversions), API-токен и rate limit.
doc_type: how-to
builds: [erp, mtk]
related: [mcp-server, glossary, architecture, meta-ads, permissions-model, landings, meta-spend-allocation, analytics, limits, permissions, ui-common, conversion-model, campaigns, domains, postback-generator, source-trackers]
language: ru
updated: 2026-09-11
---

# API / External Integrations (How-to)

Всё, что есть в UI AIO, доступно и через API — интерфейс сам общается с бэкендом через этот же API. Своя документация прямо в продукте есть у **Data API** (справочники сущностей — меню профиля → `Data API`) и у **MCP-сервера** для внешних AI-агентов (кнопка `MCP` в окне `AIO Chat`, разбор — [how-to/mcp-server.md](mcp-server.md)). Для всего остального публичной документации нет, и основной путь — снять запрос в UI через F12 и переиспользовать. Концепт-уровень (Compact API, broad API, лимиты, Heavy query limit) — в [reference/glossary.md](../reference/glossary.md) и [context/architecture.md](../context/architecture.md).

---

## TL;DR

- **API = всё, что есть в UI.** Документации мало, путь — F12 → Network → скопировать запрос → подменить параметры.
- **Data API** (`/api/v1/data/*`) — плоские справочники сущностей (кампании, домены, лендинги, юзеры…) со спекой прямо в интерфейсе (профиль → `Data API`). Не аналитика — метрик там нет.
- **MCP-сервер** (`POST /api/v1/mcp`) — подключить внешнего AI-агента (Claude Code и другие MCP-клиенты) к данным трекера на чтение; ссылка и инструкция собираются кнопкой `MCP` в окне `AIO Chat` ([how-to/mcp-server.md](mcp-server.md)).
- **Tables Data endpoint** — единый endpoint для большинства таблиц (Visits / Campaigns / Meta / Adsets); меняется только ключ `table` (`Namespace\Entity`).
- **Actions API** (`/actions/process`) — не только чтение: купить домен, обновить косты кампании и т.д.
- **External Report → CSV** — два типа: `Visits CSV` и `Google Conversions CSV` (загрузка офлайн-конверсий в Google Ads).
- **Токены AIO API — бесконечные**, доступы = доступам юзера.

---

## Выгрузить данные AIO в таблицу / Google Sheets

Отдельной кнопки «экспорт в Google Sheets» нет — данные тянут из AIO **через API или CSV-ссылку** и импортируют в таблицу (Google Sheets / Excel):

- **Метрики и отчёты** (визиты, кампании, Meta-косты, конверсии) — через `Tables Data` (`POST /api/v1/tables/data`) или `Pivot Report API` (оба ниже). В Google Sheets их подтягивают своим скриптом (Apps Script → UrlFetchApp) на этот endpoint со своим токеном.
- **Готовый CSV-отчёт** — `External Report → Visits CSV`: синхронная ссылка `external-reports/data`, её можно скормить прямо в Google Sheets через `=IMPORTDATA("<ссылка>")`. (`Google Conversions CSV` — про другое: загрузка офлайн-конверсий в Google Ads, а не выгрузка в таблицу.)

Если задача — тянуть **FB-косты/статистику в свою таблицу**, тяни их **из AIO** (через API), а не из FB напрямую: AIO уже собирает Meta-данные подключённой соцки ([how-to/meta-ads.md](meta-ads.md)), так не нужен отдельный FB-токен и мульти-аккаунтная возня.

Полная механика обоих путей — ниже (`Tables Data` / `Pivot Report API` и `External Report → CSV`).

---

## Data API — документированные справочники сущностей (`/api/v1/data/*`)

Справочники сущностей — часть API с **собственной документацией прямо в интерфейсе**: меню профиля → **`Data API`** (`Data API docs`). Отдельно ходить за спекой и снимать запросы через F12 для этих данных не нужно — в диалоге пять вкладок: `Overview`, `Parameters`, `Endpoints`, `Access`, `Errors`.

Что отдаёт: **плоские списки сущностей тенанта** — кампании, домены, источники, дестинейшены, лендинги, юзеры, метрики, рекламодатели, типы лендингов. Это **не аналитика**: метрик, футеров и группировок здесь нет, только атрибуты объектов. За цифрами — `Tables Data` / `Pivot Report` ниже.

База — `https://<host>/api/v1/data`, на каждый запрос нужен `X-Tenant-Id` с uuid тенанта. Если заголовок не прислать, возьмётся тенант, помеченный у юзера как выбранный; чужой или несуществующий тенант в заголовке → **401** `Auth Middleware: Unauthorized` (значение сверяется со списком тенантов юзера, а не подставляется как есть). Токен — тем же query-параметром `?token=`, что и у остальных эндпоинтов (см. «Где взять токен»).

### Data API — параметры запроса и постраничность

Общие параметры всех списочных эндпоинтов:

| Параметр | Тип | Дефолт | Что делает |
|---|---|---|---|
| `page` | int | `1` | Номер страницы |
| `limit` | int | `100` | Размер страницы, зажимается в **1…1000** |
| `uuids[]` | string[] | — | Вернуть только эти записи (фильтр по `uuid`) |
| `human_ids[]` | string[] | — | Вернуть только эти записи (фильтр по `human_id`); работает у `/campaigns`, `/domains`, `/destinations`, `/landings` |

**В `uuids` и `human_ids` — не больше 100 значений.** На 101-м придёт `500` с текстом `Table filter UUID values amount exceeded: 101 > 100` — бить на пачки по 100. Сортировка фиксированная, `created_at DESC`, снаружи не меняется.

Ответ одинаковый у всех эндпоинтов, кроме `/shared`: `rows` + `next` + `previous`. **`next` — эвристика**, а не признак наличия данных: `true`, когда вернулось ровно `limit` строк. Если размер последней страницы случайно совпал с `limit`, придёт `next: true`, а следующая страница окажется пустой — это нормальный конец списка, а не сбой. Общего количества строк API не отдаёт, поэтому листать надо по `next`, а не по счётчику. `previous` — просто `page > 1`.

### Data API — что лежит в ответе

`created_at` всегда объект из двух полей — `timestamp` (unix) и `string` (ISO); парсить строку не надо, для сортировки на клиенте берут `timestamp`. Набор полей **не унифицирован** между эндпоинтами: `human_id` есть у кампаний, дестинейшенов и лендингов, но нет у источников и доменов; `owner` есть не везде и может быть `null`.

Отдельные детали по эндпоинтам:

- `/campaigns` — `languages` и `countries` приходят массивами (в базе лежат json-строкой, наружу уже распарсены);
- `/sources` — `settings` распарсенный объект, набор ключей зависит от источника;
- `/landings` — `landing_type` может быть `null`, `folders` и `shares` — пустыми массивами;
- `/users` — `is_active` означает активность юзера **в текущем тенанте**; email тут не отдаётся.

**Архивные записи приходят вместе с активными** — параметра «только активные» нет, фильтровать по `is_archived` на своей стороне.

### Data API — кто что видит (скоупы)

Owner / Admin / роли полного доступа видят **всё в тенанте** — скоуп к ним не применяется. Остальным список режется шерингом: видно пошаренное лично, пошаренное команде, своё и принадлежащее своим подчинённым (для тимлида/head). Модель прав — [models/permissions-model.md](../models/permissions-model.md).

| Эндпоинт | Скоуп |
|---|---|
| `/campaigns`, `/landings` | по шерам и владельцу |
| `/sources`, `/destinations`, `/domains`, `/metrics` | по шерам и владельцу **+ записи с `access_type = everyone`** |
| `/landing-types`, `/users`, `/advertisers`, `/advertiser-types` | без скоупа — весь тенант |

Поверх скоупа работают права на таблицу. Права нет → **500** с телом `{"error": "Table access denied"}` (да, именно 500, а не 403). Отдельный `403` отдаётся только на нехватку read-доступа к колонкам.

### `GET /data/shared` — что видит конкретный юзер

Особый эндпоинт: не список сущностей, а **срез доступов другого юзера** — им построен экран настроек шеринга, чтобы показать, до чего человек дотягивается. Обязательный параметр `user_uuid` — uuid юзера этого же тенанта.

Доступен **только Owner / Admin**. Внутри происходит имперсонация: сервер временно переключается на указанного юзера и считает скоупы от его имени. Ответ — плоские списки uuid без имён и атрибутов, **без постраничности**: это полный набор доступных сущностей, список может быть длинным. Если у юзера роль без скоупинга (Owner / Admin), вернутся все uuid тенанта.

Ошибки: не Owner / Admin, не передан `user_uuid`, нет такого юзера в тенанте.

## MCP-сервер — подключить к данным внешнего AI-агента

`POST /api/v1/mcp` — MCP-сервер AIO: внешний AI-агент (`Claude Code`, `Claude Desktop`, любой другой MCP-клиент) подключается по ссылке и сам читает данные трекера — трафик, метрики, прогнозы, логи кампаний. Ссылку с готовой командой и конфигом собирает диалог `MCP Server` (кнопка `MCP` в окне `AIO Chat`). Авторизация та же, что у остальных эндпоинтов: `?token=<api_token>` прямо в ссылке.

Два отличия от Data API, на которых спотыкаются:

- **Тенант обязателен и явен** — заголовок `x-tenant-id` или `?tenant=<uuid>`. Фоллбэка на тенант, выбранный у юзера, здесь нет: без тенанта придёт `400` `Tenant is required: pass x-tenant-id header or ?tenant=<uuid>`.
- **Свой бюджет запросов** — серии вызовов от агента не съедают общий лимит API и наоборот.

Что именно отдаётся наружу, почему через MCP нельзя ничего изменить и как подключить конкретного агента — [how-to/mcp-server.md](mcp-server.md).

## Найти payload для пуллинга через F12

Когда: нужно автоматизировать получение данных (визиты, кампании, отчёты) через API, но публичной документации нет — берём payload из UI.

**Шаги:**

1. Открыть в UI нужную таблицу (например, `Analytics → Visits` / `Tracker → Campaigns`).
2. Открыть **DevTools (`F12`) → Network**.
3. Найти запрос с именем `Tables Data` (это единый endpoint для большинства таблиц).
4. Скопировать **payload** (Request body).
5. Использовать как шаблон в скрипте:
   - Поле `table` — определяет, какую таблицу запрашиваем. Меняй значение для других таблиц (например, `Tracker\Visits` → `Facebook\AdSets` / `Facebook\Ads` для Meta-таблиц).
   - Фильтры, groupers — копируются как часть payload.
   - Ключ `metrics` из скопированного тела **убрать**, если нужны все метрики: интерфейс кладёт в него только колонки, видимые на экране в момент копирования, и с этим ключом ответ придёт посчитанным только по ним (разбор — «`metrics` — считать только перечисленные метрики» ниже).
6. Запрос делать на тот же endpoint от своего токена.

Запросы инспектируются в `DevTools → Network → Data` и переиспользуются добавлением своего токена через query-параметр `?token=xxx`. Это **не публичный API** и может меняться без предупреждения — на стабильность полагаться нельзя.

### Tables Data — как устроен payload и что менять для разных таблиц

Endpoint: `POST /api/v1/tables/data?token=xxx`. Идентификатор таблицы задаётся ключом `table` в формате `Namespace\Entity`. Базовая структура запроса общая; меняется ключ `table`, а набор полей (`sort_key`, флаги `hide_*` и т.п.) подстраивается под конкретную таблицу. Примеры идентификаторов:

| `table` | Что |
|---|---|
| `Tech\Domains` | домены |
| `Tech\Servers` | серверы |
| `Tech\DnsProviders` | DNS-провайдеры |
| `Funnel\Landers` | лендинги |
| `Settings\Users` | юзеры |
| `Settings\Tags` | теги |
| `Tracker\Visits` | визиты |

Поля payload: `page`, `limit`, `search`, `sort_key`, `sort_direction`, `analytics_position`, `filters`, `dates` (`from` / `to`, опц. таймзона третьим элементом массива), `hide_empty_metrics`, `hide_bots`, `unwrap_tree`, `hide_trash`, `event_time_attribution`, `back_fix_attribution`, `metric_filters`, `metric_definition(s)`, и два необязательных ключа — `metrics` и `meta_spend_attribution_engine_level` (разобраны в двух секциях ниже).

`analytics_position` — номер позиции в воронке, за которую списки `Funnel\Landers` и `Tracker\Destinations` считают метрики: строка лэнда или Destination получает цифры только по визитам, где эта сущность стояла на позиции `#N`. Интерфейс шлёт `1` по умолчанию и переключает между `1` и `2` кнопкой `#1` / `#2` в тулбаре этих списков; лэнд, живущий на второй позиции, при `1` покажет нули. Разбор позиционной аналитики — [how-to/landings.md](landings.md).

### `metrics` — считать только перечисленные метрики (почему скопированный запрос возвращает не все колонки)

`metrics` — необязательный массив uuid метрик, которые нужно посчитать. Ключа в теле **нет** → считаются все метрики каталога тенанта, как раньше. Ключ **есть** → считаются только перечисленные, плюс метрики с флагом `is_main` (они нужны сортировке и футеру, считаются всегда) и зависимости формул — операнды формульной метрики бэк доразворачивает сам, перечислять их не нужно. Uuid метрики — тот же, что в `Settings → Metrics` и в ключах ответа `metric_<uuid>`, без префикса `metric_`.

Практическое следствие для F12-рецепта: интерфейс кладёт в `metrics` только **видимые колонки таблицы**, поэтому запрос, скопированный из DevTools, вернёт посчитанными лишь те метрики, что были на экране в момент копирования; скрытые в тот момент колонки бэк считать не будет, и ошибки об этом не придёт. Нужны все метрики — удалить ключ `metrics` из тела целиком.

Отдельное значение `_chart` в этом же массиве — не метрика, а сентинел для интерфейса: «считать мини-графики колонок». Для запросов по API-токену мини-графики не считаются в любом случае, так что в скрипте `_chart` можно не передавать.

### `meta_spend_attribution_engine_level` — уровень матчинга Meta Spend в запросе

`meta_spend_attribution_engine_level` — необязательный ключ со значением `ad` | `adset` | `campaign`: на каком Meta-объекте спенд сшивается с трафиком для метрик семейства `Meta Spend`. Ключа нет → уровень `campaign`. Интерфейс кладёт сюда выбор кнопки уровня в тулбаре таблицы (дефолт `campaign`, выбор хранится в браузере), поэтому скопированный из DevTools запрос уносит и его. Meta-группер или Meta-фильтр в том же запросе перебивает ключ своим уровнем. `POST /api/v1/tables/data` — единственный внешний запрос, который принимает уровень; `Pivot Report API` такого поля не имеет и всегда считает на `campaign`. Что такое уровень матчинга, чем `campaign` отличается от `ad` и где ещё он не переключается — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

### Pivot Report — какой payload слать

Endpoint: `POST /api/v1/pivot-report/data?token=xxx`. Все поля тела — **обязательные**: `dates`, пять булевых флагов, `conditions`, `definitions`. Но из пяти флагов в Pivot фактически влияют только два — `event_time_attribution` и `hide_empty_metrics`; остальные три (`back_fix_attribution`, `hide_bots`, `hide_trash`) в Pivot-пути **no-op** (тело их требует, но отчёт их не читает; см. ниже «полярность»).

- `dates` — `[from, to, timezone]`. Формат datetime — `YYYY-MM-DD HH:mm:ss`, диапазон **включителен** с обоих концов (для конца дня — `23:59:59`). Третий элемент — **IANA-таймзона** (напр. `Asia/Bangkok`, `Europe/Berlin`); всегда указывай, чтобы избежать неоднозначности.
- `conditions` — массив фильтров; каждый элемент `{ "key": "...", "values": ["<uuid>", ...] }`, где `values` — массив **UUID** выбранных сущностей. Несколько `conditions` объединяются по **AND**.
- `definitions` — массив `{"key":"..."}` для группировки (напр. `campaign_owner_uuid`, `location_country_code`). Количество — **1..7**, **порядок задаёт вложенность** (`group_1`, `group_2`, …); движок возвращает все комбинации, присутствующие в данных (разреженные комбо могут отсутствовать).
- Часть измерений мультизначные — индекс в скобках выбирает конкретный слот: `landing_uuids[1]` — первый лендинг (позиционные и роль-ключи — в следующей секции).

### Pivot Report — ключи `landing_uuids[N]` / `landing_of_type[<uuid>]` и уровень Meta Spend

Позиционный ключ `landing_uuids[N]` в `definitions` группирует по лэнду на позиции `N` воронки (`landing_uuids[1]` — первый лендинг). Роль-ключ `landing_of_type[<uuid типа лэнда>]` группирует по лэнду, сыгравшему роль этого типа на **любой** позиции воронки — аналог роль-группера `LP: <Тип>` в каталоге групперов ([how-to/analytics.md](analytics.md)); uuid типа лэнда отдаёт `Data API` эндпоинтом `/landing-types` (см. выше).

Поля уровня матчинга Meta Spend в Pivot Report **нет**: метрики семейства `Meta Spend` здесь всегда считаются на уровне `campaign` (Meta-кампания × день). Нужен уровень `adset` / `ad` — только через `Tables Data` ключом `meta_spend_attribution_engine_level` (см. выше); что такое уровень матчинга — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

### Полярность булевых флагов (что значит true/false) — путь Tables Data

Булевы флаги `back_fix_attribution`, `event_time_attribution`, `hide_bots`, `hide_empty_metrics`, `hide_trash` встречаются и в Tables Data (см. выше), и в теле Pivot Report. **Полярность ниже действует в пути Tables Data** — там читаются все пять. В Pivot Report влияют только два флага, а trash скрыт всегда (см. следующую секцию). Полярность легко перепутать:

| Флаг | `true` | `false` |
|---|---|---|
| `back_fix_attribution` | данные **без** backfix | применить backfix (рекоменд. для финальных отчётов) |
| `event_time_attribution` | атрибуция по времени конверсии | атрибуция по времени визита |
| `hide_bots` | скрыть ботов | включить ботов |
| `hide_trash` | скрыть trash (фильтр/отфильтрованные) | включить |
| `hide_empty_metrics` | только метрики, помеченные как **main** (режет non-main значения из `placeholders` → меньше payload) | все метрики |

### Pivot Report — какие флаги реально влияют (trash скрыт всегда)

В Pivot Report из пяти обязательных флагов тело реально учитывает только **два**:

- `event_time_attribution` — та же полярность, что и в Tables Data (`true` — по времени конверсии, `false` — по времени визита).
- `hide_empty_metrics` — та же полярность (`true` — только main-метрики, `false` — все).

Остальные три — `back_fix_attribution`, `hide_bots`, `hide_trash` — в Pivot-пути **no-op**: их обязательно передать в теле, но на результат они не влияют. В частности **trash в Pivot Report скрыт всегда**, независимо от значения `hide_trash`. Если нужна полярность этих флагов (включить/выключить backfix, ботов, trash) — это путь Tables Data (см. выше).

### Pivot Report — заголовок x-tenant-id и расшифровка placeholders в ответе

Заголовок `x-tenant-id` (UUID тенанта) — **опционален**; нужен, только если токен имеет доступ к нескольким тенантам. Если опущен — данные тянутся из текущего тенанта юзера-владельца токена.

В ответе ключи `placeholders` — это ID метрик вида `metric_<uuid>`. Маппинг `metric_<uuid>` → человекочитаемое имя и единицы — через `AIO → Settings → Metrics`. Тяни метрику по этому ключу, а не по названию колонки: `metric_<uuid>` стабилен, а имя команда может переименовать в любой момент.

## Actions API — как выполнять действия через API (не только читать)

Помимо чтения данных, через API доступны **действия (Actions)**: `POST /api/v1/actions/process?token=xxx`. Тело:

- `action` — имя действия (напр. `Domain\Create`, `Domain\CreateManually`, `Campaign\UpdateCosts`).
- `repository` — репозиторий (напр. `Eloquent\DomainRepository`, `MassiveEloquent\CampaignRepository`).
- `uuids` (опц.) — массив UUID для массовых действий.
- `arguments` — параметры конкретного действия.

### Купить домен в AIO

`action: Domain\Create`, `repository: Eloquent\DomainRepository`. В `arguments`:

- `domain_provider_uuid` — провайдер доменов.
- `domain_url` — URL домена.
- `server_uuid` — сервер, на который указывает A-запись.
- `dns_provider_uuid` — DNS-провайдер.
- `monitoring_user_uuid` — юзер, который получает нотификации.
- `settings` — JSON-строкой, напр. `{"robots":{"allow":false}}`.

Домен, купленный **вне** AIO, добавляется через `action: Domain\CreateManually` (тот же `repository: Eloquent\DomainRepository`). Набор полей тот же (`server_uuid` и `server_uuids` мёржатся одинаково в обоих action'ах), отличие одно — у `CreateManually` **нет** `domain_provider_uuid` (домен куплен не через провайдера AIO, покупать нечего).

### Обновить косты кампании

`action: Campaign\UpdateCosts`, `repository: MassiveEloquent\CampaignRepository`, `uuids` — массив UUID кампаний. В `arguments`:

- `from` / `to` — диапазон дат.
- `timezone` — таймзона.
- `value` — сумма (USD).
- `is_cpc` — bool.
- `field_uuid` + `field_value` — привязка к значению поля визита для агрегации костов.

### Лимиты API (rate limit, Heavy query budget)

Лимиты Pivot Report API: **100 строк / страница**, **60 запросов / минуту**, **3 одновременно**. Heavy query — **бюджет 20 сек CPU на скользящее окно последних 60 сек** (тяжёлые запросы троттлятся при исчерпании бюджета).

Rate limit — **60 запросов / минуту** на юзера по умолчанию; это значение поля `rate_limit` юзера, его меняет команда AIO по запросу. У Power-BI-эндпоинтов (`/v1/power-bi/*`) свой счётчик, но тот же потолок — `rate_limit` юзера, то есть те же 60 в минуту по умолчанию. Потолок считается на юзера, а не на IP: все запросы под одним юзером — с любых машин и из любых скриптов — делят один бюджет; симптом превышения — `429 Too Many Requests`. У MCP-сервера бюджет отдельный, общий лимит он не съедает ([how-to/mcp-server.md](mcp-server.md)). Сводка всех числовых лимитов — [reference/limits.md](../reference/limits.md).

### Где взять токен

Токен AIO API выдаёт команда AIO: сгенерировать его себе в интерфейсе нельзя — за токеном и за включением API-доступа обращаются в поддержку. Токены **бесконечные** (нет TTL). Доступы токена = доступы юзера, под которым он создан. Для API-автоматизации обычно заводят **отдельного юзера** с минимально нужным набором прав (см. [models/permissions-model.md](../models/permissions-model.md) → «Что НЕ делает Permissions Model»). Токен подставляется query-параметром `?token=xxx`.

### Ссылку с токеном пересылать нельзя — это пароль

Токен в URL — это полный доступ к API от вашего имени, поэтому передать ссылку с ним всё равно что передать пароль. Ссылка уезжает в историю терминала, конфиги скриптов и репозитории, так что не пересылайте её и не коммитьте. Особенно это касается ссылки MCP-сервера, где токен зашит в адрес по конструкции ([how-to/mcp-server.md](mcp-server.md)). Перевыпустить утёкший токен самостоятельно нельзя — новый выдаёт команда AIO.

### «Дайте read-only токен» — отдельного read-only флага нет

Отдельного «read-only токена» как флага в AIO нет. Токен всегда **наследует права своего юзера** ([models/permissions-model.md](../models/permissions-model.md) → «Что НЕ делает Permissions Model»), поэтому read-only API-доступ делается через права: завести отдельного юзера, у которого в Positions (`Settings → Positions`) сняты все edit-экшены (Deny на Action-императивах вроде `Edit Landing` / `Campaign\UpdateCosts`, оставить только view-императивы), и использовать его токен. Такой токен даёт только чтение, потому что чтением и ограничен его юзер. Настройка прав юзера — [how-to/permissions.md](permissions.md).

## External Report → CSV-экспорт

`External Reports` — раздел **только ERP-вида; в MTK его НЕТ**. Спрашивающему из MTK: выгрузка конверсий у него делается **только через API** (Tables Data — `POST /api/v1/tables/data`, таблица `Tracker\Conversions`, см. выше «Tables Data»), а НЕ через External Report / `Conversion List`.

CSV-экспорт доступен только для **двух** Integration Type: `Visits CSV` (экспорт визитов) и `Google Conversions CSV` (загрузка офлайн-конверсий в Google Ads).

Создание отчёта: `Settings → External reports` → `+ External report` → пресет `Conversions list`. У готового отчёта в списке есть колонки `UUID` и `Secret` — их значения подставляются в экспорт-ссылку. Если колонок не видно — включить их через кнопку `Settings` в тулбаре таблицы → `Table settings` (устройство настроек таблицы и пресетов — [reference/ui-common.md](../reference/ui-common.md)).

Поле `Secret` чувствительное и по умолчанию скрыто. Чтобы посмотреть, нужен включённый 2FA (`Profile Settings → 2FA`); затем клик по полю `Secret` + ввод 2FA-кода.

### Visits CSV

Поля при создании (`Integration Type = Visits CSV`):

- `Name`, `Description` — произвольные.
- `Lookup Hours` — за сколько часов отчёт.
- `Time Attribute` — `Created` (визиты по времени создания) или `Updated` (по времени создания конверсии).
- `Timezone` — напр. `Europe/Berlin`.

Базовая ссылка экспорта:

```
https://app.aio.tech/api/v1/external-reports/data?uuid=REPORT_UUID&secret=REPORT_SECRET
```

Ссылка вводится в адресную строку браузера; файл скачивается как `export.csv`.

### Google Conversions CSV

Когда: нужно загрузить офлайн-конверсии (`Lead` / `Purchase`) в Google Ads.

Поля при создании (`Integration Type = Google Conversions CSV`):

- `Name`, `Description`.
- `Conversion Types` — какие типы конверсий включить в отчёт.

Полная ссылка собирается из UUID + Secret отчёта, UUID gclid-поля и (опц.) фильтра. UUID полей (фильтр-поле, gclid-поле) — в `Settings → Fields`, колонка `UUID`:

```
https://app.aio.tech/api/v1/external-reports/data?uuid=REPORT_UUID&secret=REPORT_SECRET&rename[Lead]=...&rename[Purchase]=...&filters[FIELD_UUID]=VALUE&gclid=GCLID_FIELD_UUID&ngsw-bypass&deep=30&timezone=Europe/Rome
```

Параметры:

- `rename[Lead]=...` / `rename[Purchase]=...` — (опц.) переименовать тип конверсии в выгруженном CSV.
- `filters[FIELD_UUID]=VALUE` — (опц.) выгрузить только конверсии, где поле `FIELD_UUID` равно `VALUE` (напр. Ad ID = 123).
- `gclid=GCLID_FIELD_UUID` — UUID поля gclid в тенанте.
- `ngsw-bypass` — служебный параметр: запрос идёт мимо service worker приложения (важно, когда ссылку открывают в браузере на домене приложения); оставить как есть.
- `deep=30` — сколько прошедших дней включить.
- `timezone=Europe/Rome` — таймзона отчёта; если опущена — дефолт **GMT+3 (`Europe/Moscow`)**.

Ссылка вводится в адресную строку, файл скачивается как `export.csv`.

После генерации `app.aio.tech` в ссылке можно заменить на любой домен из `Tech → Domains` (обычно — домен, с которого льётся трафик), и уже эту ссылку вставлять в Google Pixel как URL для загрузки офлайн-конверсий.

Google подтягивает CSV по этой ссылке сам и делает это нечасто — оперативных конверсий этим путём не получить. Поэтому Google Conversions CSV — лишь один из двух параллельных путей передачи конверсий в Google Ads (второй — gtag, для оперативности); полный разбор, что каким путём закрывается и почему нет дублей — [models/conversion-model.md](../models/conversion-model.md) → «Google: два пути конверсий — gtag и Google Conversions CSV».

При генерации Google-ссылки в `Link Generator` нужно указывать **отдельно `Google Account ID`** — это **основной** путь, айди вписывается в URL при генерации и потом доступен в CSV для разделения конверсий по аккаунту. Альтернатива через `Fill Field` ноды во флоу — см. [how-to/campaigns.md](campaigns.md) → секция «Google Ads — Account ID и Conversion ID для атрибуции».

## Pull Data (FB-аккаунт) — ручной триггер

Связанная процедура: ручной триггер пуллинга данных FB (Insights, кампании, адсеты), не дожидаясь следующего автоматического интервала пуллинга.

**Путь:** `Meta → Social profiles` → ПКМ на соцке → **`Pull data`**. В диалоге два поля глубины в днях: `Backward for data` (созданные кампании / адсеты / объявления) и `Backward for insights` (спенд).

Интервал пуллинга соцки — `Pulling Interval`: `Auto 30 Minutes` или `Manually`. При `Manually` данные сами не подтягиваются — дёргать вручную: ПКМ на соцке → `Fetch data from fb` (и `Fetch costs from fb` для костов) либо тот же `Pull data`.

Также см. [how-to/campaigns.md](campaigns.md) → Polling Interval.

Не дёргать чаще раза в 30 минут — иначе можно упереться в rate limit Meta.

## Частые проблемы с API

### «Зачем мне F12, дайте Swagger» — публичного Swagger нет

Публичного Swagger нет. Но если вопрос про **справочники сущностей** (список кампаний, доменов, лендингов, юзеров), спека есть прямо в интерфейсе: меню профиля → **`Data API`** — параметры, эндпоинты, скоупы и коды ошибок (см. «Data API» выше). А если задача — «пусть AI-агент сам смотрит мои цифры», код писать не нужно вовсе: инструкция на каждого агента лежит в диалоге `MCP Server` ([how-to/mcp-server.md](mcp-server.md)). Для аналитики (`Tables Data`, `Pivot Report`) и всего остального документации нет — путь через F12.

### Лимит 60 запросов / минуту бьёт на batch-задаче

Нужно дробить или увеличивать таймаут между запросами. Альтернатива (для очень больших объёмов) — выделенная инфраструктура под тенант (за отдельную плату, детали через саппорт); полный разбор варианта — [context/architecture.md](../context/architecture.md).

### Токен API не работает

Симптом — `401` `Auth Middleware: Unauthorized` вместо данных. Причины: (1) **у юзера выключен API-доступ** — это отдельный флаг, колонка `API` в `Settings → Users`; по умолчанию он выключен, включает его команда AIO по запросу; (2) юзер деактивирован; (3) неверный сам токен. Если авторизация проходит, а отказ приходит на конкретном запросе — причина другая: (4) у юзера токена не хватает прав на эту таблицу или действие; (5) неверный HTTP-метод / endpoint.

### Формат ошибки API — как парсить неуспешный ответ

Любая необработанная ошибка API возвращается единым JSON-конвертом:

```json
{"type": "<ExceptionClass>", "message": "<текст ошибки>"}
```

- `type` — класс исключения (техническое имя, для диагностики).
- `message` — человекочитаемый текст ошибки (то же, что показывает UI).
- HTTP-код ответа = код ошибки; если у исключения нет валидного HTTP-статуса, отдаётся **500**.

В скрипте ориентируйся на HTTP-код и поле `message` (не на `type` — оно может меняться).

### `Statistics request interval exceeded` — троттл stats-запросов по токену

Симптом: запрос статистики по API-токену отбивается ошибкой `Statistics request interval exceeded, current: <интервал>`. Причина: у запросов статистики через токен есть минимальный интервал между вызовами; чаще него дёргать нельзя. `current` в тексте — сколько нужно подождать. Проверка: увеличить паузу между stats-запросами до указанного интервала.

### Heavy query исчерпал бюджет (20 сек CPU за последние 60 сек)

Дробить временной диапазон, использовать groupers с меньшей детализацией.

## Смежные темы

- [reference/glossary.md](../reference/glossary.md) — `AIO API / API 2.0 (Compact)`, `API-токен`, `Heavy query limit`, `Pivot Report API`, `Tables Data endpoint`, `Compact API`, `broad API / front-API`, `External Reports`, `Google Conversions CSV`, `Google Account ID`.
- [context/architecture.md](../context/architecture.md) — multi-tenancy ограничения, dedicated-инфра.
- [models/permissions-model.md](../models/permissions-model.md) — токен наследует доступы юзера.
- [how-to/campaigns.md](campaigns.md) — Pull Data, AIO Meta, Google Conversion ID / Label, косты кампаний.
- [how-to/domains.md](domains.md) — покупка / привязка доменов в UI (Actions API дублирует это через `Domain\Create` / `Domain\CreateManually`).
- [how-to/postback-generator.md](postback-generator.md) — кнопка `Postback generator` на странице конверсий собирает URL **входящего** постбэка, который отдают рекламодателю. Это обратное направление: не вызов API AIO, а ссылка, по которой внешняя система стучится в AIO; F12 и токен там не нужны.
- [how-to/source-trackers.md](source-trackers.md) — постбэк-трекеры (другой механизм отправки наружу).
