---
id: business-model
title: Business Model (формула Revenue/Payout) — концепт (модель)
description: Что такое Business Model в AIO как сущность — преднастроенная формула, которая по полю конверсии/постбэка пишет деньги в Revenue или Payout; восемь моделей, засеянных шаблоном тенанта (Arrived / Static / Zero / Cancel × Revenue / Payout), дефолты Arrived Revenue и Zero Payout, функции usd() и {{node}} в Formula, поля Type + Format + Formula, где создаётся (Settings) vs где применяется (Finance). НЕ путать с Business Value. Концепт; движок дистрибуций — distributions-model.md.
doc_type: model
builds: [erp]
related: [distributions-model, conversion-model, analytics, sdk, architecture, metric, meta-spend-allocation, distributions, postback-generator, glossary, ui-map]
language: ru
updated: 2026-09-23
---

# Business Model (формула Revenue/Payout) — концепт (модель)

> Модель сущности **Business Model**. Движок дистрибуций (Folder / Rule to pass / Strategy / лист) — [models/distributions-model.md](distributions-model.md) + Finance-addendum. Поля конверсии, семантика Revenue vs Payout, 4-шаговый кастомный паттерн — [models/conversion-model.md](conversion-model.md). Отображение денег в отчётах (Metrics) — [how-to/analytics.md](../how-to/analytics.md).

## Что такое Business Model и как она считает деньги

**Business Model — это преднастроенная формула расчёта, которая по своему правилу читает поле конверсии/постбэка и пишет результат в `Revenue` или `Payout` конверсии.** Сама конверсия приходит «голой» — деньги на неё навешивает Revenue/Payout Distribution, а *чем именно* считать деньги в листе дерева — это и есть выбранная Business Model. Создаётся модель в `Settings → Business models`, применяется в листе `Finance → Revenue/Payout Distribution`.

## Три сцепленные сущности: Conversion Type → Distribution → Business Model

Деньги в AIO считаются цепочкой из трёх звеньев, и путать их роли — главный источник вопросов:

- **Conversion Type** имеет **поля** (`arrived_revenue`, кастомные суммы и т.п.) — туда постбэк кладёт значения по имени поля (в постбэке пишут `<имя поля>=<значение>`).
- **Revenue/Payout Distribution** (вкладки в `Finance`) — дерево правил, которое для пришедшей конверсии **обращается к Business Model**.
- **Business Model** — по своей `Formula` читает нужное поле и **пишет результат в `Revenue` или `Payout`** конверсии.

Коротко: у конверсии есть поле → дистрибуция обращается к бизнес-моделям → бизнес-модель по формуле пишет деньги. Структуру самого дерева (папки, Rule to pass, Strategy) трогать для этого не надо — она описана в [models/distributions-model.md](distributions-model.md).

## Где создаётся ≠ где применяется Business Model

Это два разных раздела UI, и их постоянно мешают:

- **Создание:** `Settings → Business models → + Business model`. Модалка: `Name`, `Abbreviation` (опциональный короткий код для поиска, напр. `AR` для Arrived Revenue), `Color`, `Description` (короткое описание назначения модели), `Type` (`Revenue` или `Payout` — для какой стороны модель), `Format` (`Money` — фиксированная сумма, или `Percentage` — значение считается как процент по формуле), `Formula` (само правило расчёта). После заполнения — `Confirm`. Поле `Type` — **create-only**: в Edit оно недоступно, сторону модели после создания не переключить. Нужна другая сторона — создавай новую модель.
- **Применение:** `Finance → Revenue Distribution` / `Payout Distribution` (либо `Settings → Distributions`) → лист дерева → модалка `Edit revenue` / `Edit payout` → обязательное поле `Business model` (выбор из моделей тенанта) + поле суммы `Revenue`/`Payout`.

## Поле `Formula` — синтаксис (SDK-плейсхолдеры)

`Formula` пишется на SDK-плейсхолдерах ([reference/sdk.md](../reference/sdk.md)) плюс подстановка `{{node}}` и функция `usd()`. Четыре типовых шаблона — те же, что стоят в моделях, засеянных шаблоном тенанта (раздел «Какие Business Model заведены в тенанте из коробки» ниже):

- **Сумма из постбэка** — `usd({{aio.conversion.fields.arrived_revenue}})`: читает поле конверсии `arrived_revenue` и приводит его к USD. Функция `usd(...)` конвертирует число по параметру `currency=` постбэка (единственное имя функции — `usd`, других написаний формула не понимает); плейсхолдер без `usd()` берёт число как есть, без конвертации. Так устроены `Arrived Revenue` и `Arrived Payout`.
- **Сумма из листа дерева** — `{{node}}`: подставляет число, введённое в листе `Revenue/Payout Distribution` (модалка `Edit revenue` / `Edit payout` → поле `Revenue` / `Payout`). Одна такая модель обслуживает любую фиксированную ставку — сумма задаётся в дереве, а не в формуле. Так устроены `Static Revenue` и `Static Payout`.
- **Константа** — `0` (или любое число): всегда одно значение, независимо от постбэка и дерева. Так устроены `Zero Revenue` и `Zero Payout`.
- **Процент от revenue в payout-дереве** — `{{aio.conversion.revenue}} * ({{node}} / 100)`, где `{{node}}` подставляет введённую в листе дерева сумму как процент (используется с `Format = Percentage`).

`Format` модели влияет и на **отображение значения ноды в дереве**: при `Format = Money` лист дерева показывает фиксированную (статичную) сумму, при `Format = Percentage` — процент.

## Механизм по шагам (на примере конверсии Lead)

Как это проигрывается на живой конверсии в тенанте с засеянными деревьями:

1. приходит конверсия (Conversion Type, напр. `Lead`), у типа в `Revenue Settings` привязано дерево `Revenue Distribution`;
2. AIO идёт в это дерево (вкладка `Finance`);
3. папка без условий `Revenue by Postback for All Conversions` (в шаблонах iGaming — `Revenue by Postback`) ловит любую конверсию, в её листе выбрана Business Model `Arrived Revenue`;
4. модель `Arrived Revenue` берёт поле `arrived_revenue` из постбэка, приводит к USD функцией `usd()` по параметру `currency=` постбэка и записывает результат в поле конверсии `Revenue`.

То же самое для Payout-стороны — только через `Payout Distribution`, где такую же роль играет папка `Default 0$ Payout` с моделью `Zero Payout`.

### У типа не привязано дерево — payout и revenue считает первая модель списка

Если у Conversion Type не выбран `Payout Tree` / `Revenue Tree` (у нового типа поля пустые, это не обязательные поля), дерево не участвует: AIO берёт **первую по списку** Business Model нужной стороны (`Payout` / `Revenue`) из моделей тенанта и считает по её формуле. Какая из моделей окажется первой, в интерфейсе не видно и не настраивается, поэтому полагаться на этот путь нельзя — дерево типу привязывают явно.

## Какие Business Model заведены в тенанте из коробки — восемь моделей во всех шаблонах

**Тенант, созданный из любого из семи публичных шаблонов (ERP / MTK × Finance / iGaming / Nutra / Sweeps), получает один и тот же набор из восьми Business Model — по четыре на каждую сторону.** Имена и `Abbreviation` — как в `Settings → Business models`:

- **`Arrived Revenue` (`AR`)** / **`Arrived Payout` (`AP`)** — `usd({{aio.conversion.fields.arrived_revenue}})`: сумма из поля `arrived_revenue` постбэка, приведённая к USD. `Arrived Revenue` стоит в единственном включённом листе засеянного `Revenue Distribution`, поэтому это дефолт revenue-стороны.
- **`Static Revenue` (`SR`)** / **`Static Payout` (`SP`)** — `{{node}}`: фиксированная сумма, введённая в листе дерева (описание `Static Payout` в шаблонах Finance / Nutra / Sweeps — `Fixed payout set in the distribution - the usual CPL deal`, в iGaming — `Fixed payout set in the distribution node`).
- **`Zero Revenue` (`ZR`)** / **`Zero Payout` (`ZP`)** — `0`. `Zero Payout` стоит в единственном включённом листе засеянного `Payout Distribution` (папка `Default 0$ Payout`), поэтому это дефолт payout-стороны: пока дерево не перестроено, `Payout` у всех конверсий равен 0.
- **`Cancel Revenue` (`CREV`)** / **`Cancel Payout` (`CPAY`)** — `{{aio.visit.revenue}} * (-1)` / `{{aio.visit.payout}} * (-1)`: минус сумма revenue / payout всех конверсий визита на момент расчёта, то есть конверсия с этой моделью обнуляет уже засчитанные деньги визита (описание `Cancel Revenue` в шаблонах Finance / Nutra / Sweeps — `Zeroes out the revenue already counted - lead rejected by CRM`, в iGaming — `Zeroes out the revenue already counted for the visit`).

Рядом с этими папками в засеянных деревьях лежат папки-примеры: в Finance / Nutra / Sweeps — `Example: fixed revenue per lead`, `Example: zero out revenue - lead rejected by CRM`, `Example: pay the buyer what the CRM sent back` и `Example Buyer Payouts` (подпапки `Buyers FB` / `Buyers Google` с правилом по источнику и `Static Payout`); в iGaming — `Example: fixed revenue for Offer #1`, `Example: zero out revenue (corrections, rejects)`, `Example: fixed payout for Offer #1`, `Example: pay the buyer what the partner sent`. Сами папки включены, но их листы **выключены**, поэтому на расчёт они не влияют, пока лист не включить. Модели и деревья — обычные сущности тенанта: их правят и дополняют, поэтому в живом тенанте набор может отличаться от засеянного. Как устроен шаблон тенанта и что ещё из него копируется — [context/architecture.md](../context/architecture.md).

«Статичной» называют модель, чья `Formula` не зависит от постбэка: в засеянном наборе это `Static Revenue` / `Static Payout` с `{{node}}` (сумма берётся из листа дерева), константа прямо в `Formula` (напр. `100`) — тоже вариант. Это **не тип и не режим в `Settings`**, а описательный ярлык: создаётся такая модель обычным набором `Type` + `Format` + `Formula`.

## Статичная vs динамическая Business Model — чем отличаются Formula

«Статичная» и «динамическая» — это **не режимы создания и не пункты дропдауна в `Settings`**, а описательные категории того, *откуда модель берёт сумму*. Создаётся любая модель одинаково — `Type` + `Format` + `Formula`; различает их только то, что стоит в `Formula`:

- **Статичная** — сумма не зависит от постбэка. Готовый вариант уже засеян: `Static Revenue` / `Static Payout` с `Formula` = `{{node}}` — ставка вводится в листе дерева, одна модель на любое число ставок. Второй вариант — **константа** прямо в `Formula` (напр. `100`), тогда под каждую ставку нужна своя модель. Применяют, когда рекламодатель не передаёт ставку или передаёт некорректно: папку с правилом (напр. `Conversion Type = X AND Destination = рекламодатель`) заводят в Distribution, в листе выбирают статичную модель и вписывают сумму. В засеянных деревьях такая папка уже есть как выключенный пример: в шаблонах Finance / Nutra / Sweeps — `Example: fixed revenue per lead` (`Static Revenue`, правило `Conversion Type = Lead`), в iGaming — `Example: fixed revenue for Offer #1` (правило по Destination).
- **Динамическая** — `Formula` содержит **плейсхолдер поля конверсии** (напр. `usd({{aio.conversion.fields.<имя_поля>}})`), и значение читается из этого поля. По сути маппинг «поле постбэка → Payout/Revenue»; засеянные примеры — `Arrived Revenue` / `Arrived Payout`.

Полный кастомный паттерн (на примере суммы платежа) требует увязать 4 элемента — поле конверсии, Conversion Type (обязательно **Multiple**, не Single, если сумм по визиту несколько), Business Model и правило в Payout Distribution; без любого из них значение **видно в конверсии, но в Payout не пишется**. Пошаговая процедура — [models/conversion-model.md](conversion-model.md).

## Семантика стороны жёсткая: Revenue ≠ Payout

`Type` модели (Revenue / Payout) — это не косметика, а смысл денег:

- **`Revenue`** = деньги, которые **получаем** (наша комиссия от рекламодателя, наш доход).
- **`Payout`** = деньги, которые **платим** / прочие денежные суммы события (напр. сумма платежа, которую внёс покупатель у рекламодателя).

Сумму платежа покупателя кладут в **Payout**, а не в Revenue, чтобы не путать «сколько заработали» и «сколько прошло через событие». Положить сумму платежа в `arrived_revenue` нельзя — AIO посчитает это доходом, и ROI/метрики взорвутся в плюс. Подробно про поля и семантику — [models/conversion-model.md](conversion-model.md).

## Записать деньги ≠ показать их в аналитике (Business Model vs Metric)

Business Model пишет деньги **в поле конверсии**. Чтобы это значение появилось отдельной колонкой/средним чеком в отчётах — нужна **отдельная Metric** поверх поля (`Settings → Metrics → + Metric`). Для сумм revenue/payout берётся `Data feed metric` с источником `Conversions By Type Revenue` / `Conversions By Type Payout` (не `Conversions count`, который считает только число конверсий), а для производных вроде среднего чека — `Computable metric` с форматом `Money`; типы метрик — [models/metric.md](metric.md). У метрики есть переключатель атрибуции: по времени визита vs по времени события (конверсии). Это отдельный шаг — частая путаница «записалось в конверсию, но не вижу в листах кампаний». Детали — [how-to/analytics.md](../how-to/analytics.md).

## Как Business Model связана с остальными сущностями AIO

- **Conversion Type → имеет поля** (`arrived_revenue`, кастомные суммы…), куда постбэк кладёт значения; Business Model эти поля читает ([models/conversion-model.md](conversion-model.md)).
- **Revenue/Payout Distribution → выбирает Business Model** в листе (модалка `Edit revenue`/`Edit payout`); движок дерева — [models/distributions-model.md](distributions-model.md).
- **Business Model → пишет `Revenue`/`Payout`** конверсии по своей `Formula`.
- **Metric** (`Settings → Metrics`) **агрегирует** Revenue/Payout в аналитику через `Data feed metric` с денежным источником (`Conversions By Type Revenue` / `Conversions By Type Payout`) ([how-to/analytics.md](../how-to/analytics.md), типы — [models/metric.md](metric.md)).
- **Profit = `Revenue − Cost`, не `− Payout`** — формула прибыли и что входит в Cost (рекламный спенд) — *Медиабаинг и место AIO в нём*. По FB основной способ увидеть Cost и профит — метрика `Meta Spend` в отчётах (считается на лету в момент отчёта, revenue стоит рядом) — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).
- **Tracker / Source-постбэки** доставляют значения полей, которые читает модель.
- **ОТДЕЛЬНО от Business Value** (атрибут Conversion Type, AI-оптимизация) — НЕ путать (см. ниже).

## Чем Business Model отличается от смежных понятий

### Business Model vs Business Value — разные сущности

Business Model — денежная **формула** (Revenue/Payout по `Type` Revenue\|Payout, `Format` Money\|Percentage, `Formula`), per-tenant, в `Settings → Business models`. Business Value — атрибут Conversion Type (Good/Neutral/Bad), влияет на **Reward в AI-оптимизации**, к деньгам отношения не имеет. Похожие названия, разные сущности.

### Revenue (сторона) vs Payout (сторона) — семантика стороны

Revenue = деньги, которые МЫ получаем (наша комиссия, доход). Payout = деньги, которые МЫ платим / прочие денежные суммы события (сумма платежа покупателя). Сумму платежа в Revenue класть нельзя — ломает ROI (прибыль = `Revenue − Cost`, а не `− Payout`).

### Zero Payout vs Arrived Revenue — дефолтные модели тенанта

Zero Payout — всегда 0 (в `Formula` записан `0`), стоит в единственном включённом листе засеянного `Payout Distribution` (папка `Default 0$ Payout`). Arrived Revenue — `usd({{aio.conversion.fields.arrived_revenue}})`: приводит `arrived_revenue` к USD и пишет в Revenue, стоит в единственном включённом листе засеянного `Revenue Distribution` (папка `Revenue by Postback for All Conversions`, в iGaming — `Revenue by Postback`). Остальные шесть засеянных моделей (Static / Zero / Cancel обеих сторон и Arrived Payout) в деревьях по умолчанию не участвуют. «Статичная»/«динамическая» — описательные ярлыки по содержимому `Formula` (`{{node}}` или константа vs плейсхолдер поля), а не пункты UI (см. выше).

### Где создаётся vs где применяется Business Model

Создаётся в `Settings → Business models`. Применяется (выбирается) в листе `Finance → Revenue/Payout Distribution` (модалка Edit revenue/payout). Два разных раздела UI.

### Business Model (запись денег) vs Metric (аналитика)

Business Model пишет деньги в поле конверсии. Чтобы увидеть их колонкой/средним чеком в отчётах — нужна отдельная Metric (`Settings → Metrics`). Запись и отображение — два разных шага.

### Business Model vs поля Revenue/Reward на Conversion Type

У Conversion Type есть статичные поля (Revenue + Reward, оставляют пустыми если нет выплаты). Business Model — это **формула расчёта в дистрибуции**, а не статичное поле на типе.

## Подводные камни Business Model — частые ошибки настройки

### Конверсия пришла, но Revenue/Payout = 0

Payout у любого типа, привязанного к засеянному `Payout Distribution`, считает папка `Default 0$ Payout` → `Zero Payout`; у типа без привязанного дерева — первая Payout-модель списка тенанта. И то и другое даёт «конверсия пришла, но Payout = 0». Revenue при этом считается: включённая папка `Revenue by Postback for All Conversions` (в iGaming — `Revenue by Postback`) → `Arrived Revenue` читает `arrived_revenue` постбэка, поэтому `Revenue = 0` значит, что постбэк пришёл без `arrived_revenue` или тип не привязан к дереву (и первой Revenue-моделью списка оказалась `Zero Revenue`). Лечится папкой с правилом в Distribution и своей Business Model. См. *уточните у поддержки*.

### Постбэк с `revenue=` выключает расчёт по бизнес-модели

Симптом: формула настроена и выглядит рабочей, а `Revenue` в конверсиях ей не соответствует. Параметр постбэка `revenue=<сумма>` пишется в `Revenue` конверсии как есть, и для такой конверсии формула Business Model не применяется вовсе. Параметр `payout=` так не работает — `Payout` всё равно считается по формуле. Разбор параметров постбэка и штатный способ принять сумму формулой (`arrived_revenue=`) — [models/conversion-model.md](conversion-model.md).

### Формула содержит `0` — всегда пишет 0

Формула содержит `0` (как Zero Payout) → всегда пишет 0, даже если поле постбэка заполнено. Проверять `Formula` модели.

### Значение видно в конверсии, но не пишется в Payout/Revenue

Не настроена Business Model или не привязана в Distribution (пропущен один из 4 шагов кастомного паттерна).

### Серийная сумма с Uniqueness = Single теряет серию

Серийная конверсия (несколько сумм на один визит) с Uniqueness = Single → создаётся одна конверсия на визит, серия теряется. Для серии сумм нужен Multiple ([models/conversion-model.md](conversion-model.md)).

### Сумма платежа в `arrived_revenue` ломает ROI

Сумма платежа в `arrived_revenue` (вместо отдельного поля + Payout) → AIO считает это доходом, ROI/метрики взрываются в плюс. Разделять Revenue (наша комиссия) и Payout (сумма события).

### Изменили модель/правило Distribution — применяется только к новым конверсиям

Изменили модель/правило Distribution — применяется только к **новым** конверсиям (по дате конверсии); старые остаются с прежним значением.

### Деньги записались, но в листах кампаний/аналитике не видно

Забыли завести Metric (`Settings → Metrics`) поверх поля, или неверный переключатель атрибуции (по времени визита vs по времени события). Computable-метрика (средний чек) даёт странное число → неверный `Format` (надо `Money`) или деление не на ту метрику.

### Business Model — per-tenant: нельзя предполагать наличие конкретной модели

Business Model — per-tenant: шаблон засевает восемь моделей (раздел «Какие Business Model заведены в тенанте из коробки»), но их правят и удаляют, поэтому в живом тенанте наличие конкретной модели проверяют в `Settings → Business models`, а не предполагают.

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
