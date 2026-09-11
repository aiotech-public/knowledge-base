---
id: flow-editor
title: Flow Editor (How-to)
description: Редактирование Flow — модалка настроек шага и Apply settings, drag-хендл ноды, View flow, удаление блоков, смена Settings Availability, добавление Handle-link, fallback на переполнение капы.
doc_type: how-to
builds: [erp]
related: [flow-model, campaign-defaults, placeholders, glossary, conversion-model, campaigns, distributions]
language: ru
updated: 2026-09-11
---

# Flow Editor (How-to)

## Краткая шпаргалка по Flow Editor

Мелкие, но регулярные процедуры в Flow Editor. Концепция Flow (States, Transitions, Settings Availability) — в [models/flow-model.md](../models/flow-model.md).

- Настройки шага — клик по ноде открывает модалку; `Apply settings` только закрывает её, флоу сохраняет `Save`.
- Передвинуть ноду — тянуть за drag-хендл (крестик слева в строке ноды), не за тело.
- Удалить «висящий» блок — выделить + клавиша `Delete` (только до сохранения флоу).
- `Settings Availability` шага — `Campaign Only` / `Flow Only` / `Campaign Template`. Меняется в `Edit Flow → <Step> → Settings Availability`.
- `Handle-link` — нумерованные кнопки на лэнде через `{{link:N}}`.
- Fallback при переполнении капы — `Destination Full` transition.
- Посмотреть схему без правки — `View flow` (полный экран, цепочки шагов свёрнуты, разворачиваются кликом).

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

Схема открывается на весь экран, сверху вниз. Во флоу от 10 нод линейные цепочки шагов свёрнуты в одну ноду со списком — клик по ней разворачивает цепочку, повторный клик сворачивает; шаг `Filter` не сворачивается никогда, пара `Start` → `Source` — всегда. Внизу — легенда «иконка + цвет → тип шага» по фактическому составу схемы. Настройки шагов из `View flow` не открываются. Правила сворачивания и легенды — [models/flow-model.md](../models/flow-model.md).

## Удалить висящий блок из Flow Editor

Когда: при редактировании флоу остался несохранённый блок, не подключённый к остальным.

**Шаги (только пока флоу не сохранён):**

1. Выделить блок кликом (клик заодно открывает модалку настроек — закрыть её `Apply settings`).
2. Нажать клавишу `Delete` (или `Backspace`), пока блок выделен.

Работает **только до сохранения флоу**. **После сохранения шаг удалить нельзя** — только **архивировать** тумблером `Archive state` в блоке `Visual settings` настроек шага. `Disable node` / `Clear recursively` — это механика **дистрибуций**, а не флоу; во флоу её нет.

## Архивировать шаг сохранённого флоу

Когда: флоу уже сохранён, и шаг больше не нужен (полностью удалить нельзя — он уже часть графа). Это **единственный** способ убрать шаг из работающего флоу.

**Путь:** `Edit Flow → клик по ноде шага → Visual settings → Archive state → Apply settings → Save`.

Архивировать шаг можно, только сняв все transitions к нему и от него — иначе архивирование не сработает ([models/flow-model.md](../models/flow-model.md) → состояние `Archived`). Заархивированные шаги показываются тумблером `Show archived states` в панели `Flow details`.

**Не существует** «Delete для сохранённого шага» — кнопка `Delete` после сохранения работать перестаёт. Не путать с дистрибуциями (там `Disable node` + `Clear recursively`).

Отдельный случай — блок, который вообще не удаляется, потому что ещё не сохранён: несохранённый блок убирается только перезагрузкой редактора без сохранения.

## Сменить Settings Availability шага

Когда: настройка должна быть видна / скрыта на уровне кампании или флоу. Типовой случай — настройку нужно жёстко зафиксировать на уровне Flow, чтобы она была одинакова во всех кампаниях и не правилась в каждой отдельно → `Flow Only`.

**Путь:** `Edit Flow → <Step> → Settings Availability → Campaign Only / Flow Only / Campaign Template`.

Три значения и их механика — [models/flow-model.md](../models/flow-model.md). См. [heuristics/campaign-defaults.md](../heuristics/campaign-defaults.md) для эвристик.

## Add Handle-link (нумерованные кнопки на лэнде)

Когда: на одном лэнде несколько Call-to-Action кнопок, каждая ведёт на свой шаг флоу.

**Путь:** `Edit Flow → <Content step> → +Handle-link → выбрать номер (1, 2, …)`.

В HTML лэнда:

```html
<a href="{{link:1}}">Кнопка 1</a>
<a href="{{link:2}}">Кнопка 2</a>
```

Каждый `{{link:N}}` соответствует Handle-link N во флоу. Доступно до 5 нумерованных линков.

Каждый номер (Link 1, Link 2, Link 3 … до Link 5) расширяет палитру транзишенов шага: под каждый заводится отдельный transition — `Handle link 1`, `Handle link 2`, `Handle link 3`, …, `Handle link 5`. Дополнительно есть `Handle all links` — один transition для группировки всех пронумерованных линков сразу. Базовый `Handle` остаётся как fallback: срабатывает, если ни один пронумерованный `Handle link N` не подошёл.

Также см. [reference/placeholders.md](../reference/placeholders.md) → `{{link:N}}`, [reference/glossary.md](../reference/glossary.md) → `Handle-link N`.

## Fallback при переполнении капы (Destination Full)

Когда: Destination достиг капы, новые визиты должны уходить в fallback-Destination или fallback-дистрибуцию.

**Путь:** `Flow Editor → Destination node → transition Destination Full → выбрать другой Destination / Distribution`.

Уже зашедшие, но ещё не дошедшие до Destination — пойдут в fallback. Переход срабатывает одинаково на любом типе капы с пределом — и на `Daily`, и на `Lifetime`. Механика капов (`Infinity` / `Daily` / `Lifetime`, блокирующая логика) — [models/conversion-model.md](../models/conversion-model.md).

## Связанные материалы

- [models/flow-model.md](../models/flow-model.md) — концепт: States, Transitions, Settings Availability, Split Groups.
- [how-to/campaigns.md](campaigns.md) — создание кампании на флоу.
- [how-to/distributions.md](distributions.md) — Content Distribution в Content step.
- [heuristics/campaign-defaults.md](../heuristics/campaign-defaults.md) — Campaign Template и Skip-Skip защита.
