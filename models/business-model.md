---
id: business-model
title: Business Model (формула Revenue/Payout) — концепт (модель)
description: Что такое Business Model в AIO как сущность — преднастроенная формула, которая по полю конверсии/постбэка пишет деньги в Revenue или Payout; дефолты Arrived Revenue$ / Zero Payout, поля Type + Format + Formula, где создаётся (Settings) vs где применяется (Finance). НЕ путать с Business Value. Концепт; движок дистрибуций — distributions-model.md.
doc_type: model
builds: [erp]
related: [distributions-model, conversion-model, analytics, sdk, metric, meta-spend-allocation, distributions, postback-generator, glossary, ui-map]
language: ru
updated: 2026-08-11
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

`Formula` пишется на SDK-плейсхолдерах ([reference/sdk.md](../reference/sdk.md)). Три типовых шаблона:

- **Arrived Revenue из постбэка** — `{{aio.conversion.fields.arrived_revenue}}` (читает поле `arrived_revenue`, см. раздел «Дефолтные модели тенанта: Arrived Revenue$ и Zero Payout» ниже).
- **Статичное значение** (напр. Zero Payout) — фиксированное число (`0` или любая константа).
- **Процент от revenue в payout-дереве** — `{{aio.conversion.revenue}} * ({{node}} / 100)`, где `{{node}}` подставляет введённую в листе дерева сумму как процент (используется с `Format = Percentage`).

`Format` модели влияет и на **отображение значения ноды в дереве**: при `Format = Money` лист дерева показывает фиксированную (статичную) сумму, при `Format = Percentage` — процент.

## Механизм по шагам (на примере конверсии Purchase)

Как это проигрывается на живой конверсии:

1. приходит конверсия (Conversion Type, напр. `Purchase`);
2. AIO идёт в **Revenue Distribution** (вкладка `Finance`);
3. дефолтная папка `Accept everything` применяет Business Model `Arrived Revenue`;
4. модель `Arrived Revenue` берёт поле `arrived_revenue` из постбэка, конвертит в USD по текущему курсу и записывает в поле конверсии `Revenue`.

То же самое для Payout-стороны — только через Payout Distribution и Payout-формулу.

## Дефолтные модели тенанта: Arrived Revenue$ и Zero Payout

По дефолту в тенанте есть (нельзя предполагать наличие *других* моделей — набор per-tenant):

- **`Arrived Revenue$`** — берёт `arrived_revenue` из постбэка, конвертит в USD, пишет в `Revenue`. Дефолт ревенью-стороны.
- **`Zero Payout`** — всегда возвращает `Payout = 0` (в формуле просто записан `0`). Fallback «ничего не платим».

«Статичной» называют модель, чья `Formula` — фиксированное число (напр. `100` вместо плейсхолдера поля), без зависимости от постбэка (напр. фиксированная ставка за целевое действие). Это **не дефолт и не тип/режим в `Settings`**, а описательный ярлык: создаётся такая модель обычным набором `Type` + `Format` + `Formula`, где в `Formula` стоит константа.

## Статичная vs динамическая Business Model — чем отличаются Formula

«Статичная» и «динамическая» — это **не режимы создания и не пункты дропдауна в `Settings`**, а описательные категории того, *откуда модель берёт сумму*. Создаётся любая модель одинаково — `Type` + `Format` + `Formula`; различает их только то, что стоит в `Formula`:

- **Статичная** — `Formula` содержит **фиксированное число** (константу, напр. `100`), без зависимости от постбэка. Применяют, когда рекламодатель не передаёт ставку или передаёт некорректно: завели модель с константой и привязали правилом в Distribution (напр. `Conversion Type = X AND Destination = рекламодатель`).
- **Динамическая** — `Formula` содержит **плейсхолдер поля конверсии** (напр. `{{aio.conversion.fields.<имя_поля>}}`), и значение читается из этого поля. По сути маппинг «поле постбэка → Payout/Revenue».

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

### Zero Payout vs Arrived Revenue$ — дефолтные модели тенанта

Zero Payout — всегда 0 (fallback, в `Formula` записан `0`). Arrived Revenue$ — конвертит `arrived_revenue` в USD и пишет в Revenue (дефолт ревенью-стороны). «Статичная»/«динамическая» — описательные ярлыки по содержимому `Formula` (константа vs плейсхолдер поля), а не пункты UI (см. выше).

### Где создаётся vs где применяется Business Model

Создаётся в `Settings → Business models`. Применяется (выбирается) в листе `Finance → Revenue/Payout Distribution` (модалка Edit revenue/payout). Два разных раздела UI.

### Business Model (запись денег) vs Metric (аналитика)

Business Model пишет деньги в поле конверсии. Чтобы увидеть их колонкой/средним чеком в отчётах — нужна отдельная Metric (`Settings → Metrics`). Запись и отображение — два разных шага.

### Business Model vs поля Revenue/Reward на Conversion Type

У Conversion Type есть статичные поля (Revenue + Reward, оставляют пустыми если нет выплаты). Business Model — это **формула расчёта в дистрибуции**, а не статичное поле на типе.

## Подводные камни Business Model — частые ошибки настройки

### Конверсия пришла, но Revenue/Payout = 0

Новый Conversion Type по дефолту = `Zero Payout` → конверсия приходит, но Revenue/Payout = 0. Симптом «конверсия пришла, но $0». Нужно завести Business Model и привязать в Distribution. См. *уточните у поддержки*.

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

Business Model — per-tenant: набор у каждого тенанта свой; нельзя предполагать наличие конкретной модели кроме дефолтных (`Arrived Revenue$`, `Zero Payout`).

## Куда идти за деталями Business Model — смежные темы

- **Движок дистрибуций** (Folder / Rule to pass / Strategy / лист, Revenue/Payout Distribution, экшены ноды) → [models/distributions-model.md](distributions-model.md) (родитель-движок).
- **Поля конверсии, семантика Revenue vs Payout, 4-шаговый кастомный паттерн, Uniqueness** → [models/conversion-model.md](conversion-model.md).
- **Траблшутинг «деньги не считаются»** (Zero Payout) → *уточните у поддержки*.
- **Отображение денег в отчётах** (Settings → Metrics: `Data feed metric` с источником `Conversions By Type Revenue`/`Payout`, `Computable`) → [how-to/analytics.md](../how-to/analytics.md).
- **Пошаговые процедуры создания Revenue/Payout Distribution** → [how-to/distributions.md](../how-to/distributions.md).
- **Собрать ссылку постбэка под модель** (в селекте генератора только модели типа `Revenue`, поля формулы обязательны, превью суммы) → [how-to/postback-generator.md](../how-to/postback-generator.md).
- **Profit = Revenue − Cost, payout-модели медиабаинга** → *Медиабаинг и место AIO в нём*.
- **Термины** (Business Model / Arrived Revenue$ / Zero Payout / Business Value) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (`Settings → Business models`; применение — `Finance → Revenue/Payout Distribution`).
