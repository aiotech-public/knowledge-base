---
id: metric
title: Metric / Custom Metric — концепт (модель)
description: Что такое Metric в AIO как сущность — настраиваемый числовой показатель-колонка в отчётах аналитики; 3 типа (Conversions count = штуки, Computable = формула, Data feed = фид событий; деньги = Data feed + source Conversions By Type Revenue/Payout), адресация по UUID, ось всех отчётов. Концепт; где и как используются метрики — how-to/analytics.md.
doc_type: model
builds: [erp]
related: [analytics, conversion-model, visit-field, custom-fields, glossary, api, visit-lifecycle, campaigns, conversion-ai-testing, ui-map]
language: ru
updated: 2026-08-11
---

# Metric / Custom Metric — концепт (модель)

> Модель сущности **Metric (числовой показатель в отчётах)**. Где и как метрики используются в отчётах (Roll Up, Cohorts, Compare, Colorize) + флоу создания — [how-to/analytics.md](../how-to/analytics.md). Conversion Type, который считает метрика `Conversions count` — [models/conversion-model.md](conversion-model.md). Поля/групперы, по которым метрики считаются — [models/visit-field.md](visit-field.md), [how-to/custom-fields.md](../how-to/custom-fields.md).

## Что такое Metric в AIO и какие типы бывают

**Metric — это настраиваемый числовой показатель, который отображается колонкой в отчётах аналитики (Roll Up / Cohorts / Comparative).** Набор доступных метрик per-tenant: системные метрики + заведённые кастомные. Кастомные заводятся в `Settings → Metrics → + Metric` выбором типа (считать число конверсий / считать деньги конверсий / вычислять формулой / считать по фиду событий). Метрика — это **ось колонок** отчёта: то, что измеряется в каждой строке. Каждая метрика имеет **UUID** и адресуется по нему и в формулах, и в API.

Имя метрики произвольное — в `Settings → Metrics` его свободно создают, переименовывают и редактируют. Поэтому одна и та же метрика в разных тенантах может называться по-разному (напр. earnings per lead — `EPL` или `EPL$`, разница только в приставке), и на само имя опираться нельзя: адрес метрики — её UUID, а не подпись колонки.

## Типы метрики (`+ Metric`) — Conversions count, Computable, Data feed

При создании тип-чузер `+ Metric` предлагает **ровно 3 типа**: `Conversions count` (число конверсий), `Computable metric` (формула) и `Data feed metric` (фид событий). Отдельного «денежного» типа НЕТ — **денежные суммы конверсий (revenue/payout) считаются через `Data feed metric`**, выбрав в нём источник `Conversions By Type Revenue` / `Conversions By Type Payout` (см. секцию «`Data feed metric`» ниже). Детали полей форм и каталог — [how-to/analytics.md](../how-to/analytics.md):

- **`Conversions count`** (помечен *Most used*) — считает **только число конверсий** по условию (штуки, не деньги). Конфиг: `Name` + `Conversion type` (какой тип конверсий считать). Так заводятся метрики вроде «Registration Count», «Lead Count». Флоу: `Settings → Metrics → +Metric → Conversions Count`, задать `Name` (опц. `Description`) и выбрать ранее созданный тип из дропдауна `Conversion Type` ([models/conversion-model.md](conversion-model.md)). Для **денежных** сумм конверсий (revenue/payout) `Conversions count` не годится — берётся `Data feed metric` с источником `Conversions By Type Revenue` / `Conversions By Type Payout` (см. секцию «`Data feed metric`» ниже).
- **`Computable metric`** — вычисляет значение **формулой** из чисел и других метрик (напр. ratio двух метрик).
- **`Data feed metric`** — считает по внутреннему **фиду событий AIO**: базовая системная метрика-источник + опц. `Flag`-фильтр. Через него же считаются **денежные** суммы: источник `Conversions By Type Revenue` (сумма Revenue) / `Conversions By Type Payout` (сумма Payout). Полная структура (варианты источника, `Flag`-enum, `values`) — в секции «`Data feed metric` — базовая метрика-источник + `Flag`-фильтр» ниже.

Колонки списка метрик в `Settings → Metrics`: `Metric`, `Formula` (у формульных — формула с именами метрик), `Access Type`, `Owner`, `Order`, `Main For`, `Visible`, `Created`.

## `Data feed metric` — базовая метрика-источник + `Flag`-фильтр

`Data feed metric` в живой форме = **базовая системная метрика-источник + опциональный `Flag`-фильтр**, а не агрегация median/sum/average. Структура формы:

- **Источник** — базовая системная метрика (дефолт `Visits`). Прочих вариантов **~29** (полный список приходит с бэка) — среди них `Conversions By Type Payout`, `Conversions By Type Revenue`, `Conversions By Type Count`, `Once Conversions By Type Count`, `Events By Group Count`. Денежные суммы делаются именно здесь: источник `Conversions By Type Revenue` (Revenue) / `Conversions By Type Payout` (Payout).
- **`values`** — адрес под источник: UUID `Conversion Type` для источников `... By Type ...` либо id групп событий для `Events By Group Count`.
- **`Flag`-фильтр** (опц.) — сужает выборку по флагу визита. Enum — **8 дискретных значений**: `Trash` / `BackFix` / `Interested` / `Qualified` / `Engaged` + три предопределённых `Interested BackFix` / `Qualified BackFix` / `Engaged BackFix`. Выбирается **один** флаг из списка (свободного комбинирования нет — комбинации существуют только как эти 3 готовых `X BackFix`); `noFlag` = без фильтра по флагу.

Так строятся метрики вида «Qualified-визиты», «конверсии типа X по payout», «события группы Y» — источник задаёт что считать, `Flag` сужает по состоянию визита.

## Общие атрибуты формы метрики — Business value, Main for, Hidden at groupers, Categories

Помимо `Name` / `Format` / `Order` / `Visible`, форма создания/редактирования метрики несёт ещё несколько полей:

- **`Business value`** (select) — служебное поле метрики. Не путать с `Business Value` конверсии (лейблы `Good`/`Neutral`/`Bad`, влияет на AI-`Reward` — [models/conversion-model.md](conversion-model.md)).
- **`Main for`** (select) — служебное поле механизма approximation (`Approximate metrics` — пропорция от более полной метрики).
- **`Hidden at groupers`** (multi-select) — список групперов, при разбивке по которым колонка метрики **скрывается**. Объясняет практику «метрика в списке есть, а в конкретной разбивке колонки нет»: при чтении отчёта проверь `Hidden at groupers` метрики, прежде чем считать колонку пропавшей.
- **`Categories`** (multi-select) — теги-категории метрики; справочник категорий **per-tenant**. Служит для группировки/фильтрации метрик в списке.

## `Conversions count` vs `Conversion Type` — Metric считает, Type — это событие

`Conversion Type` — это **тип события** (создаётся в `Settings → Conversion Types`, см. [models/conversion-model.md](conversion-model.md)). Metric типа `Conversions count` — это уже **подсчёт числа** конверсий выбранного `Conversion Type`, отображаемый колонкой. Одна сущность считает другую: сначала есть тип конверсии, потом метрика, которая его считает.

Практическое следствие: чтобы завести промежуточную («наивную») конверсию и метрику на ней, сначала создаётся `Conversion Type`, и только потом метрика `Conversions count` на нём (подробнее — раздел «Как завести кастомную промежуточную конверсию и метрику» ниже).

## `Computable metric` — формула из других метрик, адресация по UUID

`Computable metric` ссылается на другие метрики **по UUID**, а не по человеческим именам. Флоу создания: `+Metric` → `Computable` → название → формула расчёта (для соотношения двух метрик берутся их uuid и составляется выражение вида `[uuid метрики 1] / [uuid метрики 2]`) → формат отображения (`Format`) → порядок (`Order`) → при необходимости отображение для конкретных групперов и общая видимость (`Visible`).

В UX формулы пользователь пишет не голые UUID, а **буквы-переменные**: пул из ~21 буквы (`x`, `y`, `z`, `a`..`d3`), каждая маппится дропдауном на конкретную метрику. Система при сохранении подставляет за каждой буквой UUID выбранной метрики — то есть буквы это удобный алиас, а хранится и считается всё равно по UUID.

Тот же UUID — адрес метрики в API: в **Pivot Report API** метрики возвращаются в leaf-объекте как мапа placeholders — ключ вида `metric_<uuid>` → значение. То есть UUID метрики — единый адрес и в формулах Computable, и в API ([reference/glossary.md](../reference/glossary.md) → `Pivot Report API`; [how-to/api.md](../how-to/api.md)).

## Как завести кастомную промежуточную конверсию и метрику — паттерн оптимизации

Сильный практический паттерн: **активно заводить кастомные метрики И кастомные конверсии**. Логика: для быстрой оптимизации (и в рекламном кабинете, и в AIO) нужно как можно больше ивентов/данных. Целевых конверсий (регистраций, покупок) обычно мало — поэтому заводят **промежуточную конверсию**, привязанную к лэнду (напр. «провёл 30 секунд» = engaged-визит), строят на ней метрику и шлют событие в рекламный кабинет.

Метрики можно **комбинировать в составную**: напр. «визит 2 минуты + style lead» → назвать `quality lead` — так реализовать можно, надстраивая одну метрику над другой.

## На какую метрику смотреть в отчётах — Qualified Visits, не сырые визиты

Рекомендуется смотреть в основном на метрику **`Qualified Visits`** (а не на сырые визиты, «чистая база воронки»); определение и механика — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md); считается через `Visit Loss` ([reference/glossary.md](../reference/glossary.md) → `Qualified Visits`, `Visit Loss`).

## Системные метрики: User Behavior и Approximate

- **`User Behavior` метрики (группа)** — поведенческие: записи сессий, хитмапы, время на лэнде, скроллинг-метрики (LP1/LP2 Scrolling — `average`), `Bounce Rate` (<10 сек И <10% скролла, границы 10/10), `Engagement Rate` (>30 сек И >30% скролла, границы 30/30). См. [reference/glossary.md](../reference/glossary.md).
- **`Approximate metrics`** (подчёркнутые в таблице) — высчитываются как **пропорция** от более полной метрики (не у всех визитов есть реальные данные для целевого шага). Подчёркивание = аппроксимация, не точное значение.

## Метрика с исключением по источнику данных — одна метрика вычитает другую

**Метрика может быть настроена с исключением по источнику: одна денежная метрика читает свой набор конверсий, минус то, что уже считает другая метрика.** То есть настраивается взаимное исключение между метриками, чтобы одни и те же деньги не считались дважды в разных колонках.

Практический вывод при чтении: если сумма в колонке кажется «недосчитанной» относительно другой денежной колонки — проверь, не настроено ли у этой метрики исключение источника (она может намеренно не включать конверсии, которые считает соседняя метрика). Это конфигурация метрики, а не потеря данных.

## Метрики варианта для AI — `Select`, `Reward`, `Metrics AI`

`Select` и `Reward` — отдельные служебные метрики варианта для Conversions AI (`Thompson Sampling`): `Select` = сколько раз вариант показан; `Reward` = сколько раз получена целевая конверсия (или сумма Business Value). Отдельно `Metrics AI` оптимизирует под **произвольную числовую метрику** (CTR, EPL, ROAS) за `TimeFrame` через `Epsilon-Greedy`. То есть метрика — это ещё и цель AI-оптимизации ([reference/glossary.md](../reference/glossary.md) → `Select`, `Reward`, `Metrics AI`).

## Где используется Metric — связи и смежные сущности

- **Metric опирается на `Conversion Type`** — тип `Conversions count` считает конверсии выбранного `Conversion Type`; `Reward Weight` / `Business Value` типа влияют на AI-метрики `Reward`/`Select` ([models/conversion-model.md](conversion-model.md)).
- **Metric — ось колонок отчётов** — Roll Up / Cohorts (`Top Level × Target Metric`) / Comparative (`Groupers × Metrics`); `Colorize` красит таблицу по выбранной метрике ([how-to/analytics.md](../how-to/analytics.md)).
- **Metric смежна с Visit Field / групперами** — метрики считаются по данным полей визита; `Availability as grouper` и `Make analytic` — про поля, не про метрики ([models/visit-field.md](visit-field.md), [how-to/custom-fields.md](../how-to/custom-fields.md)).
- **Metric питает `Metrics AI`** — на уровне кампании AI оптимизирует под произвольную числовую метрику за `TimeFrame` ([reference/glossary.md](../reference/glossary.md)).
- **Metric адресуется по UUID** — в Pivot Report API (`metric_<uuid>`) и в формулах `Computable metric` ([reference/glossary.md](../reference/glossary.md) → Pivot Report API).
- **`Data feed metric` считает по внутреннему фиду событий AIO** — форма = базовая метрика-источник (дефолт `Visits`) + опц. `Flag`-фильтр (один из 8) + адрес в `values`; денежные суммы = источник `Conversions By Type Revenue`/`Payout`. Полный список источников и `Flag`-enum — в секции «`Data feed metric` — базовая метрика-источник + `Flag`-фильтр» выше.

## Metric vs Conversion Type, vs Grouper, vs Approximate — где путают

| Не путать | Разница |
|---|---|
| **Metric vs Conversion Type** | `Conversion Type` — тип события (создаётся в `Settings → Conversion Types`). Metric `Conversions count` — **подсчёт числа** конверсий этого типа, колонка в отчёте. Одна сущность считает другую. |
| **Metric vs Grouper** | Grouper — **измерение** для разбивки строк отчёта (по полю/кампании/гео). Metric — **числовое значение** в колонке. `Availability as grouper` и `Make analytic` относятся к Fields-как-групперам, НЕ к Metric. |
| **`Computable metric` vs `Approximate metric`** | Computable — кастомная метрика-**формула** из других метрик. Approximate (подчёркнутые) — **системные** метрики, посчитанные как пропорция от более полной из-за неполноты данных. Разные вещи. |
| **Conversions count метрика vs Conversion Type** | См. строку 1: тип события vs его подсчёт. Не одно и то же — метрика существует поверх типа. |

## Подводные камни конфигурации метрики

- **Conversions count считает только выбранный Conversion Type** — чтобы метрика считала промежуточную («наивную») конверсию, сначала создаётся сам `Conversion Type`, и только потом метрика на нём.
- **Computable ссылается на UUID, не на имена** — в формуле используются именно uuid метрик (`[uuid1]/[uuid2]`).
- **Видимость управляется флагом `Visible`** — метрика может существовать, но быть скрытой из колонок. «Метрики не видно» → проверить `Visible`/`Order`, а не считать, что её нет.
- **Колонка пропала только в конкретной разбивке** — у метрики заполнен `Hidden at groupers` (список групперов, при которых колонка скрывается). Метрика есть, но в этой разбивке намеренно спрятана — проверь `Hidden at groupers`, прежде чем считать колонку пропавшей.

## Подводные камни чтения значений метрики

- **Approximate (подчёркнутые) метрики — не точные значения, а пропорция** — стоит предупреждать, что подчёркивание = аппроксимация.
- **`Data feed metric` = источник + `Flag`-фильтр** — в живой форме выбирается базовая метрика-источник (дефолт `Visits`) и опц. один `Flag` из 8 (`Trash`/`BackFix`/`Interested`/`Qualified`/`Engaged`/`Interested BackFix`/`Qualified BackFix`/`Engaged BackFix`), а не агрегация (медиана/сумма/среднее здесь не выбираются).
- **Числа AIO ≠ числа рекламного кабинета.** Раздел `Meta` показывает спенд/косты/FB-ad-метрики из кабинета, а не конверсии к нам. По объёму AIO-визитов меньше, чем FB-кликов (часть кликов до агента не доходит) — это норма, не баг метрик ([how-to/campaigns.md](../how-to/campaigns.md)).

## Вглубь и вбок — связанные материалы

- **Где и как метрики используются в отчётах** (Roll Up, Cohorts, Compare, Colorize) + флоу создания → [how-to/analytics.md](../how-to/analytics.md).
- **Conversion Type**, на котором строится `Conversions count`; `Reward Weight` / `Business Value` для AI → [models/conversion-model.md](conversion-model.md).
- **Поля/групперы**, по которым считаются метрики (`Availability as grouper`, `Make analytic`) → [how-to/custom-fields.md](../how-to/custom-fields.md), [models/visit-field.md](visit-field.md).
- **Pivot Report API**, адресация метрик по `metric_<uuid>` → [reference/glossary.md](../reference/glossary.md) (Pivot Report API); лимиты API — [how-to/api.md](../how-to/api.md).
- **Как метрики связаны с AI-оптимизацией** → [heuristics/conversion-ai-testing.md](../heuristics/conversion-ai-testing.md).
- **Точные термины** (`Conversions Count`, `Money`, `Approximate metrics`, `Qualified Visits`, `Visit Loss`, `Select`/`Reward`, `Metrics AI`, `TimeFrame`, `Epsilon-Greedy`) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (`Settings → Metrics` `/metrics`; потребление — в `/analytics/*`).
