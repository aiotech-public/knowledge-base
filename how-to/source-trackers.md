---
id: source-trackers
title: Sources / Trackers / Postbacks (How-to)
description: Source Rewrites (UTM → slug), создание постбэк-трекеров, FB CAPI кастомные события, чтение логов трекеров.
doc_type: how-to
builds: [erp, mtk]
related: [source, tracker, glossary, conversion-model, ui-map, custom-fields, campaigns, sdk, placeholders, permissions-model, ui-common, visit-lifecycle, test-campaign, debug-with-logs]
language: ru
updated: 2026-10-16
---

# Sources / Trackers / Postbacks (How-to)

Процедуры по Source-уровню (rewrites / mapping) и Trackers (постбэк-трекеры, FB CAPI). Концепт-модель Source — [models/source.md](../models/source.md), Tracker — [models/tracker.md](../models/tracker.md) (общий контекст также в [reference/glossary.md](../reference/glossary.md) и [models/conversion-model.md](../models/conversion-model.md)).

> Навигация ниже дана для ERP. В MTK группы `Tracker` в топ-нав нет — Trackers лежат в `Settings → Trackers` (Sources — отдельная топ-таба `Sources`). Создание и редактирование Source и Tracker используют те же формы; доступные пункты меню и создание полей различаются по билду.

---

## TL;DR

- **Создать Source** — `Tracker → Sources → + Source` (зелёная кнопка). Сначала диалог `Choose source template` — выбираете шаблон площадки (FB CAPI / Google Ads / TikTok …), затем в форме `Create source` правите `Name` и код источника.
- **Source Rewrites** = маппинг входящего параметра ссылки на slug поля визита. `Tracker → Sources → <Source> → правый клик → Edit source` → ключ `rewrites` в JSON.
- **Постбэк-трекер** создаётся в `Tracker → Trackers → + Tracker`. Обязательно указать Source-фильтр, тип конверсии, URL.
- **FB CAPI кастомное событие** — поле `Event` FB CAPI трекера; дополнительные данные события — стандартные поля `custom_data` (`Content Ids`, `Contents`, `Predicted LTV` …) и свои пары `Custom Key N` / `Custom Value N`.
- Логи отправки в FB CAPI — `Tracker → Trackers → ПКМ → Tracker logs`.

---

## Создать Source

Когда: завести новый источник трафика (новый рекламный канал / новый пиксель / новая таксономия событий).

**Путь:** `Tracker → Sources` → зелёная кнопка **`+ Source`** (верхний-левый угол).

**Шаги:**

1. Открывается диалог **`Choose source template`** (`Выбор шаблона источника`) — выбираете шаблон площадки, под которую делается Source (FB CAPI, Google Ads, TikTok и т.д.). Шаблон **подгружает преднастроенный код источника** — ссылки и параметры уже разложены, с нуля ничего писать не нужно. Клик по карточке выбирает шаблон, двойной клик подтверждает сразу; кнопка **`Create source`** внизу диалога неактивна, пока шаблон не выбран.
2. Открывается форма **`Create source`** с полями:
   - **`Name`** — имя источника.
   - **Структурированный редактор** с вкладками: **`Links`** — готовые ссылки для рекламного кабинета и их `Parameters` (маппинг URL-параметров на поля визита через плейсхолдеры), **`Rewrites`** — правила «ключ URL → slug поля», **`Replaces`**, **`Raw`** — сырой JSON источника целиком.
3. `Confirm` — Source готов и доступен в `Campaigns → Link Generator`.

> Отдельного «типа источника» в форме нет: под какую площадку заточен пресет, определяет выбранный шаблон — его ссылки, поля Link Generator и `rewrites` ([models/source.md](../models/source.md)). Source-ы создаются и из быстрого меню на Dashboard (верхний-левый угол) — открывается тот же диалог выбора шаблона. В ERP и MTK диалог одинаковый.

### Как найти нужный шаблон в `Choose source template`

Слева в диалоге — сайдбар категорий со счётчиком у каждого пункта, справа — карточки шаблонов.

- **`All templates`** (`Все шаблоны`) — все шаблоны сразу; популярные вынесены наверх отдельной группой и в категориях ниже не повторяются.
- **`Popular`** (`Популярные`) — шаблоны, помеченные звездой (тултип на звезде — `Popular template`); пункт появляется, только если помеченные шаблоны есть.
- **Категории** — по алфавиту, за ними **`Other`** (`Другое`) для шаблонов без категории; `Other` показывается только рядом с настоящими категориями.

Поиск в шапке списка мгновенный и **глобальный**: пока в строке есть запрос, выбранная слева категория выдачу не сужает, а ищет он по имени, описанию и категории шаблона. Клик по категории во время поиска очищает строку. Пусто по запросу — `No templates match your search`; в системе нет ни одного шаблона — `No source templates yet`; список не загрузился — `Couldn't load source templates` и кнопка `Retry`.

Если в сайдбаре нет ни одной категории и остался единственный пункт `All templates` — это не поломка пикера: категории у шаблонов просто не проставлены, поэтому все они лежат одним списком.

### После создания Source — поля, расходы, постбэки

Код источника **обязан содержать поля, заведённые в AIO под этот Source** — иначе параметры из tracking-ссылки не захватятся. В ERP поля заводятся заранее в `Settings → Fields`; порядок создания нового поля в MTK — [how-to/custom-fields.md](custom-fields.md). Часть полей может приехать из самого шаблона: если в коде источника есть блок `fields`, AIO при сохранении Source заводит недостающие поля сам — механика и её ограничение в [models/source.md](../models/source.md).

> Source может принимать **URL-параметры** (напр. CPC, tracking-плейсхолдеры рекламных платформ) через `rewrites` / JSON-конфиг и маппить их в поле `Cost` визита. Это **не** автоматический pullback расходов через Meta-интеграцию — тот работает на **уровне кампании** через `Cost Update Strategy` (см. [how-to/campaigns.md](campaigns.md)).

> После настройки Source ему почти всегда нужны **свои постбэки** — это **ключевой шаг при создании нового Source** (отбивка конверсий наружу). См. секцию «Создать постбэк-трекер» ниже.

### Структура JSON источника

Сырой JSON виден во вкладке **`Raw`** редактора (обычная настройка делается в структурированных вкладках `Links` / `Rewrites` / `Replaces`). JSON источника — это не только `rewrites`. На примере источника **`FB w/ CAPI`**, который приезжает в тенант из шаблона (структура одна во всех вертикалях; различаются лейблы и предвыбранные значения селектов, а iGaming добавляет третье событие), блоки такие:

- **`replace`** — массив определений полей, которые заполняются в Link Generator. Каждый элемент: `required`, `key`, `label`, `placeholder`, `description`, `tooltip`, `values`, `dynamic-type`, `static-data-key`, `dynamic-data-type`, `value`.
  - **`dynamic-type`**: `text` (свободный ввод) или `select` (дропдаун — тогда `values` = map опций `ключ→label`, а `value` — вариант, предвыбранный при открытии Link Generator).
  - В `FB w/ CAPI` четыре поля: `token` (`FB CAPI Token`, `EAAd…`-токен из BM), `pixel` (`FB Pixel`, напр. `1000000000000000`) — оба `text`, не обязательные; `fb_action` и `fb_action_f` — селекты стандартных событий Meta (`Lead`, `Purchase`, `CompleteRegistration` … + пункт `Nothing`). В iGaming есть пятое — `fb_action_i` под событие установки. Лейблы и предвыбранные значения селектов различаются по вертикали — [how-to/campaigns.md](campaigns.md) → «Lead Action и событие покупки».

### Блок `links` и остальные ключи JSON источника

- **`links`** — у засеянного `FB w/ CAPI` одна ссылка `Facebook Launch Link` (type `link`, `luuid: true`): в параметрах `cuuid` / `suuid` / `launcher`, replace-параметры (`pixel`, `token`, `fb_action`, `fb_action_f`) и FB-макросы `{{ad.id}}`, `{{ad.name}}`, `{{adset.id}}`, `{{adset.name}}`, `{{campaign.id}}`, `{{campaign.name}}`, `{{placement}}`. Ссылок типа `url_parameters` (отдельный хвост параметров под объявление) в нём нет — только эта полная ссылка.
- **`settings`** / **`placeholders`** / **`rewrites`** — доп. настройки источника (у засеянного `settings` и `placeholders` пустые) и маппинг URL→поля (секция «Source Mapping / Rewrites» ниже).

### Флаг `luuid` у ссылки типа `link`

У элемента `links` типа `link` есть булевый флаг `luuid`; во вкладке `Links` он показан тумблером `Use luuid`.

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

После обновления настроек нетрэш-визит с этим Source переносит значение непустого URL-параметра в поле с целевым slug. Если целевой параметр уже есть в URL, rewrite его не заменяет.

### Пример rewrites — засеянный источник «FB w/ CAPI»

**Пример — источник `FB w/ CAPI` из шаблона тенанта** (его `rewrites`, одинаковые во всех вертикалях): `ad_id→fb_ad_id`, `ad_name→fb_ad_name`, `adset_id→fb_ad_set_id`, `adset_name→fb_ad_set_name`, `campaign_id→fb_campaign_id`, `campaign_name→fb_campaign_name`, `placement→fb_placement`, `pixel→fb_pixel`, `token→fb_capi_token`, `fb_action→fb_lead_action`, `fb_action_f→fb_ftd_action`; в iGaming добавляется `fb_action_i→fb_install_action`.

> Поля `fb_pixel` и `fb_lead_action` читает SDK на лэнде (`fbPixel` / `fbCapi`, см. [reference/sdk.md](../reference/sdk.md)); `fb_capi_token` используется CAPI-трекером. Цепочка начинается одинаково: параметр в ссылке → Source rewrite → поле визита.

Это **Source-level rewrites** — действует только для визитов с этим Source. Каждый Source — свой набор rewrites. Если нужно маппить параметр **для всех source-ов** — заводи поле с тем же slug, что и URL-параметр (см. [reference/placeholders.md](../reference/placeholders.md)).

### Экшены источника (правый клик)

**Экшены источника (правый клик):** `Edit source`, `Copy source`, `Source logs`, `Source visits`, `Source conversions`, `Show sessions` и `Build Roll Up report` (эти два пункта — только ERP), `Archive`; в группе `Folders` (только ERP — в MTK у этой таблицы папок нет) — `Assign to folder` / `Unassign from folder`; в группе `Teams & permissions` — `Share` и `Change ownership` (при выделении нескольких строк — ещё `Share also`, `Force share`, `Unshare`, `Mass change ownership`). Набор пунктов зависит от уровня доступа к строке: `Share` виден при уровне `Own` или шаре с `Can share further`, `Change ownership` — при `Own`; свой уровень — строка `Your access: <уровень>` внизу меню ([models/permissions-model.md](../models/permissions-model.md)).

Правку можно открыть и без меню: иконка-карандаш у первой колонки строки открывает то же окно `Edit source` в один клик ([reference/ui-common.md](../reference/ui-common.md)).

Также см. [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) → как заполняются поля визита.

## Создать постбэк-трекер

Постбэк-трекер — это **исходящая** отбивка: когда нужно из AIO отправлять постбэки наружу (в рекламную платформу / партнёрку / Source), напр. трекер типа `HTTP Get`. Не путать со входящим постбэком рекламодателя, который создаёт конверсию (это Destination-постбэк, см. [models/conversion-model.md](../models/conversion-model.md)). Трекер всегда исходящий — направление и модель в [models/tracker.md](../models/tracker.md).

**Путь:** `Tracker → Trackers → + Tracker` (или правый клик по трекеру → `Edit tracker`).

**Поля редактора:**

1. **Name** — имя трекера.
2. **Sources** — для каких Source-ов трекер срабатывает (мульти-селект). **Обязательно**.
3. **Conversion type** — тип конверсии тенанта; в ERP типы настраиваются в `Settings → Conversion types` (засеянный набор зависит от вертикали шаблона: например, `Lead`, `Registration`, `Approved`, `Install`, `Push Subscribe`; свои в ERP добавляются там же; создание типа в MTK — [models/conversion-model.md](../models/conversion-model.md)).
4. **Tracker type** — механизм отправки (полный список типов — [models/tracker.md](../models/tracker.md)). **От типа зависят поля payload ниже.** Если у Source нет conversion API (как Facebook CAPI / TikTok Events API) — выбирают `HTTP Get`.
5. **Delay in seconds** — задержка перед отправкой (`No delay` / N секунд).
6. `Confirm`. Вкл/выкл трекера — тоггл `Switch activity` в редакторе (или экшен `Switch activity` правым кликом).

### Поля payload трекера — что ставить для FB CAPI и HTTP Get

Поля payload зависят от выбранного Tracker type.

Для `Facebook Conversion API`: `Placement` (`{{aio.visit.fields.fb_placement}}`), `Exclude Placement`, `City` (`{{aio.visit.location_city}}`), `Zip Code` (`{{aio.visit.location_postal_code}}`), `Revenue` (`{{aio.conversion.revenue}}`), необязательные стандартные поля `custom_data` (секция «Поля `custom_data` в FB CAPI» ниже) и пары **`Custom Key N` / `Custom Value N`** — свои свойства внутри `custom_data` события.

Для `HTTP Get` / постбэк-URL — поле **`Url`** с эндпоинтом партнёрки. Макросы партнёрки в URL заменяют на AIO-плейсхолдеры, напр. `click_id={{aio.visit.fields.<source>_click_id}}` — поле визита, куда для данного Source пишется click ID. Список доступных плейсхолдеров открывается кликом по иконке-шестиграннику (hexagon) в поле `Url`.

> В UI **два разных «type»**: `Conversion type` (что за конверсия — `Lead` / `Registration` / …) и `Tracker type` (механизм отправки — FB CAPI / постбэк). Не путать.

Также см. [models/conversion-model.md](../models/conversion-model.md) → Conversion Types и постбэк-URL формат.

### Поля `custom_data` в FB CAPI — `Content Ids`, `Contents`, `Num Items`, `Predicted LTV` и другие

**В трекере `Facebook Conversion API` есть необязательные стандартные поля `custom_data` события — их значения AIO кладёт в событие, которое уходит в Meta.** Все поля, кроме двух селектов, принимают текст и плейсхолдеры (`{{aio.visit.fields.<slug>}}`):

| Поле в форме | Что передаёт |
|---|---|
| `Content category` | категория товара или контента |
| `Content Ids` | идентификаторы через запятую: `id1,id2,id3` — уходят списком |
| `Content Name` | название товара или страницы |
| `Content Type` | селект: `Empty`, `Product`, `Product group` |
| `Contents` | JSON-массив объектов вида `[{"id": "sku-1", "quantity": 1, "item_price": 10}]` |
| `Delivery category` | селект: `Empty`, `In store`, `Curbside`, `Home delivery` |
| `Num Items` | число позиций — число, дробная часть отбрасывается |
| `Predicted LTV` | прогнозная ценность клиента — только число |
| `Search string` | поисковый запрос |
| `Status` | статус, например регистрации |

Пустое поле в событие не попадает. Пары `Custom Key N` / `Custom Value N` добавляют в тот же `custom_data` свои свойства с произвольным именем.

### Кривое значение в `custom_data` не роняет событие FB CAPI — смотрите `warnings` в логе трекера

**Если значение поля `custom_data` не подошло (в том числе после подстановки плейсхолдера), AIO пропускает только это поле, а событие уходит в Meta без него.** Причина пропуска пишется в запись `Tracker logs` в блок `warnings`: `num_items "<значение>" is not a number, skipped`, `predicted_ltv "<значение>" is not a number, skipped`, `contents is not valid JSON (<ошибка>), skipped`, `contents[<N>] is not an object, skipped`.

`Contents` редактируется в JSON-редакторе формы: невалидный JSON подсвечивается при вводе и не даёт сохранить трекер. Одиночный объект без квадратных скобок AIO принимает как список из одного элемента; у элемента с `id` и без `product_id` значение `id` дублируется в `product_id`. Плейсхолдер внутри `Contents` ставится в кавычках — `"id": "{{aio.visit.fields.offer_id}}"`, иначе JSON не пройдёт проверку.

### Поля payload `TikTok Events API` — `Price` и `Quantity`

Цену и количество внутри товарной позиции события `TikTok Events API` задают поля payload **`Price`** и **`Quantity`** в настройках трекера. Оба принимают плейсхолдеры полей визита/конверсии, запятая как десятичный разделитель допустима.

Пустое или нечисловое значение откатывается на дефолт — подсказки в самих полях об этом и говорят: `Leave blank to use conversion revenue` у `Price` (тогда цена = выручка конверсии) и `Leave blank to use 1` у `Quantity`; значение `Quantity` меньше 1 поднимается до 1. Сумму самого события TikTok получает из выручки конверсии независимо от `Price` — это поле её не подменяет.

### Поля payload `ChatGPT Conversion API` — обязательные и `Event Type`

Форма трекера не даст сохранить без четырёх полей: **`Pixel ID`**, **`API Key`**, **`Event Type`** и **`Click ID (oppref)`**.

`Event Type` — обычное текстовое поле, но принимаются только тринадцать значений: `app_installed`, `app_opened`, `appointment_scheduled`, `checkout_started`, `contents_viewed`, `custom`, `items_added`, `lead_created`, `order_created`, `page_viewed`, `registration_completed`, `subscription_created`, `trial_started`. Любое другое значение — отправка падает, а допустимые перечислены в тексте ошибки в `Tracker logs`. Для `Event Type = custom` дополнительно обязательно **`Custom Event Name`**.

Остальные поля payload: `Action Source` (по умолчанию `web`), `Browser Reference (obref)`, `Email Field` / `Phone Field` / `External ID`, `First Name` / `Last Name`, `City` / `Zip Code` / `Region`, `Revenue` / `Currency` / `Plan ID`, `Event Source Domain`, `Validate Only (test mode)`.

### Поля payload `Bing Conversion API` — обязательные и имя события

Обязательны три поля: **`UET Tag ID`**, **`CAPI Token`** и **`Event Name`**. В отличие от `ChatGPT Conversion API`, закрытого списка событий здесь нет — `Event Name` пишется произвольной строкой.

Остальные поля payload: `Event Category` / `Event Label`, `MSCLKID Field`, `Anonymous ID Field`, `Email Field` / `Phone Field`, `Value` / `Currency`.

### Поля payload `Reddit Conversion API` — обязательные и `Reddit Event`

Трекер `Reddit Conversion API` (Reddit CAPI, конверсии в Reddit Pixel) шлёт событие в Reddit собственным запросом AIO — постбэк-URL никуда вставлять не нужно. Форма не сохранится без трёх полей: **`Pixel ID`** (подсказка `Ex: a2_xxxxxxxxxxxx`), **`Conversion Access Token`** (подсказка `Events Manager → Conversions API → Generate Access Token`) и **`Reddit Event`**.

`Reddit Event` — текстовое поле с плейсхолдерами, событие можно взять из поля визита. AIO обрезает пробелы и переводит значение в верхний регистр. Принимаются девять значений: `LEAD`, `SIGN_UP`, `PURCHASE`, `PAGE_VISIT`, `VIEW_CONTENT`, `SEARCH`, `ADD_TO_CART`, `ADD_TO_WISHLIST`, `CUSTOM`. Другое значение — отправка падает, в `Tracker logs` ошибка `Reddit Event "<X>" is not supported, use one of: …`. Пустое значение — ошибка `There is no Reddit Event at settings`, событие не уходит. Для `CUSTOM` обязательно **`Custom Event Name`** (берутся первые 64 символа).

Значение поля, в котором после подстановки остался `{{` (у визита нет такого поля), считается пустым: необязательное поле не отправляется, обязательное даёт ошибку `There is no … at settings`.

### Что `Reddit Conversion API` отправляет сам — IP, User-Agent, хеши email и телефона, сумма

Необязательные поля: `Click ID` (предзаполнено `{{aio.visit.fields.rdt_cid}}`, подсказка `Where you store rdt_cid?`), `Test ID` (тестовый код Reddit — [how-to/test-campaign.md](test-campaign.md)), `Email Field`, `Phone Field`, `External ID` (пусто — UUID визита), `Value` (пусто или не число — revenue конверсии; итог ≤ 0 — сумма не отправляется), `Currency` (предзаполнено `USD`; если очистить — валюта конверсии, без неё `USD`).

Сам AIO добавляет IP и User-Agent визита, адрес страницы `https://<домен визита><путь>` (если собрался валидный URL), источник `WEBSITE`, UUID конверсии как `conversion_id` и время конверсии (старше 7 суток — подрезается под окно Reddit). Email и телефон AIO хеширует SHA-256 сам: email — в нижнем регистре, без точек и `+…` в части до `@`; телефон — только цифры с `+`, без добавочного и ведущего `00`, короче 7 цифр не отправляется. Код страны AIO не дописывает. Что видно в логах — [models/tracker.md](../models/tracker.md), секция «Что пишет Tracker logs».

### Поля payload `X Conversion API` — пиксель, событие и ключи приложения X

Трекер `X Conversion API` (X / Twitter Conversions API) шлёт событие в X собственным запросом AIO — постбэк-URL никуда вставлять не нужно. Форма не сохранится без шести полей:

- **`Pixel ID`** — пиксель из Events Manager X (подсказка `Ex: {{aio.visit.fields.x_pixel_id}} or o8z6j`, можно плейсхолдером из поля визита);
- **`Event ID`** — событие пикселя (подсказка `Ex: tw-o8z6j-o87ne`);
- **`Consumer Key`** и **`Consumer Secret`** — ключ и секрет приложения X (подсказки `API Key of the X app` / `API Key Secret of the X app`);
- **`Access Token`** и **`Access Token Secret`** — токен пользователя с доступом к рекламному аккаунту (подсказки `Access Token of the ads account user` / `Access Token Secret of the ads account user`).

Запрос AIO подписывает этими четырьмя ключами сам (OAuth 1.0a) — отдельный Bearer-токен не нужен. Пустое обязательное поле — событие не уходит, в `Tracker logs` ошибка вида `There is no Event ID at settings`.

### Что `X Conversion API` отправляет — `twclid`, хеши email и телефона, IP, сумма

Необязательные поля: `TWCLID Field` (поле пустое, подсказка `Ex: {{aio.visit.fields.twclid}}` — сюда ставят плейсхолдер поля визита, в которое Source пишет клик-id X `twclid`; маппинг параметров — [models/source.md](../models/source.md)), `Email Field`, `Phone Field`, `Value` (пусто или не число — revenue конверсии), `Currency` (пусто — `USD`), `Description` (подсказка `Ex: Lead`).

**Событию нужен хотя бы один идентификатор, по которому X его сопоставит: `twclid`, email или телефон.** Нет ни одного — событие не уходит, в `Tracker logs` ошибка `There is no twclid, email or phone for the conversion - X can not match it`. IP и User-Agent визита AIO добавляет дополнительным идентификатором — только парой и только если IP в формате IPv4.

Email AIO хеширует SHA-256 после обрезки пробелов по краям и перевода в нижний регистр (строка без `@` не отправляется); телефон — только цифры с `+` в начале, короче 7 цифр не отправляется. UUID конверсии уходит как `conversion_id`, время — время создания конверсии. Сумма уходит всегда, с двумя знаками после точки (в том числе `0.00`). Что видно в логах — [models/tracker.md](../models/tracker.md), секция «Что пишет Tracker logs».

### Когда нужен отдельный трекер на каждое событие

Один трекер — один Conversion Type (правило и детали — [models/tracker.md](../models/tracker.md)). Если у тебя `Install / Trial / Subscription` — нужно **три** трекера, дублируют руками. См. также [models/conversion-model.md](../models/conversion-model.md) → секция «Подписочная модель».

### Подключить ещё один Source к существующему CAPI-трекеру

Если завёл новый FB-источник и хочешь, чтобы по нему тоже уходили FB CAPI отбивки, не обязательно создавать трекер с нуля — можно добавить Source в уже существующий: `Tracker → Trackers → ПКМ по трекеру → Edit tracker` → добавить FB-источник в поле `Sources` → `Confirm`. Засеянные FB CAPI-трекеры привязаны только к `FB w/ CAPI`; какие типы конверсий они отбивают в каждой вертикали — [models/tracker.md](../models/tracker.md) («Какие трекеры заведены из коробки»).

## FB CAPI трекер с кастомным событием — поле `Event`

Когда: дефолтное событие FB CAPI (например, `Lead`) не подходит — нужно отправить кастомное событие (custom event, заведённое в пикселе) или событие из плейсхолдера поля визита.

**Имя события, которое уходит в Meta, задаёт поле `Event` трекера** — статикой (`Lead`, `Purchase`, имя своего custom event) или плейсхолдером (`{{aio.visit.fields.offer_name}}` — тогда событие зависит от значения поля визита). Пары `Custom Key N` / `Custom Value N` имя события не меняют: они добавляют в `custom_data` события свойство с именем из `Custom Key` и значением из `Custom Value` (текст или плейсхолдер).

**Путь:** `Tracker → Trackers → ПКМ по трекеру → Edit tracker` → поле `Event` → `Confirm`.

### Поле `Event` в FB CAPI трекере

В FB CAPI трекере поле **`Event`** обычно ссылается на плейсхолдер по типу конверсии:

- `{{aio.visit.fields.fb_lead_action}}` — событие лида, выбранное как **Lead Action** в Link Generator;
- `{{aio.visit.fields.fb_ftd_action}}` — парное событие целевого действия / покупки, выбранное в Link Generator;
- `{{aio.visit.fields.fb_install_action}}` — событие установки (третий селект, есть только у `FB w/ CAPI` iGaming-шаблона; его читает засеянный `FB AIO Install`).

Так событие определяется тем, что выбрали при генерации ссылки (см. [how-to/campaigns.md](campaigns.md) → Lead Action и событие покупки). **Альтернатива** — захардкодить `Event` статикой прямо в трекере; тогда для этого Conversion Type всегда отправляется одно и то же событие, независимо от выбора в Link Generator.

Большинство плейсхолдеров — общие для всех FB CAPI трекеров, в т.ч. `{{aio.visit.fields.fb_capi_token}}` и `{{aio.visit.fields.fb_pixel}}`.

## Посмотреть отправку в FB CAPI (логи трекера)

Когда: «AIO отправляет ли реально Lead в FB?» / «FB говорит, что не получает события».

**Путь:** `Tracker → Trackers → ПКМ → Tracker logs` (дословный лейбл пункта — `Tracker logs`).

Полная механика логов — [models/tracker.md](../models/tracker.md). Ключевые UI-детали: полный текст поля **`Message`** копируется кликом по нему. Если включена защита данных через 2FA, для чтения нужно ввести код; при выключенной защите текст доступен без него. `Info` означает, что отправка завершилась без ошибки на стороне AIO, `Error` — что с ошибкой; принято ли событие платформой, смотрите в её ответе. Работает и для постбэк-трекеров (`HTTP Get` и др.), не только FB CAPI.

Также см. [how-to/debug-with-logs.md](debug-with-logs.md) → Trackers → Tracker logs.

## Типичные ошибки при настройке трекеров и Source Rewrites

### Source Rewrites не работает — rewrite стоит не на том Source

Проверить, что rewrite **на нужном Source** (а не на засеянном `FB w/ CAPI`, если новый источник скопирован с него). Каждый Source — свой набор rewrites.

### «Платформа не видит лиды» — смотреть Tracker logs

`Trackers → Tracker logs` → проверь outgoing-запрос. Возможные причины: (1) токен рекламной платформы инвалидирован — нужно перегенерить и пересоздать ссылку; (2) трекер не для нужного Source; (3) не тот тип конверсии.

### `Install` и `Registration` в одном трекере не работают — нужно два трекера

Один трекер на один Conversion Type. Создать два трекера — так и засеяно в iGaming-шаблоне: `FB AIO Install` под `Install`, `FB AIO Registration` под `Registration`, третий трекер под целевое действие вертикали ([models/tracker.md](../models/tracker.md)).

### `Tracker logs` пуст — проверить конверсию и условия трекера

Проверьте, создана ли конверсия, активен ли трекер и совпали ли его `Conversion type` и фильтр `Sources` с конверсией и источником визита.

## Смежные темы

- [models/conversion-model.md](../models/conversion-model.md) — Conversion Types, формат постбэк-URL, обновление полей визита через `visit[<field>]=`.
- [models/source.md](../models/source.md) — концепт-модель Source (suuid, rewrites, двунаправленность).
- [models/tracker.md](../models/tracker.md) — концепт-модель Tracker (исходящая отбивка, типы Tracker type, Re trigger trackers).
- [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) — как Source формирует визит, какие поля он пишет.
- [how-to/campaigns.md](campaigns.md) — генерация ссылки, Lead Action и событие покупки.
- [how-to/debug-with-logs.md](debug-with-logs.md) — чтение логов FB CAPI и Destination Handler.
- [reference/glossary.md](../reference/glossary.md) — Trackers, Re trigger trackers, Sale Status Mapping, Source-level rewrites.
