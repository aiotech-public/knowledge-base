---
id: auto-rules
title: Автоправила Meta (раздел Automations) — шаблон, ассайн и запуск
description: Как настроить автоправила над Meta-сущностями в разделе Automations — шаблон правил (ось фаз, условия, переменные $NAME, шесть вердиктов), ассайн со скоупом и режимами Observe/Approve/Auto, кто видит ассайн (владение, Access type, Change ownership), проверка через Live check и Backtest Rules, права automation.*, бейдж Rolling out. Только ERP.
doc_type: how-to
builds: [erp]
related: [meta-ads, auto-rules-engine, ui-map, distributions-model, meta-spend-allocation, limits, permissions-model, permissions, notification-center]
language: ru
updated: 2026-09-11
---

# Автоправила Meta (раздел Automations) — шаблон, ассайн и запуск

Автоправило — это связка «шаблон правил (`Rule Template`) + ассайн (`Assignment`) на кампании, адсеты или объявления Meta»: движок сам считает метрики сущности и делает то же, что делают руками экшены `Stop` и `Change budget` на страницах Meta — выключает сущность или двигает бюджет. Ручные экшены Meta описаны в [how-to/meta-ads.md](meta-ads.md).

Порядок работ: собрать шаблон → повесить ассайн на скоуп → посмотреть вердикты в `Live check` → перевести `Mode` из дефолтного `Observe` в `Approve` или `Auto`. Тумблер `Enabled` у нового ассайна уже включён — выключенной создаётся только копия через `Duplicate assignment`.

Раздел называется `Automations` (`/app/automations`) и есть только в ERP-билде. Почему собранное правило может не сделать ничего — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

## Четыре страницы раздела `Automations` — что где лежит

**`Status`** (заголовок страницы — `Tenant Automation Status`) — обзор движка: KPI-полоса (виджет `Savings` и пять карточек — `Actions created`, `Executed`, `Stops`, `Budget changes`, `Failed`), счётчик `Pending approvals` с отметкой, сколько подтверждений протухнет в ближайший час, и список ассайнов с временем прогонов и кнопкой `Live check`. `Declined` и `Observed changes` в полосу не попадают — это серии графика `Actions by day`. В карточках главное число — за последние 30 суток, рядом мельче вчера, текущий и прошлый месяц. `Savings` — оценка по темпу расхода остановленных сущностей (`Estimate by the running spend rate of stopped entities`), а не факт из рекламного кабинета. Подтверждают действия не здесь, а на `History` или из уведомления; в `Live check` у ждущей сущности только бейдж `Needs approval`.

**`Assignments`** — ассайны: какой шаблон на каком скоупе работает. Список ограничен владением: чужой ассайн виден руководителю команды владельца, роли полного доступа и всем — если у него `Access type = Everyone` (разбор — секция «Кто видит чужой ассайн»).

**`Rule Templates`** — таблица шаблонов; дерево фаз и правил правится на отдельной странице `Rule Template Tree`.

**`History`** — журнал вердиктов, вкладки `All` и `Pending`; на pending-строках доступны `Approve` и `Decline`. Поиск идёт по имени сущности, её facebook id и по UUID — сущности или самой записи. Быстрые фильтры — `Status`, `Action`, `Assignment`, `Level`. Видны записи только тех ассайнов, что доступны на просмотр по владению и `Access type` (секция «Кто видит чужой ассайн»). Маршруты страниц и поведение пункта меню — [reference/ui-map.md](../reference/ui-map.md).

### Бейдж `Rolling out` на страницах `Automations` — плашка, а не ограничение

На всех четырёх страницах раздела стоит бейдж `Rolling out` (в таблицах — в шапке рядом с поиском, на `Status` — в шапке страницы) с подсказкой `Auto Rules are rolling out gradually — enabled for selected clients step by step.`; в русской локали — `Раскатывается` / `Auto Rules раскатывается постепенно — включаем клиентам пошагово.`. Плашка информационная: она зашита в страницы раздела, не привязана к настройке тенанта и ничего в разделе не блокирует — если раздел открыт, шаблоны, ассайны и журнал работают штатно. Доступность самого раздела задают права ветки `automation.*`.

## Как навесить правило прямо со страниц Meta — `Apply Auto Rules`

Раздел `Automations` можно не открывать вообще: на страницах Meta `Campaigns`, `Ad Sets`, `Ads` и `Ad Accounts` правый клик по выделенным строкам даёт `Apply Auto Rules` — «Create an assignment for the selected rows — the scope is prefilled»: скоуп ассайна уже заполнен выделением. Пункт открывается по праву `automation.assignments.edit`.

Премодалка предлагает два пути:

- **`New simple rule`** — одна метрика, окно, сравнение и порог; шаблон с переменной `$THRESHOLD`, одна фаза и одно правило собираются сами. Кнопка `Check on history` тут же показывает, на скольких из выделенных сущностей правило сработало бы за последний 31 день; горизонт этой быстрой проверки фиксированный, поля периода в форме нет (поле `Window` рядом — окно метрики условия, а не горизонт проверки). Выбрать горизонт даёт `Backtest Rules` полем `Days (1–31)`. Ось фаз в этом пути жёстко `Meta Spend`, вердикт `change_budget` недоступен.
- **`Existing template`** — «Assign one of your templates to the selection»: взять готовый шаблон и настроить ассайн.

### Колонка `Auto Rules` в таблицах Meta — под какими правилами строка

В таблицах `Campaigns`, `Ad Sets` и `Ads` есть колонка `Auto Rules` с подсказкой «Auto Rules assignments covering this entity (resolved live for the visible page)» — какие ассайны накрывают эту строку.

Покрытие считается вживую и только пока на видимой странице не больше 300 строк. На более длинной странице колонка у каждой строки показывает `No rules` независимо от того, накрыта строка правилами или нет, — чтобы читать её, уменьшите размер страницы.

В счётчике ячейки — все неархивные ассайны уровня, накрывающие строку, включая выключенные. Наведение раскрывает их поимённо; ассайны, недоступные вам на просмотр, сведены там в один счётчик `+N others` без имён. Клик по ассайну в списке открывает его форму. Ручные экшены тех же страниц — [how-to/meta-ads.md](meta-ads.md).

## Шаблон правил (`Rule Template`) — из чего он состоит

Шаблон автоправил — это дистрибуция типа `Auto Rules`. В шапке: `Name`, `Description`, `Phase axis`, `Tags` и `Variables`. Дерево собирается только из двух нод — `Phase` в корне и `Auto Rule` внутри фазы: папок (`Folder`) и стратегий (`Strategy`) у этого типа нет, глубже двух уровней не вложить. Это исключение из общей схемы дерева дистрибуций — [models/distributions-model.md](../models/distributions-model.md).

Создание идёт через премодалку `New rule template` («Start simple or build the full template.») с двумя карточками: `Simple rule` — «One metric and one threshold — the phase and the rule are built for you.» и `Advanced` — «Full editor: variables, phases and multiple rules.».

В таблице `Rule Templates` у каждого шаблона видно `Phase axis`, `Phases` (границы фаз по возрастанию и число правил в каждой) и `Variables` (переменные с дефолтами).

### `Phase axis` — ось, по которой сущность движется между фазами

Ось одна на шаблон и обязательна. Выбор — из четырёх накопительных Meta-метрик: `Meta Spend` (значение по умолчанию), `Meta Impressions`, `Meta Clicks`, `Meta Age (hours)`. Метрику тенанта осью взять нельзя — форма отбивает такой выбор ошибкой `"metric.<uuid>" cannot be a phase axis: only spend, impressions, clicks or age_hours`.

Значение оси считается за всю жизнь сущности, независимо от окон в условиях правил, — как из оси и правил собирается вердикт, в [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

Не путайте с отчётной колонкой `Meta Spend`: в условии правила это расход рекламного кабинета на саму сущность Meta, а в отчётах — аллокация расхода на срезы визитов, разбор которой — в [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

### Фаза — диапазон `[from, to)` по оси

Нижняя граница входит в фазу, верхняя принадлежит следующей, пустое `To` = ∞. Подсказка формы: «The range is [from, to): the upper bound belongs to the next phase.»

Ограничения: `From` ≥ 0, `To` больше `From`, диапазоны фаз не пересекаются — иначе сохранение отбивается ошибкой вида `Range overlaps phase "Testing" (0 – 100)`.

Порядок фаз производный от диапазона (по возрастанию `From`) и руками не переставляется: попытка поменять фазы местами возвращает `Phases are ordered by their range — edit the phase range instead`. Фаза без диапазона уезжает в конец списка и в оценке не участвует. `Duplicate` фазы создаёт копию без диапазона и с суффиксом ` - Copy`, правила внутри сохраняются. Пустое имя фазы собирается из оси и границ — например `Meta Spend 0 – 100`.

Что происходит с сущностью, чьё значение оси попало в непокрытый интервал, — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

### Условие правила — метрика, окно, сравнение и порог

Левый операнд берётся из каталога двумя группами. `Meta` — `Meta Spend`, `Meta Impressions`, `Meta Clicks`, `Meta CPC`, `Meta CPM`, `Meta CTR`, `Meta Age (hours)`. `Metrics` — любые неархивные метрики тенанта.

Окно приклеивается к операнду через `@` (`meta.spend@today`); без суффикса окно равно `Last 31 days`. Сравнения только числовые: `=`, `<`, `<=`, `>`, `>=`. Справа ровно одно значение — число или переменная шаблона.

`Meta Clicks` здесь — клики по ссылке (`Meta Inline Link Clicks`), а не все клики по объявлению.

Все условия одного правила соединяются логическим И — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md). Правило без условий — else-ветка фазы («A rule without conditions is the else-branch»), поэтому его ставят в фазе последним.

### Переменные шаблона `$NAME` — пороги, которые задаёт ассайн

Переменная — именованное число, на которое ссылаются правила; в дереве она рендерится как `$NAME` (например `$CPL_BAD`). Имя начинается с заглавной латинской буквы, дальше заглавные буквы, цифры и подчёркивание, до 64 символов; имена уникальны внутри шаблона. У каждой переменной есть `Default` и `Description`.

Сами числа задаются не в шаблоне, а в ассайне — блок `Variable values`, дефолты подставляются автоматически, кнопка `Copy values from…` переносит значения из другого ассайна того же шаблона. Каждая переменная обязана быть заполнена числом, иначе сохранение отбивается ошибкой вида `Variable THRESHOLD is required` или `Variable THRESHOLD must be a number`.

Переменную, на которую ссылается хотя бы одно правило, из шапки не удалить: `Variables referenced by rules cannot be removed: THRESHOLD`.

### Вердикт правила — шесть действий

У правила ровно одно действие (`Action`); в интерфейсе оно показывается коротким чипом:

- `stop` — чип `Stop`, «Turn the entity off»;
- `start` — чип `Run`, «Turn the entity back on»;
- `keep` — чип `Keep`, «Leave running, looks good»;
- `wait` — чип `Wait`, «Leave running, waiting for data»;
- `change_budget` — чип `Bdgt`, «Adjust the budget within bounds»;
- `duplicate` — чип `Dupe`, «Create a copy, approve only».

`keep` и `wait` — явные no-op: ничего не исполняют, тумблер `Require approval` у них скрыт. У `duplicate` тот же тумблер, наоборот, включён принудительно и не снимается (`Approval is mandatory for this action`) — но это только состояние формы правила: судьбу действия решает режим ассайна, и в режиме `Auto` дубликат уходит на исполнение сразу, без человека ([mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md)).

До Meta доезжают `stop` и `change_budget`; что происходит с остальными вердиктами — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

Новое правило создаётся **выключенным** и включается вручную. Правило без выбранного действия в оценке не участвует.

### `Change budget` — три режима и границы бюджета

У вердикта `change_budget` появляется блок `Budget` с полем `Mode` и тремя режимами:

- `Set to value` — «The budget becomes exactly this value.»;
- `Multiply by` — «The budget is multiplied by this factor (1.2 = +20%, 0.8 = −20%).»;
- `Add / subtract` — «This amount is added to the budget; a negative value decreases it.».

Рядом `Min budget` и `Max budget`: «The resulting budget never leaves the min–max bounds.» Пустая граница подписана `No limit`.

Шаблон с `change_budget` не вешается на уровень `Ads`: у объявления своего бюджета нет, и форма ассайна отдаёт ошибку `This template changes budgets — it cannot be assigned to ads (no budget on ad level)`.

## Ассайн (`Assignment`) — на какие сущности повесить шаблон

Ассайн связывает шаблон с конкретным скоупом Meta-сущностей одного уровня. В форме: `Name`, `Description`, `Template`, `Mode`, тумблер `Enabled`, `Variable values`, `Target` и `Limits`.

Ассайн выключается двумя способами: тумблером `Enabled` («Disabled assignments are skipped by the tick») и архивацией («Archiving switches the assignment off»). `Duplicate assignment` создаёт копию выключенной, в режиме `Observe` и на того, кто копирует (имя по умолчанию — `<имя> Copy`); шаблон, таргет, значения переменных и лимиты переносятся.

Сущность, которую ассайн уже остановил, он больше не оценивает, и ручное включение обратно в Meta этого не меняет — разбор в [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

В таблице `Assignments` у ассайна видно `Template`, `Level`, `Target` (режим таргета и «сейчас в скоупе»), `Values`, `Mode`, `Enabled`, `Last run`, а также колонки владения `Access type` и `Owner`; `Next run`, `Last error` и `Shares` (кому ассайн пошарен) по умолчанию скрыты и включаются в настройках таблицы. Быстрые фильтры таблицы — `Template`, `Level`, `Mode`, `Enabled only`, `Owner`, `Team` и `Archived`. Значения переменных — снапшот на момент сохранения: значение, переменной для которого в шаблоне больше нет, помечается подписью «This variable is no longer in the template». Что означают колонки владения — секция «Кто видит чужой ассайн».

### `Level` и `Select by` — как описать скоуп ассайна

`Level` — уровень сущностей, один на ассайн: `Campaigns`, `Ad sets` или `Ads`. В новой форме подставляется `Campaigns`; если ассайн создаётся из выделения на странице Meta, уровень берётся из типа выделенных строк — для строк `Ad Accounts` это снова `Campaigns`.

`Select by` — способ описать скоуп:

- **`By filters`** — «Live filter set of the level table — new entities matching the filters join the scope automatically»: берутся фильтры и поиск таблицы своего уровня, и всё новое, что под них подходит, попадает под правило само;
- **`Selected`** — «Hand-picked entities; a node above the level includes all its descendants dynamically»: выбранные узлы, причём узел выше уровня (например `Ad account`) динамически покрывает всех своих потомков этого уровня.

Форма на лету считает `Now in scope: N` — счётчик доходит до 1000 и дальше показывает, что сущностей больше. Она же показывает пересечения с другими ассайнами с подписью «Not a blocker: the more specific target wins; if equal — the newer assignment»; как движок ведёт себя на пересечениях на самом деле — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

### Режимы `Observe` / `Approve` / `Auto` — лестница доверия

Режим ассайна (`Mode`) решает, что происходит с вердиктом: `Observe` — «Log verdicts only, no actions are executed»; `Approve` — «Actions wait for a human approval»; `Auto` — «Actions are executed immediately».

По умолчанию в форме стоит `Observe`, поэтому «правила настроены, а ничего не происходит» чаще всего означает именно его — `Mode` и тумблер `Enabled` смотрят первыми.

Ждущее подтверждения действие видно не только на страницах `Status` и `History`: владельцу ассайна приходит уведомление в колокольчик с кнопками `Approve` и `Decline` — какие уведомления шлёт движок, в [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

Два предупреждения. Отметка `Require approval` у отдельного правила на исполнение не влияет: единственный рычаг «ждать апрува» — режим ассайна. И на устаревших данных рекламного кабинета режим `Auto` сам уводит действие в очередь апрувов. Оба поведения разобраны в [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

### `Limits` — сколько действий ассайн делает за прогон и за сутки

Блок `Limits` — предохранители прогона («Safety caps for the tick; empty means the tenant default»). Работающих полей пять, пустое поле означает дефолт:

- `Max actions per run` и `Max actions per day` — потолки числа действий за один прогон и за календарные сутки;
- `Pending approve TTL, min` — сколько живёт действие, ждущее подтверждения;
- `Budget min` и `Budget max` — границы бюджетного действия; при сохранении проверяется `Budget max must be ≥ min`.

Сами дефолты, которые подставляются вместо пустых полей, собраны в [reference/limits.md](../reference/limits.md).

Шестое поле блока, `Entity cooldown, min`, движок не читает — поведением ассайна оно не управляет. Как считается суточный лимит и что происходит при упоре в потолок — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

## Кто видит ассайн и как передать его другому — владение, `Access type`, `Change ownership`

### Кто видит чужой ассайн — владелец, руководитель команды, `Access type = Everyone`

Видимость ассайна определяется владением, а не правами ветки `automation.*`: право `automation.assignments.view` открывает страницу `Assignments`, но список в ней ограничен. Ассайн видят его владелец, руководитель команды владельца (`Team Head` / `Team Leader`), роль полного доступа тенанта (Owner / Admin) и — если у ассайна `Access type = Everyone` — все пользователи тенанта. Новый ассайн создаётся с `Access type = By Share`, то есть по умолчанию его видит только владелец. Как устроено владение и команды — [models/permissions-model.md](../models/permissions-model.md).

Та же граница действует везде, где ассайны перечисляются: в списке на странице `Status`, в журнале `History` и его фильтре `Assignment`, в селекте `Copy values from…` формы ассайна и в колонке `Auto Rules` таблиц Meta, где недоступные ассайны свёрнуты в счётчик `+N others`.

Колонка `Access type` в таблице `Assignments` подписана `Using for visibility only, everyone shared cannot edit anyway.`, но действующее поведение шире: ассайн с `Everyone` попадает и в выборку на редактирование — править его может любой пользователь тенанта с правом `automation.assignments.edit`. Нужно «все видят, правит только владелец» — через `Access type` это не собрать; разбор в [how-to/permissions.md](permissions.md). У колонки `Owner` подсказка `Owner — metrics are read and actions are executed on their behalf`: под владельцем ассайна движок и считает метрики, и исполняет действия в Meta ([mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md)).

### Сотрудник с полными правами `automation.*` не видит ассайны коллег — права-обхода нет

Симптом: роли выданы все шестнадцать императивов `automation.*`, а в `Assignments`, `History` и `Status` человек видит только свои ассайны. Причина — видимость ассайна задаётся владением и `Access type`, а права ветки `automation.*` её не расширяют: ни одно из них не означает «видеть все ассайны тенанта».

Обхода через ветку `Scopes` у ассайнов тоже нет: в ней есть кейс для флоу (`scopes.tracker.flows`) и для шаблонов автоправил, которые как дистрибуции открываются правом `Scopes: settings distributions` (`scopes.settings.distributions`), но не для ассайнов. Рычага два: выставить ассайну `Access type = Everyone` или передать его через `Change ownership`; третий — руководство командой владельца.

### `Change ownership` — как передать ассайн другому человеку

Смена владельца и `Access type` — одно действие `Change ownership` в контекстном меню строки ассайна, группа `Teams & Permissions`; форма (`Transfer ownership and update the access level.`) из двух обязательных селектов — `Access type` (`Everyone` / `By Share`) и `Tenant users`. Массового варианта у ассайнов нет: `Mass change ownership` есть у шаблонов, но не здесь.

Пункт показывается по праву `automation.assignments.share.ownership` и только на своей строке либо при полном доступе. На сервере решает владение: сменить владельца может владелец записи или роль полного доступа, остальным возвращается `You don't have permission to change ownership`.

Передача ассайна меняет не только видимость: метрики и действия в Meta идут под новым владельцем — с его скоупом сущностей и его правами на ручные экшены Meta ([mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md)). Перед передачей стоит проверить, что новый владелец видит нужные рекламные кабинеты и имеет права вида `meta.campaigns.edit.stop`.

### Пошарить ассайн конкретному человеку или команде из интерфейса нельзя

Адресного шаринга ассайна в продукте нет: в контекстном меню `Assignments` нет ни `Share`, ни `Share also`, ни `Unshare`, а право `automation.assignments.share` объявлено в ветке, но рычага в интерфейсе не имеет. Колонку `Shares` в таблице ассайнов из интерфейса поэтому не заполнить.

Шаблоны автоправил, наоборот, шарятся полностью: страница `Rule Templates` — таблица дистрибуций с зашитым типом `Auto Rules`, и от неё шаблонам достаётся весь набор действий владения — `Share`, `Share also`, `Force share`, `Unshare` под правом `automation.templates.share` плюс `Change ownership` и массовый `Mass change ownership` под `automation.templates.share.ownership`. Как работает шаринг сущностей — [how-to/permissions.md](permissions.md).

Чтобы коллега увидел ваш ассайн, рычага два: `Access type = Everyone` или передача через `Change ownership`; чтобы видел все ассайны команды — роль руководителя команды.

## Пять готовых шаблонов, которые AIO поставляет сам

Кроме собственных шаблонов доступны глобальные заготовки AIO, общие для всех тенантов:

- `Meta CPC > $THRESHOLD → Stop`;
- `Meta CPM > $THRESHOLD → Stop`;
- `Meta CTR < $THRESHOLD → Stop`;
- `Meta $MIN_SPEND spent, no clicks → Stop`;
- `Meta CPC ladder → Stop`.

Все построены только на Meta-метриках, поэтому работают в любом тенанте без донастройки. У четырёх одиночных — одна фаза `All spend` (0 → ∞), у ладдера две: `Testing $0–100` и `Scaling $100+`. Переменные заготовок — `MIN_SPEND`, `THRESHOLD`, `MIN_IMPRESSIONS`, `MAX_CLICKS`, `TEST_THRESHOLD`, `SCALE_THRESHOLD`.

Во всех пяти заготовках у правил проставлена отметка `Require approval`, но на исполнение она не влияет: ассайн с заготовкой в режиме `Auto` останавливает сущности без подтверждения. Нужен человеческий шаг — ставьте ассайн в `Approve`.

В таблице `Rule Templates` заготовок нет — она показывает только шаблоны тенанта. Выбрать заготовку можно в селекте шаблона в форме ассайна и в `Backtest Rules`: там она стоит рядом с собственными шаблонами и помечена значком AIO с подсказкой «Shared across all — the same distribution is used everywhere». Чтобы поменять под себя границы фаз или условия, шаблон форкают в свой тенант через `Duplicate` — копия создаётся уже в тенанте.

## Как проверить правило, ничего не запуская — `Live check` и `Backtest Rules`

Два инструмента отвечают на разные вопросы. `Live check` — «что правило сделало бы с текущим скоупом прямо сейчас»: им разбирают ситуацию «правило не срабатывает» на конкретной кампании или адсете. `Backtest Rules` — «когда бы правило срабатывало на реальной истории»: им подбирают порог до того, как ассайн переведён в `Auto`.

Ни один из них ничего не исполняет, не меняет сущности в Meta и не пишет записей в журнал `History`.

### `Live check` — вердикты по текущему скоупу прямо сейчас

Кнопка `Live check` стоит в строке ассайна на странице `Status` и требует право `automation.status.view`.

Проверка считает вердикты по текущему скоупу и показывает по каждой сущности либо вердикт с раскрытием «фаза › правило › входные значения › пороги», либо названную причину, по которой сущность не оценивается (разбор причин — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md)). Сверху идут рекламные кабинеты скоупа с отметкой свежести данных.

Два ограничения. За раз показывается не больше 200 сущностей, список помечается усечённым. И считается проверка под правами того, кто её запустил, а боевой прогон — под владельцем ассайна: при разных скоупах видимости цифры могут разойтись.

### `Backtest Rules` — прогон шаблона по реальной истории

`Backtest Rules` — «Dry-run the template on real history of specific entities — no assignment involved»: дерево правил прогоняется по почасовой истории выбранных сущностей, ассайн в этом не участвует и ничего не исполняется. Право — `automation.assignments.edit.test-run`.

Три входа: правый клик по строке шаблона (в `Rule Templates` и в `Settings → Distributions`), правый клик по строкам на `Campaigns` / `Ad Sets` / `Ads` и правый клик на `Ad Accounts` — в последнем случае берутся топ-25 сущностей уровня по расходу внутри выбранных кабинетов.

Границы формы: `Days (1–31)`, по умолчанию 31; от 1 до 25 сущностей уровня либо от 1 до 10 рекламных кабинетов. Значения переменных задаются поверх дефолтов шаблона.

Главное допущение цитируется прямо в результатах: «The simulation does not stop entities — history is real, so events may continue after a “stop” verdict». То есть бэктест отвечает на вопрос «когда бы правило сработало», а не «сколько бы удалось сэкономить».

## Права `automation.*` — своя ветка, отдельная от дистрибуций

Раздел закрыт собственной веткой прав `automation.*`, независимой и от `settings.distributions.*`, и от трафиковых `tracker.*`, и от `meta.*`. Императивов шестнадцать, четырьмя группами — шаблоны, ассайны, журнал вердиктов и страница статуса; объявлены они только в ERP, а в матрице `Settings → Positions` собраны в секцию `Automation section` с подписями вида `Automation: view templates` и `Automation: approve verdicts`. Полный список слагов и какая страница каким из них открывается — [models/permissions-model.md](../models/permissions-model.md), выдача прав под типовые роли — [how-to/permissions.md](permissions.md).

Пункта `Automations` не будет в меню, если из ветки выдан только `automation.status.view`: меню поднимают view-права на шаблоны, ассайны или журнал — разбор в [models/permissions-model.md](../models/permissions-model.md). Права ветки открывают страницы и действия, но не расширяют видимость: чужие ассайны показываются по владению и `Access type` (секция «Кто видит чужой ассайн»), а `automation.assignments.share` рычага в интерфейсе не имеет.

Само действие в Meta исполняется правами владельца ассайна на такой же ручной экшен — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

### Пункт `Backtest` виден, а сам запуск отбивается

Пункт `Backtest` в контекстных меню Meta-страниц и в таблице шаблонов показывается по праву на просмотр шаблонов (`automation.templates.view`), а сам запуск бэктеста бэкенд проверяет по `automation.assignments.edit.test-run`. Роль без `test-run` видит пункт и получает отказ уже на запуске — недостающее право и надо выдать.

### Карточка `New simple rule` открывается, а правило не создаётся — нужно `automation.templates.edit`

Зеркальный случай на `Apply Auto Rules`: карточка `New simple rule` в премодалке показывается по праву `automation.assignments.edit`, но сама она сначала создаёт шаблон — действие `Distribution\CreateAutoRules`, а его сервер проверяет по `automation.templates.edit`; следом по тому же праву создаются фаза и правило, и только потом ассайн по `automation.assignments.edit`. Роль с правом на ассайны, но без права на шаблоны, откроет карточку и получит отказ на первом шаге. Простому пути нужны оба права — `automation.templates.edit` и `automation.assignments.edit`; карточка `Existing template` обходится одним `automation.assignments.edit`.

### Кто может подтверждать вердикт

Кнопки `Approve` и `Decline` на странице `History` и в карточке уведомления в колокольчике проверяются по правам `automation.actions.edit.approve` и `automation.actions.edit.decline`.

В Telegram кнопки приходят с уведомлением `auto_rule.pending` в личный привязанный чат владельца ассайна; подтверждать действие из этого чата может владелец. Как выглядят кнопки и что бот отвечает по просроченному действию — [mechanics/notification-center.md](../mechanics/notification-center.md).
