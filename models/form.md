---
id: form
title: Form (захват лида) — концепт (модель)
description: Что такое Form в AIO как сущность — настраиваемая форма захвата лида со своим Form Builder (Info/Controls/Steps/Layouts/Languages/Preview), маппингом контрол↔поле визита и привязкой к флоу; SDK-форма vs Non-SDK Handler, два контекста Required. Концепт; точный разбор по коду (lifecycle/хуки/intl-tel-input) — reference/sdk.md, раздел «SDK-форма — полный разбор».
doc_type: model
builds: [erp, mtk]
related: [sdk, forms, architecture, landing, placeholders, flow-model, visit-field, destination, conversion-model, custom-fields, glossary, ui-map]
language: ru
updated: 2026-10-09
---

# Form (захват лида) — концепт (модель)

## Суть в одном абзаце

**Form — это настраиваемая форма захвата лида: самостоятельная сущность со своим Form Builder UI, которая определяет, *какие* поля вводит визит и *на какие поля визита* они мапятся.** В ERP управляется через `Settings → Forms` (`/app/settings/forms`), привязывается к кампании через флоу (нода `Fill form`) и рендерится на лэнде в плейсхолдере `{{form}}`. При этом сама форма — это «лицо» (контролы, шаги, переводы, раскладка), а почти всё «поведение» (как валидируется, как сабмитится, как пушит лид) живёт в **AIO SDK** (`config.form`) — поэтому Form неотделима от [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор») и работает только при подключённом SDK.

## Что такое Form и Form Builder — сущность с собственным UI

У формы есть top-level подсекция `Settings → Forms` — список ваших форм со своими экшенами (правый клик по строке: `Edit form`, `Form builder`, `Copy form`). `+Form` и `Form builder` открывают **Form Builder** с вкладками `Settings` и `Preview`. В `Settings` находятся панель `Info` и разделы `Controls`, `Steps`, `Layouts`, `Languages`. `Edit form` открывает отдельный диалог с JSON-настройками. Состав Form Builder:

- **Info** — Name, Description, Tags, Base language, Color, Icon;
- **Controls** — список инпутов + правила валидации (см. раздел Control ниже);
- **Steps** — многошаговость;
- **Layouts** — визуальное расположение контролов;
- **Languages** — переводы;
- **Preview** — превью.

`Copy form` открывает Form Builder с Controls, Steps, Layouts, Languages, Tags и Description исходной формы. Перед сохранением их можно изменить; сохраняется новая форма. Подробно процедуры — [how-to/forms.md](../how-to/forms.md).

### Какие формы уже есть в тенанте из шаблона

Набор форм приходит с шаблоном тенанта и зависит от его вертикали ([context/architecture.md](../context/architecture.md)):

- **Finance** — `Default Leads Form` (First Name, Last Name, Email, Phone), `Leads Form w/ Consent` (те же четыре + чекбокс `consent`), `2-Step Leads Form` (шаг 1 — email, шаг 2 — имя, фамилия, телефон), `Qualifying Form w/ Investment Range` (шаг 1 — выбор суммы `investment_amount`, шаг 2 — имя, телефон, согласие), `Short Form - Name & Phone`.
- **Sweeps** — `Email Only Form`, `Default SOI Form` (First Name, Last Name, Email, Phone), `Quiz Form - Prize & Age` (три шага: приз → возраст → контакты и согласие), `Shipping Form - Physical Prize` (контакты, затем адрес доставки: `address`, `city`, `state`, `zip_code`).
- **Nutra** — `Order Form - Name & Phone`, `Order Form - Package & Address` (три шага: количество `quantity` → имя и телефон → адрес), `Order Form - Full Address` (контакты, затем адрес с квартирой и комментарием), `Quiz Form - Age & Contacts` (возраст, затем имя и телефон).
- **iGaming** — форм в шаблоне нет: воронка шаблона идёт на PWA и редирект-офферы без формы захвата.

У всех шаблонных форм базовый язык `English`, стандартные контролы (`first_name`, `last_name`, `email`, `phone`) смаплены на одноимённые поля визита, а языки уже добавлены — несколько десятков (у Nutra набор короче). Что именно добавляют языки и что остаётся пустым у нового контрола — [how-to/forms.md](../how-to/forms.md).

## Что такое Control — инпут плюс маппинг на поле визита

**Control** описывает элемент формы: поле ввода, выбор, чекбокс или служебный элемент. У контрола есть атрибуты `key`, `placeholder`, `label`, `value`, `type`, `rules`, а в `Control settings` (по шестерёнке) — поле **Field**: на *какое поле визита* пишется значение (напр. `Visit → First Name`). **Это и есть slug-маппинг контрол↔поле визита** — именно он определяет, какие данные и куда уходят.

Тип контрола задаёт вид инпута (текстовые, email, phone, number, date, выбор/чекбокс и служебные); полный перечень типов — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).

Правила (`rules`) превращаются в HTML-атрибуты: `required`, `minlength:N`, `maxlength:N`, `pattern` (особое `pattern:no-local-emails` отсекает local-адреса). Точные дефолты валидации каждого контрола (First Name / email / phone …) и полный разбор типов — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).

## Как работают Steps — многошаговая форма

**Steps** = массив массивов ключей полей: каждый под-массив = один шаг (напр. шаг 1: email, шаг 2: phone). На промежуточном шаге сабмит = *отправить поля текущего шага в визит + перейти на следующий шаг* (это **не** пуш лида). Валидация применяется только к полям активного шага. Индикатор шагов управляется опцией `stepsShowing`.

## Layouts — только визуальное расположение, не маппинг данных

**Layouts** отвечают исключительно за визуальное расположение. Layout = массив массивов: внутри одного массива поля идут side-by-side, между массивами — на новой строке. Ключевое следствие: если push успешен, но у получателя данные перепутаны (firstname вместо телефона), проверьте Field и `key` в Controls; Layouts не меняет имена отправляемых полей. См. *уточните у поддержки*.

## Как работают Languages — мульти-язычные переводы формы

**Один Base Language + N additional; язык вычисляется один раз при загрузке и переключателя языка для визитёра нет.**

Наборы языков — два разных, их легко перепутать:

- **Form Builder** предлагает 53 языка с готовым авто-переводом (сегмент `Auto-translated`) из полного справочника в 185 языков (сегмент `All languages` — приходит **пустым**).
- **У SDK есть свой встроенный пак на 33 языка.** Когда к лэнду привязана форма из ERP, её словарь заменяет пак SDK целиком; сам пак остаётся крайним фолбэком только для четырёх строк `SUBMIT*`.

При добавлении языка из `Auto-translated` авто-переводятся непустые label и placeholder базового языка у контролов `first_name` / `last_name` / `email` / `phone` с выбранным Field, а также непустые Submit-строки; кастомные контролы и опции `Drop list` остаются пустыми. Кастомные переводы **не** копируются из SDK defaults автоматически.

Фолбэк по строке: нужный язык → `en` → сам ключ. Полный список ключей строк — [reference/sdk.md](../reference/sdk.md) (раздел «i18n и переводы формы»); процедуры перевода — [how-to/forms.md](../how-to/forms.md).

## Два пути: SDK-форма vs Non-SDK Form Handler — чем отличаются

- **SDK-форма** — основной/рекомендуемый путь. Рендерится в контейнер формы; в ERP настраивается в `Settings → Forms`, маппинг авто, активно развивается.
- **Non-SDK Form Handler** (макрос `AIO Non-SDK Form Handler`, слаг `aio_nonsdk_form_handler`, вставка `{{aio:macros:aio_nonsdk_form_handler}}`; есть в шаблонах Finance, Nutra и Sweeps, в iGaming нет) — фолбэк для лэнда с родной HTML-`<form>`; цепляется **только** к `<form>` (если форма свёрстана как `<div>` — сабмит не перехватывается). Активно **не** развивается. Механика, slug-маппинг и побочка со стилями `intl-tel-input` — [how-to/forms.md](../how-to/forms.md); там же третий путь для сложной своей формы — `AIO Pre-Fill Form Handler` (только Sweeps).

## Как Form проходит путь: рендер → submit → лид

Форма «оживает» только при подключённом SDK:
1. SDK подключён, например через `{{aio}}` или `{{aio:head}}`, и включена фича `features.form` (встроенный дефолт SDK для `form` = OFF; шаблонный `AIO SDK Macros Collection` во всех вертикалях ставит `form: true` поверх);
2. в HTML есть `{{form}}` либо готовый `<form class="aio-form"></form>` — контейнер для формы; селектор можно изменить через `form.selector`;
3. SDK ищет элементы по `form.selector` (дефолт `.aio-form`) и достраивает контролы;
4. данные поддерживаемых input/textarea и select собираются **прогрессивно**, если включён `collectDataEvent` (дефолт `blur`). Пустые значения и сокращённые строки этот обработчик не отправляет; телефон преобразуется и собирается при submit ([reference/sdk.md](../reference/sdk.md)). Это объясняет вопрос «поля визита имя/почта заполнились, хотя юзер форму не отправил» — так и задумано;
5. финальный submit отправляет GET на `form.url`. В штатной конфигурации hidden-controls передают `_aio_handle=form` (или `form-N`) и `handle.key=handle.value`; выбранный transition определяет дальнейший путь визита. Если путь ведёт в `Destination`, выполняется попытка пуша.

### Полный lifecycle сабмита — где разбор

Полный lifecycle сабмита (валидация телефона → `disableForms` → хук `beforeSubmitPromise` → `submitPromises`/`formPromise` → fetch → success/reject/reload → редирект по возвращённому URL) — [reference/sdk.md](../reference/sdk.md) (раздел «Lifecycle сабмита SDK-формы»).

## Связанные макросы Form Loader и Static Language Country

Поведение формы (`config.form`), `intl-tel-input` и CSS-переменные настраиваются в `Content → Macros → AIO SDK Macros Collection`. В MTK это устроено иначе — макросы доступны через `Settings → Macros` (см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*). Шаблонные макросы вокруг формы — в Finance, Nutra и Sweeps (в iGaming-шаблоне нет ни форм, ни этих макросов); оба не глобальные — в шаблоне подключены к лэндам-примерам с формой, на свой лэнд вставляются строкой `{{aio:macros:<slug>}}`:
- **AIO Form Loader** (`aio_form_loader`) — на время отправки накрывает форму оверлеем с лоадером и локализованным сообщением; повторные нажатия не проходят — снимает «юзер нажал 3 раза → 3 заявки-дубля»;
- **AIO Static Language Country for Form** (`aio_static_language__country_for_form` в Finance, `aio_static_language_country_for_form` в Nutra и Sweeps) — берёт страну телефона у лэнда, если у него задана ровно одна страна, и язык формы — если ровно один язык (каждое независимо), иначе у визита; без него под VPN подставляется страна VPN-сервера. Механика и рекомендации — [reference/sdk.md](../reference/sdk.md);
- валидация телефона — сторонняя `intl-tel-input.com`, с гео-спецификой (DE, BR) и `intlParameters`.

Детали — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»), [how-to/forms.md](../how-to/forms.md).

## Как Form связан с остальными сущностями

### Landing — форма рендерится через `{{form}}`

`{{form}}` создаёт контейнер SDK-формы; можно также добавить готовый `<form class="aio-form"></form>`. Для отрисовки должен быть подключён SDK и включена `features.form`. → [models/landing.md](landing.md), [reference/placeholders.md](../reference/placeholders.md)

### AIO SDK — форма это фича `features.form`

Форма это фича `features.form`; конфиг через `config.form`; lifecycle / хуки / `intl-tel-input` живут в коде SDK. Главный компаньон → [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).

### Flow — форма подключается нодой `Fill form`

Форма подключается нодой `Fill form` (подпись `Select form` / `Choose Form`) либо через Content-шаг; разные формы можно сплитовать по весам. Submit формы запускает transition `Handle-form` (vs `Handle-link` при клике `{{link}}`). Для нескольких форм доступны `{{form:1}}` … `{{form:5}}`. Эти контейнеры обрабатываются, когда в HTML также есть обычный `{{form}}`; для номерной формы настройте соответствующий переход `Handle Form 1` … `Handle Form 5`. В меню также есть `Handle All Forms`. Параллель к поведению `{{link:N}}` в transition'ах флоу. → [models/flow-model.md](flow-model.md)

### Visit Field — маппинг контрола на поле визита

Field в Control settings задаёт поле визита, а UI синхронизирует `key` со slug этого поля. Поддерживаемые элементы собирают значения прогрессивно на `blur`; финальный submit собирает форму, включая телефон. Выбранная форма записывается в `form_uuid`. → [models/visit-field.md](visit-field.md)

### Destination — submit может вести к попытке пуша

Если transition формы ведёт в Destination, submit запускает попытку пуша; `rejectedMessage` показывается при `success:false`; проверки перед пушем могут отрезать лид. → [models/destination.md](destination.md), *уточните у поддержки*

### Conversion — на успешном пуше шлётся `form-submit-fn`

При ответе SDK-форме `success:true` запускаются `form-submit-fn` callbacks. Включённые FB/TT Pixel с заданным pixel ID отправляют настроенное `formSubmitEvent`; макрос `AIO GTAG` тоже шлёт конверсию Google на успешной форме (прежняя версия макроса слала её при submit, до проверки лида — [reference/sdk.md](../reference/sdk.md)). → [models/conversion-model.md](conversion-model.md)

### Antifraud на сабмите — проверки перед пушем

Для проверок перед пушем в путь формы добавляют `AIO Anti Fraud Filter`: он записывает результаты в поля визита, а Condition-шаг задаёт ветвление перед Destination. Сам submit не включает фильтр автоматически. Состав и настройка проверок описаны.

## Чем Form отличается от смежных понятий

### SDK-форма и Non-SDK Form Handler

| Не путать | Разница |
|---|---|
| **SDK-форма vs Non-SDK Form Handler** | SDK — контейнер `{{form}}` или `.aio-form`, в ERP настраивается в `Settings → Forms`, маппинг авто, развивается. Non-SDK — макрос-фолбэк для родной `<form>`, `name` инпутов = слаг поля буквально, цепляется только к `<form>`, **не** развивается (Pushing modal и хуков нет; после пуша — редирект по URL ответа (autologin, если интеграция вернула такую ссылку), реджект — `alert`). |

### Form Controls, Layouts и другие настройки

| Не путать | Разница |
|---|---|
| **Controls vs Layouts** | Controls определяют `key` инпутов и маппинг на поля визита — *какие* данные и *куда* уходят. Layouts — **только** визуальное расположение. Данные у получателя перепутаны → сначала проверить Field и `key` в Controls. |
| **Form Control Required vs Link Generator Required** | Form Builder `required` — HTML-валидация SDK-формы. `required` у подстановочного параметра Source — обязательность заполнения параметра в UI Link Generator; это не отдельный атрибут Visit Field. Детали — [how-to/forms.md](../how-to/forms.md). |
| **Fill form нода vs `{{form}}` плейсхолдер** | `Fill form` (нода во флоу) **выбирает**, какая форма привязана к кампании/шагу. `{{form}}` — контейнер в HTML лэнда. Выбор возможен также настройкой Content; без выбранной формы SDK использует встроенные controls. |
| **Поля захвата лида vs трекинговые поля визита SDK** | Контролы формы (`first_name`/`last_name`/`email`/`phone`) — данные, которые вводит визит. Включённые SDK-фичи дополнительно пишут трекинговые поля (fingerprint, browser_time_zone, scroll, time_on_landings) — это **не** контролы формы. |

## Подводные камни Form — частые ошибки настройки

### Подводные камни подключения и поведения

- **Нет контейнера SDK-формы** — добавьте `{{form}}` или `<form class="aio-form"></form>`. `Fill form` выбирает форму, но не обязателен для всех сценариев: выбор есть также в Content, а без него SDK использует встроенные controls.
- **`features.form` = OFF по дефолту SDK** — форма работает только если фича включена (шаблонный `AIO SDK Macros Collection` ставит `form: true` поверх; выключили или заменили макрос своим — форма пропадёт). Без этого — пустое место.
- **Прогрессивный сбор (blur) ≠ submit.** При включённом `collectDataEvent` поддерживаемые поля могут заполниться до отправки формы; сам сбор не означает submit.
- **Required — два контекста** (Form Builder vs параметр Source в Link Generator) — детали в [how-to/forms.md](../how-to/forms.md), [how-to/custom-fields.md](../how-to/custom-fields.md).

### Подводные камни кода, версий и стилей

- **Хук сабмита — `beforeSubmitPromise`, в единственном числе.** Вариант с `beforeSubmitPromises` (во множественном) SDK **не** подхватит. Полный разбор поведения формы — [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»).
- **`?v=` в ссылке на бандл — не версия SDK.** Параметр только сбивает кэш браузера: на статике лежит один бандл, и агент отдаёт его независимо от query. Разные числа на разных лэндах и в превью формы не означают разных версий SDK — поведение опции сверяй с кодом актуального бандла, а не с этим параметром ([reference/sdk.md](../reference/sdk.md), раздел «Ключи, на которых чаще всего спотыкаются»).
- **Non-SDK Form Handler вешает на инпут телефона SDK-шный `intl-tel-input`** — инпут получает его стили, форма может «съезжаться»; лечить точечной адаптацией стилей или переходом на SDK (стили лэнда не удалять).

## Куда идти за деталями — смежные темы

- **Полное поведение формы** (lifecycle, controls, шаблоны, хуки, `intl-tel-input`, CSS-переменные) → [reference/sdk.md](../reference/sdk.md) (раздел «SDK-форма — полный разбор»); три способа обработки → раздел «Три способа обработки формы — что выбрать»; `UTILS` для хуков → раздел «UTILS — что доступно хукам формы».
- **Процедуры** (создать/дублировать SDK-форму, Form Builder вкладки, Control settings, Non-SDK setup, Required-контексты, валидация телефона) → [how-to/forms.md](../how-to/forms.md).
- **Плейсхолдеры** (`{{form}}`, `{{aio}}`, `{{aio:macros:aio_nonsdk_form_handler}}`; вариант для нескольких форм — в `placeholders.md`) → [reference/placeholders.md](../reference/placeholders.md).

### Куда подключается форма и где UI

- **Подключение к пути** (нода `Fill form`, Content-шаг, transition `Handle-form`) → [models/flow-model.md](flow-model.md).
- **Маппинг контрол↔поле визита, Required-контексты** → [models/visit-field.md](visit-field.md), [how-to/custom-fields.md](../how-to/custom-fields.md).
- **Куда уходит лид после submit** → [models/destination.md](destination.md).
- **Термины** (SDK-форма, Form Builder, Controls, Steps, Layouts, Languages, Form Behavior settings, Pushing modal, Advanced Form Sample, CSS-переменные) → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (в ERP — `Settings → Forms` `/app/settings/forms`; Form Builder: Settings (Info и Controls/Steps/Layouts/Languages) и Preview; поведение — в ERP `Content → Macros → AIO SDK Macros Collection`, в MTK `Settings → Macros`; привязка в ERP — `Tracker → Flows → Edit Flow → Fill form`; рендер — `{{form}}` в коде лэнда).
