---
id: distributions
title: Distributions (How-to)
description: Создание Distributions всех типов — Direct Traffic, Content, Fill Field, Message templates (тексты ремаркетинг-рассылки), money-деревья в Finance. Управление деревом (Duplicate, Disable, Clear recursively).
doc_type: how-to
builds: [erp, mtk]
related: [distributions-model, flow-model, remarketing-campaigns, flow-editor, push-notifications, domains, glossary]
language: ru
updated: 2026-08-12
---

# Distributions (How-to)

Пошаговые процедуры создания и редактирования Distributions всех типов. Концептуальное устройство (типы, дерево, Strategy, Business Models) — в [models/distributions-model.md](../models/distributions-model.md).

В ERP раздел доступен всем; **в MTK — только с платным модулем `MTK Distribution`** (отдельная вкладка `Distributions` с `Content Library` внутри), его подключает команда AIO — *Билды, тарифы, триал и статусы тенанта*.

---

## TL;DR

- Все Distributions — в `Settings → Distributions → +Distribution`.
- Самые частые: **Direct Traffic** (голый домен → кампания), **Campaign content / Flow content** (лэнд по правилу), **Fill Field** (заполнение полей визита по правилу).
- Внешний сайт как payload — нода `Add Reflect` **внутри** Campaign content; механика внешних лэндов.
- На каждом домене для DTD/CD должна быть **явно выбрана** дистрибуция в `Tech → Domains → Edit Domain` — иначе домен её не использует.
- **Message templates** — дерево текстов для ремаркетинг-рассылки; у него есть своя страница `Marketing → Template distributions` (см. секцию ниже).
- Управление нодами — ПКМ или scroll: `Duplicate recursively`, `Disable node` → `Clear recursively` (активные ноды нельзя удалить).
- Глобальной (системной) дистрибуции в списке нет — её выбирают в селекте тумблером `Show global only`, править её из тенанта нельзя ([models/distributions-model.md](../models/distributions-model.md)).

---

## Как создать дистрибуцию и построить дерево

Шаги одинаковы для всех типов; отличия типов — в секциях ниже.

### Создать дистрибуцию (Name, Description, Tags, тип)

`Settings → Distributions → +Distribution → выбрать тип`. В окне создания: `Name`, `Description` (опц.), `Tags` (опц. — для поиска: гео, тест-метка и т.п.). Подтверждение — `Confirm`. Для типа `Flow Content` дополнительно появляется обязательное поле выбора `Flow` (флоу, к которому применяется дистрибуция). Готовую `Flow Content`-дистрибуцию можно подключить **к одному конкретному шагу кампании или сразу ко всем шагам** (в отличие от `Campaign Content`, которая управляет одним элементом за раз). Для money-дистрибуций сначала выбирается плитка `Payout settings` / `Revenue settings`, затем те же `Name` / `Description` / `Tags`.

### Построить дерево дистрибуции (Manage Tree, Add Folder, ноды)

В `Settings → Distributions` правый клик по дистрибуции → `Manage Tree`. Внутри начинать с `Add Folder` (папка под правило), затем складывать ноды внутрь. Доступные типы нод зависят от типа дистрибуции. Исключение — money-дистрибуции `Payout` / `Revenue`: их дерево строится в разделе `Finance`, а не в `Manage Tree` (см. ниже).

Без прав на правку дерево тоже открывается: в том же меню есть экшен `View distribution` — то же дерево в режиме только чтение, по праву `settings.distributions.view` (правка требует `settings.distributions.edit`). Что именно в нём отключено — [models/distributions-model.md](../models/distributions-model.md).

- **Элемент внутрь ноды** — клик по полю `Please Configure` (например, `Configure Landing`).
- **Правило** — клик по полю `Rule to pass` (открывается Rule Builder). Пустое правило открывается экраном плиток-пресетов; полный конструктор «поле + оператор + значение» — плитка `Custom` ([models/flow-model.md](../models/flow-model.md)).
- **Ноду внутрь** — клик по иконке нужного типа ноды либо ПКМ → выбор типа из меню.

Правила в дереве проверяются сверху вниз: чем выше нода, тем выше приоритет. См. концепт-уровень в [models/distributions-model.md](../models/distributions-model.md).

### Где строить дерево для Payout / Revenue — в `Finance`, не в `Manage Tree`

Money-дистрибуции (`Payout` / `Revenue`) создаются в `Settings → Distributions` как обычно, но дерево строится в разделе **`Finance`**: найти там созданную дистрибуцию → `Add Folder` → правило (`Edit Rule`) → внутри `Add Folder Inside` (подпапки под conversion type) и/или `Add Revenue Inside` / `Add Payout Inside` (лист со значением). У листа в `Edit Settings` выбирается `Business model` + сумма `Revenue` / `Payout`. Строже/специфичнее правило — выше в папке. Концепт-уровень и дефолтное дерево тенанта — в [models/distributions-model.md](../models/distributions-model.md) § «Как настроить Revenue и Payout Distribution в разделе Finance».

### Подключение дистрибуции во флоу — два способа

1. **Отдельный шаг-нода.** Для `Fill Field` — нода `Fields by Distribution`; для рассылки — узел `Drip schedule` во флоу типа `Notifications`, он выбирает дистрибуцию типа `Message Templates` ([how-to/remarketing-campaigns.md](remarketing-campaigns.md)). Открывается шестерёнкой (cogwheel) шага, дистрибуция выбирается из выпадающего списка.
2. **Новый payload внутри существующего шага.** На шаге кликнуть cogwheel, в поле `Payload type` выбрать `Content Distribution`, затем выбрать дистрибуцию. Выбор доступен в самом Flow либо в кампаниях через `+Add Another Variant → Content Distribution` (зависит от того, flow-only шаг или доступен в кампаниях).

## Direct Traffic Distribution + привязка к домену

Когда: голый URL без UTM (`mydomain.com`) должен открыть конкретную кампанию. Механика DTD vs Default Query, конфликт при совместном использовании — [models/distributions-model.md](../models/distributions-model.md).

**Путь:** `Settings → Distributions → +Distribution → Direct traffic`.

**Шаги:**

1. `Settings → Distributions → +Distribution → Direct traffic`, задать `Name` (+ опц. `Description` / `Tags`), `Confirm`.
2. ПКМ по созданной дистрибуции → `Manage Tree` → `Add Query`.
3. В поле `Manage Rule` выбрать домен → `Please configure query` → в окне `Add Query` добавить `Campaign` (обязательно) и `Source`.
4. Для разделения трафика внутри домена — правило по `initial_path` (`/fr` → одна кампания, `/de` → другая); поддерживаются и правила по `IP` / `city` / `country`.
5. Перейти `Tech → Domains → <Domain> → Edit Domain → Direct Traffic Distribution` → выбрать созданную Direct Traffic Distribution. Без этого шага домен дистрибуцию не использует.
6. Если на домене был **Default Query Code** — очистить (иначе конфликт).

После привязки домена к кампании через DTD она **сразу доступна** по прямому переходу — в неё пойдёт любой заход на голый домен.

## Content Distribution (домен → лэнд)

Когда: один Content-шаг во флоу должен показать разный лэнд в зависимости от домена/гео визита.

**Путь:** `Settings → Distributions → +Distribution → Campaign content` (для нескольких шагов одного флоу — `Flow content`).

**Шаги:**

1. Выбрать `Campaign content` как тип, задать имя.
2. Построить **дерево**:
   - `Add Landing` — какой лэнд показывать.
   - Условие прохода ноды задаётся через поле **`Rule to pass`** (клик по нему открывает редактор правила).
3. В правиле: поле визита (напр. `country_code`) + оператор + значение → сохранить.
4. Подключить к Content step во флоу (см. секцию «Подключение Content Distribution в Content step флоу» ниже).

### Подключение Content Distribution в Content step флоу

**Путь:** `Edit Flow → <Content step> → +Variant → Payload Type: Content Distribution → выбрать`.

Также см. [models/flow-model.md](../models/flow-model.md).

### Шаг, которым управляет дистрибуция, переводят в `Flow Only`

Если контент шага выбирает дистрибуция, шаг переводится из настройки уровня кампании в `Flow Only` (Settings Availability): шаг становится одинаковым для всех кампаний этого Flow, а внутри шага вариантом стоит дистрибуция. Из `Edit Campaign` шаг пропадает — на уровне кампании его не настраивают вовсе. Как сменить Settings Availability шага — [how-to/flow-editor.md](flow-editor.md).

## Fill Field Distribution

Когда: заполнить поле визита по правилам (метка баера `buyer_utm`, `Alter ID`, `Affiliate ID`, API-токен партнёрки, ID-маркеры и т.п.).

**Путь:** `Settings → Distributions → +Distribution → Fill field`.

**Шаги:**

1. Выбрать плитку `Fill field` как тип.
2. `Add Folder` — папка-контейнер.
3. На папке задать `Rule to pass` (например, `Campaign Owner = X`).
4. Внутри папки — `Add Fill Field` → выбрать поле визита + `Payload Type` (`Fill Text` для строки) → значение.
5. `Save`. Подключить во флоу через шаг `Fill Fields` или `Fields by Distribution`.

Дерево строй по крупному признаку сверху, по мелкому — внутрь (например, верхний уровень `Campaign Owner`, гео — внутри папки владельца). Эвристику «сначала по байеру, потом по гео» см. в [models/distributions-model.md](../models/distributions-model.md).

## Как создать дистрибуцию шаблонов сообщений (`Message templates`)

Когда: ремаркетинг-рассылке нужен набор текстов, из которого она возьмёт сообщение для конкретного визита. Чем этот тип отличается от остальных (какие ноды допускает, почему у него нет `Split Group` и `Flow State`) — [models/distributions-model.md](../models/distributions-model.md).

**Путь:** `Marketing → Template distributions` → кнопка создания. Пикер типов на этой странице не показывается — сразу открывается создание нужного типа. Тот же тип создаётся и из общего пикера: `Settings → Distributions → +Distribution → Message templates`.

### Собрать дерево шаблонов и подключить его к рассылке

1. Задать `Name` (обязательно), опционально `Description` и `Tags` → `Confirm`. Флоу при создании **не выбирается**: привязки к флоу у этого типа нет.
2. ПКМ по созданной дистрибуции → `Manage Tree`.
3. Разложить шаблоны: `Add Folder` под группу, внутрь — ноды `Push Template` (лист-шаблон можно положить и прямо в корень дерева).
4. В ноде заполнить `Name` (это имя строки в дереве, а не текст сообщения), `Push title` и `Push message`; `Push icon` — опционально. Плейсхолдеры и спинтакс в текстах — [how-to/push-notifications.md](push-notifications.md).
5. Развести шаблоны условиями: на каждой ноде `Rule to pass` (например, по `aio.visit.language_code`). Других способов отбора у этого типа нет — `Split Group` и `Flow State` выключены.
6. Выбрать готовую дистрибуцию в шаге серии рассылок (`Drip schedule`) ремаркетинг-кампании — [how-to/remarketing-campaigns.md](remarketing-campaigns.md).

Ноды `Email Template` / `Sms Template` / `Telegram Template` дерево создаст, но из него они не отправляются, а нода `Strategy` обрывает шаг рассылки — обе оговорки разобраны в [models/distributions-model.md](../models/distributions-model.md).

## Внешний сайт как payload дистрибуции — нода `Add Reflect`

Когда: контент шага должен отдаваться с внешнего сайта, без миграции файлов в AIO.

Нода **`Add Reflect`** внутри `Campaign content` (как и `Add Redirect`) даёт тот же payload-тип, что одноимённая опция шага во флоу: в самой ноде указывается целевой внешний URL.

Подключается дистрибуция в Content step флоу так же, как обычная Content Distribution (через cogwheel шага → выбор дистрибуции, см. выше).

## Как управлять нодами дерева дистрибуций (Duplicate, Disable, Clear)

### Duplicate (рекурсивно)

Дублирует ноду со всеми вложенными.

**Путь:** scroll на ноде **или** ПКМ → `Duplicate recursively` → переименовать копию.

### Disable node + Clear recursively

**Активные ноды нельзя удалить** напрямую. Чтобы удалить — сначала деактивировать:

1. ПКМ → `Disable node` (нода красится серым).
2. ПКМ → `Clear recursively` — удалить со всеми вложенными.

Остальные действия (`Toggle activity`, `Toggle locked`, `Move Up` / `Move down`, `Show children`) — полная таблица в [models/distributions-model.md](../models/distributions-model.md) § «Как управлять нодами дерева».

## Частые ошибки и грабли при работе с Distributions

- **Создал DTD, но домен не использует.** Не выбрана дистрибуция в атрибуте домена `Tech → Domains → <D> → Edit Domain → Direct Traffic Distribution`.
- **Голый домен с DTD доступен сразу.** Прямые заходы идут в кампанию с момента привязки — настройка шага `Filter` для такой кампании.
- **Active нода не удаляется.** Сначала `Disable node`, потом `Clear recursively`.
- **В дереве видно не все ноды уровня.** Это окно отрисовки, а не потеря данных: под последней строкой уровня есть строка `Show more` со счётчиком `<показано> / <всего>` — дожимай её, пока не исчезнет (механика — [models/distributions-model.md](../models/distributions-model.md)).
- **`Default Query Code` + `Direct Traffic Distribution` на одном домене.** Конфликт — очистить `Default Query Code`, использовать только DTD.
- **Разделение Direct Traffic по путям (`initial_path`), где оно не нужно.** Возможность есть (см. шаг 4 DTD), но для голых Direct-Traffic-запусков обычно не требуется — лишнее усложнение.
- **Большие пулы доменов (10k+) на одну дистрибуцию.** UI тормозит — поднимать через API.

## Смежные темы

- [models/distributions-model.md](../models/distributions-model.md) — концепт-уровень: 8 типов, дерево, Strategy, Business Models, Buyer UTM Distribution.
- [models/flow-model.md](../models/flow-model.md) — Content step, Fill Fields step, Fields by Distribution.
- [how-to/domains.md](domains.md) — `Default Query Code`, привязка домена к серверу.
- [how-to/push-notifications.md](push-notifications.md) — как собирается текст сообщения (плейсхолдеры, спинтакс).
- [how-to/remarketing-campaigns.md](remarketing-campaigns.md) — рассылка по аудитории визитов: где выбирается дистрибуция `Message Templates`.
- [reference/glossary.md](../reference/glossary.md) — `Direct Traffic Distribution`, `Content Distribution`, `Fill Field Distribution`, `Add Folder`, `Clear recursively`, `Strategy First/Weights/ML`.
