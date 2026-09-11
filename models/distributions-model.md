---
id: distributions-model
title: Distributions Model AIO
description: Как устроены Distributions в AIO — 9 типов (включая Message Templates — дерево текстов ремаркетинг-рассылки и Auto Rules — шаблон автоправил Meta с деревом из фаз и правил), структура дерева (Folder + Strategy + Rule + Payload), Business Models, типичные use case'ы (Buyer UTM Distribution, Direct Traffic, Payout rules).
doc_type: model
builds: [erp, mtk]
related: [auto-rules, distributions, remarketing-campaigns, notifications-flow, push-notifications, permissions, glossary, flow-model, business-model, visit-lifecycle, conversion-model]
language: ru
updated: 2026-09-11
---

# Distributions Model AIO

**Distribution** — централизованное дерево правил (`Settings → Distributions`), которое автоматически подставляет значения по визиту: payout, revenue, buyer-метка, лэнд по гео, поле визита. Вместо настройки каждой кампании отдельно, правило задаётся один раз и применяется ко всем визитам. Поддерживается 9 типов: `Payout`, `Revenue`, `Campaign Content`, `Flow Content`, `Fill Field`, `Direct Traffic`, `Remarketing Content`, `Message Templates`, `Auto Rules`. Структура дерева: `Folder` + `Strategy` (полный набор в дереве Distribution — `First` / `Weights` / `Conversions AI` / `Metrics AI`) + `Rule to pass` + `Payload`-ноды. Девятый тип `Auto Rules` (только ERP) под это описание не подходит: значения по визиту он не подставляет, а держит автоправила над Meta-сущностями, дерево у него собрано из двух других нод (`Phase` и `Auto Rule`), и настраивают его в собственном разделе `Automations` — хотя создать и увидеть его можно и в `Settings → Distributions` ([how-to/auto-rules.md](../how-to/auto-rules.md)). Конкретные процедуры — в [how-to/distributions.md](../how-to/distributions.md). В MTK-билде раздел открывается платным модулем `MTK Distribution` (*Билды, тарифы, триал и статусы тенанта*), в ERP доступен всем.

## Зачем нужны Distributions

**Без Distributions:** каждая кампания настраивается отдельно. Когда баеров 50 — в каждой кампании руками вписывать buyer-метку. Меняется payout — лезть в каждый Destination. Запутанно, ошибки.

**С Distributions:** правила вынесены централизованно. «Buyer X — метка Y» / «Гео IT, источник FB — payout $5» / «Голый домен без UTM — этой кампании» — заданы один раз в `Settings → Distributions`, применяются ко всем визитам.

## Какие 9 типов Distributions существуют и когда каждый применяется

Все 9 создаются из `Settings → Distributions → + Distribution`; плитки пикера в порядке показа: **Payout settings · Revenue settings · Campaign content · Fill field · Direct traffic · Message templates · Auto rules · Flow content · Remarketing content** (плитка `Auto rules` есть только в ERP-билде, и у этого типа есть второй вход — своя страница `Automations → Rule Templates`, [how-to/auto-rules.md](../how-to/auto-rules.md)). На плитках `Flow content` и `Remarketing content` стоит бейдж **`Advanced`** — плитка при этом остаётся видимой и кликабельной, бейдж только помечает «не для беглого выбора»; из пикера плитки убираются исключительно по правам. `Payout settings` / `Revenue settings` — те же money-деревья, что в разделе `Finance` (см. раздел «Как настроить Revenue и Payout Distribution в разделе Finance» ниже). В фильтре `Type` таблицы `Settings → Distributions` значений больше девяти: фильтр отдаёт весь внутренний список типов, включая те, которые в пикере создания не предлагаются.

### Payout — автоматический payout по правилам

Автоматически применяет нужный payout по правилам. Используется, когда payout-правила разные — payout по гео/источнику/рекламодателю.

### Revenue — автоматический revenue по правилам

Автоматически применяет revenue по правилам. По дефолту тенант имеет preconfigured (`Arrived Revenue$`) для приёма revenue из постбэков. Используется для кастомных правил revenue.

### Campaign Content — значения для одного шага кампании

Назначает значения для **одного шага** кампании (лэнд по гео, дестинейшн по owner-у). Используется для тонкой настройки одной кампании.

### Flow Content — значения для нескольких шагов одного Flow

Назначает значения для **нескольких шагов одного Flow**. Набор нод тот же, что у Campaign Content (Landing, Destination, Redirect, Reflect), **плюс** `Split Groups` и `Flow State` — именно они задают, к какому шагу применяется нода внутри дерева. Готовую дистрибуцию можно навесить на **один** шаг кампании или сразу на **все**. Используется, когда все кампании на этом Flow должны вести себя одинаково.

### Fill Field — заполнение поля визита через Fields by Distribution

Заполняет поля визита (используется в шаге `Fields by Distribution` во Flow). Под разные цели: Buyer UTM, FB Pixel, CAPI Token, поля-плейсхолдеры Content Library (`Offer Name`, `Price` и т.п.), а также подстановка лэндов и дестинейшнов. Пять типов fill-узлов, каждый под свой тип контента поля: `Add Fill Field Text` / `Image` / `Landing` / `Library` / `Destination`. Используется для UTM-меток баеров (`buyer_utm`), AffID, API-токенов, ID-маркеров.

Один шаг `Fields by Distribution` заполняет **несколько полей за один проход**: `Fill Field`-дистрибуция работает в режиме `Multiple Nodes` (см. «Две оси выбора» ниже) — дерево обходится до конца, каждый прошедший fill-узел пишет своё значение в своё целевое поле. Если **два узла целятся в одно и то же поле**, выигрывает **первый по порядку дерева** (узлы читаются сверху вниз), поздний узел на то же поле игнорируется. Значит порядок узлов в дереве — это приоритет: более общий fallback держи ниже конкретных правил.

### Direct Traffic — маршрутизация голых ссылок (домен без UTM)

Для голых ссылок (только домен, без UTM). Альтернатива `Default Query Code` на домене. Поддерживает `initial_path` для разделения трафика внутри одного домена. Используется при прямом заходе на голый домен. Если на домене не настроено ни `Default Query Code`, ни `Direct Traffic Distribution`, заход на голый домен отдаёт `502`.

### Remarketing Content — сообщения для нод пуш-сценария, собранного во флоу

Дерево сообщений для нод `Notifications Flow`: ноды `Add Push Message` и `Add Telegram Message`, каждая ссылается на готовый шаблон из `Marketing → Message Templates`. Привязывается к флоу при создании.

Дистрибуция `Remarketing Content` — **кастомный путь**: рассылку по визитам сегодня собирают ремаркетинг-кампанией, и тексты она берёт из дистрибуции типа `Message Templates` ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)). `Remarketing Content` остаётся для пуш-сценариев, которые собирают во флоу вручную — [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

### Message Templates — дерево шаблонов сообщений для ремаркетинг-рассылок

Дерево нод-шаблонов, из которого ремаркетинг-рассылка берёт текст сообщения: шаг серии рассылок (`Drip schedule`) выбирает дистрибуцию этого типа и на каждую отправку достаёт из неё подходящую ноду. Процедура рассылки — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md), сборка самого текста (плейсхолдеры, спинтакс) — [how-to/push-notifications.md](../how-to/push-notifications.md). Контент сообщения хранится **прямо в настройках ноды**, а не ссылкой на сущность из `Marketing → Message Templates` — нода `Push Template` называется так же, но это не она.

Чем этот тип отличается от остальных дистрибуций:

- **Листья — четыре типа шаблонов:** `Email Template` / `Push Template` / `Sms Template` / `Telegram Template`. Нод лэнда, дестинейшна и `Push Message` в этом дереве нет.
- **`Split Group` и `Flow State` выключены** — у нод этого типа обеих осей нет вовсе (в отличие от `Flow Content` и `Remarketing Content`), единственный отбор ноды — её `Rule to pass`.
- **Привязки к флоу нет** — флоу при создании не выбирается: запрашиваются только `Name` (обязателен), `Description` (до 255 символов) и `Tags`.
- **Режим выбора зашит кодом:** дистрибуция всегда работает как `Multiple Nodes` (см. «Две оси выбора» ниже), поля в форме нет, и после создания режим не меняется — дерево отдаёт все прошедшие ноды, одну из них выбирает отправка.

### Почему из дерева `Message Templates` уходят только push — email, SMS и Telegram не отправляются

Рассылка отбирает из дерева только push-ноды — в дереве этого типа это `Push Template`. Ноды `Email Template`, `Sms Template` и `Telegram Template` создаются, валидируются и хранятся, но отправлять их из этого дерева нечем.

**Симптом:** в дереве собраны шаблоны письма / SMS / Telegram, серия рассылки отрабатывает, но сообщение не приходит и запись об отправке не появляется.

**Что делать:** держать в дереве push-ноды. Если push-нод в дереве нет вовсе, шаг рассылки пишет в лог ошибку `Drip: no push nodes selected from distribution tree` — дальше по симптому «пуши не приходят» *уточните у поддержки*.

### Где создать Message Templates-дистрибуцию — плитка в Settings и страница `Template distributions`

Два входа в один и тот же тип:

- **`Settings → Distributions → + Distribution` → плитка `Message templates`** — право то же, что у `Remarketing content` (`settings.distributions.edit.marketing-content`).
- **`Marketing → Template distributions`** (`/app/remarketing/message-template-distributions`) — отдельная страница со своей веткой прав (`marketing.template-distributions.*`). На выделенной странице пикер типов не показывается: создание открывается сразу.

**«Право на дистрибуции шаблонов выдал, а раздела у юзера нет».** Пункт `Marketing` в главном меню открывается по другим правам раздела (`marketing.messages.view`, `marketing.message-templates.view`, `marketing.sender-providers.view`, `marketing.flows.view`) — `marketing.template-distributions.view` в этот список не входит. Роли, у которой есть только оно, до страницы не дойти: выдавай его вместе с одним из прав, открывающих сам раздел `Marketing` ([how-to/permissions.md](../how-to/permissions.md)).

### Auto Rules — шаблон автоправил над Meta-сущностями, живущий в разделе `Automations`

Дистрибуция типа `Auto Rules` — шаблон автоправил (`Rule Template`): дерево фаз и правил, по которому движок сам останавливает, запускает и меняет бюджет кампаний, адсетов и объявлений Meta. Значения по визиту этот тип не подставляет — с визитами он не работает вообще. Тип есть только в ERP-билде.

Основной дом типа — раздел `Automations` со страницей `Rule Templates` и своей веткой прав `automation.templates.*`. Плитка `Auto rules` («Stop, start or change budget of Meta entities by phases and rules.») в пикере `Settings → Distributions → + Distribution` открывается по тому же праву `automation.templates.edit`, а не по `settings.distributions.*`. Как собрать шаблон, повесить ассайн и запустить — [how-to/auto-rules.md](../how-to/auto-rules.md).

Готовые шаблоны видны в обоих местах: таблица `Settings → Distributions` показывает шаблоны автоправил тенанта наравне с остальными дистрибуциями, а значение `Auto Rules` есть в её фильтре `Type`. Дерево у этого типа устроено иначе, чем у остальных типов, — см. «Дерево `Auto Rules` — только `Phase` и `Auto Rule`» ниже.

## Как устроено дерево дистрибуций: Folder, Strategy, Rule, Payload

Distribution = **дерево**. Чтение сверху вниз. Каждый Distribution состоит из `Folder`, `Strategy`, `Rule to pass` и `Payload`-нод, описанных ниже. Единственное исключение — тип `Auto Rules`: у него ни `Folder`, ни `Strategy`, ни payload-нод нет (см. «Дерево `Auto Rules` — только `Phase` и `Auto Rule`» ниже).

### Две оси выбора: сколько нод берётся из дерева vs как выбирается вариант внутри Strategy-узла

Выбор в дереве идёт по **двум независимым осям**, и путать их не нужно.

**Ось 1 — сколько нод дерево отдаёт (уровень всей дистрибуции), поле `Single Node` / `Multiple Nodes`.**

- **`Single Node`** — как только первый лист прошёл все свои проверки, обход дерева **останавливается**, и этот лист выигрывает. Так работают money- и маршрутные дистрибуции: `Payout`, `Revenue`, `Direct Traffic` (первый проходящий лист сверху вниз — результат).
- **`Multiple Nodes`** — дерево обходится до конца и собираются **все** прошедшие листья. Так работают `Fill Field` (за один проход можно заполнить несколько полей), `Remarketing Content` и `Message Templates`.

**Ось 2 — как Strategy-узел выбирает один вариант среди своих детей** (First / Weights / Conversions AI / Metrics AI, ниже). Эта ось живёт **внутри** `Strategy`-узла и решает тай-брейк между его дочерними payload-нодами; ось 1 решает, сколько таких выборов дерево в итоге вернёт визиту.

### Folder — контейнер правил

Контейнер для правил. Группирует Strategy + Payload-ноды.

### Strategy — как выбирается нода внутри Folder

Стратегия выбора внутри Folder. Полный набор из четырёх вариантов доступен в дереве Distribution:
- **First** — проход сверху вниз. Пробует первый вариант, если его правила не сработали — следующий. Если ни один — `No Payload`.
- **Weights** — распределение по весам. Одинаковые веса = равномерное распределение.
- **Conversions AI** — AI-оптимизация (`Thompson Sampling`) по выбранному `Conversion Type`. Не отключает остальные варианты — ретестирует.
- **Metrics AI** — AI-оптимизация (`Epsilon-Greedy`) по выбранной метрике за окно времени. Тот же механизм выбора варианта, что и на шаге, отличается целевым сигналом (метрика вместо конверсии).

Дерево использует ту же машинерию выбора вариантов, что и campaign-сплит на шаге Flow: одна фабрика стратегий, `Fill First` работает и в дереве, выбранный вариант записывается в реестр вариантов визита (`variants`).

**Граница редактора шага:** в редакторе шага Flow дестинейшна/контента (Campaign Split) доступны только `First` / `Weights`; полный набор из четырёх — в дереве Distribution.

См. [reference/glossary.md → Стратегии выбора](../reference/glossary.md).

### Когда лист применяется к визиту — fail-closed конвейер проверок

Лист (payload-нода) применяется, только если проходит **все** проверки в строгом порядке; любой промах **тихо** роняет узел (он ведёт себя как непрошедшее правило), обход продолжается со следующего:

1. **`active`** — узел не деактивирован.
2. **`configured`** — у узла заполнен обязательный по его типу ключ настроек (`Destination` без `destination_uuid`, `Landing` без `landing_uuid`, `Redirect`/`Reflect` без `url`, `Fill Field Text` без `text` и т.п. — считаются не сконфигурированными). Это и есть частая причина «настроил, но не применяется»: узел без цели скипается как непрошедшее правило.
3. **`flow_state`-матч** — только для `Flow Content` и `Remarketing Content`: `Flow State` узла (если задан) должен совпасть с текущим шагом.
4. **`split_group`-матч** — только для `Flow Content` и `Remarketing Content` (у `Campaign Content` этого гейта нет): если у визита уже выставлен цвет `Split Group`, цвет узла должен совпасть либо быть `Rainbow`.
5. **`Rule to pass`** — условия правила. Ноль условий = авто-pass.

У дерева `Message Templates` проверки 3 и 4 не применяются вовсе: `Flow State` и `Split Group` у этого типа выключены, и отбор ноды идёт только по `active` + `configured` + `Rule to pass`.

### Fill First — приоритетный вариант поверх весового распределения

На вариантах шага (variant), которые распределяются по стратегии `Weights`, рядом с полем веса есть тоггл **Fill First** (поле `is_fill_first`). Вариант с включённым Fill First попадает в отдельную корзину и становится приоритетнее весовых: пока в этой корзине есть хотя бы один Fill First-вариант, выбор идёт **случайно только среди них**, минуя весовое распределение. Весовые варианты рассматриваются, только когда Fill First-корзина пуста (например, все Fill First-варианты отфильтрованы правилами или капами).

Если Fill First включён на нескольких вариантах — между ними выбор **равновероятный** (вес при этом игнорируется полностью). Настройка живёт в строке варианта рядом с весом, редактируется вместе с ним.

Веса зажимаются в диапазон `0..100` и работают как относительные счётчики слотов, а не как проценты: вес `0` у обычного (не-Fill First) варианта **не гарантирует** его исключение из розыгрыша — чтобы вариант точно не участвовал, его **деактивируют**, а не ставят вес `0`.

### Rule to pass — условие применения ноды к визиту

Условие, при котором payload-нода применяется к визиту. Состоит из: поле визита + оператор + значение. Можно комбинировать через AND.

Правило **без условий** редактор открывает экраном плиток-пресетов (простые пресеты по одному полю визита, комбинированные с бейджем `Advanced` и плитка `Custom` — прежний полный конструктор); разбор пресетов — [models/flow-model.md](flow-model.md).

Пример: `Campaign Owner = Иван` AND `Visit Country = IT` → применить эту payload-ноду.

### Payload-ноды — результат, применяемый к визиту

Результат, который применяется к визиту. Тип payload зависит от типа Distribution.

### Дерево `Auto Rules` — только `Phase` и `Auto Rule`, без `Folder`, `Strategy` и payload-нод

Дерево дистрибуции типа `Auto Rules` собирается ровно из двух нод: в корне — только `Phase` (фаза), внутри фазы — только `Auto Rule` (правило). Правило — лист, фазы друг в друга не вкладываются, глубже двух уровней дерево не уходит. Нод `Folder`, `Strategy` и payload-нод (`Landing`, `Destination`, `Fill Field` и остальных) у этого типа нет вовсе.

Форму дерева проверяет не только интерфейс, но и сервер, поэтому нода, положенная не туда, отбивается сообщением:

- `Only Phase nodes are allowed at the root of an Auto Rules template` — в корень кладут не фазу;
- `Auto Rules nodes can be added only inside a Phase` — ноду кладут не в фазу;
- `Only Auto Rule nodes are allowed inside a Phase` — внутрь фазы кладут не правило.

Ось фаз (`Phase axis`) и переменные шаблона задаются в шапке самой дистрибуции, а не в нодах дерева. Что настраивается в фазе и в правиле — [how-to/auto-rules.md](../how-to/auto-rules.md).

## Какие Distribution Nodes можно добавить в дерево

| Нода | Иконка | Что | Где используется |
|---|---|---|---|
| Add Folder | | контейнер для правил | в любом типе Distribution, кроме `Auto Rules` |
| Add Strategy | | стратегия выбора | внутри Folder |
| Add Destination | | нода дестинейшна | Campaign Content, Flow Content |
| Add Landing | | нода лэнда | Campaign Content, Flow Content |
| Add Redirect | ↗ | URL-редирект | Campaign Content, Flow Content |
| Add Reflect | | внешний URL как payload (Reflect) | Campaign Content, Flow Content |
| Add Query | | default query (привязать source+campaign к домену) | Direct Traffic |
| Add Push Message | | push-шаблон из `Marketing → Message Templates` | Remarketing Content |
| Add Telegram Message | | telegram-сообщение рассылки | Remarketing Content |
| Add Revenue | | revenue-значение | Revenue |
| Add Payout | | payout-значение | Payout |
| Add Fill Field | | заполнение поля визита (с подтипами `Text`/`Image`/`Landing`/`Library`/`Destination`) | Fill Field |

Листья дерева `Message Templates` — отдельной таблицей ниже. Ноды `Phase` и `Auto Rule` в таблицу не входят: они бывают только в дереве типа `Auto Rules` ([how-to/auto-rules.md](../how-to/auto-rules.md)) и ни в одном другом типе не предлагаются.

### Ноды дерева `Message Templates` — четыре листа-шаблона и где их ставить

Дерево типа `Message Templates` допускает четыре листа, которых нет ни в одном другом типе дистрибуции:

| Нода | Что |
|---|---|
| `Email Template` | лист-шаблон письма (`email_subject` + `email_text`) |
| `Push Template` | лист-шаблон push: `push_title` и `push_message` обязательны (до 255 символов), `push_icon` опционален |
| `Sms Template` | лист-шаблон SMS (`sms_text`); в дереве и диалоге подписан `SMS template` |
| `Telegram Template` | лист-шаблон telegram-сообщения (`telegram_message`) |

У каждой из четырёх нод есть ещё поле `Name` — это имя строки в дереве, а не текст сообщения. Контент хранится **прямо в настройках ноды**: они не ссылаются на сущность из `Marketing → Message Templates`, хотя нода `Push Template` называется так же. Как собирается сам текст — [how-to/push-notifications.md](../how-to/push-notifications.md).

**Где ставить.** Лист-шаблон кладётся в корень дерева или внутрь `Folder`; вложить что-либо внутрь самого листа нельзя. Вложение в `Strategy` дерево тоже разрешает, но рассылка на такой ветке падает — см. раздел про `Strategy` ниже. `Folder` вкладывается в `Folder`, `Strategy` — в корень или в `Folder`.

### `Strategy` в дереве `Message Templates` — веса не работают, шаг рассылки обрывается целиком

Ноду `Strategy` дерево предлагает, но обработать её на рассылке нечем: как только под `Strategy` проходит условия хотя бы одна нода-шаблон, обработка дерева обрывается ошибкой — и шаг не отправляет **ничего**, включая шаблоны в соседних папках того же дерева. Взвешенного выбора шаблонов в дистрибуции этого типа нет.

**Симптом:** в дереве стоит нода `Strategy`, серия идёт по расписанию, но из этой дистрибуции не приходит ни одного сообщения.

**Что делать:** убрать `Strategy` из дерева — складывать шаблоны списком в `Folder` и разводить их правилами `Rule to pass` (например, по `aio.visit.language_code`). Среди всех нод, прошедших условия, отправка выбирает одну **равновероятно** — веса (`weight`, `Fill First`) на листьях-шаблонах не учитываются.

## Как управлять нодами дерева — элементы управления

Управляющие элементы каждой ноды:

| Элемент | Что |
|---|---|
| **Rule to pass** | условие применения ноды |
| **Toggle activity** | вкл/выкл ноду |
| **Show children** | раскрыть/свернуть |
| **Toggle locked** | заблокировать (нельзя disable, пока заблочена) |
| **Move Up / Move Down** | переставить порядок |
| **Clear recursively** | очистить со всеми вложенными (требует disabled) |
| **Duplicate recursively** | дублировать со всеми вложенными |

### `View distribution` — как посмотреть дерево, не имея прав на правку

`View distribution` (описание в меню — `View distribution tree`) открывает то же полноэкранное дерево дистрибуции, что и `Manage tree`, но **в режиме только чтение**. Экшен лежит в контекстном меню по правому клику на дистрибуции в `Settings → Distributions`, рядом с `Manage tree`.

Различие в правах: `View distribution` доступен по праву `settings.distributions.view`, `Manage tree` требует `settings.distributions.edit`. Юзеру, у которого есть только просмотр дистрибуций, дерево открывается — через `View distribution`.

В режиме чтения недоступны все редактирующие элементы управления: создание нод, редактирование ноды, `Edit weight`, `Split Group`, `Flow State`, `Rule to pass`, `Toggle locked`, `Toggle activity` — они скрыты или отключены. Навигация по дереву и раскрытие детей (`Show children`) работают как обычно.

### «В дереве видно не все ноды» — уровень грузится порциями через `Show more`

Уровень дерева (все дети одного родителя) рендерится **окном, а не целиком**, поэтому длинный уровень выглядит обрезанным — данные при этом целы, это не потеря нод.

**Признак:** под последней строкой уровня стоит служебная строка **`Show more`** со счётчиком вида `<показано> / <всего>`. Клик по ней расширяет окно; когда показанное упирается в уже загруженное, с сервера подтягивается следующая порция нод.

**Проверка:** дожимай `Show more`, пока строка не исчезнет — она пропадает ровно тогда, когда показаны все `N` из `N` нод уровня. Если строки `Show more` под уровнем не было изначально — уровень показан целиком, и недостающих нод действительно нет.

## Глобальная дистрибуция — общая для всех, из тенанта не редактируется

**Глобальная (системная) дистрибуция** — дистрибуция без тенанта-владельца: одна и та же для всех, заводит её команда AIO. В списках `Settings → Distributions` и `Marketing → Template distributions` её **нет** — она появляется только в селектах выбора дистрибуции.

В таком селекте, если хотя бы одна глобальная дистрибуция заведена, появляется тумблер **`Show global only`** («Только глобальные»). Он работает как режим: либо обычные дистрибуции, либо только глобальные — вперемешку они не показываются. Глобальная строка помечена логотипом AIO с тултипом `Shared across all — the same distribution is used everywhere`. Пока глобальных дистрибуций нет, ни тумблера, ни пометки в интерфейсе не видно.

### «Правлю глобальную дистрибуцию, а `Save` отбивается» — `System distribution is read-only`

Глобальную дистрибуцию можно открыть и даже начать править — отказ приходит **на сохранении**, а не при открытии: правка самой дистрибуции отбивается сообщением `System distribution is read-only`, правка её дерева — тем же сообщением с кодом `403`. Это ожидаемое поведение, а не сбой: из тенанта такая дистрибуция только для чтения, её выбирают, но не меняют.

## Что такое Business Models и как они связаны с Payout и Revenue Distribution

Преднастроенный набор бизнес-правил, на котором строятся `Payout` и `Revenue` distribution. По дефолту тенант имеет `Arrived Revenue$` (`AR`) — для приёма revenue из постбэков. Создаются в `Settings → Business models`, нужны при настройке Payout/Revenue distribution-ов. Детали определения, дефолтов (`Arrived Revenue$`, `Zero Payout`) и полей (`Type`/`Format`/`Formula`) — [models/business-model.md](business-model.md). Именно эту модель выбираешь в листе `Finance → Revenue/Payout Distribution` (`Edit revenue` / `Edit payout` → поле `Business model`, см. раздел «Как настроить Revenue и Payout Distribution в разделе Finance» ниже).

## Типичные сценарии применения Distributions

### Buyer UTM Distribution — как автоматически проставлять UTM-метки баеров через Fill Field

Самый частый use case `Fill Field Distribution`.

Без Distribution: каждый баер вручную вписывает свою UTM-метку в URL ссылки кампании. Ошибки, расхождения, потери в аналитике.

С Distribution:
1. В шаблоне тенанта обычно уже есть преднастроенная **`Buyer UTMs`** Distribution как пример — её не создают с нуля, а открывают через ПКМ → `Manage Tree` и переконфигурируют под себя. Тип — `Fill Field`, поле визита — `buyer_utm` (должно быть заведено в `Settings → Fields`, обычно есть в шаблоне).
2. Дерево правил по `Campaign Owner`:
   - Если `Campaign Owner = Иван` → `Fill Text: buyer_ivan`
   - Если `Campaign Owner = Маша` → `Fill Text: buyer_masha`
   - и т.д.

   Значение метки внутри ветки задаётся элементом **`Fill Field Text`** (поле `Text`). Новые ветки под других юзеров добавляются через `Add Folder → Add Fill Field Text` либо копированием существующей через `Duplicate recursively`. Отдельного режима-селектора «по владельцу vs по лаунчеру» нет — это просто разные `Rule to pass`: ветка проверяет `Campaign Owner UUID` (дефолт — владелец кампании, из которой пришёл визит) либо `Launcher` (лаунчер ссылки; для одной кампании, запускаемой несколькими юзерами) — какое поле поставить слева в правиле, по тому и делится (см. ниже «`Campaign Owner` vs `Launcher`»).
3. Во Flow ставится шаг **`Fill Field`** (в начале флоу, до фильтра). Он применяет правила автоматически — каждый визит получает правильную метку без участия баера. В шаблонном Flow обычно уже есть преднастроенный шаг **`Fill Buyer UTM`**; если его нет — добавляется нодой **`Fields by Distribution`**, в которой через cogwheel выбирается нужная Buyer UTM Distribution.
4. После того как `buyer_utm` заполнен, он передаётся не только в `Tracker → Destinations`, но и в Advertisers (`Settings → Advertisers → Edit`) — плейсхолдером `{{aio.visit.fields.buyer_utm}}` в нужном поле.

В шаблоне тенанта из `Fill Field`-дистрибуций есть **только** `Buyer UTM Distribution` — она и мапит `Campaign Owner UUID` на метку в поле `buyer_utm`. Заполнение других полей — `aff_id` / Offer ID, API-токены, fileId-ы под каждого баера — **не часть шаблона**: эти поля заводятся вручную в `Settings → Fields` и создаёт под них отдельные `Fill Field Distribution`.

#### По какому полю делить Buyer UTM — `Campaign Owner` vs `Launcher`

- **Дефолт** — `Campaign Owner` (владелец кампании). Срабатывает для всех визитов кампании.
- **`Launcher` (юзер, сгенерировавший ссылку)** — переключаются на него в **multi-buyer**-сценариях: одна кампания, ссылку из неё генерят несколько байеров через `Generate Link`, каждый со своим UTM/sub. `Launcher` фиксируется в момент генерации ссылки — у разных линков на ту же кампанию разный Launcher.
- У Direct Traffic визитов `Launcher = Unknown` (ссылки не генерили) — для Direct делить только по `Campaign Owner`. См. *уточните у поддержки* → «Launcher = Unknown».

### Payout Distribution по гео и источнику — разные ставки без правки кампаний

Use case: $3 за лида из IT, $5 за лида из DE на FB, $2 за всё остальное.

С Payout Distribution:
- Дерево с правилами:
  - `Visit Country = IT` AND `Source = Facebook` → `Add Payout: 3`
  - `Visit Country = DE` AND `Source = Facebook` → `Add Payout: 5`
  - (без правил, fallback) → `Add Payout: 2`

Применяется автоматически ко всем конверсиям. Меняется payout — правим только Distribution, кампании трогать не надо.

### Direct Traffic Distribution — как направить визит на голый домен без UTM

Use case: на рекламе стоит голый URL `mydomain.com` (без `?cuuid=...&suuid=...`). Куда направить такой визит?

Опция 1 (на уровне домена) — `Default Query Code` в `Edit Domain`:
- Указать там нужный `cuuid + suuid` → все визиты на голый домен идут в одну кампанию.

Опция 2 (централизованно) — `Direct Traffic Distribution`:
- В Settings → Distributions создать тип `Direct Traffic`.
- Дерево правил с `Add Query` нодами.
- Можно использовать `initial_path` в правилах:
  - `initial_path = /promo1` → кампания A
  - `initial_path = /promo2` → кампания B
  - (fallback) → кампания C

Один домен → разные кампании в зависимости от пути. Без Distribution это невозможно (Default Query Code один на домен).

### Как привязать домен к кампании связкой DTD + Campaign Content

Схема, когда голый URL домена должен открывать кампанию, а контент на Content-шаге выбирается централизованно: `Direct Traffic Distribution` привязывает домен к кампании (голый URL без параметров открывает её) + `Campaign Content Distribution` задаёт, какой контент показать на Content-шаге. На уровне кампании этот шаг при этом **не настраивается** — выбор идёт из дистрибуции.

- Под одну кампанию можно привязать **очень много** доменов через `Direct Traffic Distribution` (жёсткого лимита нет; на больших пулах — **10k+** — UI начинает тормозить, такие пулы поднимают **через API**). Управление пулом на этом объёме — вне AIO: свои скрипты, менеджащие дистрибуцию через API, который предоставляет AIO; отдельного инструмента или плейбука под это в AIO нет.
- В `Tech → Domains → Edit Domain → Direct Traffic Distribution` дистрибуция должна быть **выбрана на самом домене**, иначе домен её не использует — правил «кампания + домен» в `Direct Traffic Distribution` недостаточно.
- После привязки домена кампания **сразу доступна** по прямому переходу — на неё пойдёт любой заход на голый домен.
- По дефолту (без DTD), чтобы визит попал в кампанию, в URL обязательно **два** параметра: `UUID кампании` + `UUID source`. DTD снимает это требование — вход идёт по голому домену без параметров.

### Многостраничный лэнд через DTD — поведение init и 404 на подпутях

Для многостраничного лэнда ссылка выглядит как `домен/article-slug`: подстраница открывается, только если у лэнда есть Sub-page с таким `Path` **и** этот путь в наборе разрешённых (`Allowed paths`). Такой подстраницы нет — открывается `index.html` того же лэнда.

На `init` (первый заход) DTD кидает любой подпуть на `index.html`; **внутри сессии** несуществующий подпуть отдаёт `404` — это штатное поведение маршрутизации: подставляется только явно объявленный `Path`, неизвестный путь не резолвится в реальную страницу. Механика `init` / `Allowed paths` / Sub-pages — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) и *уточните у поддержки*.

### Почему `aio.*`-плейсхолдеры не подставляются на лэнде-заглушке

Лэнд-заглушка (нейтральная страница, которую флоу отдаёт вместо основного контента) открывается **вне контекста визита**, поэтому SDK-плейсхолдеры на ней **не подставляются** — `aio.*` не резолвятся. Практическое следствие: динамические подстановки на такой странице не сработают, вёрстка должна быть самодостаточной.

### Рассылка пушей по визитам — какую дистрибуцию под неё заводить

Use case: визит был на лэнде, через N часов хотим послать ему push.

**Дефолтный путь — ремаркетинг-кампания:** она собирает аудиторию визитов и рассылает по расписанию, а тексты берёт из дистрибуции типа `Message Templates`. Процедура целиком — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md).

Дистрибуция `Remarketing Content` (дерево из `Add Push Message` / `Add Telegram Message`, привязанное к `Notifications Flow`) — кастомный путь для сценариев, которые собирают во флоу вручную: [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

### Campaign Content Distribution — разные лэнды по гео в одной кампании

Use case: один Flow + один Content step + лэнды на 10 языках.

С Campaign Content Distribution (для этой одной кампании):
- В Settings → Distributions тип `Campaign Content`, привязать к Campaign.
- Дерево с `Add Landing` нодами по правилам `Visit Country` / `Visit Language`.

Альтернатива — настроить State Rules прямо на Content step во Flow. Distribution предпочтителен, если правил много или нужны Flow Content (несколько шагов одной логикой).

## Что важно знать про Distributions на практике

Каждый пункт — частая точка путаницы; заголовок и первая строка ведут симптомом.

### «Изменил Distribution — не вижу эффекта» — Flow не использует Fields by Distribution

Distribution — мощный инструмент, но важно различать: где значение правят **на самой кампании** (per-campaign) vs где через **Distribution** (централизованно). Симптом «изменил Distribution, эффекта нет» почти всегда значит, что Flow использует статичный шаг `Fill Fields`, а не `Fields by Distribution` — проверь, что во Flow стоит именно `Fields by Distribution`.

### Fields by Distribution ставить в начало Flow — до фильтра

Шаг `Fields by Distribution` в начале Flow (**до** фильтра) — критично. Если шаг стоит после `Filter`, метки могут не примениться к отклонённым визитам.

### `Direct Traffic Distribution` vs `Default Query Code` на домене

Два пути для одной задачи. `Default Query Code` проще, `Direct Traffic Distribution` мощнее (разделение по `initial_path`). На **одном** домене их не миксовать — конфликтуют; при переходе на DTD очистить `Default Query Code` (см. [how-to/distributions.md](../how-to/distributions.md)).

### Метки баеров — через `Campaign Owner`-правила, не ручным вводом UTM

Метки баеров заводить через правила по `Campaign Owner` в `Fill Field Distribution`, а не ручным прописыванием UTM в каждой ссылке. Это убирает источник ошибок и расхождений в аналитике.

### `Payout Distribution` обновляется по дате конверсии

Поменялось правило `Payout Distribution` — оно применится только к **новым** конверсиям. Старые конверсии остаются с тем payout, что был на их дату.

### `Retrigger trackers` — не то же, что правка Distribution

`Retrigger trackers` на конверсии — отдельное от Distribution действие. Перепушивает постбэки в Source (рекламную платформу, партнёрку), но **не меняет** payout/revenue в AIO. Правку сумм делает Distribution.

## Как настроить Revenue и Payout Distribution в разделе Finance

Revenue- и Payout-дистрибуции редактируются в разделе **`Finance`** (`/app/finance/distributions`) — две вкладки: **`Revenue Distribution`** (доход с конверсии) и **`Payout Distribution`** (выплата баеру / сорсу). *(Business Models, на которых строятся листья, создаются в Settings, см. раздел «Что такое Business Models» выше.)*

**Структура (обе вкладки одинаковы).** Rule-routed дерево. Колонки: `Node` · **`Rule to pass`** (условие, напр. `( Accept everything )`) · `Touched by` · `Touched at` · `Node UUID` · `Created`. Тулбар: **`Folder`** (добавить корневую папку) + **`Revenue`** / **`Payout`** (добавить корневой лист).

**Дефолт в шаблоне тенанта.** Revenue-дерево = папка `Revenue by Postback for All Conversions` → лист `Arrived Revenue $` (Business Model `AR – Arrived Revenue`, сумма 0,00). Payout-дерево = папка `Example Buyer Payouts`. Оба правила — `Accept everything`.

**Лист (Revenue/Payout node) — модалка `Edit revenue` / `Edit payout`:**
- **`Business model`** (обяз.) — выбор из бизнес-моделей тенанта (в шаблоне одна — `AR – Arrived Revenue`; модели per-tenant, заводятся в Settings → Business Models).
- **`Revenue`** / **`Payout`** (обяз.) — сумма (число).

**Экшены ноды (правый клик):** `Edit settings` · `Edit rule` (правило прохода) · **`Add Folder Inside`** · **`Add Revenue Inside`** / **`Add Payout Inside`** (строят дерево вглубь) · `Open node` (раскрыть) · `Disable` / `Enable node` · `Lock node` · **`Clear recursively`** (удалить ветку; единственная операция удаления) · `Duplicate recursively` · `Move up` / `Move down` · `Show logs`. *(Клик по имени папки = быстрый `Edit folder` — только имя.)*

Концептуально это тот же distribution-движок (раздел «Как устроено дерево» + Strategy + действия с нодами), применённый к деньгам. `Revenue` и `Payout` — две денежные стороны конверсии: оба назначаются конверсиям автоматически по правилам дерева (кампании трогать не надо — см. «Payout Distribution по гео и источнику» выше). Важно: **Profit = Revenue − Cost**, а не − Payout (Cost — рекламный спенд; подробнее — *Медиабаинг и место AIO в нём*). `Payout Distribution` обновляется по дате конверсии (см. раздел «Что важно знать про Distributions на практике» выше).

## Общие принципы применения Distributions — централизация и капы

### Принцип «выноси в дистрибуцию»

Общий принцип AIO: если что-то (оффер, поле, настройка) повторяется во множестве кампаний — выносить в **дистрибуцию** (или уровень флоу), а не править по одной. В продвинутых сетапах выбор офферов/PWA забирают у байеров в **централизованную дистрибуцию** менеджера. Байер — «кнопка запуска».

### Капы и поведение Destination Full

При переполнении капы визиты уходят в fallback через `Destination Full` **в момент открытия Destination**, не захода в кампанию — уже зашедшие, но не дошедшие до Destination пойдут в fallback. Типы кап (`Daily` / `Infinity`) и механика подсчёта — [models/conversion-model.md](conversion-model.md).

Архивация Destination **не убирает** его из кампаний, где он уже используется — продолжает распределять трафик. Чтобы отключить — убрать/деактивировать **в самой кампании** (или через дистрибуцию). Архивация как «фильтр во флоу» сейчас не работает.

### Деактивация варианта vs нулевой вес — что выбрать

Лучше **деактивировать** вариант, чем ставить вес 100/0: деактивированный красится серым в аналитике, сразу видно.

## Смежные темы

- [models/flow-model.md](flow-model.md) — Flow и шаг `Fields by Distribution`
- [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) — поля визита, как их заполняют
- [reference/glossary.md](../reference/glossary.md) — определения (`Distribution`, `Distribution Nodes`, `Business Model`, и т.д.)
- [how-to/distributions.md](../how-to/distributions.md) — пошаговое создание Distribution (вкл. Buyer UTM / Fill Field)
- [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md) — рассылки по аудитории визитов: где выбирается дистрибуция `Message Templates`
- [how-to/push-notifications.md](../how-to/push-notifications.md) — как собирается текст сообщения в нодах-шаблонах
- [how-to/auto-rules.md](../how-to/auto-rules.md) — автоправила Meta: шаблон типа `Auto Rules`, ассайн, режимы запуска

## Как правильно структурировать Distribution и Flow дерево — эвристики

### Split Groups vs Rules — простота vs гибкость

**Когда:** дизайн флоу с множественными вариантами (например, разные офферы под разные прилэнды).

**Делай:** выбирай между двумя путями:

- **`Split Groups` (связка по цветам)** — упрощение. Вариант на прилэнде с цветом `green` → автоматически идёт на оффер с цветом `green`. UI понятный, настройка быстрая.
- **`Rules` (`All` + сопоставление по полю)** — гибче, но многословнее. Каждое правило прописывается явно через Allowance Rules.

**Когда что:**

- Простые «один-к-одному» матчи по цветам — **Split Groups**.
- Сложная логика (несколько условий, AND/OR, кросс-поля) — **Rules**.

**Важно в Rules:** забивай **по внутреннему UUID-полю**, не по человекочитаемому имени. UUID не меняется при переименовании, имя — может, и правило сломается.

**Почему:** Split Groups — простота за счёт жёсткой модели «цвет = поле визита `Split Group`», который пишет любая нода-выборщик (`Content` / `Fill fields` / `Fill form` / `Fields by distribution`). Rules — это полная мощь Allowance Rules, ценой verbose-настройки.

Также см. [models/flow-model.md](flow-model.md) → «Split Groups — маршрутизация цветами».

### Fill Field Distribution — сначала по байеру, потом по гео

**Когда:** проектируешь дерево `Fill Field Distribution` (или `Payout Distribution`) под команду баеров — типично заполняешь `Buyer Team`, `Alter ID`, `Buyer Subaccount` по правилам.

**Делай:** дели **сначала по байеру** (top folder = `Campaign Owner`), **потом по гео** внутри байера.

Если одно и то же значение (например, один `Alter ID`) повторяется во всех гео байера — **выноси его на уровень выше** (на саму папку байера, не на каждую гео-папку).

**Пример структуры:**

```
Fill Field Distribution
├── Buyer = Alex
│   ├── Alter ID = "alex_default"   ← на уровне байера
│   ├── Geo IT → ... (специфичное только для IT)
│   ├── Geo DE → ... (специфичное только для DE)
│   └── (fallback) → ...
├── Buyer = Bob
│   └── ...
```

**Не делай:** плоское дерево с десятками одинаковых правил (`Buyer=Alex, Geo=IT → Alter=alex_default`, `Buyer=Alex, Geo=DE → Alter=alex_default`...) — это **антипаттерн**, дублирование значений → правки в N мест.

**Почему:** дерево читается сверху вниз, более общее наверху, конкретное — внутри. Дублирование значений = когда меняется одно — приходится менять во всех листьях.

Также см. раздел «Как устроено дерево дистрибуций» выше.
