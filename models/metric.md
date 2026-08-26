---
id: metric
title: Metric / Custom Metric — концепт (модель)
description: Что такое Metric в AIO как сущность — настраиваемый числовой показатель-колонка в отчётах аналитики; 4 типа (Conversions count = штуки, Remarketing count = события рассылок, Computable = формула, Data feed = фид событий; деньги = Data feed + source Conversions By Type Revenue/Payout), адресация по UUID, ось всех отчётов; Remarketing count считает события отправок, а не визиты (Delivered/Opened только у пушей remarketing-кампании, по SMS/Email событий не бывает вовсе), заводится руками и правится диалогом Edit data feed metric под правом settings.metrics.edit.data-feed; процент по рассылкам (CTR/open-rate) — формула над Remarketing count, и на страницах раздела Marketing (рассылки) видны только такие метрики и формулы целиком над ними; список метрик в Settings → Metrics — папками, два вида Groups / Order, колонка Definition (из чего метрика собрана), порядок перетаскиванием в Order под правом settings.metrics.edit. Концепт; где и как используются метрики — how-to/analytics.md.
doc_type: model
builds: [erp]
related: [analytics, conversion-model, visit-field, custom-fields, remarketing-campaigns, debug-with-logs, permissions-model, glossary, api, visit-lifecycle, campaigns, conversion-ai-testing, ui-map]
language: ru
updated: 2026-08-11
---

# Metric / Custom Metric — концепт (модель)

> Модель сущности **Metric (числовой показатель в отчётах)**. Где и как метрики используются в отчётах (Roll Up, Cohorts, Compare, Colorize) + флоу создания — [how-to/analytics.md](../how-to/analytics.md). Conversion Type, который считает метрика `Conversions count` — [models/conversion-model.md](conversion-model.md). Поля/групперы, по которым метрики считаются — [models/visit-field.md](visit-field.md), [how-to/custom-fields.md](../how-to/custom-fields.md).

## Что такое Metric в AIO и какие типы бывают

**Metric — это настраиваемый числовой показатель, который отображается колонкой в отчётах аналитики (Roll Up / Cohorts / Comparative).** Набор доступных метрик per-tenant: системные метрики + заведённые кастомные. Кастомные заводятся в `Settings → Metrics → + Metric` выбором типа (считать число конверсий / считать события рассылок / вычислять формулой / считать по фиду событий; деньги конверсий — через `Data feed metric`). Метрика — это **ось колонок** отчёта: то, что измеряется в каждой строке. Каждая метрика имеет **UUID** и адресуется по нему и в формулах, и в API.

Имя метрики произвольное — в `Settings → Metrics` его свободно создают, переименовывают и редактируют. Поэтому одна и та же метрика в разных тенантах может называться по-разному (напр. earnings per lead — `EPL` или `EPL$`, разница только в приставке), и на само имя опираться нельзя: адрес метрики — её UUID, а не подпись колонки.

## Типы метрики (`+ Metric`) — Conversions count, Remarketing count, Computable, Data feed

При создании тип-чузер `+ Metric` (заголовок диалога — `Add metric to AIO`) предлагает **ровно 4 типа**: `Conversions count` (число конверсий), `Remarketing count` (события рассылок), `Computable metric` (формула) и `Data feed metric` (фид событий). Отдельного «денежного» типа НЕТ — **денежные суммы конверсий (revenue/payout) считаются через `Data feed metric`**, выбрав в нём источник `Conversions By Type Revenue` / `Conversions By Type Payout` (см. секцию «`Data feed metric`» ниже). Детали полей форм и каталог — [how-to/analytics.md](../how-to/analytics.md):

- **`Conversions count`** (помечен *Most used*) — считает **только число конверсий** по условию (штуки, не деньги). Конфиг: `Name` + `Conversion type` (какой тип конверсий считать). Так заводятся метрики вроде «Registration Count», «Lead Count». Флоу: `Settings → Metrics → +Metric → Conversions Count`, задать `Name` (опц. `Description`) и выбрать ранее созданный тип из дропдауна `Conversion Type` ([models/conversion-model.md](conversion-model.md)).
- **`Remarketing count`** — считает **события рассылок** (в UI тип подписан «Sends, errors and deliveries of remarketing messages»): два необязательных мультиселекта `Events` и `Channels`, пустой список = все. Форма, как читать колонку и кто может её править — секции «`Remarketing count`» ниже.
- **`Computable metric`** — вычисляет значение **формулой** из чисел и других метрик (напр. ratio двух метрик).
- **`Data feed metric`** — считает по внутреннему **фиду событий AIO**: базовая системная метрика-источник + опц. `Flag`-фильтр. Через него же считаются **денежные** суммы: источник `Conversions By Type Revenue` (сумма Revenue) / `Conversions By Type Payout` (сумма Payout). Полная структура (варианты источника, `Flag`-enum, `values`) — в секции «`Data feed metric` — базовая метрика-источник + `Flag`-фильтр» ниже.

### `Remarketing count` — счётчик событий рассылок (`Events` / `Channels`)

**`Remarketing count` считает события рассылок, а не визиты.** Форма: `Name`, `Description` и два необязательных мультиселекта — `Events` (`Sent` / `Failed` / `Delivered` / `Opened`, плейсхолдер пустого выбора `All events`) и `Channels` (каналы подписаны `WebPush` / `Telegram` / `SMS` / `Email`, плейсхолдер `All channels`). Пустой список = без фильтра, то есть «все». Модификатора `Flag` у этого типа нет — он к нему неприменим.

**Готовых метрик рассылок в системе нет — их заводят руками.** Новый тенант создаётся из тенанта-шаблона и метрики копируются вместе с ним, поэтому «набор из коробки» — это набор шаблона, а не системный набор AIO. Нет колонки по рассылкам в отчёте — сначала проверь, заведена ли метрика в `Settings → Metrics`.

Тот же счётчик собирается и вторым путём — `Data feed metric` с источником `Remarketing Count` (секция «`Data feed metric`» ниже). Метрика получается одна и та же; разница в том, что пресет `Remarketing count` зашивает служебные поля (формат `Number`, порядок, видимость), а в `Data feed metric` они задаются руками.

### Как читать колонку `Remarketing count` — считаются события, а не визиты

**Колонка считает события отправки, а не уникальные визиты:** повторные отправки одному и тому же визиту не схлопываются, каждая отправка — своё событие. В строках, где рассылок не было, стоит 0; фильтры и групперы отчёта применяются к метрике как к любой другой.

- **`Delivered` и `Opened` есть только у пушей remarketing-кампании** — их присылает сервис-воркер лэнда. У `Telegram` таких событий не бывает вовсе: по нему наполняются только `Sent` и `Failed` ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)).
- **По каналам `SMS` и `Email` колонка всегда пустая** — отправителей под эти каналы в системе нет, поэтому по ним не возникает ни `Sent`, ни `Failed`. Отказ молчаливый: нули здесь не признак сбоя доставки, отправки просто не было.
- **`Failed` внутри метрики не разбит по причине** — в него попадают и отправки, срезанные лимитом, и мёртвые push-подписки, и обычные сбои. Разложить по причине — группировкой `Send Result` ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)); текст ошибки — в логах namespace `Remarketing` ([how-to/debug-with-logs.md](../how-to/debug-with-logs.md)).

**В отчётах трекера колонка рассылок ставится рядом с обычными метриками.** В Roll Up и Tracker-таблицах `Remarketing count` считается вместе с визитными и конверсионными метриками, и трекерные фильтры, скоупы и групперы применяются к ней так же, как к остальным. Обратное неверно: на страницах раздела `Marketing` метрики других источников не показываются вовсе — см. секцию про формулы ниже.

### Процент по рассылкам (CTR, open-rate) — формула над `Remarketing count`

**Готовой колонки-процента у рассылок нет: CTR / open-rate собирается `Computable metric`-формулой над метриками `Remarketing count`** — например метрика с событием `Opened`, делённая на метрику с событием `Delivered`. Сам тип `Remarketing count` даёт только счётчики событий, отношение он не считает.

**Ограничение, из-за которого формула может не появиться.** На страницах раздела `Marketing` (`/app/remarketing` — рассылки) показываются **только** метрики типа `Remarketing count` и формулы, построенные **целиком** над ними: формула проходит, если все её зависимости — в том числе через вложенные формулы — сами такие же. Формула, в которую подмешана метрика любого другого источника (визиты, конверсии, метрики `Meta`), просто **не появится колонкой** — без ошибки и без объяснения. Отсюда типичный симптом: «формулу завёл, а колонки на страницах рассылок нет» → проверить, что каждая метрика формулы — `Remarketing count`.

Формула отсеивается ещё в двух случаях: если у неё нет ни одной метрики-зависимости и если метрика, на которую она ссылается, **заархивирована** — архивные метрики в отбор не попадают.

Пошаговый рецепт под рассылку — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md).

### Метрику `Remarketing count` завёл, а изменить её не могу — какие нужны права

**Отдельного права под тип `Remarketing count` нет: он живёт под общими правами метрик** — создание и правка `settings.metrics.edit`, просмотр списка `settings.metrics.view` ([models/permissions-model.md](permissions-model.md)). Сама карточка `Remarketing count` в диалоге `Add metric to AIO` отдельно не гейтится: её видит любой, кто дошёл до кнопки создания метрики.

**Правка уже созданной метрики требует другого права.** Своего диалога редактирования у этого типа нет — созданная метрика открывается диалогом `Edit data feed metric`, а он гейтится правом `settings.metrics.edit.data-feed`. Отсюда симптом «метрику завёл, а поменять `Events` / `Channels` не могу»: не хватает именно `settings.metrics.edit.data-feed`, а не общего права на метрики.

## `Data feed metric` — базовая метрика-источник + `Flag`-фильтр

`Data feed metric` в живой форме = **базовая системная метрика-источник + опциональный `Flag`-фильтр**, а не агрегация median/sum/average. Правило не универсально: у источника `Remarketing Count` `Flag` не предлагается вовсе (см. буллет ниже). Структура формы:

- **Источник** — базовая системная метрика (дефолт `Visits`); полный список вариантов приходит с бэка — среди них `Conversions By Type Payout`, `Conversions By Type Revenue`, `Conversions By Type Count`, `Once Conversions By Type Count`, `Events By Group Count`. Денежные суммы делаются именно здесь: источник `Conversions By Type Revenue` (Revenue) / `Conversions By Type Payout` (Payout).
- **`values`** — адрес под источник: UUID `Conversion Type` для источников `... By Type ...` либо id групп событий для `Events By Group Count`.
- **Источник `Remarketing Count`** — особый случай: при его выборе форма показывает вместо блока `Flag` те же два мультиселекта `Events` и `Channels`, что у типа `Remarketing count` (секция выше). `Flag` для этого источника не предлагается вовсе.
- **`Flag`-фильтр** (опц.) — сужает выборку по флагу визита. Enum — **8 дискретных значений**: `Trash` / `BackFix` / `Interested` / `Qualified` / `Engaged` + три предопределённых `Interested BackFix` / `Qualified BackFix` / `Engaged BackFix`. Выбирается **один** флаг из списка (свободного комбинирования нет — комбинации существуют только как эти 3 готовых `X BackFix`); `noFlag` = без фильтра по флагу.

Так строятся метрики вида «Qualified-визиты», «конверсии типа X по payout», «события группы Y» — источник задаёт что считать, `Flag` сужает по состоянию визита.

## Общие атрибуты формы метрики — Business value, Main for, Hidden at groupers, Categories

Помимо `Name` / `Format` / `Order` / `Visible`, форма создания/редактирования метрики несёт ещё несколько полей:

- **`Business value`** (select) — служебное поле метрики. Не путать с `Business Value` конверсии (лейблы `Good`/`Neutral`/`Bad`, влияет на AI-`Reward` — [models/conversion-model.md](conversion-model.md)).
- **`Main for`** (select) — служебное поле механизма approximation (`Approximate metrics` — пропорция от более полной метрики).
- **`Hidden at groupers`** (multi-select) — список групперов, при разбивке по которым колонка метрики **скрывается**. Объясняет практику «метрика в списке есть, а в конкретной разбивке колонки нет»: при чтении отчёта проверь `Hidden at groupers` метрики, прежде чем считать колонку пропавшей.
- **`Categories`** (multi-select) — теги-категории метрики; справочник категорий **per-tenant**. Служит для группировки/фильтрации метрик в списке.

## Как устроен список метрик в `Settings → Metrics` — папки и два вида, `Groups` / `Order`

**Список метрик — не плоская таблица: по умолчанию он раскладывается по папкам, а переключатель вверху справа даёт два вида — `Groups` и `Order`.**

- **`Groups`** — метрики разложены по папкам, режим под поиск нужной метрики. Верхний уровень — тип метрики, как он хранится: `Database` и `Formula`. Папка `Formula` — это `Computable metric`; `Conversions count`, `Remarketing count` и `Data feed metric` все хранятся как `Database`. Внутри типа — папки по базовой метрике-источнику: у самых частых источников подпись короткая (`Conversions`, `Revenue`, `Payout`, `Events`, `Remarketing`), у остальных подпапка называется именем источника целиком (`Visits`, `Cost`, `Clicks`, `Formula` и т.д.). В `Ungrouped` попадают только метрики, у которых источник не задан вовсе.
- **`Order`** — тот же список одной сплошной дорожкой, без папок. Только здесь строки перетаскиваются мышью.

Быстрый фильтр **`Type`** над списком фильтрует по метрике-источнику, а не по папке: метрики рассылок в нём собраны под пунктом `Remarketing Count`.

Вид не запоминается: следующий заход снова открывает `Groups`. Свёрнутые папки запоминаются в браузере, поэтому в другом браузере список снова раскрыт целиком.

### Какие колонки в списке метрик и что показывает `Definition`

Колонки по умолчанию: `Metric`, `Definition`, `Access Type`, `Owner`, `Main For`, `Visible`, `Created`. Колонки `UUID`, `Type`, `Order`, `Categories`, `Business value`, `Shares`, `Hidden at groupers`, `Archived`, `Updated` тоже есть, но выключены и включаются в настройке колонок.

**`Definition` показывает, из чего метрика собрана.** У `Computable metric` — сама формула, где вместо UUID подставлены имена метрик-слагаемых (числа, операторы `+ − × ÷` и скобки остаются как есть). У остальных типов — имя источника (`Conversions By Type Count`, `Remarketing Count`, `Events By Group Count`, …) и в скобках его аргументы: тип конверсии, группа событий, событие рассылки (`Sent` / `Delivered` / `Opened` / `Failed`), канал, `Flag`.

Имена подставляются по **всем** метрикам тенанта — в том числе по тем, которых нет в твоей видимости. Если в формуле стоит `Unknown` — метрику, на которую она ссылается, разрешить не удалось: она удалена или заархивирована. UUID такой ссылки виден в тултипе чипа.

### Как поменять порядок метрик — перетаскивание в виде `Order`

**Порядок меняется перетаскиванием строк в виде `Order`; в `Groups` строки не таскаются.** Сохраняется сразу, без диалога подтверждения, и весь список перенумеровывается с 1. Это тот же порядок, что задаётся полем `Order` в форме метрики, и он же определяет, в каком порядке метрики идут колонками в отчётах. `order` — атрибут метрики, а не личная настройка смотрящего: порядок один на тенант и меняется у всех.

Перетаскивание требует права `settings.metrics.edit` ([models/permissions-model.md](permissions-model.md)) — без него список читается, но не переставляется.

**Порядок не сохранился, пришла ошибка `Some UUIDs were not found in this model. No access to: '<uuid>'`.** Сохранение уходит **одним запросом на весь список**, поэтому одна метрика, которую тебе нельзя редактировать, роняет сохранение целиком — не сохраняется ничего. В списке видны все метрики, доступные на просмотр, а редактировать можно не все: это определяет `Access Type` метрики.

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
- **`Data feed metric` считает по внутреннему фиду событий AIO** — форма = базовая метрика-источник (дефолт `Visits`) + опц. `Flag`-фильтр (один из 8; у источника `Remarketing Count` вместо `Flag` — `Events`/`Channels`) + адрес в `values`; денежные суммы = источник `Conversions By Type Revenue`/`Payout`. Полный список источников и `Flag`-enum — в секции «`Data feed metric`» выше.

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
- **`Data feed metric` = источник + `Flag`-фильтр** — в живой форме выбирается базовая метрика-источник (дефолт `Visits`) и опц. один `Flag` из 8 (`Trash`/`BackFix`/`Interested`/`Qualified`/`Engaged`/`Interested BackFix`/`Qualified BackFix`/`Engaged BackFix`), а не агрегация (медиана/сумма/среднее здесь не выбираются). У источника `Remarketing Count` `Flag` не предлагается — вместо него мультиселекты `Events` / `Channels`.
- **Числа AIO ≠ числа рекламного кабинета.** Раздел `Meta` показывает спенд/косты/FB-ad-метрики из кабинета, а не конверсии к нам. По объёму AIO-визитов меньше, чем FB-кликов (часть кликов до агента не доходит) — это норма, не баг метрик ([how-to/campaigns.md](../how-to/campaigns.md)).

## Вглубь и вбок — связанные материалы

- **Где и как метрики используются в отчётах** (Roll Up, Cohorts, Compare, Colorize) + флоу создания → [how-to/analytics.md](../how-to/analytics.md).
- **Conversion Type**, на котором строится `Conversions count`; `Reward Weight` / `Business Value` для AI → [models/conversion-model.md](conversion-model.md).
- **Поля/групперы**, по которым считаются метрики (`Availability as grouper`, `Make analytic`) → [how-to/custom-fields.md](../how-to/custom-fields.md), [models/visit-field.md](visit-field.md).
- **Pivot Report API**, адресация метрик по `metric_<uuid>` → [reference/glossary.md](../reference/glossary.md) (Pivot Report API); лимиты API — [how-to/api.md](../how-to/api.md).
- **Как метрики связаны с AI-оптимизацией** → [heuristics/conversion-ai-testing.md](../heuristics/conversion-ai-testing.md).
- **Точные термины** (`Conversions Count`, `Money`, `Approximate metrics`, `Qualified Visits`, `Visit Loss`, `Select`/`Reward`, `Metrics AI`, `TimeFrame`, `Epsilon-Greedy`) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (`Settings → Metrics` `/metrics`; потребление — в `/analytics/*`).
