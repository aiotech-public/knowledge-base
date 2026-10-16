---
id: business-model
title: Business Model (формула Revenue/Payout) — концепт (модель)
description: Что такое Business Model в AIO как сущность — формула расчёта Revenue или Payout конверсии; восемь моделей, засеянных шаблоном тенанта (Arrived / Static / Zero / Cancel × Revenue / Payout), дефолты Arrived Revenue и Zero Payout, функции usd() и {{node}} в Formula, поля Type + Format + Formula, где создаётся (Settings) vs где применяется (Finance). НЕ путать с Business Value. Концепт; движок дистрибуций — distributions-model.md.
doc_type: model
builds: [erp, mtk]
related: [distributions-model, conversion-model, analytics, sdk, architecture, metric, meta-spend-allocation, distributions, postback-generator, glossary, ui-map]
language: ru
updated: 2026-10-09
---

# Business Model (формула Revenue/Payout) — концепт (модель)

> Модель сущности **Business Model**. Движок дистрибуций (Folder / Rule to pass / Strategy / лист) — [models/distributions-model.md](distributions-model.md) + Finance-addendum. Поля конверсии, семантика Revenue vs Payout, 4-шаговый кастомный паттерн — [models/conversion-model.md](conversion-model.md). Отображение денег в отчётах (Metrics) — [how-to/analytics.md](../how-to/analytics.md).

## Что такое Business Model и как она считает деньги

**Business Model — формула расчёта `Revenue` или `Payout` конверсии.** Она может использовать поля конверсии и визита, значение листа Distribution или константу. В Revenue/Payout Distribution выбирают модель для подходящего листа; при отсутствии дерева используется отдельный fallback. Присланная готовая `Revenue` может обойти расчёт Revenue-модели; условия и отличие `Payout` разобраны ниже.

Создаётся модель в `Settings → Business models`, применяется в листе `Finance → Revenue/Payout Distribution`.

## Три сцепленные сущности: Conversion Type → Distribution → Business Model

Деньги в AIO считаются цепочкой из трёх звеньев, и путать их роли — главный источник вопросов:

- **Conversion Type** имеет **поля** (`arrived_revenue`, кастомные суммы и т.п.) — туда постбэк кладёт значения по имени поля (в постбэке пишут `<имя поля>=<значение>`).
- **Revenue/Payout Distribution** — дерево правил, которое для пришедшей конверсии **обращается к Business Model**.
- **Business Model** — по своей `Formula` рассчитывает **`Revenue` или `Payout`** конверсии; для суммы из постбэка формула читает нужное поле.

Для расчёта суммы из поля: у конверсии есть поле → дистрибуция выбирает бизнес-модель → её формула читает значение поля и пишет деньги. Структуру самого дерева (папки, Rule to pass, Strategy) трогать для этого не надо — она описана в [models/distributions-model.md](distributions-model.md).

Деревья `Revenue Distribution` и `Payout Distribution` — вкладки раздела `Finance`.

## Где создаётся ≠ где применяется Business Model

Это два разных раздела UI, и их постоянно мешают:

- **Создание:** `Settings → Business models → + Business model`. Модалка: `Name`, `Abbreviation` (обязательный короткий код, напр. `AR` для Arrived Revenue), `Color`, `Description` (короткое описание назначения модели), `Type` (для расчёта денег конверсии выбирают `Revenue` или `Payout`), `Format` (`Money` или `Percentage` — отображение значения листа; сам расчёт задаёт `Formula`), `Formula` (само правило расчёта). После заполнения — `Confirm`. Поле `Type` — **create-only**: в Edit оно недоступно, сторону модели после создания не переключить. Нужна другая сторона — создавай новую модель.
- **Применение:** `Finance → Revenue Distribution` / `Payout Distribution` (либо `Settings → Distributions`) → лист дерева → модалка `Edit revenue` / `Edit payout` → обязательное поле `Business model` (выбор из моделей тенанта) + поле суммы `Revenue`/`Payout`.

В MTK-виде Business Model не создаются и не правятся: в нём нет ни `Settings → Business models`, ни раздела `Finance`. Восемь моделей и деревья `Revenue Distribution` / `Payout Distribution` приходят из шаблона тенанта и считают деньги в фоне — состав в разделе «Какие Business Model заведены в тенанте из коробки». Ставку и выбор существующей модели можно менять в доступном денежном дереве раздела `Distributions`, если включён этот модуль, дерево доступно в MTK и есть права просмотра и редактирования. Для новой Business Model нужна настройка со стороны команды AIO.

## Поле `Formula` — синтаксис (SDK-плейсхолдеры)

`Formula` пишется на SDK-плейсхолдерах ([reference/sdk.md](../reference/sdk.md)) плюс подстановка `{{node}}` и функция `usd()`. Ниже — примеры формул. Сумма из поля, `{{node}}` и `0` используются в засеянных моделях; процентный пример — отдельная формула.

- **Сумма из постбэка** — `usd({{aio.conversion.fields.arrived_revenue}})`: читает поле конверсии `arrived_revenue` и приводит его к USD. Функция `usd(...)` конвертирует число по параметру `currency=` постбэка (функция конвертации валюты называется именно `usd`); плейсхолдер без `usd()` берёт число как есть, без конвертации. Так устроены `Arrived Revenue` и `Arrived Payout`.
- **Сумма из листа дерева** — `{{node}}`: подставляет число, введённое в листе `Revenue/Payout Distribution` (модалка `Edit revenue` / `Edit payout` → поле `Revenue` / `Payout`). Одна такая модель обслуживает любую фиксированную ставку — сумма задаётся в дереве, а не в формуле. Так устроены `Static Revenue` и `Static Payout`.
- **Константа** — `0` (или любое число): всегда одно значение, независимо от постбэка и дерева. Так устроены `Zero Revenue` и `Zero Payout`.
- **Процент от revenue в payout-дереве** — `{{aio.conversion.revenue}} * ({{node}} / 100)`, где `{{node}}` подставляет введённую в листе дерева сумму как процент (используется с `Format = Percentage`).

`Format` модели задаёт **формат отображения значения листа дерева**: `Money` — денежный формат, `Percentage` — процент. Фиксированную или динамическую сумму рассчитывает `Formula`.

## Механизм по шагам (на примере конверсии Lead)

Пример расчёта по модели в тенанте с засеянными деревьями, если постбэк не передал готовую Revenue, обходящую формулу:

1. приходит конверсия (Conversion Type, напр. `Lead`), у типа в `Revenue settings` привязано дерево `Revenue Distribution`;
2. AIO идёт в это дерево;
3. папка без условий `Revenue by Postback for All Conversions` (в шаблонах iGaming — `Revenue by Postback`) ловит любую конверсию, в её листе выбрана Business Model `Arrived Revenue`;
4. модель `Arrived Revenue` берёт поле `arrived_revenue` из постбэка, приводит к USD функцией `usd()` по параметру `currency=` постбэка и записывает результат в поле конверсии `Revenue`.

То же самое для Payout-стороны — только через `Payout Distribution`, где такую же роль играет папка `Default 0$ Payout` с моделью `Zero Payout`.

Дерево из шага 2 открывается во вкладке `Finance`.

### У типа не привязано дерево — используется первая найденная модель нужной стороны

Если у Conversion Type не выбраны `Payout settings` / `Revenue settings` (у нового типа поля пустые и необязательны), AIO берёт **первую найденную** Business Model нужной стороны из моделей тенанта. Для Revenue этот fallback используется, если готовая Revenue не обошла формулу. Отдельного выбора fallback-модели нет, поэтому дерево типу привязывают явно.

## Какие Business Model заведены в тенанте из коробки — восемь моделей во всех шаблонах

**Тенант, созданный из любого из семи публичных шаблонов (ERP: Finance / iGaming / Nutra / Sweeps; MTK: Finance / iGaming / Sweeps), получает один и тот же набор из восьми Business Model — по четыре на каждую сторону.** Имена и `Abbreviation` моделей:

- **`Arrived Revenue` (`AR`)** / **`Arrived Payout` (`AP`)** — `usd({{aio.conversion.fields.arrived_revenue}})`: сумма из поля `arrived_revenue` постбэка, приведённая к USD. `Arrived Revenue` стоит в единственном включённом листе засеянного `Revenue Distribution`, поэтому это дефолт revenue-стороны.
- **`Static Revenue` (`SR`)** / **`Static Payout` (`SP`)** — `{{node}}`: фиксированная сумма, введённая в листе дерева (описание `Static Payout` в шаблонах Finance / Nutra / Sweeps — `Fixed payout set in the distribution - the usual CPL deal`, в iGaming — `Fixed payout set in the distribution node`).
- **`Zero Revenue` (`ZR`)** / **`Zero Payout` (`ZP`)** — `0`. `Zero Payout` стоит в единственном включённом листе засеянного `Payout Distribution` (папка `Default 0$ Payout`), поэтому это дефолт payout-стороны: пока дерево не перестроено, `Payout` у всех конверсий равен 0.
- **`Cancel Revenue` (`CREV`)** / **`Cancel Payout` (`CPAY`)** — `{{aio.visit.revenue}} * (-1)` / `{{aio.visit.payout}} * (-1)`: минус сумма revenue / payout всех конверсий визита на момент расчёта, то есть конверсия с этой моделью обнуляет уже засчитанные деньги визита (описание `Cancel Revenue` в шаблонах Finance / Nutra / Sweeps — `Zeroes out the revenue already counted - lead rejected by CRM`, в iGaming — `Zeroes out the revenue already counted for the visit`).

Под этими именами и `Abbreviation` модели видны в `Settings → Business models`.

Рядом с этими папками в засеянных деревьях лежат папки-примеры: в Finance / Nutra / Sweeps — `Example: fixed revenue per lead`, `Example: zero out revenue - lead rejected by CRM`, `Example: pay the buyer what the CRM sent back` и `Example Buyer Payouts` (подпапки `Buyers FB` / `Buyers Google` с правилом по источнику и `Static Payout`); в iGaming — `Example: fixed revenue for Offer #1`, `Example: zero out revenue (corrections, rejects)`, `Example: fixed payout for Offer #1`, `Example: pay the buyer what the partner sent`. Сами папки включены, но их листы **выключены**, поэтому на расчёт они не влияют, пока лист не включить. Модели и деревья — обычные сущности тенанта: их правят и дополняют, поэтому в живом тенанте набор может отличаться от засеянного. Как устроен шаблон тенанта и что ещё из него копируется — [context/architecture.md](../context/architecture.md).

«Статичной» называют модель, чья `Formula` не зависит от постбэка: в засеянном наборе это `Static Revenue` / `Static Payout` с `{{node}}` (сумма берётся из листа дерева), константа прямо в `Formula` (напр. `100`) — тоже вариант. Это **не тип и не режим в `Settings`**, а описательный ярлык: создаётся такая модель обычным набором `Type` + `Format` + `Formula`.

## Статичная vs динамическая Business Model — чем отличаются Formula

«Статичная» и «динамическая» — это **не режимы создания и не пункты дропдауна в `Settings`**, а описательные категории того, *откуда модель берёт сумму*. Создаётся любая модель одинаково — `Type` + `Format` + `Formula`; различает их только то, что стоит в `Formula`:

- **Статичная** — сумма не зависит от постбэка. Готовый вариант уже засеян: `Static Revenue` / `Static Payout` с `Formula` = `{{node}}` — ставка вводится в листе дерева, одна модель на любое число ставок. Второй вариант — **константа** прямо в `Formula` (напр. `100`), тогда под каждую ставку нужна своя модель. Применяют, когда рекламодатель не передаёт ставку или передаёт некорректно: папку с правилом (напр. `Conversion Type = X AND Destination = рекламодатель`) заводят в Distribution, в листе выбирают статичную модель и вписывают сумму. В засеянных деревьях такая папка уже есть как выключенный пример: в шаблонах Finance / Nutra / Sweeps — `Example: fixed revenue per lead` (`Static Revenue`, правило `Conversion Type = Lead`), в iGaming — `Example: fixed revenue for Offer #1` (правило по Destination).
- **Динамическая** — `Formula` содержит **плейсхолдер поля конверсии** (напр. `usd({{aio.conversion.fields.<имя_поля>}})`), и значение читается из этого поля. По сути маппинг «поле постбэка → Payout/Revenue»; засеянные примеры — `Arrived Revenue` / `Arrived Payout`.

Для суммы из отдельного поля удобно увязать четыре элемента: поле конверсии, Conversion Type (один из вариантов **Multiple**, если нужно сохранить несколько событий на визит), Business Model и подходящий лист Payout Distribution. Если поле заполнено, а Payout неверен, проверяют выбранную модель, формулу и условие листа. Пошаговая процедура — [models/conversion-model.md](conversion-model.md).

## Семантика стороны жёсткая: Revenue ≠ Payout

`Type` модели (Revenue / Payout) — это не косметика, а смысл денег:

- **`Revenue`** = деньги, которые **получаем** (наша комиссия от рекламодателя, наш доход).
- **`Payout`** = деньги, которые **платим** / прочие денежные суммы события (напр. сумма платежа, которую внёс покупатель у рекламодателя).

Сумму платежа покупателя кладут в **Payout**, а не в Revenue, чтобы не путать «сколько заработали» и «сколько прошло через событие». Для суммы платежа рекомендуется отдельное поле и расчёт Payout: `arrived_revenue` с моделью Arrived Revenue будет посчитан как доход и исказит ROI. Подробно про поля и семантику — [models/conversion-model.md](conversion-model.md).

## Записать деньги ≠ показать их в аналитике (Business Model vs Metric)

Business Model пишет деньги **в поле конверсии**. В отчётах это значение показывает подходящая **Metric**. Используют существующую метрику или создают отдельную для нужного типа конверсии/среднего чека; создание Business Model само по себе не добавляет колонку отчёта. Детали — [how-to/analytics.md](../how-to/analytics.md).

Metric заводится в `Settings → Metrics → + Metric`. Для сумм revenue/payout берётся `Data feed metric` с источником `Conversions By Type Revenue` / `Conversions By Type Payout` (не `Conversions count`, который считает только число конверсий), а для производных вроде среднего чека — `Computable metric` с форматом `Money`; типы метрик — [models/metric.md](metric.md). В отчёте переключают атрибуцию по времени визита или события (конверсии).

В MTK-виде метрики не создаются: колонки отчёта приходят из шаблона тенанта, новую метрику по запросу заводит команда AIO.

## Как Business Model связана с остальными сущностями AIO

- **Conversion Type → имеет поля** (`arrived_revenue`, кастомные суммы…), куда постбэк кладёт значения; Business Model эти поля читает ([models/conversion-model.md](conversion-model.md)).
- **Revenue/Payout Distribution → выбирает Business Model** в листе (модалка `Edit revenue`/`Edit payout`); движок дерева — [models/distributions-model.md](distributions-model.md).
- **Business Model → пишет `Revenue`/`Payout`** конверсии по своей `Formula`.
- **Profit = `Revenue − Cost`, не `− Payout`** — формула прибыли и что входит в Cost (рекламный спенд) — *Медиабаинг и место AIO в нём*. По FB основной способ увидеть Cost и профит — метрика `Meta Spend` в отчётах (считается на лету в момент отчёта, revenue стоит рядом) — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).
- **Входящий постбэк** заполняет поля для формулы. **Tracker** передаёт уже рассчитанную конверсию во внешнюю систему.

- **Metric** (`Settings → Metrics`) **агрегирует** Revenue/Payout в аналитику через `Data feed metric` с денежным источником (`Conversions By Type Revenue` / `Conversions By Type Payout`) ([how-to/analytics.md](../how-to/analytics.md), типы — [models/metric.md](metric.md)).
- **ОТДЕЛЬНО от Business Value** (атрибут Conversion Type, AI-оптимизация) — НЕ путать (см. ниже).

## Чем Business Model отличается от смежных понятий

### Business Model vs Business Value — разные сущности

Business Model — денежная **формула** (Revenue/Payout по `Type` Revenue\|Payout, `Format` Money\|Percentage, `Formula`), per-tenant, в `Settings → Business models`. Business Value — атрибут Conversion Type (Good/Neutral/Bad), влияет на **Reward в AI-оптимизации**, к деньгам отношения не имеет. Похожие названия, разные сущности.

### Revenue (сторона) vs Payout (сторона) — семантика стороны

Revenue = деньги, которые МЫ получаем (наша комиссия, доход). Payout = деньги, которые МЫ платим / прочие денежные суммы события (сумма платежа покупателя). Сумму платежа рекомендуется записывать в Payout, чтобы не искажать ROI (прибыль = `Revenue − Cost`, а не `− Payout`).

### Zero Payout vs Arrived Revenue — дефолтные модели тенанта

Zero Payout — всегда 0 (в `Formula` записан `0`), стоит в единственном включённом листе засеянного `Payout Distribution` (папка `Default 0$ Payout`). Arrived Revenue — `usd({{aio.conversion.fields.arrived_revenue}})`: приводит `arrived_revenue` к USD и пишет в Revenue, стоит в единственном включённом листе засеянного `Revenue Distribution` (папка `Revenue by Postback for All Conversions`, в iGaming — `Revenue by Postback`). Остальные шесть засеянных моделей (Static Revenue/Payout, Zero Revenue, Cancel Revenue/Payout и Arrived Payout) в деревьях по умолчанию не участвуют. «Статичная»/«динамическая» — описательные ярлыки по содержимому `Formula` (`{{node}}` или константа vs плейсхолдер поля), а не пункты UI (см. выше).

### Где создаётся vs где применяется Business Model

Создаётся в `Settings → Business models`. Применяется (выбирается) в листе `Finance → Revenue/Payout Distribution` (модалка Edit revenue/payout). Два разных раздела UI.

### Business Model (запись денег) vs Metric (аналитика)

Business Model пишет деньги в поле конверсии. В отчётах их показывает подходящая Metric; если нужной ещё нет, её создают в `Settings → Metrics`. Запись и отображение — два разных шага.

### Business Model vs поля Revenue/Reward на Conversion Type

В настройках Conversion Type выбирают `Revenue settings` / `Payout settings` для денежных деревьев. `Business Value` и `Reward weight` задают награду AI-оптимизации. Business Model рассчитывает деньги конверсии по формуле.

## Подводные камни Business Model — частые ошибки настройки

### Конверсия пришла, но Revenue/Payout = 0

Payout у любого типа, привязанного к засеянному `Payout Distribution`, считает папка `Default 0$ Payout` → `Zero Payout`; у типа без привязанного дерева — первая найденная Payout-модель тенанта. Zero Payout даёт 0; результат первой модели при отсутствии дерева зависит от её Formula. Revenue при этом считается: включённая папка `Revenue by Postback for All Conversions` (в iGaming — `Revenue by Postback`) → `Arrived Revenue` читает `arrived_revenue` постбэка. При `Revenue = 0` проверяют значение `arrived_revenue`, выбранную модель и привязку дерева. Пустое поле, формула `0` или ошибка расчёта могут дать нулевой результат.

Лечится папкой с правилом в Distribution и своей Business Model. См. *уточните у поддержки*.

В MTK Business Model не редактируют. Ставку и выбранную существующую модель можно менять в доступном денежном дереве `Distributions` при включённом модуле и необходимых правах; разбор.

### Готовая Revenue из постбэка может обойти Revenue-модель

Симптом: формула настроена и выглядит рабочей, а `Revenue` в конверсиях ей не соответствует. Готовая `Revenue` из параметра `revenue=` принимается напрямую, если значение не считается пустым. Например, `revenue=0.00` передаёт готовый ноль, а `revenue=0` оставляет расчёт по модели. Пропускается только Revenue-модель: Payout рассчитывается отдельно. Параметр `payout=` не заменяет этот расчёт. Разбор параметров постбэка и штатный способ принять сумму формулой (`arrived_revenue=`) — [models/conversion-model.md](conversion-model.md).

### Formula равна `0` — модель рассчитывает ноль

Если вся Formula равна `0` (как у Zero Payout), результат расчёта модели — 0, даже если поле постбэка заполнено. Проверять `Formula` модели.

### Значение видно в конверсии, но не пишется в Payout/Revenue

Проверьте, какая Business Model выбрана, читает ли её Formula нужное поле и подходит ли условие листа Distribution. Если дерево не привязано, проверьте fallback-модель.

### Серийная сумма с Uniqueness = Single теряет серию

Серийная конверсия (несколько сумм на один визит) с Uniqueness = Single → создаётся одна конверсия на визит, серия теряется. Для серии сумм нужен Multiple ([models/conversion-model.md](conversion-model.md)).

### Сумма платежа в `arrived_revenue` ломает ROI

Сумма платежа в `arrived_revenue` (вместо отдельного поля + Payout) → AIO считает это доходом, ROI/метрики взрываются в плюс. Разделять Revenue (наша комиссия) и Payout (сумма события).

### Изменили модель/правило Distribution — когда используется новая формула

Изменение модели или правила само по себе не пересчитывает уже созданные конверсии. Новая конверсия считается при создании по текущим настройкам, даже если передана старая дата события. Повторный постбэк с настроенным `Correction type` может заново рассчитать Revenue и записать разницу отдельной конверсией; ручное изменение денег — отдельные экшены `Update Revenue` / `Update Payout` ([models/conversion-model.md](conversion-model.md)).

### Деньги записались, но в листах кампаний/аналитике не видно

Забыли завести Metric (`Settings → Metrics`) поверх поля, или неверный переключатель атрибуции (по времени визита vs по времени события). Если средний чек странно отображается, проверьте формат `Money`; если неверна сама сумма — формулу деления и выбранные метрики.

В MTK-виде метрики самостоятельно не заводятся — недостающую колонку отчёта по запросу добавляет команда AIO.

### Business Model — per-tenant: нельзя предполагать наличие конкретной модели

Business Model — per-tenant: шаблон засевает восемь моделей (раздел «Какие Business Model заведены в тенанте из коробки»), но их редактируют и дополняют, поэтому в живом тенанте наличие конкретной модели проверяют в `Settings → Business models`, а не предполагают.

## Куда идти за деталями Business Model — смежные темы

- **Движок дистрибуций** (Folder / Rule to pass / Strategy / лист, Revenue/Payout Distribution, экшены ноды) → [models/distributions-model.md](distributions-model.md) (родитель-движок).
- **Поля конверсии, семантика Revenue vs Payout, 4-шаговый кастомный паттерн, Uniqueness** → [models/conversion-model.md](conversion-model.md).
- **Траблшутинг «деньги не считаются»** (Zero Payout) → *уточните у поддержки*.
- **Отображение денег в отчётах** (Settings → Metrics: `Data feed metric` с источником `Conversions By Type Revenue`/`Payout`, `Computable`) → [how-to/analytics.md](../how-to/analytics.md).
- **Пошаговые процедуры создания Revenue/Payout Distribution** → [how-to/distributions.md](../how-to/distributions.md).
- **Собрать ссылку постбэка под модель** (в селекте генератора только модели типа `Revenue`, поля формулы обязательны, превью суммы) → [how-to/postback-generator.md](../how-to/postback-generator.md).
- **Profit = Revenue − Cost, payout-модели медиабаинга** → *Медиабаинг и место AIO в нём*.
- **Термины** (Business Model / Arrived Revenue / Zero Payout / Business Value) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (`Settings → Business models`; применение — `Finance → Revenue/Payout Distribution`).
