---
id: flow-editor
title: Flow Editor (How-to)
description: Редактирование Flow — удаление блоков, смена Settings Availability, добавление Handle-link, fallback на переполнение капы.
doc_type: how-to
builds: [erp]
related: [flow-model, campaign-defaults, placeholders, glossary, conversion-model, campaigns, distributions]
language: ru
updated: 2026-08-11
---

# Flow Editor (How-to)

## Краткая шпаргалка по Flow Editor

Мелкие, но регулярные процедуры в Flow Editor. Концепция Flow (States, Transitions, Settings Availability) — в [models/flow-model.md](../models/flow-model.md).

- Удалить «висящий» блок — выделить + клавиша `Delete` (только до сохранения флоу).
- `Settings Availability` шага — `Campaign Only` / `Flow Only` / `Campaign Template`. Меняется в `Edit Flow → <Step> → Settings Availability`.
- `Handle-link` — нумерованные кнопки на лэнде через `{{link:N}}`.
- Fallback при переполнении капы — `Destination Full` transition.

## Удалить висящий блок из Flow Editor

Когда: при редактировании флоу остался несохранённый блок, не подключённый к остальным.

**Шаги (только пока флоу не сохранён):**

1. Выделить блок кликом.
2. Нажать клавишу `Delete`.

Работает **только до сохранения флоу**. **После сохранения шаг удалить нельзя** — только **архивировать** через `Visual Settings → Archive` на самом шаге. `Disable node` / `Clear recursively` — это механика **дистрибуций**, а не флоу; во флоу её нет.

## Архивировать шаг сохранённого флоу

Когда: флоу уже сохранён, и шаг больше не нужен (полностью удалить нельзя — он уже часть графа). Это **единственный** способ убрать шаг из работающего флоу.

**Путь:** `Edit Flow → <Step> → Visual Settings → Archive`.

Архивировать шаг можно, только сняв все transitions к нему и от него — иначе архивирование не сработает ([models/flow-model.md](../models/flow-model.md) → состояние `Archived`). Заархивированные шаги показываются тумблером `Show archived states` в панели `Flow details`.

**Не существует** «Delete для сохранённого шага» — кнопка `Delete` после сохранения работать перестаёт. Не путать с дистрибуциями (там `Disable node` + `Clear recursively`).

Отдельный случай — блок, который вообще не удаляется, потому что ещё не сохранён: несохранённый блок убирается только перезагрузкой редактора без сохранения. Разбор — *Флоу не сохраняется — несохранённый блок в редакторе*.

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
- *Флоу не сохраняется — несохранённый блок в редакторе* — почему флоу не сохраняется.
- [heuristics/campaign-defaults.md](../heuristics/campaign-defaults.md) — Campaign Template и Skip-Skip защита.
