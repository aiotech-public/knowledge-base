---
id: custom-fields
title: Custom Fields (How-to)
description: Создание полей визита в Settings → Fields — шесть значений Type, атрибуты и тогглы карточки поля, папки по Type и Group, состав колонок и экшенов, пачка полей под интеграцию обогащения, Geo Code как select.
doc_type: how-to
builds: [erp]
related: [visit-lifecycle, glossary, user-fields, forms, visit-field, sdk, landings, flow-model, placeholders, source-trackers, meta-spend-allocation, campaigns, analytics]
language: ru
updated: 2026-08-12
---

# Custom Fields (How-to)

Как заводить поля визита, какие у поля атрибуты, типичные пачки полей под интеграции обогащения и под формы. Концептуальная модель полей и плейсхолдеров — в [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) и [reference/glossary.md](../reference/glossary.md) → Fields & Groupers Catalog.

---

## TL;DR

- Все поля визита — в `Settings → Fields → +Field`. Slug автогенерируется по имени. Сама страница — не плоская таблица, а папки по `Type` и `Group`.
- Атрибуты в карточке поля: `Is Visible`, `Is Registry`, `Is Macro Visible`, `2FA Protected`.
- Вписать значение поля прямо в карточку `Source`, `Campaign`, лендинга, `Destination` (и раздать его из `Flow` / `Lander Type` / `Advertiser`) — секция `User fields`: [how-to/user-fields.md](user-fields.md).
- Под интеграцию обогащения (`Traffic Filter`, дозаполняющий поля визита внешними данными) пачку полей нужно завести **заранее** — иначе ответ интеграции некуда записать и поля останутся пустыми; сами интеграции — *уточните у поддержки*.
- `Geo Code` (и другие строки) можно превратить в **select** через `Available Values`.
- `required` напрямую на **Visit Field** сделать **нельзя** — только через `Link Generator` (помечает параметр обязательным в ссылке). Не путать с `Required` на **Form Control** в `Settings → Forms` — там обязательность инпута SDK-формы работает (см. [how-to/forms.md](forms.md) § Required — два контекста).

---

## Создать поле визита

**Путь:** `Settings → Fields → +Field`.

### Два режима создания — `Visit string field` vs `Create manually`

При `+Field` система предлагает два режима:

- **`Visit string field`** — базовый, самый быстрый путь для обычного строкового поля визита. Заполняете только `Name` (+ опц. `Description`), остальные атрибуты — по дефолтам.
- **`Create manually`** — полный набор атрибутов для более тонкой настройки.

### Что выставлять в `Create manually` для типового строкового поля

Для типового строкового поля визита в `Create manually` выставляют: `Type = Visit`, `Format = String`, `Data source = Agent Init`. `Type` и `Format` — **два раздельных select** (`Type` = сущность-владелец поля и неймспейс адресации — `Visit` / `Campaign` / `Source` / `Conversion` / `Landing` / `Destination`; для обычного поля визита это `Visit`; концепт и роутинг неймспейсов — [models/visit-field.md](../models/visit-field.md). `Format` = как хранить/показать + дефолт/formatter, select из ~20 значений — `String`/`Number`/`Boolean`/`Placeholder`/`Variant`/`Money`/`Phone`/`Ip`/… и др.); подробно — в шагах создания поля. `Is Visible` обычно on; `Is Macro Visible` — on только если поле читается на лэнде через AIO SDK (см. [reference/sdk.md](../reference/sdk.md)); `Is 2FA Protected` обычно off. Чтобы поле сразу было доступно как группер в отчётах — в `Availability as grouper` выбрать `Analytics` и `Tables` (см. секцию «Поле как группер» ниже). Если при создании `Is Visible` не включён, поле можно добавить в таблицы позже через `Presets → Table Settings`.

### Какие атрибуты заполнять при создании поля

**Шаги:**

1. **Name** — имя поля. Slug автогенерируется по имени (напр. `Postal Code` → slug `postal_code`).
2. **Description** — опционально.
3. **Group** — опционально. Категория/тег (напр. `Google`, `FB`, `Landings`), задаётся при создании, чтобы группировать поля для удобной навигации в таблицах и аналитике. Эта же категория = категория поля как группера (секция «Поле как группер» этого документа) и папка второго уровня на странице `Settings → Fields`.
4. **Type** (`fieldTypes`) — **сущность-владелец поля**, она же определяет неймспейс адресации (значение в любом случае хранится на визите). Шесть значений: `Visit`, `Campaign`, `Source`, `Conversion`, `Landing`, `Destination`; для обычного поля визита — `Visit`. Откуда у каждого типа берётся дефолт и каким плейсхолдером поле читается — [models/visit-field.md](../models/visit-field.md); где значение вписывается в карточку сущности — [how-to/user-fields.md](user-fields.md).
5. **Format** (`fieldFormats`) — отдельный select (не `Type`): хранение значения + дефолт + formatter + тип плейсхолдера. Значений ~20; что выбрать — секция «Форматы поля» этого документа. Задаётся только при создании: в `Edit Field` этого select нет.
6. **Data source** — select, который **ни на что не влияет**: выбрать значение можно, эффекта у выбора нет, для любого поля можно оставить `Agent Init`. Кто на самом деле штампует источник записи — [models/visit-field.md](../models/visit-field.md).

### Форматы поля (`Format`) — что выбрать

`Format` — не косметика: выбирает дефолт значения, formatter и тип плейсхолдера, а не только отображение. Список **не исчерпывающий** (~20 значений); грузонесущие:

- `String` — строка (дефолт пусто); `Number` — число (дефолт `0`); `Boolean` — true/false.
- `Placeholder` — поле-плейсхолдер (используется в Content Library).
- `Variant` — часто для `LP_*` сплитов; для поля под **Split Key** контент-сплитов `Format = Variant`, slug вставляется в `Split Key` лэнда, бэкфилла нет ([how-to/landings.md](landings.md) → Content Splits).
- `Money`, `Percentage`, `Time Seconds`, `Progress Number` — числовые (дефолт `0`).
- `Phone`, `Ip` — типизируют плейсхолдер под формат (валидация).
- `Destination`, `Lander`, `Country`, `Language`, `Domain` и др. — `Destination`/`Lander` подставляют связанную сущность, а не голый uuid.

### Тогглы поля (`Is Registry`, `Is Macro Visible`, `2FA Protected`) и `Available Values`

7. **Атрибуты** (toggle):
   - **`Is Visible`** — отображать в UI визита.
   - **`Is Registry`** — поле хранит секвенцию значений (история всех записей `1,2,3...`); если выключено — каждый постбэк перезаписывает.
   - **`Is Macro Visible`** — поле доступно через объект `aio` на лэнде (JS-доступ в рантайме, не плейсхолдеры).
   - **`2FA Protected`** — значение скрыто в UI без 2FA-кода (для phone, email).

8. **Available Values** — если поле должно быть select: список значений (секция «Geo Code как select (Available Values)» этого документа).
9. `Save`.

### `Type = Campaign` — тоже поле визита, дефолт из кампании

`Campaign` — **не отдельное «поле кампании», а то же поле визита**, только его значение по умолчанию берётся не из ссылки, а из карточки кампании: задаётся один раз на кампанию и **копируется на каждый её визит** при заходе. Удобно для FB Pixel / CAPI Token — один общий дефолт на весь трафик кампании, не светится в URL.

Значение всё равно живёт **на визите** и **может быть перезаписано для конкретного визита** — как любое поле визита, в том числе выбрав его целью в шаге `Fill Fields` во флоу ([models/flow-model.md](../models/flow-model.md)). Это не одно неизменяемое значение на всю кампанию. Концепт и разница Visit-vs-Campaign — [models/visit-field.md](../models/visit-field.md).

Заводится поле по-прежнему здесь, в `Settings → Fields`, но в карточке кампании оно не появляется само: форма кампании показывает только поля, добавленные кнопкой `Add field` в секции `User fields`, плюс унаследованные от `Flow`. Как добавить поле в карточку и что при этом происходит со старыми кампаниями — [how-to/user-fields.md](user-fields.md).

### Slug — буквальное совпадение

Slug используется в плейсхолдерах (`{{aio.visit.fields.<slug>}}`), Source Mapping, Non-SDK Form Handler. **Изменить slug после создания обычно нельзя** — поле архивируется и создаётся новое. См. [reference/placeholders.md](../reference/placeholders.md).

### Куда вставлять slug поля

- **`Tracker → Trackers` (Edit Tracker, поле URL)** и **`Tracker → Destinations` (Edit Destination)** — slug вставляется как плейсхолдер через **hexagon-иконку** в редакторе. Иконка открывает список доступных плейсхолдеров в формате `{{aio.visit.fields.xxx}}` с поиском по имени поля. См. [how-to/source-trackers.md](source-trackers.md).
- **`Tracker → Sources` (Edit Source)** — slug вписывается **вручную** в код источника, чтобы смапить поля источника на поля AIO (напр. `CLICK_ID` источника = `<source>_click_id` в AIO). См. [how-to/source-trackers.md](source-trackers.md) → Source Rewrites.

### `Is Used For Cost` — только для ручных апдейтов, и тоггла в карточке поля больше нет

Атрибут нужен для **ручного** `Update Costs By <Field>` (правый клик → Update Costs). Для FB **не актуален** — по FB-костам и профиту основной способ — метрика `Meta Spend` ([mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md)); старый перелив через `AIO Meta` в метрику `Costs` — устаревший путь (см. [how-to/campaigns.md](campaigns.md) → секция Update Costs).

**Выставить флаг из интерфейса нельзя:** тоггла `Is Used For Cost` нет ни в `Create manually`, ни в `Edit Field`, колонки `Used For Costs` в `Settings → Fields` тоже нет. Механика при этом жива — в выборе `Update Costs By <Field>` остаются неархивные поля, у которых флаг был включён раньше. Новому полю его сейчас не включить.

### Поле как группер — `Availability as grouper`

Доступность поля как **группера** задаётся **внутри поля**: `Edit Field → **Availability as grouper**` — мульти-селект со значениями **`Analytics`** (группер в Roll Up / Analytics) и **`Tables`** (группер-разбивка в Tracker-таблицах, напр. Campaigns drill-down). Поле встаёт группером под своей категорией **`Group`** (`Landings` / `Facebook` / `Quora` / `AdForm` / `LP Content Variables`…).

**Не путать с экшеном `Make analytic`** (правый клик по полю): он выделяет полю **отдельную колонку в хранилище аналитики** → **ускоряет** разбивки/агрегации по полю, но сам по себе группер **не включает**. Группер — это `Availability as grouper`.

Поэтому каталог групперов **per-tenant**: системные поля + поля с включённым `Availability as grouper`. См. [how-to/analytics.md](analytics.md) → Groupers.

### Как устроена страница `Settings → Fields` — папки по `Type` и `Group`

Поля разложены по раскрывающимся папкам, а не выведены плоской таблицей. Верхний уровень — `Type`, в порядке хода воронки: `Sources` → `Campaigns` → `Landings` → `Destinations` → `Conversions` → `Visits`. Внутри типа — папки по `Group` поля (по алфавиту), поля без группы собраны в конце под заголовком `Ungrouped` (`Без группы`). Если в типе вообще нет полей с `Group`, второго уровня не будет — поля лежат прямо в папке типа. У каждой папки счётчик полей.

Свёрнутые папки запоминаются браузером: в другом браузере или после чистки данных сайта все папки снова раскрыты.

Пагинации и переключателя числа строк на этой странице нет — весь список (до 1000 полей) грузится одним запросом. На обычных таблицах выбор `10 / 25 / 50 / 100` не изменился ([how-to/analytics.md](analytics.md)).

### Какие колонки показывает `Settings → Fields`

По умолчанию видны: `Field` (имя поля), `Slug`, `Description`, `Access type`, `Shares`, `Visible Macros` (= `Is Macro Visible`), `Visible`, `Registry`, `2FA Protected`.

Скрыты по умолчанию и включаются в настройках таблицы: `UUID`, `Group`, `Format`, `Availability as grouper`, `Owner`, `Created`, `Updated`, `Archived`.

Колонок `Data source`, `Pre Processor`, `Used For Costs` и `Analytics` в таблице нет — их убрали совсем. Сделано ли поле аналитическим, теперь видно не по колонке, а по составу экшенов правого клика: у такого поля вместо `Make analytic` предлагаются `Relink analytic` и `Remove analytic`.

### Экшены правого клика в `Settings → Fields`

`Edit field`, `Field logs`, **`Make analytic`** (ускоритель разбивок по полю), `Relink analytic` и `Remove analytic` (у поля, которое уже сделали аналитическим), `Share`, `Change ownership`, `Archive` / `Unarchive`.

### Что содержит `Edit Field`

`Name`, `Group`, **`Availability as grouper`** (`Analytics` / `Tables`), `Available values`, **`Type`** (`fieldTypes`: сущность-владелец и неймспейс — `Visit` / `Campaign` / `Source` / `Conversion` / `Landing` / `Destination`, значение всегда на визите), тогглы (`Is 2FA protected` / `Is visible` / `Is registry` / `Is macros visible`) и `Description`.

### Не нахожу `Pre-Processor` в карточке поля — куда он делся

`Pre-Processor` и `Pre-Processor Settings` из карточки поля убраны: этих контролов нет ни в `Create manually`, ни в `Edit Field`, и колонки `Pre Processor` в таблице `Settings → Fields` тоже нет. Выбрать пре-процессор (`String`, `Taboola CPC`) из интерфейса больше нельзя.

Рядом в карточке поля нет ещё трёх контролов: `Is Used For Cost` убран вовсе (секция «`Is Used For Cost`» этого документа), а `Format` и `Data source` задаются только при создании поля — в `Edit Field` их нет.

## Geo Code как select (Available Values)

Когда: хотите сделать поле строкового типа выбираемым из фиксированного списка (вместо свободного ввода).

**Путь:** `Settings → Fields → Edit Field (<имя поля>) → Available Values → список значений`.

**Шаги:**

1. Открыть поле для редактирования.
2. Найти секцию `Available Values`.
3. Внести значения JSON-массивом или построчно (зависит от UI).
4. `Save`. Поле в UI кампании теперь — select из этих значений.

Также см. [reference/glossary.md](../reference/glossary.md) → Available Values.

### `required` для Visit Field — через Link Generator

Сделать **Visit Field** `required` напрямую в `Settings → Fields` **нельзя** — UI не поддерживает. Альтернатива — пометить обязательным в `Link Generator` при генерации ссылки кампании; тогда параметр в URL станет обязательным.

Это **не то же самое**, что `Required` на **Form Control** (в Form Builder инпут можно сделать обязательным валидацией SDK-формы — там работает напрямую). См. [how-to/forms.md](forms.md) § «Required — два контекста, не путать».

## Item Lists — именованные списки значений

Связанный механизм: именованный список значений одного типа, на который ссылаются правила вместо перечисления значений вручную.

**Путь:** `Settings → Item Lists → +Item List`.

Атрибуты списка при создании:

- **`Type`** — тип элементов списка, **обязателен**, значения `Ip` | `User Agent` | `Phone`, дефолт `Ip`. **Один список = один тип** (IP-список, User-Agent-список и телефон-список — раздельные Item List, смешивать нельзя).
- **`Description`** — опционально.

**Шаги:**

1. Задать имя списка.
2. Выбрать `Type` (`Ip`, `User Agent` или `Phone`).
3. Внести значения (IP / User-Agent / телефоны — построчно или через запятую).
4. `Save`.

**Как готовый список подставляется в правило:** в правиле выбирают соответствующее поле визита и оператор `In` / `Not In`, а вместо ручного перечисления значений указывают сам список. Где настраиваются такие правила — *уточните у поддержки*.

## Типичные ошибки

- **Подключил интеграцию обогащения (`Traffic Filter`), а поля визита пустые.** Поля под маппинг/интеграцию заводите ЗАРАНЕЕ — сначала создать поля в `Settings → Fields`, потом подключать интеграцию (детали — *уточните у поддержки*).
- **`required` не работает на поле.** Через `Settings → Fields` нельзя. Через `Link Generator` — отметить required при генерации.
- **Хочу поменять slug.** Нельзя на существующем — архивировать старое, создать новое. Если поле уже используется в плейсхолдерах — придётся править лэнды.
- **Поле `Is Used For Cost = true`, но FB-косты не приходят.** `Is Used For Cost` — для **ручного** Update Costs. По FB-костам и профиту основной способ — метрика `Meta Spend` ([mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md)); старый перелив через `AIO Meta` в метрику `Costs` — устаревший путь (см. [how-to/campaigns.md](campaigns.md)).

## Смежные темы

- [models/visit-field.md](../models/visit-field.md) — концепт: поле визита как слот данных и субстрат системы; в каком порядке значения ложатся на визит.
- [how-to/user-fields.md](user-fields.md) — секция `User fields` в карточках `Source` / `Campaign` / лендинга / `Destination` и режимы полей у `Flow` / `Lander Type` / `Advertiser`.
- [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) — путь визита и поля.
- [reference/glossary.md](../reference/glossary.md) — Fields & Groupers Catalog (Tracker/Location/Browser/OS/Time/Web/Landings/Destinations/Custom), `Available Values`, `Item List`.
- [reference/placeholders.md](../reference/placeholders.md) — `{{aio.visit.fields.<slug>}}`, Source Mapping.
- *уточните у поддержки* — интеграции обогащения: под какие поля их заводить.
- [how-to/source-trackers.md](source-trackers.md) — Source Rewrites (slug-маппинг).
- [how-to/campaigns.md](campaigns.md) — Update Costs By Field (использует `Is Used For Cost`).
