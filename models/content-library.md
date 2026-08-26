---
id: content-library
title: Content Library — концепт (модель)
description: Что такое Content Library в AIO — хранилище типизированных объектов (Content Type → Content Item), значения которых динамически подставляются на лэнды/в кампании через {{aio.visit.fields.<field>.<key>}}; связь с полем визита типа Placeholder, Fill Fields и Distribution. Концепт; процедуры — how-to/landings.md.
doc_type: model
builds: [erp]
related: [landings, custom-fields, placeholders, visit-field, flow-model, distributions-model, limits, landing, ui-map, destination, advertiser, campaign-defaults, glossary]
language: ru
updated: 2026-08-11
---

# Content Library — концепт (модель)

> Модель сущности **Content Library**. Процедуры (создать Content Type/Item, использовать на лэнде) — [how-to/landings.md](../how-to/landings.md) (секции про Content Library). Поле визита под неё — [how-to/custom-fields.md](../how-to/custom-fields.md). Плейсхолдер-контракт — [reference/placeholders.md](../reference/placeholders.md).

## Что такое Content Library

**Content Library — это хранилище типизированных объектов с настраиваемой структурой, значения которых динамически подставляются на лэнды и в кампании через плейсхолдеры полей визита.** Вместо того чтобы хардкодить на лэнде название бренда, картинку или текст, вы заводите объект в Content Library, привязываете его к **полю визита типа `Placeholder`**, а на лэнде ставите `{{aio.visit.fields.<field>.<key>}}` — и AIO в runtime подставляет нужное значение, выбранное по правилам флоу/дистрибуции. Это отдельная сущность данных (одна из 6 вкладок раздела Content), со своей моделью структуры, доступа и плейсхолдер-контрактом — её потребляют лэнды, флоу (`Fill Fields`) и дистрибуции, поэтому это не под-тема Landing, а самостоятельный объект на спайне.

## Как работает Content Library: Content Type, Content Item, плейсхолдеры

### 1. Двухуровневая структура: Content Type → Content Item

Внутри Content Library данные лежат двумя уровнями:
- **Content Type** — группа верхнего уровня: именованный Placeholder (напр. `Offer Names`) + **структура** (атрибуты/ключи, задаётся через `Edit Structure`) + **данные** (через `Manage Data`). Создаётся кнопкой `+Content Type`; при создании выбираете **поле визита**, к которому Items применяются, и задаёте Content Name, Tooltip, Placeholder, Description, Key, Group.
- **Content Item** (в интерфейсе также `Placeholder Data`) — одна единица контента под этим типом: image / text string / HTML / country / language value. Например, под типом `Offer Names` Item — это конкретный набор строк оффера; под типом с картинками — изображение и его атрибуты (напр. `FullName` / `image` / `Description`). Набор атрибутов Item задаёте вы сами через `Edit Structure`. Создаётся через `Manage Data → +Placeholder Data`.

Тип хранимого контента широкий: images, text strings, HTML, country и language values — всё, что можно динамически использовать через AIO-поля. Процедуры создания — [how-to/landings.md](../how-to/landings.md).

### 2. Контракт подстановки: Content Item ↔ Key ↔ поле визита ↔ плейсхолдер

Это центральная связка, ради которой существует вся сущность. У Content Type есть **Key** — slug, который формирует часть плейсхолдера на лэнде. Content Type привязан к **полю визита типа `Placeholder`**. На лэнде значение читается так:

```
{{aio.visit.fields.<field>.<key>}}
```

Например, Content Type привязан к полю `offer_name` и имеет key `for_visitor` → на лэнде доступен `{{aio.visit.fields.offer_name.for_visitor}}`, который в runtime подставит значение из выбранного Content Item. Под картинку — `{{aio.visit.fields.content_storage.image}}` заменит `src`. Синтаксис и список ключей — [reference/placeholders.md](../reference/placeholders.md).

**Важно:** если у Content Type несколько ключей (напр. `for_visitor` / `for_advertiser`), плейсхолдер **без key** не сработает — AIO не знает, что подставлять. Key указывать обязательно.

### 3. Канонический use case — Offer Name (for_visitor / for_advertiser)

Самый частый пример: статичное название бренда на лэнде заменяется на `{{aio.visit.fields.offer_name.for_visitor}}`. Поле `offer_name` хранит объект с двумя строками:
- **`for_visitor`** — то, что показываем визиту на лэнде;
- **`for_advertiser`** — то, что уходит в Destination/интеграцию через `{{aio.visit.fields.offer_name.for_advertiser}}`.

«Offer Name For Visitor» и «Offer Name For Advertiser» существуют параллельно как поля внутри одного Content Item. Это позволяет показывать визиту одно имя оффера, а партнёру отдавать другое.

### 4. Привязка к полю визита типа Placeholder

Content Type не висит в воздухе — он привязан к **полю визита**, которое вы создаёте сами в `Settings → Fields` с типом `Placeholder`. Встроенных storage-полей нет — под любой набор заводите своё Placeholder-поле: напр. `offer_storage` под оффер-неймы, `content_storage` под картинки лэнда (это просто имена-примеры кастомных полей, не предзаданные поля системы). На лэнде объект читается через `{{aio.visit.fields.<storage>.<key>}}`. Модель самого поля — [models/visit-field.md](visit-field.md); как его завести — [how-to/custom-fields.md](../how-to/custom-fields.md).

### 5. Кто выбирает Item: Fill Fields и Distribution

Content Library — это хранилище; **выбор конкретного Item** делают два механизма пути визита:
- **Flow-нода `Fill Fields`** (серверная, невидима визиту): заполняет поля визита «по правилам или из Content Library». Атрибут `Payload Type` = `Fill Content` (значение из Content Library) или `Fill Text` (статичный текст). Из какого набора Items брать — определяет атрибут **`Placeholder Group`**. Детали ноды — [models/flow-model.md](flow-model.md).
- **Distribution-нода `Add Fill Field`** подтип **`Library`** (наряду с Text/Image/Landing/Destination): значение для поля визита берётся из Content Library через дерево дистрибуции, конкретный Item выбирается стратегией (First / Weights / Conversions AI). Детали — [models/distributions-model.md](distributions-model.md).

Вес варианта в стратегии `Weights` (Content / Fill Fields) задаётся числом **0..100** ([reference/limits.md](../reference/limits.md)).

**Placeholder Group** (она же Lander Placeholder Group) — мост между Content Item и шагом `Fill Fields`: она указывается и в атрибуте шага, и в атрибутах самого Content Item, и определяет, из какого набора берутся значения.

### 6. Подстановка в runtime

Содержимое подставляется не на этапе сборки, а при отдаче лэнда: значение берётся из Content Item, выбранного по правилам дистрибуции/флоу (с учётом матчинга Countries/Languages у Item-а) — и выбор, и подстановку значения в плейсхолдер делает AIO: на агент контент уходит уже отрендеренным, агент отдаёт его как есть (плейсхолдер-контракт — [reference/placeholders.md](../reference/placeholders.md)). Поэтому динамический контент из Content Library работает только там, где грузится SDK — на White-лэнде он не подставляется by design ([models/landing.md](landing.md)).

Самые ходовые лэнды AIO кэширует агрессивнее (атрибут `cache_priority` проставляется автоматически по трафику) — руками это включать не нужно (когда и по какому расписанию — *Что AIO проставляет и делает сам — фоновые задачи и их расписание*).

### 7. Модель доступа

У каждого Content Item есть **Access Type**:
- **`Everyone`** — доступен всем байерам всегда;
- **`By Share`** — нужно расшарить (ПКМ → `Share`).

Сменить — ПКМ → `Change Ownership`. По умолчанию байеры **не** имеют доступа к вкладке Content Library и не заводят Items сами; им либо шарят объект, либо ставят `Access Type = Everyone`.

### 8. Встроенных Content Type-ов нет — структура целиком кастомная

Предзаданных/«коробочных» Content Type-ов в Content Library нет: и набор атрибутов, и сами типы вы задаёте сами через `Edit Structure`. «Content Type» здесь — это **ваша именованная структура** (кнопка `+Content Type` → `Edit Structure`), а не выбор из готового списка типов.

## Как Content Library связан с остальными сущностями

- **Content (раздел) ⊃ Content Library** — одна из 6 вкладок (рядом: Landings, Creatives, Macros, Uploaded Files, CDN files). → [reference/ui-map.md](../reference/ui-map.md)
- **Content Library → Content Type → Content Item** — структура через `Edit Structure`, данные через `Manage Data`.
- **Content Library → поле визита типа `Placeholder`** (заводите сами через `Settings → Fields`; встроенных storage-полей нет — имена вроде `offer_storage` / `content_storage` это примеры кастомных полей). → [models/visit-field.md](visit-field.md), [how-to/custom-fields.md](../how-to/custom-fields.md)
- **Content Library → Landing** — потребляется через `{{aio.visit.fields.<field>.<key>}}`. → [models/landing.md](landing.md), [reference/placeholders.md](../reference/placeholders.md)
- **Content Library ← Fill Fields-нода** (`Payload Type = Fill Content`, через `Placeholder Group`) **и ← Distribution-нода `Add Fill Field`** подтип `Library`. → [models/flow-model.md](flow-model.md), [models/distributions-model.md](distributions-model.md)
- **Content Library → Destination** — ключ `for_advertiser` оффер-нейма уходит в Destination/интеграцию. → [models/destination.md](destination.md), [models/advertiser.md](advertiser.md)
- **Content Library → Showcase Flow** — Showcase Site/Item как частный случай поверх Content Library. → *уточните у поддержки*

## Чем Content Library отличается от смежных понятий

### Content Library vs Creatives — в чём разница

Creatives — статичное файловое хранилище креативов (баннеры/картинки). Content Library — типизированные объекты, значения которых динамически подставляются через поля визита. Обе — вкладки Content.

### Content Library vs Uploaded Files / CDN files — в чём разница

Uploaded Files — личные файлы юзера (видит только свои). CDN files — файлы лэндов на CDN (CSS/HTML/JS). Content Library — структурированные объекты-данные, не файловое хранилище лэнда.

### Content Library vs Content Splits (LP-сплиты) — в чём разница

Content Splits — A/B-варианты кусков **одного** лэнда (`{{split.<key>}}`, стратегии First/Weights/Conversions AI), задаются в Manage landing. Content Library — централизованное хранилище объектов под поля визита. Для разводки «оффер → контент» рекомендуется Content Library, а не Content Splits. → [how-to/landings.md](../how-to/landings.md)

### Placeholder Group vs Key vs поле визита (slug) — три разных понятия

Три разных «ключа»: **Placeholder Group** — группа Items для выбора в Fill Fields; **Key** — slug внутри Content Type, формирует часть плейсхолдера; **поле визита (slug)** — само поле типа Placeholder, к которому привязан Content Type.

### Content Type (структура) vs «встроенных Content Type-ов нет» — в чём различие

«Content Type» как структура/кнопка существует; предзаданных/встроенных Content Type-ов нет — структура задаётся целиком кастомно. См. «Встроенных Content Type-ов нет — структура целиком кастомная»: кнопка `+Content Type` → `Edit Structure`.

### Content Library vs Showcase Site/Item — в чём разница

Showcase — частный паттерн поверх Content Library (массив карточек, рендер через Vue.js), не отдельная фича. → *уточните у поддержки*

## Подводные камни Content Library — частые ошибки настройки

- **Key обязателен при нескольких ключах.** `{{aio.visit.fields.offer_name}}` без key (когда есть `for_visitor`/`for_advertiser`) не подставится — AIO не знает, что брать.
- **White-лэнд не подставляет `aio.*`** — SDK там не грузится, динамический контент из Content Library не работает by design.
- **Абсолютные CDN-ссылки на картинки из Content Library не переписываются.** При процессинге лэнда AIO переписывает на CDN только **относительные** пути внутри HTML-ZIP; абсолютная ссылка из Content Library остаётся в исходниках лэнда и продолжает вести на исходный хост, а не на ваш CDN-домен. Переписывайте на relative вручную (актуально для Showcase Item).
- **`Placeholder Group`, не «Payload Group».** Корректное имя поля — `Placeholder Group`.
- **Байеры по дефолту не видят вкладку и не заводят Items.** Нужно либо `Share`, либо `Access Type = Everyone`, иначе Item байеру не виден.
- **Чтобы лэнд показал контент под конкретный Destination — предзаполните Destination через `Fill Fields` ДО шага Landing.** На лэнде нельзя «заранее» знать Destination без записи в поле визита.

## Куда углубиться: связанные материалы по Content Library

- **Процедуры (создать Content Type/Item, использовать на лэнде)** → [how-to/landings.md](../how-to/landings.md).
- **Поле визита типа Placeholder, кастомные storage-поля (`offer_storage` / `content_storage` — примеры)** → [how-to/custom-fields.md](../how-to/custom-fields.md), [models/visit-field.md](visit-field.md).
- **Плейсхолдер-контракт `{{aio.visit.fields.<field>.<key>}}`, ключи** → [reference/placeholders.md](../reference/placeholders.md).
- **Fill Fields-нода (Fill Text / Fill Content, Placeholder Group)** → [models/flow-model.md](flow-model.md).
- **`Add Fill Field` подтип `Library`, выбор Item стратегиями** → [models/distributions-model.md](distributions-model.md).
- **Как Content Library накладывается поверх лэнда (динамика)** → [models/landing.md](landing.md).
- **Content Library как способ «убрать возможность ошибиться» у байера** → [heuristics/campaign-defaults.md](../heuristics/campaign-defaults.md).
- **Точные имена (Content Library, Key, Placeholder Group)** → [reference/glossary.md](../reference/glossary.md).
- **UI** → [reference/ui-map.md](../reference/ui-map.md) (раздел Content `/app/landers-creatives` → вкладка Content Library, внутренний URL `/lander-placeholders`; `+Content Type`; ПКМ: Edit Structure / Manage Data / Assign to Folder / Share / Change ownership / Archive / Freeze).
