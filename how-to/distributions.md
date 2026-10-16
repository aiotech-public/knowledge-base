---
id: distributions
title: Distributions (How-to)
description: Создание Distributions всех типов — Direct Traffic, Content, Fill Field, Message templates (тексты ремаркетинг-рассылки), money-деревья в Finance. Управление деревом (Duplicate, Disable, Clear recursively).
doc_type: how-to
builds: [erp, mtk]
related: [distributions-model, auto-rules, flow-model, remarketing-campaigns, flow-editor, push-notifications, domains, glossary]
language: ru
updated: 2026-10-16
---

# Distributions (How-to)

Пошаговые процедуры создания и редактирования Distributions всех типов. Концептуальное устройство (типы, дерево, Strategy, Business Models) — в [models/distributions-model.md](../models/distributions-model.md).

В ERP просмотр раздела требует `settings.distributions.view`. **В MTK основной раздел `Distributions` доступен с платным модулем `Distribution`** и правами на дистрибуции или Content Library. Внутри показываются доступные дистрибуции и группы плейсхолдеров, включённые для MTK. Owner или Admin тенанта включает модуль кнопкой `Enable` в секции `Modules` биллинг-модалки; как он попадает в счёт — *Билды, тарифы, триал и статусы тенанта*.

---

## TL;DR

- В ERP новые Distributions создают в `Settings → Distributions → +Distribution`; доступные типы зависят от прав. В MTK готовые деревья открывают в разделе `Distributions`.
- Самые частые: **Direct Traffic** (голый домен → кампания), **Campaign content / Flow content** (лэнд по правилу), **Fill Field** (заполнение полей визита по правилу).
- Внешний сайт как payload — нода `Add Reflect` **внутри** Campaign content; механика внешних лэндов.

### Где привязывают DTD и открывают Message templates

- На каждом домене для DTD должна быть **явно выбрана** дистрибуция в `Tech → Domains → Edit domain` (поле `Direct traffic distribution`) — иначе домен её не использует. Content Distribution к домену не привязывают: домен в ней — условие правила.
- **Message templates** — дерево текстов для ремаркетинг-рассылки. В ERP страницу открывают через `Marketing → Template distributions`, в MTK — через `PWA → Template distributions` при включённом PWA-модуле (см. секцию ниже).

### Как устроены Auto Rules и управление деревом

- **Auto rules** — плитка шаблона автоправил над Meta-сущностями (только ERP). Дерево у типа другое (`Phase` → `Auto Rule`), а настраивают и запускают его в разделе `Automations` — [how-to/auto-rules.md](auto-rules.md).
- Управление нодами — ПКМ: `Duplicate recursively`, `Disable node` → `Clear recursively` (активные ноды нельзя удалить).
- Глобальной (системной) дистрибуции в списке нет — её выбирают в селекте дистрибуции, где глобальные идут в конце списка после разделительной линии; править её из тенанта нельзя ([models/distributions-model.md](../models/distributions-model.md)).

---

## Как создать дистрибуцию и построить дерево

Общий порядок — создать дистрибуцию, настроить дерево и подключить его там, где этот тип используется. Окна создания и доступные ноды различаются по типам.

### Создать дистрибуцию (Name, Description, Tags, тип)

`Settings → Distributions → +Distribution → выбрать тип`. В окне создания: `Name`, `Description` (опц.), `Tags` (опц. — для поиска: гео, тест-метка и т.п.). Подтверждение — `Confirm`. Для типа `Flow Content` дополнительно появляется обязательное поле `Select flow`: из этого Flow берётся список шагов для настройки `Flow state` у нод дерева. Оба типа — `Flow Content` и `Campaign Content` — выбираются вариантом `Content distribution` на Content-шаге. У `Flow Content` листья можно развести по `Flow state`, чтобы на разных шагах выбирался разный контент. Для money-дистрибуций сначала выбирается плитка `Payout settings` / `Revenue settings`, затем те же `Name` / `Description` / `Tags`.

Плитка `Auto rules` в пикере `Settings → Distributions → +Distribution` открывает не общую форму создания дистрибуции, а свой редактор шаблона автоправил (`Create auto rules template`): кроме `Name`, `Description` и `Tags` в нём задаются ось фаз и переменные шаблона. Премодалка выбора `Simple rule` / `Advanced` есть только на странице `Automations → Rule Templates` — из пикера настроек сразу открывается полный редактор. Процедура целиком — [how-to/auto-rules.md](auto-rules.md).

### Построить дерево дистрибуции (Manage tree, Add Folder, ноды)

В ERP: `Settings → Distributions` → правый клик по дистрибуции → `Manage tree` открывает дерево отдельной страницей. В MTK дерево открывается из меню `Distributions`.

`Add Folder` создаёт папку под правило; внутрь добавляются ноды разрешённых типов. Payload-ноды можно добавлять и прямо в корень. У `Auto Rules` другое дерево: `Phase` → `Auto Rule`, без папок. Money-дистрибуции `Payout` / `Revenue` редактируются тем же деревом; вход через `Finance` описан ниже.

В ERP без прав на правку дерево тоже открывается: в том же меню есть экшен `View distribution` — то же дерево в режиме только чтение, по праву `settings.distributions.view`; действия правки в этом UI требуют `settings.distributions.edit`. Что именно в нём отключено — [models/distributions-model.md](../models/distributions-model.md).

- **Настройки ноды** — клик по предупреждению `Please configure Landing...` или по её текущему содержимому; откроется форма, например `Edit landing`.
- **Правило** — клик по полю `Rule to pass` (открывается Rule Builder). Пустое правило открывается экраном плиток-пресетов; полный конструктор «поле + оператор + значение» — плитка `Custom` ([models/flow-model.md](../models/flow-model.md)).
- **Ноду внутрь** — клик по иконке нужного типа ноды либо ПКМ → выбор типа из меню.

Новый payload-лист создаётся выключенным. После настройки включи его через ПКМ → `Enable node`, если он ещё неактивен. Папки, `Strategy` и `Phase` при создании уже активны.

В шапке дерева — имя дистрибуции (тип — в подсказке) и меню `Settings` с двумя переключателями: `Auto-open` — при открытии дерева раскрываются ветки, открытые в прошлый раз (`Branches opened last time are restored on load`), и `Auto scroll` — после загрузки дерево прокручивается туда, где вы остановились (`After loading, the tree scrolls back to where you left off`).

В режиме `Single Node` обход идёт сверху вниз и останавливается на первом прошедшем листе. В `Multiple Nodes` собираются все прошедшие листья; внутри `Strategy` результат зависит от выбранной стратегии. См. концепт-уровень в [models/distributions-model.md](../models/distributions-model.md).

### Как построить дерево Payout / Revenue через Finance

Money-дистрибуции (`Payout` / `Revenue`) создаются в `Settings → Distributions`; открыть дерево можно через `Manage tree` или раздел **`Finance`**: выбрать созданную дистрибуцию → `Add Folder` → правило (`Edit rule`) → внутри `Add Folder Inside` (подпапки под conversion type) и/или `Add Revenue Inside` / `Add Payout Inside` (лист со значением). У листа в `Edit settings` выбирается `Business model` + сумма `Revenue` / `Payout`; настроенный выключенный лист включают через `Enable node`. Строже/специфичнее правило — выше в папке. Концепт-уровень и дефолтное дерево тенанта — в [models/distributions-model.md](../models/distributions-model.md) § «Как настроить Revenue и Payout Distribution в разделе Finance».

### Подключение дистрибуции во флоу — два способа

1. **Отдельный шаг-нода.** Для `Fill Field` — нода `Fields by distribution`; для рассылки — узел `Drip schedule` во флоу типа `Notifications`, он выбирает дистрибуцию типа `Message Templates` ([how-to/remarketing-campaigns.md](remarketing-campaigns.md)). Настройки шага открываются **модалкой** по клику на ноду в редакторе флоу (кроме портов переходов и ручки перетаскивания), дистрибуция выбирается из выпадающего списка.
2. **Новый payload внутри существующего шага.** В редакторе флоу кликнуть по ноде шага — откроется модалка настроек; в поле `Payload type` выбрать `Content distribution`, затем выбрать дистрибуцию. Выбор доступен в самом Flow либо в кампаниях через `+Add another variant → Content distribution` (зависит от того, flow-only шаг или доступен в кампаниях).

Кнопка `Apply settings` внизу модалки шага ничего не применяет — форма пишет в шаг живьём, кнопка только закрывает окно; флоу после этого всё равно нужно сохранить ([models/flow-model.md](../models/flow-model.md)).

## Direct Traffic Distribution + привязка к домену

Когда: голый URL без UTM (`mydomain.com`) должен открыть конкретную кампанию. Механика DTD и Default Query, порядок дополнения параметров — [models/distributions-model.md](../models/distributions-model.md).

**Путь:** `Settings → Distributions → +Distribution → Direct traffic`.

**Шаги:**

1. `Settings → Distributions → +Distribution → Direct traffic`, задать `Name` (+ опц. `Description` / `Tags`), `Confirm`.
2. ПКМ по созданной дистрибуции → `Manage tree` → `Add Query`.
3. Через `Rule to pass` задать правило по домену. Кликнуть `Please configure Query...` → в форме `Edit query` выбрать `Campaign` и `Source` (оба обязательны) → `Confirm`. Если лист ещё выключен — ПКМ → `Enable node`.
4. Для разделения трафика внутри домена — правило по `initial_path` (`fr` → одна кампания, `de` → другая). В URL это `/fr` и `/de`, но в сравниваемом значении крайние `/` убраны; поддерживаются и правила по `IP` / `city` / `country`.
5. Перейти `Tech → Domains → <Domain> → Edit domain → Direct traffic distribution` → выбрать созданную Direct Traffic Distribution. Без этого шага домен дистрибуцию не использует.
6. Проверь `Default query` на домене. DTD первой дополняет пустые параметры запроса; затем подходящий `Default query` дополняет оставшиеся пустые параметры. Если вся атрибуция должна идти только из DTD — очисти `Default query`.

После привязки домена к кампании через DTD она **доступна по прямому переходу без отдельного запуска** — в неё пойдёт заход, прошедший правила дерева и проверки доступности домена, кампании и тенанта; привязка доходит до трафика в пределах нескольких минут (настройки домена — до ~10 минут, *Что AIO проставляет и делает сам — автоматические обновления и их лаг*).

## Content Distribution (домен → лэнд)

Когда: один Content-шаг во флоу должен показать разный лэнд в зависимости от домена/гео визита.

**Путь:** `Settings → Distributions → +Distribution → Campaign content` (для нескольких шагов одного флоу — `Flow content`).

**Шаги:**

1. Выбрать `Campaign content` как тип, задать имя.
2. Построить **дерево**:
   - `Add Landing` — какой лэнд показывать.
   - Условие прохода ноды задаётся через поле **`Rule to pass`** (клик по нему открывает редактор правила).
3. В правиле: поле визита (напр. `country_code`) + оператор + значение → сохранить. Настроенный выключенный лист включить через ПКМ → `Enable node`.

Готовую `Campaign content`-дистрибуцию подключают к Content step во флоу — секция «Подключение Content Distribution в Content step флоу» ниже.

### Подключение Content Distribution в Content step флоу

**Путь:** `Edit flow → <Content step> → добавить вариант → Payload type: Content distribution → выбрать`. Тот же тип варианта — `Content distribution` («Дистрибуция контента») — выбирается и в `Edit campaign` на карточке шага, если шаг настраивается на уровне кампании.

Также см. [models/flow-model.md](../models/flow-model.md).

### Когда Content-шаг переводят в Flow Only

Если всем кампаниям нужен один набор настроек Content-шага, можно выбрать `Flow only` в `Settings availability`: шаг становится одинаковым для всех кампаний этого Flow, а внутри шага вариантом стоит дистрибуция. Из `Edit campaign` шаг пропадает — на уровне кампании его не настраивают вовсе. Для выбора дистрибуции в отдельной кампании оставляют `Campaign template` или `Campaign only`; обязательного требования `Flow only` у дистрибуции нет. Как сменить Settings availability шага — [how-to/flow-editor.md](flow-editor.md).

## Fill Field Distribution

Когда: заполнить поле визита по правилам (метка баера `buyer_utm`, `Alter ID`, `Affiliate ID`, API-токен партнёрки, ID-маркеры и т.п.).

**Путь:** `Settings → Distributions → +Distribution → Fill field`.

**Шаги:**

1. Выбрать плитку `Fill field` как тип.
2. `Add Folder` — папка-контейнер.
3. На папке задать `Rule to pass` (например, `Campaign Owner = X`).
4. Внутри папки — `Add Fill field text Inside` → выбрать `Field` (пользовательское поле типа Visit с форматом String, UrlEncode или Phone) и заполнить `Text`.
5. `Confirm`. Если лист ещё выключен — ПКМ → `Enable node`. Подключить во флоу через шаг `Fields by distribution`.

Дерево строй по крупному признаку сверху, по мелкому — внутрь (например, верхний уровень `Campaign Owner`, гео — внутри папки владельца). Эвристику «сначала по байеру, потом по гео» см. в [models/distributions-model.md](../models/distributions-model.md).

## Как создать дистрибуцию шаблонов сообщений (`Message templates`)

Когда: ремаркетинг-рассылке нужен набор текстов, из которого она возьмёт сообщение для конкретного визита. Чем этот тип отличается от остальных (какие ноды допускает, почему у него нет `Split group` и `Flow state`) — [models/distributions-model.md](../models/distributions-model.md).

**Путь:** `Marketing → Template distributions` → кнопка создания. Пикер типов на этой странице не показывается — сразу открывается создание нужного типа. Тот же тип создаётся и из общего пикера: `Settings → Distributions → +Distribution → Message templates`.

В MTK страница находится в `PWA → Template distributions`, когда у тенанта включён модуль `PWA + Push`. Для просмотра нужно `marketing.template-distributions.view`, для перехода к дереву — `marketing.template-distributions.edit`. Создание на этой странице также открывает форму типа `Message templates`.

### Создать Message templates и настроить push-лист

1. Задать `Name` (обязательно), опционально `Description` и `Tags` → `Confirm`. Флоу при создании **не выбирается**: привязки к флоу у этого типа нет.
2. ПКМ по созданной дистрибуции → `Manage tree`.
3. Разложить шаблоны: `Add Folder` под группу, внутрь — ноды `Push template` (лист-шаблон можно положить и прямо в корень дерева).
4. В ноде заполнить `Name` (это имя строки в дереве, а не текст сообщения), `Push title` и `Push message`; `Push icon` — опционально. Плейсхолдеры и спинтакс в текстах — [how-to/push-notifications.md](push-notifications.md). Подтвердить форму; настроенный выключенный лист включить через `Enable node`.

### Как Message templates выбирает сообщение для серии

Развести шаблоны условиями: на каждой ноде `Rule to pass` (например, по `aio.visit.language_code`). `Split group` и `Flow state` у этого типа выключены. В шаге серии из прошедших правила push-листьев случайно выбирается один.

Выбрать готовую дистрибуцию в шаге серии рассылок (`Drip schedule`) ремаркетинг-кампании — [how-to/remarketing-campaigns.md](remarketing-campaigns.md).

Ноды `Email template` / `SMS template` / `Telegram template` в ERP можно создать, но шаг серии их не отправляет. Если внутри `Strategy` проходит лист шаблона сообщения, обработка завершается ошибкой. Для push-шаблонов используй `Folder` с `Push template`-листами — [models/distributions-model.md](../models/distributions-model.md).

## Внешний сайт как payload дистрибуции — нода `Add Reflect`

Когда: контент шага должен отдаваться с внешнего сайта, без миграции файлов в AIO.

Нода **`Add Reflect`** внутри `Campaign content` (как и `Add Redirect`) даёт тот же payload-тип, что одноимённая опция шага во флоу: в самой ноде указывается целевой внешний URL.

Подключается дистрибуция в Content step флоу так же, как обычная Content Distribution (клик по ноде шага → модалка настроек → выбор дистрибуции; см. «Подключение дистрибуции во флоу — два способа»).

## Как управлять нодами дерева дистрибуций (Duplicate, Disable, Clear)

### Duplicate (рекурсивно)

Дублирует ноду со всеми вложенными.

**Путь:** ПКМ → `Duplicate recursively`. Копия появляется в дереве; её параметры настраиваются через `Edit settings`. Имя можно задать в формах тех типов, где есть поле `Name`, например у папки или push-шаблона.

### Disable node + Clear recursively

**Активные ноды нельзя удалить** напрямую. Чтобы удалить — сначала деактивировать:

1. ПКМ → `Disable node` (строка визуально приглушается).
2. ПКМ → `Clear recursively`. Удалятся только неактивные ноды, у которых нет активных потомков. Для удаления всей ветки сначала деактивируй все её ноды.

Остальные действия (`Toggle activity`, `Toggle locked`, `Move Up` / `Move down`, `Show children`) — полная таблица в [models/distributions-model.md](../models/distributions-model.md) § «Как управлять нодами дерева».

## Частые ошибки и грабли при работе с Distributions

- **Создал DTD, но домен не использует.** Не выбрана дистрибуция в атрибуте домена `Tech → Domains → <D> → Edit domain → Direct traffic distribution`.
- **Голый домен с DTD доступен без отдельного запуска.** Прямые заходы, прошедшие правила дерева и проверки доступности, идут в кампанию, как только привязка дошла до трафика (до ~10 минут), — настройка шага `Filter` для такой кампании.
- **Active нода не удаляется.** Сначала `Disable node`, потом `Clear recursively`.
- **В дереве видно не все ноды уровня.** Это окно отрисовки и подгрузка страниц, а не потеря данных: под последней строкой уровня есть строка `Show more` со счётчиком `<показано> / <всего>` — дожимай её, пока не исчезнет (механика — [models/distributions-model.md](../models/distributions-model.md)).
- **`Default query` вместе с `Direct traffic distribution`.** DTD дополняет пустые параметры первой, затем `Default query` — оставшиеся пустые. Очисти `Default query`, если он больше не должен дополнять атрибуцию.
- **Разделение Direct Traffic по путям (`initial_path`), где оно не нужно.** Возможность есть (см. шаг 4 DTD), но для голых Direct-Traffic-запусков обычно не требуется — лишнее усложнение.
- **Большие пулы доменов (порядка 10k) на одну дистрибуцию.** Работать с таким деревом в UI может быть неудобно; большой пул можно создавать и управлять им своими скриптами через API.

## Смежные темы

- [models/distributions-model.md](../models/distributions-model.md) — концепт-уровень: 9 типов, дерево, Strategy, Business Models, Buyer UTM Distribution.
- [how-to/auto-rules.md](auto-rules.md) — дистрибуция типа `Auto Rules`: шаблон автоправил Meta, ассайн и запуск в разделе `Automations`.
- [models/flow-model.md](../models/flow-model.md) — Content step, Fill Fields step, Fields by Distribution.
- [how-to/domains.md](domains.md) — `Default Query Code`, привязка домена к серверу.
- [how-to/push-notifications.md](push-notifications.md) — как собирается текст сообщения (плейсхолдеры, спинтакс).
- [how-to/remarketing-campaigns.md](remarketing-campaigns.md) — рассылка по аудитории визитов: где выбирается дистрибуция `Message Templates`.
- [reference/glossary.md](../reference/glossary.md) — `Direct Traffic Distribution`, `Content distribution`, `Fill Field Distribution`, `Add Folder`, `Clear recursively`, `Strategy First/Weights/ML`.
