---
id: content-library
title: Content Library — концепт (модель)
description: Что такое Content Library в AIO — хранилище типизированных объектов (Content Type → Content Item), значения которых динамически подставляются на лэнды/в кампании через {{aio.visit.fields.<field>.<key>}}; связь с полем визита формата Placeholder, Fill Fields и Distribution. В MTK — только с платным модулем Distribution (вкладка Distributions). Концепт; процедуры — how-to/landings.md.
doc_type: model
builds: [erp, mtk]
related: [landings, custom-fields, placeholders, visit-field, flow-model, distributions-model, limits, landing, permissions, ui-map, destination, advertiser, creatives, campaign-defaults, glossary]
language: ru
updated: 2026-10-19
---

# Content Library — концепт (модель)

> Модель сущности **Content Library**. Процедуры (создать Content Type/Item, использовать на лэнде) — [how-to/landings.md](../how-to/landings.md) (секции про Content Library). Поле визита под неё — [how-to/custom-fields.md](../how-to/custom-fields.md). Плейсхолдер-контракт — [reference/placeholders.md](../reference/placeholders.md).

## Что такое Content Library

**Content Library — это хранилище типизированных объектов с настраиваемой структурой, значения которых динамически подставляются на лэнды и в кампании через плейсхолдеры полей визита.** Вместо того чтобы хардкодить на лэнде название бренда, картинку или текст, вы заводите объект в Content Library, привязываете его к **полю визита формата `Placeholder`**, а на лэнде ставите `{{aio.visit.fields.<field>.<key>}}` — и AIO в runtime подставляет нужное значение, выбранное по правилам флоу/дистрибуции. Это отдельная сущность данных со своей моделью структуры, доступа и плейсхолдер-контрактом — её потребляют лэнды, флоу (`Fill Fields`) и дистрибуции.

Content Library — вкладка раздела Content.

## Content Library в MTK — только с модулем `Distribution`

**В MTK-виде Content Library открывает платный модуль `Distribution`: он добавляет в верхнее меню вкладку `Distributions`, и Content Type идут в её подменю отдельными пунктами — рядом с дистрибуциями.** Пункт открывает таблицу Content Item этого Content Type (`Placeholder Data`): новый Item добавляет кнопка `+Placeholder Data` (окно `Create content item`), в контекстном меню строки — правка (`Edit content item`), `Share`, `Change ownership` и `Archive`. Создать новый Content Type или поменять его структуру в MTK-виде нельзя. Видимость меню и доступные действия также зависят от прав пользователя и доступа к Item.

Без модуля вкладка Content Library в MTK-виде скрыта. Для динамической записи полей можно использовать свои JS-макросы на лэнде. Как подключить модуль — *Билды, тарифы, триал и статусы тенанта*; что ещё он открывает — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

## Как работает Content Library: Content Type, Content Item, плейсхолдеры

### 1. Двухуровневая структура: Content Type → Content Item

Внутри Content Library данные лежат двумя уровнями:
- **Content Type** — группа верхнего уровня: именованный Placeholder (напр. `Offer Names`) + **структура** (атрибуты/ключи) + **данные**.
- **Content Item** (в интерфейсе также `Placeholder Data`) — одна единица контента под этим типом: image / text string / HTML / country / language value. Например, под типом `Offer Names` Item — это конкретный набор строк оффера; под типом с картинками — изображение и его атрибуты (напр. `FullName` / `image` / `Description`).

Тип хранимого контента широкий: images, text strings, HTML, country и language values — всё, что можно динамически использовать через AIO-поля.

Content Type создаётся кнопкой `+Content Type`; при создании задаёте имя Content Type и выбираете **поле визита**, к которому Items применяются. Для атрибутов структуры задаются имя и Key; в расширенном режиме — Tooltip, Placeholder, Description и Group. Структуру — атрибуты/ключи, то есть набор атрибутов Item — задаёте вы сами через `Edit Structure`, данные ведутся через `Manage Data`: новый Content Item создаётся через `Manage Data → +Placeholder Data`. Процедуры создания — [how-to/landings.md](../how-to/landings.md).

### 2. Контракт подстановки: Content Item ↔ Key ↔ поле визита ↔ плейсхолдер

Это центральная связка, ради которой существует вся сущность. У каждого атрибута в структуре Content Type есть **Key**, который формирует часть плейсхолдера на лэнде. Content Type привязан к **полю визита формата `Placeholder`**. На лэнде значение читается так:

```
{{aio.visit.fields.<field>.<key>}}
```

Например, Content Type привязан к полю `offer_name`, а его структура содержит key `for_visitor` → на лэнде доступен `{{aio.visit.fields.offer_name.for_visitor}}`, который в runtime подставит значение из выбранного Content Item. Под картинку — `{{aio.visit.fields.content_storage.image}}` заменит `src`. Синтаксис и список ключей — [reference/placeholders.md](../reference/placeholders.md).

**Важно:** для чтения конкретного атрибута Content Item нужен его Key (например, `for_visitor` или `for_advertiser`). Без Key плейсхолдер может вернуть превью-картинку или имя выбранного Item.

### 3. Канонический use case — Offer Name (for_visitor / for_advertiser)

Самый частый пример: статичное название бренда на лэнде заменяется на `{{aio.visit.fields.offer_name.for_visitor}}`. Поле `offer_name` хранит ID выбранного Content Item; в этом примере Item содержит две строки:
- **`for_visitor`** — то, что показываем визиту на лэнде;
- **`for_advertiser`** — то, что уходит в Destination/интеграцию через `{{aio.visit.fields.offer_name.for_advertiser}}`.

«Offer Name For Visitor» и «Offer Name For Advertiser» существуют параллельно как поля внутри одного Content Item. Это позволяет показывать визиту одно имя оффера, а партнёру отдавать другое.

### 4. Привязка к полю визита формата Placeholder

Content Type не висит в воздухе — он привязан к **полю визита**, которое вы создаёте сами в `Settings → Fields` с форматом `Placeholder`. Встроенных storage-полей нет — под любой набор заводите своё Placeholder-поле: напр. `offer_storage` под оффер-неймы, `content_storage` под картинки лэнда (это просто имена-примеры кастомных полей, не предзаданные поля системы). На лэнде объект читается через `{{aio.visit.fields.<storage>.<key>}}`. Модель самого поля — [models/visit-field.md](visit-field.md); как его завести — [how-to/custom-fields.md](../how-to/custom-fields.md).

### 5. Кто выбирает Item: Fill Fields и Distribution

Content Library — это хранилище; **выбор конкретного Item** можно настроить через следующие механизмы пути визита:
- **Flow-нода `Fill Fields` в ERP** (серверная, невидима визиту): заполняет поля визита «по правилам или из Content Library». Атрибут `Payload Type` = `Fill Content` (значение из Content Library) или `Fill Text` (статичный текст). Из какого набора Items брать — определяет атрибут **`Placeholder Group`**. Детали ноды — [models/flow-model.md](flow-model.md).
- **Distribution-нода `Add Fill Field`** подтип **`Library`** (наряду с Text/Image/Landing/Destination): значение для поля визита берётся из Content Library через дерево дистрибуции. Детали — [models/distributions-model.md](distributions-model.md).

В дереве дистрибуции конкретный Item выбирается стратегией (First / Weights / Conversions AI). Вес варианта в стратегии `Weights` (Content / Fill Fields) в интерфейсе обозначен диапазоном **0..100** ([reference/limits.md](../reference/limits.md)).

**Placeholder Group** (она же Lander Placeholder Group) — мост между Content Item и шагом `Fill Fields`: она указывается и в атрибуте шага, и в атрибутах самого Content Item, и определяет, из какого набора берутся значения.

### 6. Подстановка в runtime

Содержимое подставляется не на этапе сборки, а при отдаче лэнда: значение берётся из Content Item, выбранного по правилам дистрибуции/флоу (по настроенным условиям выбора) — и выбор, и подстановку значения в плейсхолдер делает AIO: на агент контент уходит уже отрендеренным, агент отдаёт его как есть (плейсхолдер-контракт — [reference/placeholders.md](../reference/placeholders.md)). Серверная подстановка Content Library не требует SDK и выполняется также на White-лэнде; из JS поля можно читать или менять через SDK ([models/landing.md](landing.md)).

Правка Content Item доходит до новых визитов не мгновенно, а через несколько минут — как и правки кампании, флоу и лэнда (*Что AIO проставляет и делает сам — автоматические обновления и их лаг*, секция «Правки кампании, флоу и лэнда доходят до трафика не мгновенно — через несколько минут»).

### 7. Кто видит Content Type и Content Item — `Access type`, шаринг, `Change ownership`

**У каждого Content Type и каждого Content Item есть `Access type` — кому он доступен без шаринга; новый объект создаётся `Private`** (в форме нового Content Item это поле `Access type`, по умолчанию `Private`). Значения:

- **`Private`** — владельцу, тем, кому объект пошарили (ПКМ → `Share`), вышестоящим по команде владельца, Owner / Admin и позициям со `Scopes` на Content Library (ниже);
- **`Use`** — даёт пользователям тенанта доступ к объекту в списках и селектах (например, Content Type — в выборе на шаге `Fill Fields` флоу), но сам по себе не даёт чтение карточки;
- **`View`** — все ещё и открывают карточку на чтение.

Править чужой объект `Access type` сам по себе не позволяет. Для редактирования нужен уровень `Edit` или `Own`: его дают, в частности, шара `Editor`, владение, доступ через команду, `Editor Scopes` на Content Library (`edit-scopes.content.content-library`), а также роль Owner / Admin тенанта. Видеть все объекты раздела без шаринга позволяет `scopes.content.content-library`.

Сменить `Access type` и владельца — ПКМ → `Change ownership` (для выделенных строк — `Mass change ownership`), пункт показывается только при уровне `Own` на объект. Уровни доступа и роли шары — [how-to/permissions.md](../how-to/permissions.md).

По умолчанию байеры **не** имеют доступа к вкладке Content Library и не заводят Items сами: вкладку открывают права позиции, а конкретный объект — шаринг или `Access type` `Use` / `View`.

### 8. Встроенных Content Type-ов нет — структура целиком кастомная

Предзаданных/«коробочных» Content Type-ов в Content Library нет: и набор атрибутов, и сами типы вы задаёте сами через `Edit Structure`. «Content Type» здесь — это **ваша именованная структура** (кнопка `+Content Type` → `Edit Structure`), а не выбор из готового списка типов.

## Как Content Library связан с остальными сущностями

- **Content (раздел) ⊃ Content Library** — вкладка раздела Content (рядом: Landings, Creatives, Macros, Uploaded Files, CDN files). → [reference/ui-map.md](../reference/ui-map.md)
- **Content Library → Content Type → Content Item** — структура через `Edit Structure`, данные через `Manage Data`.
- **Content Library → поле визита формата `Placeholder`** (заводите сами через `Settings → Fields`; встроенных storage-полей нет — имена вроде `offer_storage` / `content_storage` это примеры кастомных полей). → [models/visit-field.md](visit-field.md), [how-to/custom-fields.md](../how-to/custom-fields.md)
- **Content Library → Showcase Flow** — Showcase Site/Item как частный случай поверх Content Library. → *уточните у поддержки*

- **Content Library → Landing** — потребляется через `{{aio.visit.fields.<field>.<key>}}`. → [models/landing.md](landing.md), [reference/placeholders.md](../reference/placeholders.md)
- **Content Library ← Fill Fields-нода** (`Payload Type = Fill Content`, через `Placeholder Group`) **и ← Distribution-нода `Add Fill Field`** подтип `Library`. → [models/flow-model.md](flow-model.md), [models/distributions-model.md](distributions-model.md)
- **Content Library → Destination** — значение ключа `for_advertiser` оффер-нейма можно передать в Destination/интеграцию через плейсхолдер. → [models/destination.md](destination.md), [models/advertiser.md](advertiser.md)

## Чем Content Library отличается от смежных понятий

### Content Library vs Creatives — в чём разница

Creatives — медиатека рекламных креативов (картинки и видео, кадрирование, видеоредактор — [how-to/creatives.md](../how-to/creatives.md)); на лэнд креатив не подставляется. Content Library — типизированные объекты, значения которых динамически подставляются через поля визита. Обе — вкладки Content.

### Content Library vs Uploaded Files / CDN files — в чём разница

Uploaded Files — личные файлы юзера (видит только свои). CDN files — файлы лэндов на CDN (CSS/HTML/JS). Content Library — структурированные объекты-данные, не файловое хранилище лэнда.

### Content Library vs Content Splits (LP-сплиты) — в чём разница

Content Splits — A/B-варианты кусков **одного** лэнда (`{{split.<key>}}`, стратегии First/Weights/Conversions AI), задаются в Manage landing. Content Library — централизованное хранилище объектов под поля визита. Для разводки «оффер → контент» рекомендуется Content Library, а не Content Splits. → [how-to/landings.md](../how-to/landings.md)

### Placeholder Group vs Key vs поле визита (slug) — три разных понятия

Три разных «ключа»: **Placeholder Group** — группа Items для выбора в Fill Fields; **Key** — ключ атрибута в структуре Content Type, формирует часть плейсхолдера; **поле визита (slug)** — само поле формата Placeholder, к которому привязан Content Type.

### Content Type (структура) vs «встроенных Content Type-ов нет» — в чём различие

«Content Type» как структура/кнопка существует; предзаданных/встроенных Content Type-ов нет — структура задаётся целиком кастомно. См. «Встроенных Content Type-ов нет — структура целиком кастомная»: кнопка `+Content Type` → `Edit Structure`.

### Content Library vs Showcase Site/Item — в чём разница

Showcase — частный паттерн поверх Content Library (массив карточек, рендер через Vue.js), не отдельная фича. → *уточните у поддержки*

## Подводные камни Content Library — частые ошибки настройки

- **Для конкретного атрибута нужен Key.** `{{aio.visit.fields.offer_name.for_visitor}}` читает строку `for_visitor`; форма без Key возвращает превью-картинку или имя Item.
- **Серверная подстановка Content Library не требует SDK** и работает также на White-лэнде. Из JS поля можно читать или менять через SDK.
- **Проверяйте хост картинок из Content Library.** Внешние абсолютные URL не импортируются вместе с картинками HTML-ZIP и продолжают вести на исходный хост. При отдаче лэнда AIO заменяет собственный CDN URL на `/_cdn` уже после подстановки полей. Для файлов из ZIP используйте пути внутри загруженного архива.
- **`Placeholder Group`, не «Payload Group».** Корректное имя поля — `Placeholder Group`.
- **Байеры по дефолту не видят вкладку и не заводят Items.** Вкладку открывают права позиции; сам Item байеру виден, если его пошарили или у Item `Access type` `Use` (виден в списках) либо `View` (открывается на чтение); остальные источники доступа — в разделе «Кто видит Content Type и Content Item». Новый Item создаётся `Private`.

- **Если контент лэнда зависит от выбора Destination, выберите его до шага Landing**, например через `Fill Fields`, и сохраните выбор в поле визита. Это поле позволяет показать соответствующий контент на лэнде.

## Куда углубиться: связанные материалы по Content Library

- **Процедуры (создать Content Type/Item, использовать на лэнде)** → [how-to/landings.md](../how-to/landings.md).
- **Поле визита формата Placeholder, кастомные storage-поля (`offer_storage` / `content_storage` — примеры)** → [how-to/custom-fields.md](../how-to/custom-fields.md), [models/visit-field.md](visit-field.md).
- **Плейсхолдер-контракт `{{aio.visit.fields.<field>.<key>}}`, ключи** → [reference/placeholders.md](../reference/placeholders.md).
- **Fill Fields-нода (Fill Text / Fill Content, Placeholder Group)** → [models/flow-model.md](flow-model.md).
- **`Add Fill Field` подтип `Library`, выбор Item стратегиями** → [models/distributions-model.md](distributions-model.md).
- **Как Content Library накладывается поверх лэнда (динамика)** → [models/landing.md](landing.md).
- **Content Library как способ «убрать возможность ошибиться» у байера** → [heuristics/campaign-defaults.md](../heuristics/campaign-defaults.md).
- **Точные имена (Content Library, Key, Placeholder Group)** → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (раздел Content `/app/landers-creatives` → вкладка Content Library, внутренний URL `/lander-placeholders`; `+Content Type`; ПКМ: Edit Structure / Manage Data / Assign to Folder / Share / Change ownership / Archive).
