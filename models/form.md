---
id: form
title: Form (захват лида) — концепт (модель)
description: Что такое Form в AIO как сущность — настраиваемая форма захвата лида со своим Form Builder (Info/Controls/Steps/Layouts/Languages/Preview), маппингом контрол↔поле визита и привязкой к флоу; SDK-форма vs Non-SDK Handler, два контекста Required. Концепт; точный разбор по коду (lifecycle/хуки/intl-tel-input) — reference/sdk.md, раздел «SDK-форма — полный разбор».
doc_type: model
builds: [erp, mtk]
related: [sdk, forms, landing, placeholders, flow-model, visit-field, destination, conversion-model, custom-fields, glossary, ui-map]
language: ru
updated: 2026-08-12
---

# Form (захват лида) — концепт (модель)

## Суть в одном абзаце

**Form — это настраиваемая форма захвата лида: самостоятельная сущность со своим Form Builder UI, которая определяет, *какие* поля вводит визит и *на какие поля визита* они мапятся.** Живёт в `Settings → Forms` (`/forms`), привязывается к кампании через флоу (нода `Fill form`) и рендерится на лэнде в плейсхолдере `{{form}}`. При этом сама форма — это «лицо» (контролы, шаги, переводы, раскладка), а почти всё «поведение» (как валидируется, как сабмитится, как пушит лид) живёт в **AIO SDK** (`config.form`) — поэтому Form неотделима от [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор») и работает только при подключённом SDK.

## Что такое Form и Form Builder — сущность с собственным UI

У формы есть top-level подсекция `Settings → Forms` — список ваших форм со своими экшенами (правый клик по строке: `Edit form`, `Form builder`, `Copy form`). `+Form` / `Edit Form` открывает **Form Builder** с 6 вкладками:

- **Info** — Name, Description, Tags, Base language, Color, Icon;
- **Controls** — список инпутов + правила валидации (см. раздел Control ниже);
- **Steps** — многошаговость;
- **Layouts** — визуальное расположение контролов;
- **Languages** — переводы;
- **Preview** — превью.

Дефолтная **Default Leads Form** = 4 контрола: First Name, Last Name, Email, Phone — каждый смаплен на одноимённое поле визита (совпадает с дефолтом SDK-формы). Дублирование формы (`Copy form`) делает **полную копию** формы, включая Controls (маппинги контрол↔поле визита), Steps, Layout, Languages (можно отредактировать), но **не** копирует Tags и Description. Подробно процедуры — [how-to/forms.md](../how-to/forms.md).

## Что такое Control — инпут плюс маппинг на поле визита

Каждый **Control** — это один инпут. У контрола есть атрибуты `key`, `placeholder`, `label`, `value`, `type`, `rules`, а в `Control settings` (по шестерёнке) — поле **Field**: на *какое поле визита* пишется значение (напр. `Visit → First Name`). **Это и есть slug-маппинг контрол↔поле визита** — именно он определяет, какие данные и куда уходят.

Тип контрола задаёт вид инпута (текстовые, email, phone, number, date, выбор/чекбокс и служебные); полный перечень типов — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).

Правила (`rules`) превращаются в HTML-атрибуты: `required`, `minlength:N`, `maxlength:N`, `pattern` (особое `pattern:no-local-emails` отсекает local-адреса). Точные дефолты валидации каждого контрола (First Name / email / phone …) и полный разбор типов — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).

## Как работают Steps — многошаговая форма

**Steps** = массив массивов ключей полей: каждый под-массив = один шаг (напр. шаг 1: email, шаг 2: phone). На промежуточном шаге сабмит = *отправить поля текущего шага в визит + перейти на следующий шаг* (это **не** пуш лида). Валидация применяется только к полям активного шага. Индикатор шагов управляется опцией `stepsShowing`.

## Layouts — только визуальное расположение, не маппинг данных

**Layouts** отвечают исключительно за визуальное расположение. Layout = массив массивов: внутри одного массива поля идут side-by-side, между массивами — на новой строке. Ключевое следствие: если push успешен, но у получателя данные перепутаны (firstname вместо телефона) — виноваты `key` контролов, не Layouts; чинить в Controls. См. *уточните у поддержки*.

## Как работают Languages — мульти-язычные переводы формы

**Один Base Language + N additional; язык вычисляется один раз при загрузке и переключателя языка для визитёра нет.**

Наборы языков — два разных, их легко перепутать:

- **Form Builder** предлагает 53 языка с готовым авто-переводом (сегмент `Auto-translated`) из полного справочника в 185 языков (сегмент `All languages` — приходит **пустым**).
- **У SDK есть свой встроенный пак на 33 языка.** Когда к лэнду привязана форма из ERP, её словарь заменяет пак SDK целиком; сам пак остаётся крайним фолбэком только для четырёх строк `SUBMIT*`.

Язык из `Auto-translated` приходит **частично заполненным**: label и placeholder — только у контролов `first_name` / `last_name` / `email` / `phone`, смапленных на поле визита, плюс четыре Submit-строки; кастомные контролы и опции `Drop list` остаются пустыми. Кастомные переводы **не** копируются из SDK defaults автоматически.

Фолбэк по строке: нужный язык → `en` → сам ключ. Полный список ключей строк — [reference/sdk.md](../reference/sdk.md) (раздел «i18n и переводы формы»); процедуры перевода — [how-to/forms.md](../how-to/forms.md).

## Два пути: SDK-форма vs Non-SDK Form Handler — чем отличаются

- **SDK-форма** — основной/рекомендуемый путь. Рендерится в плейсхолдер `{{form}}`, настраивается в `Settings → Forms`, маппинг авто, активно развивается.
- **Non-SDK Form Handler** (макрос `aio_nonsdk_form_handler`) — фолбэк для лэнда с родной HTML-`<form>`; цепляется **только** к `<form>` (если форма свёрстана как `<div>` — сабмит не перехватывается). Активно **не** развивается. Механика, slug-маппинг и побочка со стилями `intl-tel-input` — [how-to/forms.md](../how-to/forms.md).

## Как Form проходит путь: рендер → submit → лид

Форма «оживает» только при подключённом SDK:
1. на лэнде есть `{{aio}}` (загрузка SDK) и включена фича `features.form` (встроенный дефолт SDK для `form` = OFF; шаблон обычно ставит `form: true` поверх);
2. в HTML лэнда есть `{{form}}` — место рендера; без него форма не отрисуется, даже если настроена во флоу;
3. SDK ищет элементы по `form.selector` (дефолт `.aio-form`) и достраивает контролы;
4. данные собираются **прогрессивно** — каждое поле шлётся в визит ещё до сабмита (механика `collectDataEvent` — [reference/sdk.md](../reference/sdk.md)). Это объясняет вопрос «поля визита имя/почта заполнились, хотя юзер форму не отправил» — так и задумано;
5. на сабмите — GET на `form.url`; в данные всегда добавляются `_aio_handle=form` + пара `handle.key=handle.value`, по которым запрос распознаётся как форма — визит идёт по `Handle-form`, лид уходит в `Destination`.

### Полный lifecycle сабмита — где разбор

Полный lifecycle сабмита (валидация телефона → `disableForms` → хук `beforeSubmitPromise` → `submitPromises`/`formPromise` → fetch → success/reject/reload → редирект на autologin) — [reference/sdk.md](../reference/sdk.md) (раздел «Lifecycle сабмита SDK-формы»).

## Связанные макросы Form Loader и Static Language Country

Поведение формы (`config.form`), `intl-tel-input` и CSS-переменные настраиваются в `Content → Macros → AIO SDK Macros Collection`. В MTK это устроено иначе — макросы доступны через `Settings → Macros` (см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*). Шаблонные макросы вокруг формы:
- **Form Loader** (`aio_form_loader`) — после первого сабмита блокирует повторные нажатия кнопки и показывает лоадер; снимает «юзер нажал 3 раза → 3 заявки-дубля»;
- **Static Language Country** (`aio_static_language__country_for_form`) — без него ломается под VPN; механика и рекомендации — [reference/sdk.md](../reference/sdk.md);
- валидация телефона — сторонняя `intl-tel-input.com`, с гео-спецификой (DE, BR) и `intlParameters`.

Детали — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»), [how-to/forms.md](../how-to/forms.md).

## Как Form связан с остальными сущностями

### Landing — форма рендерится через `{{form}}`

Форма рендерится на лэнде через `{{form}}`; без `{{aio}}` на лэнде SDK и форма не грузятся. → [models/landing.md](landing.md), [reference/placeholders.md](../reference/placeholders.md)

### AIO SDK — форма это фича `features.form`

Форма это фича `features.form`; конфиг через `config.form`; lifecycle / хуки / `intl-tel-input` живут в коде SDK. Главный компаньон → [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).

### Flow — форма подключается нодой `Fill form`

Форма подключается нодой `Fill form` (подпись `Select form` / `Choose Form`) либо через Content-шаг; разные формы можно сплитовать по весам. Submit формы запускает transition `Handle-form` (vs `Handle-link` при клике `{{link}}`). При нескольких формах через `{{form:1}}`, `{{form:2}}` (до 5) каждая запускает свой transition-хендл: `Handle form 1`, `Handle form 2`, 3, 4, 5; отдельно есть `Handle all forms`. Параллель к поведению `{{link:N}}` в transition'ах флоу. → [models/flow-model.md](flow-model.md)

### Visit Field — каждый Control мапится на поле визита

Каждый Control мапится на поле визита (Field в Control settings); значения собираются прогрессивно на `blur` + на submit; есть поле визита `form_uuid`. → [models/visit-field.md](visit-field.md)

### Destination — успешный submit пушит лид

Успешный submit пушит лид в Destination (через `_aio_handle=form` + handle); `rejectedMessage` показывается при `success:false`; проверки перед пушем могут отрезать лид. → [models/destination.md](destination.md), *уточните у поддержки*

### Conversion — на успешном пуше шлётся `form-submit-fn`

На успешном пуше `form-submit-fn` шлёт FB/TT lead-события; `gtag` шлётся при submit *до* проверки лида. → [models/conversion-model.md](conversion-model.md)

### Antifraud на сабмите — проверки перед пушем

На сабмите лид проходит встроенный `AIO Antifraud` — проверки перед пушем в Destination; их состав и настройка описаны.

## Чем Form отличается от смежных понятий

| Не путать | Разница |
|---|---|
| **SDK-форма vs Non-SDK Form Handler** | SDK — `{{form}}`, настраивается в `Settings → Forms`, маппинг авто, развивается. Non-SDK — макрос-фолбэк для родной `<form>`, slug инпутов обязан совпадать буквально, цепляется только к `<form>`, **не** развивается (Pushing modal / autologin / хуки не работают). |
| **Controls vs Layouts** | Controls определяют `key` инпутов и маппинг на поля визита — *какие* данные и *куда* уходят. Layouts — **только** визуальное расположение. Данные у получателя перепутаны → чинить Controls. |
| **Form Control Required vs Visit Field required** | Два разных уровня: Form Builder `Required` = валидация SDK-формы; Visit Field `required` = только через `Link Generator` (обязательность параметра в URL, не поля). Детали — [how-to/forms.md](../how-to/forms.md). |
| **Fill form нода vs `{{form}}` плейсхолдер** | `Fill form` (нода во флоу) **выбирает**, какая форма привязана к кампании/шагу. `{{form}}` — место в HTML лэнда, **куда** форма рендерится. Нужны оба. |
| **Поля захвата лида vs трекинговые поля визита SDK** | Контролы формы (`first_name`/`last_name`/`email`/`phone`) — данные, которые вводит визит. SDK дополнительно сам пишет десятки трекинговых полей (fingerprint, browser_time_zone, scroll, time_on_landings) — это **не** контролы формы. |

## Подводные камни Form — частые ошибки настройки

### Подводные камни подключения и поведения

- **Нет `{{form}}` на лэнде** — форма не отрисуется, даже если настроена во флоу. И наоборот: `Fill form` во флоу обязателен, чтобы SDK знал, *какую* форму грузить.
- **`features.form` = OFF по дефолту SDK** — форма работает только если фича включена (обычно шаблон ставит `form: true` поверх). Без этого — пустое место.
- **Прогрессивный сбор (blur) ≠ submit.** Поля визита заполняются по мере ввода, до отправки — это норма, не баг.
- **Required — два контекста** (Form Builder vs поле визита через Link Generator) — детали в [how-to/forms.md](../how-to/forms.md), [how-to/custom-fields.md](../how-to/custom-fields.md).

### Подводные камни кода, версий и стилей

- **Хук сабмита — `beforeSubmitPromise`, в единственном числе.** Вариант с `beforeSubmitPromises` (во множественном) SDK **не** подхватит. Полный разбор поведения формы — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).
- **`?v=` в ссылке на бандл — не версия SDK.** Параметр только сбивает кэш браузера: на статике лежит один бандл, и агент отдаёт его независимо от query. Разные числа на разных лэндах и в превью формы не означают разных версий SDK — поведение опции сверяй с кодом актуального бандла, а не с этим параметром ([reference/sdk.md](../reference/sdk.md), раздел «Ключи, на которых чаще всего спотыкаются»).
- **Non-SDK Form Handler тянет свои CSS `intl-tel-input`** — форма может «съезжаться»; лечить точечной адаптацией стилей или переходом на SDK (стили лэнда не удалять).

## Куда идти за деталями — смежные темы

- **Полное поведение формы** (lifecycle, controls, шаблоны, хуки, `intl-tel-input`, CSS-переменные) → [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»); три способа обработки → раздел «Три способа обработки формы — что выбрать»; `UTILS` для хуков → раздел «UTILS — что доступно хукам формы».
- **Процедуры** (создать/дублировать SDK-форму, Form Builder вкладки, Control settings, Non-SDK setup, Required-контексты, валидация телефона) → [how-to/forms.md](../how-to/forms.md).
- **Плейсхолдеры** (`{{form}}`, `{{aio}}`, `{{aio:macros:nonsdk_form_handler}}`; вариант для нескольких форм — в `placeholders.md`) → [reference/placeholders.md](../reference/placeholders.md).

### Куда подключается форма и где UI

- **Подключение к пути** (нода `Fill form`, Content-шаг, transition `Handle-form`) → [models/flow-model.md](flow-model.md).
- **Маппинг контрол↔поле визита, Required-контексты** → [models/visit-field.md](visit-field.md), [how-to/custom-fields.md](../how-to/custom-fields.md).
- **Куда уходит лид после submit** → [models/destination.md](destination.md).
- **Термины** (SDK-форма, Form Builder, Controls, Steps, Layouts, Languages, Form Behavior settings, Pushing modal, Advanced Form Sample, CSS-переменные) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (`Settings → Forms` `/forms`; Form Builder: Info/Controls/Steps/Layouts/Languages/Preview; поведение — `Content → Macros → AIO SDK Macros Collection`; привязка — `Tracker → Flows → Edit Flow → Fill form`; рендер — `{{form}}` в коде лэнда).
