---
id: flow-editor
title: Flow Editor (How-to)
description: Создание флоу (Tracker → Flows → Traffic flow → Campaigns) и почему он не сохраняется, подписи нод и стрелок на канвасе и выбор перехода у стрелки, модалка настроек шага и Apply settings, drag-хендл, View flow, удаление и архив шагов, Settings Availability, Handle-link, fallback на переполнение капы.
doc_type: how-to
builds: [erp]
related: [flow-model, campaigns, campaign-defaults, placeholders, glossary, conversion-model, distributions]
language: ru
updated: 2026-10-09
---

# Flow Editor (How-to)

## Краткая шпаргалка по Flow Editor

Мелкие, но регулярные процедуры в Flow Editor. Концепция Flow (States, Transitions, Settings Availability) — в [models/flow-model.md](../models/flow-model.md).

- Создать флоу — `Tracker → Flows` → зелёная кнопка `Traffic flow` → плитка `Campaigns`; копия готового — ПКМ по флоу → `Copy flow`.
- Переход у стрелки — клик по подписи на стрелке открывает меню переходов шага.
- Настройки шага — клик по ноде открывает модалку; `Apply settings` только закрывает её, флоу сохраняет `Save`.
- Передвинуть ноду — тянуть за drag-хендл (крестик слева в строке ноды), не за тело.
- Удалить «висящий» блок — выделить + клавиша `Delete` (пока сам блок не сохранён; кроме `Start`).
- `Settings Availability` шага — `Campaign Only` / `Flow Only` / `Campaign Template`. Меняется в `Edit Flow → <Step> → Settings Availability`.
- `Handle-link` — нумерованные кнопки на лэнде через `{{link:N}}`.
- Fallback при переполнении капы — `Destination Full` transition.
- Посмотреть схему без правки — `View flow` (полный экран, цепочки шагов свёрнуты, разворачиваются кликом).

## Как создать новый флоу (поток) — `Tracker → Flows` → `Traffic flow`

**Новый флоу для кампаний создаётся в `Tracker → Flows` зелёной кнопкой `Traffic flow` («Флоу трафика») над таблицей → плитка `Campaigns` («Кампании»).** Кнопка активна при праве `tracker.flows.edit`; без него она серая с подсказкой `No permission`. Частые формулировки: «как создать поток», «как создать Campaign Flow», «нет выбора потоков при создании кампании».

1. `Tracker → Flows` → `Traffic flow`. Откроется окно `Add flow to AIO` с двумя плитками: `Campaigns` — обычный флоу от источника к офферу (помечен рекомендуемым), и `Campaign sub-flow` («Sub-Flow кампании») — отдельный кусок флоу, который обрабатывает визит и возвращает его в основной флоу ([models/flow-model.md](../models/flow-model.md)).
2. Выбрать `Campaigns` — откроется редактор, на канвасе одна нода `Start`.
3. В панели `Flow details` («Параметры флоу») задать `Name` — обязательно; `Description` и `Tags` — по желанию.
4. Добавить шаги из палитры `Nodes to create` («Узлы для создания»): кликом по типу шага или перетаскиванием на канвас. Соединить шаги стрелками и у каждой стрелки выбрать переход — раздел «Как выбрать или сменить переход у стрелки» ниже.
5. Нажать `Save`. Флоу появится в окне `Select flow to create campaign` при создании кампании ([how-to/campaigns.md](campaigns.md)).

Быстрее начать с копии готового флоу: ПКМ по строке флоу → `Copy flow` откроет редактор с копией, `Save` создаст новый флоу. В MTK редактора флоу нет — там флоу выбирают из готовых пресетов тенанта.

### Флоу не сохраняется — что проверяет редактор при `Save`

**Кнопка `Save` не сохраняет флоу, пока в нём есть ошибка, и называет первую из них.** Проверки идут по порядку:

- `The Flows name is required` — не задано `Name`;
- `You must add at least one state` — в схеме нет шагов; `You must add at least one state and transition.` / `You must add at least one transition` — данные схемы или список стрелок ещё не получены. Пустой список стрелок сам по себе эту проверку не блокирует;
- `State <имя> is invalid` — у шага не заполнены обязательные настройки;
- `The transition does not have a label set` — у стрелки не выбран переход.

У `Campaign sub-flow` редактор перед сохранением просит подтверждение, если нет `Finish` или `Destination` с входящей стрелкой. Несохранённый блок, который мешает сохранению, разобран.

## Как читать канвас: подписи под нодами и на стрелках

**Под именем ноды канвас печатает описание шага, а если автор флоу его не задал — подпись типа шага; на стрелке — переход, по которому визит уходит дальше.** По этим подписям схему читают без открытия настроек — и так же её описывают, когда спрашивают, почему визит не идёт дальше.

### Подпись под нодой — тип шага

| Подпись (EN / RU) | Тип шага |
|---|---|
| `Show to browser` / «Показать в браузере» | `Content` — лэнд, HTML, редирект, 404 |
| `Initiate conversion` / «Запустить конверсию» | `Spawn conversion` |
| `Traffic filtering` / «Фильтрация трафика» | `Filter` и шаги Traffic Filters |
| `Check rules` / «Проверка правил» | `Condition` |
| `Flip domains` / «Сменить домены» | `Domain Change` |
| `Operate with fields` / «Работа с полями» | `Fill Fields` |
| `Use trees to fill fields` | `Fields by Distribution` |
| `Select form` / «Выбрать форму» | `Fill form` |
| `Verify field via sender provider` | `Field verification` |
| `Extend your flow` / «Расширить флоу» | `Sub flow` |
| `External target` / «Внешняя цель» | `Destination` |
| `Initial state` / «Начальное состояние» | `Source` и `Start` |

### Подпись на стрелке — переход, по которому уходит визит

| Подпись | Переход | Когда срабатывает |
|---|---|---|
| `arrived` | `Arrived` | визит вошёл из `Source` |
| `handle` | `Handle` | базовый переход шага; сюда же уходит клик по `{{link}}` и ссылка продолжения PWA, если у шага нет `all links` |
| `all links` / `link 1`…`link 5` | `Handle link` / `Handle link N` | клик по `{{link}}` / по `{{link:N}}` при наличии соответствующего перехода |
| `all forms` / `form 1`…`form 5` | `Handle form` / `Handle form N` | сабмит ненумерованной формы / формы `{{form:N}}` при наличии соответствующего перехода |
| `passed` / `rejected` | `Passed` / `Rejected` | исход `Filter`, `Condition`, `Field verification` |
| `no payload` | `No payload` | на шаге не подошёл ни один вариант |
| `pushed` / `reject` / `full` / `interact` | `Destination-pushed` / `Destination-rejected` / `Destination Full` / `Destination-interacted` | исходы пуша в Destination |
| `repeat` / `end repeat` | повтор в флоу рассылок | нода `Repeat` |

Когда какой переход выбирается и что с ним происходит дальше — [models/flow-model.md](../models/flow-model.md).

### Как выбрать или сменить переход у стрелки

**Клик по подписи на стрелке открывает меню переходов, которые допускает исходный шаг, и пункт `Delete transition` («Удалить переход»).** Новая стрелка сразу получает первый допустимый переход; если у шага их несколько, меню открывается само. Переход `handle` у шага может быть только один: если он уже занят другой стрелкой, новая стрелка из `Content` получает `all forms` — следующий по списку. Поэтому после соединения шагов проверьте подпись на стрелке: `all forms` вместо `handle` — частая причина `Command not found`.

## Открыть настройки шага и сохранить их

Когда: нужно поменять вариант, стратегию, правила или `Visual settings` шага во флоу.

**Путь:** `Edit Flow → клик по ноде шага → модалка настроек → правки → Apply settings → Save`.

Клик по любому месту ноды (имя, описание, значок-шестерёнка) открывает модалку с настройками шага; не открывают её только порты транзишенов и drag-хендл. Форма пишет в шаг живьём, поэтому кнопка `Apply settings` (RU `Применить настройки`) ничего не применяет — она закрывает окно. **Флоу при этом ещё не сохранён**: без `Save` в редакторе флоу правки шага пропадут. Полный состав модалки и настройки каждой ноды — [models/flow-model.md](../models/flow-model.md) (раздел «Настройки нод — поштучно»).

## Передвинуть ноду на канвасе

Когда: нужно разложить схему руками, а не `Re-sort`.

**Путь:** зажать drag-хендл — крестик из четырёх стрелок слева в строке ноды — и тянуть.

Тянуть за тело ноды нельзя: клик по нему открывает модалку настроек. Симптом «нода не двигается, вместо этого открывается окно настроек» — значит, тянули не за хендл. Хендл есть только в `Edit flow` — в `View flow` ноды не двигаются. Схема раскладывается сверху вниз; авто-раскладка — кнопка `Re-sort` в нижнем тулбаре канваса. Как устроена нода на канвасе — [models/flow-model.md](../models/flow-model.md).

## Посмотреть схему флоу без редактирования — `View flow`

Когда: нужно быстро понять форму флоу (ветки `Filter`, где какие шаги), не открывая редактор.

**Путь:** правый клик по строке флоу → `View flow`; из карточки кампании — кнопка `Flow preview` (иконка глаза) рядом с выбранным флоу.

Схема открывается на весь экран, сверху вниз. Во флоу от 10 нод линейные цепочки шагов свёрнуты в одну ноду со списком — клик по ней разворачивает цепочку, повторный клик сворачивает; шаг `Filter` не сворачивается никогда; линейная пара `Start` → `Source` сворачивается даже в коротком флоу. Внизу — легенда «иконка + цвет → тип шага» по фактическому составу схемы. Настройки шагов из `View flow` не открываются. Правила сворачивания и легенды — [models/flow-model.md](../models/flow-model.md).

## Удалить висящий блок из Flow Editor

Когда: при редактировании флоу остался несохранённый блок, не подключённый к остальным.

**Шаги (пока сам блок не сохранён; кроме `Start`):**

1. Выделить блок кликом (клик заодно открывает модалку настроек — закрыть её `Apply settings`).
2. Нажать клавишу `Delete` (или `Backspace`), пока блок выделен и фокус находится на ноде.

Новый блок можно удалить и при редактировании ранее сохранённого флоу. **Сохранённый шаг удалить нельзя** — только **архивировать** тумблером `Archive state` в блоке `Visual settings` настроек шага; `Start` не архивируется. `Disable node` / `Clear recursively` — это механика **дистрибуций**, а не флоу; во флоу её нет.

## Архивировать шаг сохранённого флоу

Когда: сохранённый шаг больше не нужен. Полностью удалить его в редакторе нельзя; кроме `Start`, его можно архивировать после удаления связанных стрелок.

**Путь:** `Edit Flow → клик по ноде шага → Visual settings → Archive state → Apply settings → Save`.

Архивировать шаг можно, только сняв все transitions к нему и от него — иначе архивирование не сработает ([models/flow-model.md](../models/flow-model.md) → состояние `Archived`). Заархивированные шаги показываются тумблером `Show archived states` в панели `Flow details`.

Клавиша `Delete` не удаляет уже сохранённый шаг; новые блоки остаются удаляемыми до их сохранения. Не путать с дистрибуциями (там `Disable node` + `Clear recursively`).

Если клавиша не срабатывает, проверьте, что канвас разблокирован, блок выделен, фокус находится на ноде и это не `Start`.

## Сменить Settings Availability шага

Когда: настройка должна быть видна / скрыта на уровне кампании или флоу. Типовой случай — настройку нужно жёстко зафиксировать на уровне Flow, чтобы она была одинакова во всех кампаниях и не правилась в каждой отдельно → `Flow Only`.

**Путь:** `Edit Flow → <Step> → Settings Availability → Campaign Only / Flow Only / Campaign Template`.

Три значения и их механика — [models/flow-model.md](../models/flow-model.md). См. [heuristics/campaign-defaults.md](../heuristics/campaign-defaults.md) для эвристик.

## Add Handle-link (нумерованные кнопки на лэнде)

Когда: на одном лэнде несколько Call-to-Action кнопок, каждая ведёт на свой шаг флоу.

**Путь:** `Edit Flow` → стрелка от `Content`-шага к шагу, куда ведёт кнопка → клик по подписи на стрелке → `link 1`, `link 2`, … (раздел «Как выбрать или сменить переход у стрелки»).

В HTML лэнда:

```html
<a href="{{link:1}}">Кнопка 1</a>
<a href="{{link:2}}">Кнопка 2</a>
```

Каждый `{{link:N}}` соответствует Handle-link N во флоу. Доступно до 5 нумерованных линков.

В меню шага есть отдельные переходы `link 1`…`link 5`. Для `{{link:N}}` задайте стрелку с тем же номером. Переход `all links` обслуживает ненумерованный `{{link}}`; базовый `handle` — резервный переход, когда нужного перехода по номеру нет.

Также см. [reference/placeholders.md](../reference/placeholders.md) → `{{link:N}}`, [reference/glossary.md](../reference/glossary.md) → `Handle-link N`.

## Fallback при переполнении капы (Destination Full)

Когда: Destination достиг капы, новые визиты должны уходить в fallback-Destination или fallback-дистрибуцию.

**Путь:** `Flow Editor → Destination node → transition Destination Full → выбрать другой Destination / Distribution`.

Капа проверяется при обработке Destination: поэтому и уже зашедший визит уйдёт в fallback, если к этому моменту блокирующая капа заполнена и настроен переход `full`. Проверка предела одинакова для `Daily`, `Monthly` и `Lifetime`. Conversion cap со стратегией `Nothing` трафик не блокирует и сама по себе этот переход не запускает. Механика капов (`Infinity` / `Daily` / `Monthly` / `Lifetime`, блокирующая логика) — [models/conversion-model.md](../models/conversion-model.md).

## Связанные материалы

- [models/flow-model.md](../models/flow-model.md) — концепт: States, Transitions, Settings Availability, Split Groups.
- [how-to/campaigns.md](campaigns.md) — создание кампании на флоу.
- [how-to/distributions.md](distributions.md) — Content Distribution в Content step.
- [heuristics/campaign-defaults.md](../heuristics/campaign-defaults.md) — Campaign Template и Skip-Skip защита.
