---
id: analytics
title: Analytics / Reports (How-to)
description: Highlight Row, тоггл Attribute by events time (атрибуция по времени события), Colorize Static, древовидный отчёт через Groupers (полный каталог), Cohorts / Comparative, Tracker data-секции, экспорт таблицы в CSV (что попадает в файл, где есть кнопка, право на выгрузку), Roll Up по полям антифрода.
doc_type: how-to
builds: [erp, mtk]
related: [glossary, meta-ads, meta-spend-allocation, custom-fields, limits, events-exporter, permissions-model, user-fields, visit-field, campaigns, debug-with-logs, conversion-model, metric, conversion-ai-testing, live-pulse, session-analytics]
language: ru
updated: 2026-08-12
---

# Analytics / Reports (How-to)

Частые операции в аналитике: подсветка строк, переключатель атрибуции `Attribute by events time`, цветовая разметка по диапазонам, древовидные отчёты через Groupers, Cohorts и Comparative analytics, Tracker data-секции, Roll Up по полям антифрода. Концептуальные понятия (Cohorts, Compare, Visit Loss, Qualified Visits) — в [reference/glossary.md](../reference/glossary.md).

---

## TL;DR

- **Highlight Row** = подсветить выбранные строки (фильтр по объектам). ПКМ → `Highlight Row`.
- **`Attribute by events time`** = тоггл: считать конверсии по дате события, а не дате визита.
- **Colorize → Static** = окрасить ячейки по диапазонам метрики (Roll-Up).
- **Древовидный отчёт (Groupers)** = `+` рядом с чипами групперов → каталог (Tracker/Location/Landings/Destinations/Variants/Client/Device/OS/Time/Browser/Other/Funnel + динамические категории полей); синие `►` на строках раскрывают drill-down; до 7 групперов; работает в `Campaigns` и `Roll Up report`.
- **`Unwrap tree view`** = кнопка, которая разворачивает древовидную группировку в плоский список всех комбинаций.
- **`Export as CSV`** = выгрузка таблицы в фон, готовый файл приходит в колокольчик: до 10000 строк, ссылка на файл живёт 7 дней, кнопка есть не на всех таблицах и требует права `features.table-export`.

---

## Highlight Row

Когда: выделить пачку строк в таблице, чтобы дальнейшие отчёты строились **только по ним**.

**Путь:** ПКМ на строке (или нескольких выделенных) → `Highlight Row`.

После применения — в таблицу добавляется фильтр по выбранным объектам. Снять — через панель фильтров.

**Где доступен:** не только в отчётах аналитики — тот же экшен есть в таблицах Meta-модуля (на каких именно вкладках и в ERP, и в MTK-виде — [how-to/meta-ads.md](meta-ads.md)). Механика везде одна: фильтр по выделенным строкам добавляется в таблицу, снимается через панель фильтров, работает и на нескольких выделенных строках сразу.

## Атрибуция по времени события — тоггл `Attribute by events time`

Когда: смотришь конверсии за период и хочешь разделить «лиды, пришедшие в этот день» vs «лиды по визитам, состоявшимся в этот день».

Частые формулировки: «тоггл атрибуции по времени события», «Attribution by Event Time», «переключатель атрибуции», «атрибуция по конверсии».

**Путь:** `Roll Up report` / `Campaigns` / любая таблица → тоггл **`Attribute by events time`** в верхней панели таблицы (подсказка тоггла: `Attribute by event time (Also known as conversion time attribution)`).

**Что делает:**

- **Дефолт** (тоггл выключен) — конверсия атрибутируется к **дате визита**. Лид, пришедший сегодня по визиту вчерашнего дня — попадёт во вчерашний день.
- **`Attribute by events time` включён** — конверсия атрибутируется к **дате события** (когда пришёл постбэк). Лид сегодня — в сегодняшний день.

**Когда что:**

- ROI по дню запуска кампании → тоггл выключен (дефолт).
- Дневная статистика поступления конверсий → включить `Attribute by events time`.

Также см. [reference/glossary.md](../reference/glossary.md) → Attribute by events time.

Если у команды разные toggle-настройки между байерами и тимлидом — числа не сойдутся. Договариваться единообразно: держать `Attribute by events time` в одном положении на всю команду.

## Colorize → Static в Roll-Up

Когда: визуально подсветить ячейки по диапазону (например, `engagement_rate < 25` красным, `25-60` жёлтым, `> 60` зелёным).

**Путь:** `Roll Up Report → Colorize → Static → выбрать метрику → задать ranges`.

**Шаги:**

1. Открыть Roll-Up Report.
2. В панели сверху найти `Colorize`.
3. Режим `Static` (есть ещё `Dynamic` для автомата).
4. Выбрать метрику (`engagement_rate`, `CR`, `ROI`, …).
5. Задать диапазоны и цвета.
6. Применить.

После применения — ячейки в столбце метрики окрашиваются.

## Древовидный отчёт через Groupers

Когда: разбивка таблицы по нескольким полям (drill-down), напр. `Campaigns → Source → Country` или `LP #1 → LP #2 → OfferName`. Работает **одинаково и в `Tracker → Campaigns` (и других Tracker-таблицах), и в `Analytics → Roll Up report`**.

**Как добавить:** рядом с чипами групперов (напр. чип `Campaigns`) — кнопка **`+`** → открывается **каталог групперов** (поиск + категории). Выбираешь группер — он встаёт чипом в цепочку (`Campaigns → Source → …`), `×` на чипе убирает. До **7 групперов** стеком. Очень большие разбивки могут упереться в Pivot API лимиты (100/стр, 60/мин, 3 одновременно, heavy = 20с CPU-бюджет на окно 60с; см. [reference/glossary.md](../reference/glossary.md)).

**Как раскрыть:** после добавления у каждой строки появляются **синие стрелки `►`** — клик раскрывает строку на следующий уровень группера (drill-down по дереву). Частые формулировки: «лид не в дрилдауне», «конверсия не раскрывается в дереве», «не проваливается на следующий уровень» — обычно лид есть, но не под тем группером/уровнем, на который смотрят.

### Косты и профит по FB в разрезах — колонка Meta Spend

Когда: смотришь косты, спенд или профит по FB в разбивке (по Source, Destination, гео, офферу) прямо в древовидном отчёте. Частые формулировки: «где мои косты по FB в отчёте», «спенд по разрезу», «профит по Destination», «ROI по FB в Roll-Up».

**Основной способ — колонка `Meta Spend`** в Roll Up report. Она считается на лету в момент отчёта, разносит расход из Meta по визитам под любой выбранный группер и сходится до копейки, а рядом стоит revenue → профит и ROI по FB читаются прямо в отчёте по любому разрезу. У каждой строки рядом со `Meta Spend` стоит значок-статус (fine / noisy / partial) — насколько надёжна аллокация в эту строку. Полная механика раскладки, статусы и диагностика — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

### Каталог групперов

Каталог (категория → суб-групперы) — типовые системные групперы (динамический per-tenant состав см. в подсекции ниже):

| Категория | Групперы |
|---|---|
| **Tracker** | Campaigns, Source, Flow, Split Group |
| **Location** | Country, City, Continent, Time Zone |
| **Landings** | LP #1…#3, LP Type #1…#3 |
| **Destinations** | Destination #1…#3, Advertiser #1…#3 |
| **Variants** | Variant #1…#10 |
| **Client** | Client Engine, Client Engine Version, Client Family, Client Name, Client Type, Client Version |
| **Device** | Brand Name, Device Name, Model (`Device Name` бьёт метрики по device-type: `smartphone` / `tablet` / `desktop` и др.) |
| **OS** | OS Combined, OS Name, OS Platform, OS Version |
| **Time** | Day, Hour, Month, Week, Year, Day Part, Week Part |
| **Browser** | Browser Language, Initial Domain, Initial Path |
| **Other** | Screen Dimensions, Agent Version, Trash Reason, ASN Organization, Visit UUID, Ip Address, User Agent |
| **Funnel** | Funnels Contains LP, Funnels Contains LP Type |

### Почему каталог групперов у меня другой (динамический per-tenant)

Каталог групперов **динамический и per-tenant**. Таблица выше — **системные** групперы; плюс туда попадают **кастомные поля визита** из `Settings → Fields`, у которых в `Edit Field → **Availability as grouper**` выбрано **`Analytics`** (группер в Roll Up) и/или **`Tables`** (группер в Tracker-таблицах). Поле встаёт группером под категорией из своего атрибута `Group` (пустой `Group` → категория `Fields`), подписью группера служит имя поля. При подключённых рекламных интеграциях в каталог добавляются категории `Facebook` / `Google`. Поэтому у разных тенантов список разный — напр. кастомные поля под конкретный источник или интеграцию (`Quora Campaign ID`, `AdForm Order ID`). Категории с «+N» как раз скрывают такие доп. (часто кастомные) поля.

*(Не путать: экшен `Make analytic` по полю — это **отдельная колонка для скорости** разбивок, а не вкл/выкл группера.)* См. [how-to/custom-fields.md](custom-fields.md).

### Поиск и Table Settings в Roll Up

- **Search.** Поиск ищет сущности по UUID или имени (Enter / кнопка-лупа). Какую сущность ищет — определяется **первым выбранным группером**: первый группер `Campaign` → поиск по кампаниям, и т.д.
- **Table Settings.** Путь `Presets → Table settings`. В окне: фильтры, пометка строк как **Favorites**, скрытие / переупорядочивание колонок, сохранение пресетов, число строк на странице (`10 / 25 / 50 / 100`). На страницах, которые грузят весь список одним запросом (сейчас это `Settings → Fields`), переключателя числа строк в окне нет, а в верхней панели нет пагинации — листать там нечего.
- **Номер строки.** В таблице есть колонка с порядковым номером строки. Подсказка в шапке: `Row number. Calculated on the front end — handy for naming a row while sharing your screen`. Номер считается **на фронте по текущей выдаче** — это позиция в том, что сейчас на экране, а не идентификатор записи: при смене сортировки, фильтров или страницы он у той же строки поменяется. Ссылаться на объект по нему нельзя, для этого есть `Human ID` / UUID ([reference/glossary.md](../reference/glossary.md)).

### Отчёт не строится: нужно выбрать группер

Симптом: отчёт (Roll Up / DrillDown) не строится, приходит ошибка про группировку. Причина — не выбран ни один группер. Точные сообщения:

- `Groups must be selected.` — DrillDown-отчёт запущен без единого группера.
- `Please specify group` — Tracker-таблица `Other` открыта без выбранного группера (эта секция стартует пустой, наполняется вручную).

Проверка: добавить хотя бы один группер (кнопка `+` рядом с чипами) и повторить.

### Фильтр по колонке не применяется: слишком много значений

Симптом: фильтр таблицы отбивается ошибкой про количество значений. Частые формулировки: «фильтр не применился», «слишком много значений в фильтре», «`values amount exceeded`».

Причина: в один табличный фильтр можно передать не больше **100** значений. При превышении бэк кидает `Table filter <label> values amount exceeded: <n> > 100` (где `<label>` — имя колонки, `<n>` — сколько значений пришло). Число — в [reference/limits.md](../reference/limits.md).

Проверка: сузить выбор значений в фильтре до 100 или отфильтровать другим способом (по метрике, по поиску).

## Как выгрузить таблицу в CSV и что попадает в файл

Выгрузка запускается кнопкой **`Export as CSV`** (`Экспорт в CSV`) в верхней панели таблицы: открывается панель со списком условий → `Start export` (`Начать экспорт`) → тост `Экспорт запущен — появится в уведомлениях, когда будет готов`. Файл собирается в фоне и приходит уведомлением в колокольчик.

Панель дословно перечисляет, что попадёт в файл:

- `Uses the current filters and sorting.` — берётся текущее состояние таблицы: период, фильтры, поиск, сортировка.
- `Exports a selected subset of columns (not all).` — набор колонок задаёт сама таблица; скрытие колонки в `Table settings` из файла её не убирает.
- `Only the first level of the table is exported.` — раскрытые уровни drill-down в файл не идут, только верхний уровень дерева.
- `Up to 10000 rows are exported.` — потолок строк на одну выгрузку ([reference/limits.md](../reference/limits.md)).
- `The download link is valid for 7 days.` — срок жизни ссылки на готовый файл.

Файл пишется с BOM, поэтому Excel открывает кириллицу без «кракозябр».

### На каких таблицах есть кнопка `Export as CSV`

Выгрузка в CSV поддерживается не каждой таблицей. Кнопка есть в древовидных отчётах — `Roll Up report` (в MTK-билде тот же отчёт открывается табом `Reports`) и Tracker-секция `Other` — а также в `Tracker → Campaigns` (только ERP) и в `Conversions`, причём конверсии выгружаются и в ERP, и в MTK.

На таблице визитов (`Tracker → Visits`) выгрузки в CSV нет вовсе — это частый вопрос. Нет её и на остальных Tracker-секциях (`Countries`, `Days`, `Devices`, `OS`, `Browsers`, `Day Party`), и на таблицах настроек. Запрос на экспорт такой таблицы отбивается кодом `422` и текстом `CSV export is not available for this table`. Массовая выгрузка сырых визитов и конверсий делается другим инструментом — [how-to/events-exporter.md](events-exporter.md).

Если кнопки нет на таблице, которая выгрузку поддерживает, — дело не в таблице, а в отсутствующем праве `features.table-export` ([models/permissions-model.md](../models/permissions-model.md)).

### Файл из уведомления не скачивается — ссылка устарела

Скачивание из колокольчика идёт кусками, поэтому на кнопке `Download CSV` во время загрузки показывается прогресс (`Download CSV · 42%`, а если размер файла заранее неизвестен — сколько мегабайт уже скачано), а сама кнопка на это время заблокирована — это штатная работа, а не зависание.

Ссылке больше 7 дней → скачивание падает с сообщением `The download link has expired — run the export again`. Лечится повторным запуском экспорта: старая нотификация не оживёт, ссылку не продлить. Любой другой сбой скачивания даёт `Download failed` — повторить попытку, при устойчивом повторе написать в чат поддержки. Срок жизни ссылки — [reference/limits.md](../reference/limits.md).

## CSV-экспорт таблицы обрезался / выгрузка неполная

Симптом: таблица аналитики выгружена в CSV, а в файле меньше строк, чем в отчёте. Частые формулировки: «CSV из таблицы обрезался», «выгрузка неполная», «не все строки в экспорте», «пропали строки в CSV».

Причина: in-table CSV-экспорт таблицы аналитики режется на **10000 строк**. Когда данных больше, файл содержит первые 10000 строк, а в готовой нотификации приходят `truncated:true` и `rows_total` (сколько строк было на самом деле) и встаёт предупреждение `Row limit reached — some rows were not included` — по ним видно, что выгрузка неполная. Это потолок именно табличного экспорта, не отчёта. Число — в [reference/limits.md](../reference/limits.md).

Не путать с двумя другими выгрузками:

- **Events Exporter** — массовый экспорт сырых событий (сотни млн строк, окно до 31 дня). Большие выгрузки делаются им → [how-to/events-exporter.md](events-exporter.md).
- **Pivot Report API** — программный доступ, страничный: 100 строк/страница (см. [reference/glossary.md](../reference/glossary.md)).

### Статус CSV-экспорта: running / ready / failed

Экспорт таблицы асинхронный: жмёшь выгрузку — job уходит в фон, а прогресс приходит нотификацией. Три статуса в заголовке нотификации: `CSV export running` (стартовал, идёт), `CSV export ready` (готов, в payload — `file_url` со ссылкой на файл, `rows_total`, `truncated`), `CSV export failed` (упал).

При падении в нотификацию попадает абстрактный `Something went wrong during export` — реальная причина туда не пишется (уходит во внутренний трейс). Если в нотификации `CSV export failed` без деталей — это ожидаемо, конкретику разбирает команда AIO.

Ссылка на готовый файл живёт **7 дней**, после чего экспорт нужно запустить заново — [reference/limits.md](../reference/limits.md).

### Кнопки `Export as CSV` нет или выгрузка отвечает `403` — нет права на экспорт

Выгрузка гейтится **отдельным правом** `features.table-export`, и без него кнопки `Export as CSV` в таблице просто не видно: интерфейс её скрывает, а не показывает и отбивает отказом. Данные на экране при этом видны — право читать таблицу и право забрать её файлом разные.

Если запрос всё же уходит (например, из давно открытой вкладки), ответ — `403` с текстом `Table export is not allowed for your role`. Как право подписано в матрице `Settings → Positions`, кто проходит без него и как его выдать — [models/permissions-model.md](../models/permissions-model.md).

### Meta-метрики в CSV — одно число, включая orphans

Meta-метрики (`Spend`, `Impressions` и т.п.) внутри несут не одно значение, а пару «привязанное значение + orphans» (нераспределённый спенд). В CSV такая колонка выгружается **одним числом — той же суммой, что стоит в ячейке на экране**. Расходиться с интерфейсом цифра не должна; в файл не попадает только разбор этой суммы — сколько привязано и сколько ушло в orphans (в интерфейсе это подсказка ячейки) и значок статуса аллокации. Что такое orphans и почему спенд не привязался к строке — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

## Значения пользовательских полей видно колонками в таблицах визитов и конверсий

В таблицах визитов и конверсий каждое пользовательское поле показывается своей колонкой. В `Visits` (ERP и MTK) выводятся поля типов `Visit`, `Source`, `Campaign`, `Landing`, `Destination`; в `Conversions` (ERP и MTK) — те же плюс `Conversion`. Поля типа `Conversion` берут значение с самой конверсии, все остальные типы — с визита, к которому относится строка: поэтому значение, дописанное на лендинге или на `Destination`, видно и в строке конверсии.

Колонки полей всегда попадают в CSV-выгрузку таблицы, даже если скрыты на экране; архивные поля колонок не дают вовсе. Как поле заводится и какая сущность в него пишет — [how-to/user-fields.md](user-fields.md) и [models/visit-field.md](../models/visit-field.md).

## Roll Up по полям антифрода

Поля антифрода — обычные поля визита: `fraud_score` и `triggered_rules` встают в отчёт групперами `Fraud Score` и `Trigger Rules`, как любое поле с включённым `Availability as grouper` в `Settings → Fields` ([how-to/custom-fields.md](custom-fields.md)). Дерево раскрывается синими стрелками `►` или разворачивается целиком кнопкой `Unwrap tree view`.

Как читать эти значения и что с ними делать — *уточните у поддержки*.

## Смежные темы

- [reference/glossary.md](../reference/glossary.md) — Cohorts Report, Compare Analytics, Roll Up Report, Qualified Visits, Visit Loss, Attribution Event Time, External Reports, Pivot Report API, LP1 / LP2 / Scrolling метрики, Show sessions.
- *уточните у поддержки* — антифрод: поля `fraud_score` / `triggered_rules` и их настройка.
- *Антифрод / антиспам — диагностика проблем* — разбор антифрода по отчётам.
- [how-to/debug-with-logs.md](debug-with-logs.md) — Loggable UUID для конкретного визита.
- [models/conversion-model.md](../models/conversion-model.md) — что такое конверсия, postback-формат.
- [models/metric.md](../models/metric.md) — концепт: что такое Metric как сущность (3 типа Conversions count / Computable / Data feed; деньги = Data feed + source; адресация по UUID).
- [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md) — AIO Attribution Engine: как метрика `Meta Spend` аллоцируется по разрезам Roll Up report.
- [heuristics/conversion-ai-testing.md](../heuristics/conversion-ai-testing.md) — Attribution toggle единый на команду.

---

## Где Roll Up report в MTK-интерфейсе

В MTK-билде Roll Up report открывается через топ-таб **`Reports`** (`/analytics/dd`), а не через группу `Analytics → Roll Up`. Это единственный аналитический отчёт MTK: остальные отчёты вкладки Analytics (Cohorts, Comparative analytics) в MTK отсутствуют.

## Что есть в разделе Analytics — подсекции-билдеры (`/app/analytics`)

### Roll Up report — древовидный drill-down отчёт (`/analytics/dd`)

Групперы (`+` → каталог) + панель `Filters` слева + синие `►`. Каталог — тот же, что в Tracker-таблицах (см. «Каталог групперов» выше): системные категории Tracker / Location / Landings / Destinations / Variants / Client / Device / OS / Time / Browser / Other / Funnel.

> В каталоге есть и динамические per-tenant категории — **`Fields`** и категории из атрибута `Group` кастомных полей с `Availability as grouper`, а при подключённых рекламных интеграциях — **`Facebook`** / **`Google`** (см. блок про динамический per-tenant каталог выше + [how-to/custom-fields.md](custom-fields.md)).

### Cohorts report — когортный анализ (`/analytics/cohorts-report`)

Отчёт-«лесенка»: строки — когорты по датам, столбцы — прожитые периоды. Задаётся тремя селекторами когорты в тулбаре + режимами отображения справа + сайдбаром фильтров слева. Концепт (что такое когорта, зачем) — [reference/glossary.md](../reference/glossary.md) → Cohorts Report.

#### Три селектора когорты (слева в тулбаре)

- **`Size`** — размерная метрика когорты (день-0, база строки). Список = любые метрики тенанта; дефолт — `Visits` / `LP Visits`, если есть.
- **`Target`** — целевая (событийная) метрика, чью динамику по периодам смотрим. Список сужен бэком до событийных метрик; дефолт — первая доступная событийная метрика (`Revenue` / `Leads` / `Registrations` / `Installs` и т.п.).
- **`Unit`** — шаг периода: `day` / `week` / `month`.

Каждый селектор — с встроенным поиском; смена любого перезапрашивает данные.

#### Cohort не строится: ошибка про Size / Target метрику

Симптом: когортный отчёт не открывается, приходит ошибка 422 про выбранную метрику. Причина — бэк проверяет пригодность метрик когорты (фронтовый фильтр — не защита), и две метрики отбивает дословными сообщениями:

- `Size metric is not available (trash metric, or a formula depending on one)` — в селектор `Size` попала trash-метрика (или формула, зависящая от trash-метрики). Выбрать в `Size` обычную метрику (`Visits` / `LP Visits`).
- `Target metric must be an event-based metric (conversions / payout / revenue / passed) or a formula over them` — в `Target` попала не-событийная метрика. `Target` принимает только событийные метрики (conversions / payout / revenue / passed) или формулу над ними.

Проверка: сменить проблемный селектор на подходящую метрику из подсказки в тексте ошибки.

#### Режимы отображения (справа, кнопки-группы)

- **Режим значения** — `Rate` (доля от базы) / `Value` (абсолютное значение в ячейке).
- **Цветовая схема** — тепловая карта: `Single` (моно-градиент), `Gradient` (negative→warning→positive), `Positive` (один positive-тон по насыщенности).
- **База нормализации** — `By row` (внутри строки-когорты) / `Total` (по всей сетке).
- **`Skip +0`** — тогл-кнопка: исключить день +0 из нормализации цвета (день +0 = `Size` когорты, обычно доминирует и «съедает» шкалу остальных дней; rate при этом не меняется, только раскраска).

Плюс date-range пресеты (`Last month` / `Last week` / `This week` / `This month`), выбор дат и refresh.

#### Сайдбар фильтров (слева, свёрнут по умолчанию)

Metric-фильтры по доступным полям (та же панель `Table settings → Filters`, что в таблицах) — сужают когорты по значению метрики.

### Comparative analytics — сравнение двух периодов (`/analytics/compare-analytics`)

**`First time range`** vs **`Second time range`** (свой date range у каждого) × `Groupers` × `Metrics` + `Conditions` (фильтры) → кнопка **`Compare`**. Пресеты (`Select preset` / `Save` / `Save as new` / `Reset`). Условие добавляется кнопкой **`Add condition`**: выбрать `Field` → `Contained in` (`Yes`/`No`) → `Value`; несколько условий комбинируются кнопками `And`/`Or`.

### Live Pulse — живой поток событий трафика (`/analytics/live-pulse`)

`Live Pulse` (`/app/analytics/live-pulse`, право `analytics.live-pulse`, только ERP) — экран живого потока событий трафика: `init` (визит зарегистрирован), `handle` (дискретное событие флоу — submit формы или клик по handle-ссылке), `conversion` (создана конверсия). Счётчики копятся только пока страница открыта и обнуляются при перезагрузке, поэтому **сверять цифры Live Pulse с отчётами и таблицами нельзя** — это не отчёт. Полностью — [how-to/live-pulse.md](live-pulse.md).

### Session Analytics — записи сессий и сравнение карт лэндов (`/analytics/session-analytics`)

`Session Analytics` (`/app/analytics/session-analytics`, право `analytics.session-analytics`, только ERP) — записи сессий с плеером плюс сравнение scroll- и click-карт до трёх лэндов рядом. Частые формулировки: «где посмотреть запись сессии», «тепловая карта лэнда», «куда кликают», «докуда доскроллили», «сравнить два лэнда по карте». Полностью — [how-to/session-analytics.md](session-analytics.md).

### Settings → Metrics — кастомные метрики (что можно считать в отчётах)

Метрики (числовые показатели в колонках Roll Up / Cohorts / Comparative) — **настраиваемые** в `Settings → Metrics → + Metric`. Три типа: `Conversions count` (число конверсий), `Computable metric` (формула), `Data feed metric` (фид событий; денежные суммы = источник `Conversions By Type Revenue`/`Payout`) — концепт, поля форм и `Flag`-enum в [models/metric.md](../models/metric.md). Колонки списка метрик: `Metric`, `Formula` (у формульных — формула с именами метрик), `Access Type`, `Owner`, `Order`, `Main For`, `Visible`, `Created`. Набор доступных метрик в аналитике **per-tenant** — системные + заведённые кастомные.

## Tracker data-секции (Countries / Day Party / Devices / Days / OS / Browsers / Other)

В Tracker-табе есть **7 data-секций** для просмотра деталей Visits в табличном виде, независимо друг от друга: **Countries, Day Party, Devices, Days, OS, Browsers, Other**. Каждая сегментирует Visits/Conversions по заданному критерию.

Интерфейс — как у прочих Tracker-таблиц: тогглы `Traffic` (только визиты с трафиком) и `Attribute by events time` (атрибуция по времени события), `Table View` (`Presets → Customize table view` — колонки, порядок, пресеты, 10/25/50/100 строк), date-range, поиск по UUID или имени.

### Дефолтный grouper и переключение секции

Каждая секция (кроме `Other`) применяет свой group-критерий по умолчанию: `Country` для Countries, `Day` для Days и т.д. Групперы — в левом верхнем углу, меню раскрывается кликом по имени группера. **Первый выбранный grouper автоматически переключает на соответствующую секцию**: напр. группер `Device model` перебросит в `Devices`, после чего можно добавить второй группер (`Day`) для дальнейшей разбивки. `Other` стартует **без grouper** — пустая таблица, наполняется вручную выбором групперов.

### Избранные групперы

Любой grouper помечается избранным кликом по звезде рядом с ним; избранные выносятся в отдельную линию **`Favorite`** над остальными линиями каталога. Категории каталога делятся по группам (`Favorite`, `Browser`, `Facebook` и др.). Частые формулировки: «FB-групперы скрыты по дефолту», «не вижу групперы Facebook», «где разбивка по FB», «по названию креатива», «крео», «creo», «fb ad name» — категория `Facebook` сворачивается в каталоге как и прочие; разворачивается кликом по группе (плюс «+N» скрывает доп. поля, см. динамический per-tenant каталог выше).

### Action Menu Tracker data-секции

**Action Menu** (ПКМ по строке): `Build Roll Up report` (открывает Roll Up с данными строки), `Freeze top` / `Freeze bottom` (закрепить строку вверху/внизу, обратимо через `UnFreeze top` / `UnFreeze bottom`).

> Каталог групперов здесь — тот же динамический per-tenant каталог, что и в Roll Up report (см. «Древовидный отчёт через Groupers» выше).
