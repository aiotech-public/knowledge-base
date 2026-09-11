---
id: sdk
title: AIO SDK — устройство, встроенные макросы и работа с ним
description: Единый справочник по AIO SDK — как грузится на лэнд, AIO SDK Macros Collection и его features, window.aioBus, SDK-форма, плейсхолдеры, сопутствующие макросы. Часть II — точный разбор по коду SDK (эндпоинты, слаги полей визита, протокол aioBus, lifecycle формы, фичи pwa/fingerprint/tiktok).
doc_type: reference
builds: [erp, mtk]
related: [forms, placeholders, landings, user-fields, push-notifications, glossary, mechanics-pwa, campaigns, custom-fields, server, conversion-model, how-to-pwa, notifications-flow]
language: ru
updated: 2026-09-11
---

# AIO SDK — устройство, встроенные макросы и работа с ним

Единая точка по AIO SDK: что такое SDK, как он грузится на лэнд, главный шаблонный макрос `AIO SDK Macros Collection` и все его features, `window.aioBus`, SDK-форма, плейсхолдеры, сопутствующие макросы (Static Language Country, Non-SDK Form Handler), типичные кастомизации и ошибки.

> Глубокие детали — в компаньонах: процедуры SDK-формы → [how-to/forms.md](../how-to/forms.md); полный список плейсхолдеров → [reference/placeholders.md](placeholders.md).

---

## Что такое AIO SDK и зачем он нужен

**AIO SDK** — JS-слой, который AIO автоматически подгружает на каждый лэнд через плейсхолдеры `{{aio}}` (перед `</body>`) и `{{aio:head}}` (в `<head>`). **Без `{{aio}}` SDK не загрузится** — формы, Backfix, трекинг и аналитика работать не будут.

Что SDK делает на лэнде:
- **Рендерит SDK-форму** (в `{{form}}`) и пушит лид в Destination.
- **Backfix** — ловит «Назад» в браузере и возвращает визит вперёд по флоу.
- **Трекинг для аналитики** — `landed` (Qualified Visits), скроллы (Heatmaps), время на лэнде, размеры окна, таймзона, запись сессий (Session Records).
- **FB-трекинг** — `fbPixel` (клиентский) и `fbCapi` (серверный).
- **Конверсии и поля визита** — через `window.aioBus` (`conversion`, `trigger`).
- **Палитра и отпечаток устройства** — `hueChanger` (лёгкое смещение оттенка палитры лэнда) + `fingerprint` (device-отпечаток в поле визита, **включён по умолчанию**).
- **TikTok-трекинг, PWA и web-push** — `ttPixel` / `ttCapi`, `pwa` (установка приложения + push-подписка + Android-intent).

Конфигурируется SDK **одним макросом** — `AIO SDK Macros Collection` (см. ниже). Вся настройка фич идёт там.

> Точный технический разбор устройства SDK (эндпоинты, слаги полей визита, протокол `window.aioBus`, lifecycle формы, фичи `pwa` / `fingerprint` / TikTok) — в **Части II** ниже.

---

## Что такое макрос в AIO и зачем он нужен

Макрос — это кусок кода, который хранится в `Content → Macros` и подгружается на лэнд автоматически (если Global) или по вызову (если не Global). Макросы — это JS, исполняются **только на фронтенде** (не на сервере).

Свойства макроса:
- Имеет статус **Active** (вкл/выкл).
- Имеет флаг **Global**:
  - `true` — подгружается на все лэнды автоматически.
  - `false` — вызывается локально на конкретных лэндах через `{{aio:macros:<slug>}}`.
- Имеет **Preferred Placement**: `Header` (рендерится в `{{aio:head}}`) или `Footer` (рендерится в `{{aio}}`, дефолт).
- Имеет **Slug** — используется в локальном вызове `{{aio:macros:<slug>}}`.

**Зачем существует:** чтобы куски кода (пиксели, SDK-конфиги, кастомные скрипты) не копировались в каждый лэнд вручную, а централизованно подгружались.

**Условная логика макроса:** правил «подгружать макрос только если выполнено условие на бэке» **нет** — макрос либо активен (грузится всегда), либо нет. Условная логика — JS-ифы внутри скрипта.

## Как найти все макросы тенанта: Content → Macros

Путь: `Content → Macros`. В тенанте по дефолту лежит шаблонный набор макросов (состав зависит от билда/шаблона; типичный набор):

| Макрос | Slug | Placement | Global |
|---|---|---|---|
| **AIO SDK Macros Collection** (главный, см. ниже) | `aio_sdk_macros_collection` | Footer | да |
| **AIO Non-SDK Form Handler** (фолбэк для HTML-форм) | `aio_nonsdk_form_handler` | Footer | нет |
| **AIO Static Language Country for Form** | `aio_static_language__country_for_form` | Footer | нет |
| **AIO Push Subscription** (web-push подписка) | `aio_push_subscription` | Footer | нет |
| **AIO GTAG** (Google Tag) | `aio_gtag` | **Head** | нет |
| **AIO Form Loader** | `aio_form_loader` | Footer | нет |
| **AIO Save Landing Params to Visit** | `aio_save_landing_params_to_visit` | Footer | да |

> `AIO GTAG` — единственный с `Preferred Placement: Head` (грузится в `<head>`); остальные — `Footer`.

**Редактор макроса:** правый клик по макросу → `Edit macros` (рядом `Macros logs` — история, и `Archive`). Поля редактора: Name, Description, Tags, Prefer placement (Footer/Head), Active?, Global?, и **Html** — само содержимое (код в Monaco-редакторе). Для `AIO SDK Macros Collection` в Html лежит загрузка бандла `/aio-static/sdk/main.js` + `window.aioBus.push({type:"config", …})`.

## Как устроен и настроен AIO SDK Macros Collection — главный шаблонный макрос

**Что это:** макрос с дефолтным содержимым — JS-конфиг SDK. Технически — обычный макрос в `Content → Macros`, но с особой важностью: на нём построена вся работа SDK-фич.

**Как открыть:** `Content → Macros → AIO SDK Macros Collection → Edit Macro`.

**Структура содержимого:**
```javascript
window.aioBus = window.aioBus || [];
window.aioBus.push({
  type: "config",
  config: {
    features: {
      backFix: true,
      fbCapi: true,
      fbPixel: true,
      form: true,
      browserTimezone: true,
      landed: true,
      scrolling: true,
      sessionRecords: true,
      timeOnLandings: true,
      windowDimensions: true,
      hueChanger: true,
    },
    backFix: { /* настройки backFix, см. ниже */ },
    form: { /* настройки формы, см. ниже */ },
    // другие блоки настроек, в зависимости от features
  }
});
```

### Как включить или отключить фичу SDK — блок `features`

Каждая фича включается/выключается одной строчкой. Дефолтный набор (по шаблону тенанта 2026):

| Feature | Что делает | Дефолт |
|---|---|---|
| `backFix` | Принудительный редирект при клике «Назад» в браузере | `true` |
| `fbCapi` | Facebook Conversion API (server-to-server конверсии) | `true` |
| `fbPixel` | Facebook Pixel (клиентский трекинг) | `true` |
| `form` | AIO SDK-форма (рендерится в `{{form}}`) | `true` |
| `browserTimezone` | Записывает таймзону визита (IANA, напр. `Europe/Berlin`) в поле `browser_time_zone` | `true` |
| `landed` | Boolean — успел ли визит загрузить лэнд (метрика `Qualified Visits`) | `true` |
| `scrolling` | Tracking скроллов визита по лэнду | `true` |
| `sessionRecords` | Видеозаписи сессий (Session Records) | `true` |
| `timeOnLandings` | Tracking времени, проведённого на лэнде | `true` |
| `windowDimensions` | Записывает размеры окна (ширина/высота) | `true` |
| `hueChanger` | Чуть-чуть меняет color palette лэнда (лёгкое смещение оттенка, незаметное глазу) | `true` |
| `fingerprint` | Device-fingerprint (FingerprintJS) → поле визита `fingerprint` | `true` |
| `ttPixel` / `ttCapi` | TikTok Pixel (клиентский) / TikTok CAPI (серверный, кука `_ttp`) | по тенанту |
| `pwa` | PWA-install + web-push subscribe + Android-intent | off (по запросу) |

**Чтобы отключить фичу** — поставь `false` в `features` блоке. Тонкая настройка — в соответствующем блоке настроек ниже.

### Как настроить backFix — блок `backFix` подробно

Один из самых частых блоков для кастомизации.

**Что делает:** если визит на лэнде нажмёт «Назад» в браузере — принудительно перебросит его на следующий шаг флоу (с прилэнда — на оффер, и т.д.). В трекере у такого визита поле `Backfix = Yes` и в URL появляется `&backfix=true`. `backfix=true` — служебная пометка в URL: по ней **сервер** проставляет визиту поле **`is_backfix`** (в UI — `Backfix`, плейсхолдер `{{aio.visit.is_backfix}}`), по нему и считают, кто прошёл через Backfix. Сам SDK это поле не пишет — он только добавляет параметр в ссылку перехода.

**Структура блока `backFix` в SDK Macros Collection:**
```javascript
backFix: {
  enabledBackFix: () => [
    "uuid-of-preland-type",
    "uuid-of-offer-type"
  ].includes(aio.landing.lander_type_uuid),

  enableStrangeUrlParameters: true,

  pathName: "/some-path",
  localStorageKey: "aio_bf",

  link: "{{link}}&backfix=true"
}
```

**Поля:**

| Поле | Что |
|---|---|
| `enabledBackFix` | функция-условие. Обычно проверяет UUID типа лэнда — backFix работает только если визит на нужном типе. По дефолту — только на прилэндах. |
| `enableStrangeUrlParameters` | если `true` — переписывает реальные CUUID/SUUID в URL на фейковые UTM. Защита от шаринга ссылки. По дефолту `true`. |
| `pathName`, `localStorageKey` | служебные. Менять обычно не нужно. |
| `link` | куда уходит визит при срабатывании. По дефолту — на `{{link}}` с пометкой `&backfix=true` для отслеживания. |

**Где смотреть UUID типов лэндов:** `Content → Landings` → правый клик на лэнде → данные / либо через API.

**Типичная кастомизация:** добавить тип лэнда в `enabledBackFix` (например, начать делать backFix и на офферах, а не только на прилэндах).

`enableStrangeUrlParameters` ломает шаринг ссылок. Симптом «открыл лэнд из почты, и там ничего не работает» → проверь, не активна ли эта опция.

Это настройка на уровне `AIO SDK Macros Collection` (JS-уровень), а не готовый per-tenant / per-campaign / per-landing toggle в UI AIO. Для условной логики по типам лэндов используй функцию `enabledBackFix` (см. выше). Отдельного «per-campaign toggle Backfix» как фичи AIO нет — при необходимости per-campaign управление собирается через поле кампании + JS в макросе (см. «Как включить или выключить Backfix» ниже).

### Backfix не определяет визит / перестал срабатывать — две причины

Если Backfix не определяет визит («после правок SDK перестал срабатывать», «на iframe-вайте не отдаёт визит», «работает на основном домене — не работает в iframe», «нажал „Назад“ на прилэнде — не переброшен на оффер») — причин обычно две.

1. **Функция `enabledBackFix` не пропускает текущий тип лэнда** — самая частая. По умолчанию Backfix работает только на прилэндах; для другого типа добавь его UUID в функцию `enabledBackFix` (см. блок `backFix` выше).
2. **Лэнд/вайт открыт в iframe со стороннего домена** — разобрано ниже.

### На iframe-вайте Backfix не отдаёт визит — third-party cookies недоступны

Если лэнд или вайт открывается в iframe со **стороннего** домена, Backfix не может прочитать визит: когда родительский домен и домен iframe разные, куки сессии внутри iframe недоступны (CORS, third-party cookies). Отсюда «Backfix работает на основном домене, но не в iframe».

Дополнительно жмут сами браузеры: Safari и Firefox с защитой от трекинга блокируют third-party cookies по умолчанию → у этой аудитории Backfix в iframe не работает в принципе. Это ограничение браузеров, а не баг AIO — правкой конфига не чинится, нужно менять схему встраивания.

### Чем заменить iframe, чтобы Backfix работал — редиректы или Dynamic White

Две рабочие альтернативы iframe:
- **Редиректы вместо iframe** — юзер физически переходит на домен лэнда обычной навигацией, куки сессии доступны, Backfix читает визит штатно.
- **Вайт внутри AIO (`Dynamic White`)** — вайт хостится на инфраструктуре AIO, домен страницы совпадает с доменом кампании → ограничение third-party cookies не срабатывает, Backfix работает. Типы лэндов и White Page — [how-to/landings.md](../how-to/landings.md).

Если Backfix не работает на **основном** домене (не в iframe), а блок `backFix` не трогали — это уже предмет для разбора визита, а не настройка.

### Как настроить fbPixel — блок `fbPixel` подробно

**Структура:**
```javascript
fbPixel: {
  pixel: "{{aio.visit.fields.fb_pixel}}",
  events: {
    pageView: "PageView",
    formSubmitEvent: "{{aio.visit.fields.fb_lead_action}}"
  }
}
```

**Поля:**

| Поле | Что |
|---|---|
| `pixel` | откуда брать pixel-ID. По дефолту — из поля визита `fb_pixel` (значит, нужно заполнять это поле через Fill Fields или Distribution). |
| `events.formSubmitEvent` | какое FB-событие шлём при **успешном** сабмите формы (lead). По дефолту — из поля визита `fb_lead_action`. |

По коду `fbPixel` **сам по себе PageView не отправляет** — PageView шлёт `fbCapi`. `fbPixel` грузит пиксель и шлёт только lead-событие на сабмит. Подробно — §II.10.

### Как настроить fbCapi

Facebook Conversion API. Серверная отправка событий в FB. По дефолту `true`, тонкая настройка идёт через поля кампании / интеграции.

### Как настроить form — параметры SDK-формы

`features.form: true` — SDK подгружает форму в `{{form}}`. Параметры формы — через `config.form`:

```javascript
form: {
  url: "/",                       // сабмит идёт сюда (GET с query)
  selector: ".aio-form",          // какие формы на лэнде достраивает SDK
  defaultCss: true,
  successTimeoutSeconds: 5,
  language: "navigator.language", // первые 2 буквы
  defaultCountryCode: "GB",
  stepsShowing: true,
  collectDataEvent: "blur",
  rejectedMessage: "We cant register you at this time.",
  beforeSubmitPromise: [],        // ед. число (см. §II.20)
  submitPromises: [],
  intlParameters: { nationalMode: true, autoPlaceholder: "aggressive" },
  templates: { /* HTML-шаблоны для каждого типа поля */ }
}
```

**Ключевые поля:**

| Поле | Что |
|---|---|
| `successTimeoutSeconds` | таймаут между успешным сабмитом и редиректом на autologin (дефолт 5 секунд). |
| `language` | язык формы. Дефолт — из `navigator.language` визита. |
| `defaultCountryCode` | страна для intl-tel-input (поле телефона). Дефолт `GB`. |
| `rejectedMessage` | что показать визиту, если дестинейшн отклонил лид. |
| `beforeSubmitPromise` | массив JS-хуков, выполняющихся **после валидации, но до пуша** (ключ в **ед. числе**, см. §II.20). Можно вставить кастомную логику (показать модалку подтверждения, доп. проверка). |
| `submitPromises` | массив JS-хуков **вокруг самого пуша**. Используется для управления pushing modal'ом и success modal'ом. |
| `templates` | HTML-шаблоны под каждый тип контрола формы (text, email, phone, и т.д.). Кастомизация через CSS-переменные `--aio-sdk-form-*`. |

### Как настроить атрибуты инпутов SDK-формы в Settings → Forms

Атрибуты каждого инпута SDK-формы (`Required`, min/max) настраиваются **отдельно** в `Settings → Forms → форма → инпут`.

### Валидация номера телефона и дополнительные ресурсы

Номер телефона в SDK-форме проходит валидацию **сторонней библиотекой** intl-tel-input.com.

**Расширенная кастомизация формы** — см. [how-to/forms.md](../how-to/forms.md) и полный lifecycle сабмита в **§II.12**.

### Прочие features — boolean-флаги без сложных под-настроек

Большинство — boolean-флаги:

- **landed** — записывает в визит "лэнд загрузился". Используется в `Qualified Visits` метрике.
- **scrolling** — для Heatmaps и видеосессий.
- **sessionRecords** — Session Records (видеозаписи).
- **timeOnLandings** — время на лэнде.
- **browserTimezone** — таймзона визита.
- **windowDimensions** — размеры окна.
- **hueChanger** — лёгкая модификация color palette лэнда (незаметное глазу смещение оттенка).

Если что-то из этого отключить — соответствующая метрика / фича перестаёт работать (например, выключил `scrolling` → нет Heatmaps).

## Свойства макроса в UI редактирования

В UI редактирования любого макроса:

| Свойство | Что |
|---|---|
| **Active** | вкл/выкл. Если false — макрос не подгружается. |
| **Global** | `true` — на все лэнды автоматически; `false` — только по вызову `{{aio:macros:<slug>}}`. |
| **Preferred Placement** | `Header` (рендер в `{{aio:head}}`) / `Footer` (рендер в `{{aio}}`, дефолт). |
| **Slug** | для локального вызова. |
| **Name** | для UI. |
| **Description** | произвольное описание. |

**Преимущество `Preferred Placement: head`** — для скриптов, которым нужно загрузиться до DOM (некоторые пиксели, A/B-тестинг до отрисовки).

## Когда нужен AIO Non-SDK Form Handler — макрос-фолбэк для HTML-форм

Когда нужно оставить родную HTML-форму вместо `{{form}}`. Полная механика, 3 шага подключения и ограничения — [how-to/forms.md](../how-to/forms.md). Грузит intl-tel-input со своими CSS → форма может «съезжаться». Slug макроса (`aio_nonsdk_form_handler`) может отличаться по тенантам — проверять в `Content → Macros`.

Ключевые ограничения: цепляется только к тегу `<form>` (если форма свёрстана как `<div>` — сабмит не перехватывается); если одновременно есть `{{form}}` и Non-SDK Handler — возможны конфликты; Pushing modal / autologin / хуки не работают (активно не развивается).

## Что делает Static Language Country — макрос фиксации языка и страны формы

Шаблонный макрос (часто включён по дефолту).

**Что делает:** фиксирует язык SDK-формы и страну телефона **по странам/языку лэнда**, а не по визиту.

**Зачем:** без него форма берёт `language` из `navigator.language` визита (может быть не тот язык, на котором написан лэнд), а страну телефона — из гео визита (может быть VPN). С макросом — оба значения берутся из метаданных лэнда (Country/Language заданы при загрузке лэнда).

**Рекомендация:** включать как глобальный, если таргетинг по стране фиксированный.

## Как работает window.aioBus — очередь команд для SDK

Глобальный массив-очередь команд для SDK. Через `window.aioBus.push({type, ...})` SDK получает команды (полный протокол всех типов — **§II.3**):

| Type | Что делает |
|---|---|
| `config` | конфигурация фич, формы, backFix (используется в AIO SDK Macros Collection) |
| `trigger` | заполнить поле визита (`{type:"trigger", key:"...", value:"..."}`) |
| `conversion` | отправить конверсию (`{type:"conversion", type_uuid:"...", query:{revenue: 10}}`) |
| `form-controls` | программно заполнить контролы формы |
| `vue-data-object` | объявить data-объект для Vue.js на лэнде |

`Trigger Field` может обновить **любое** поле визита. Не «бэкдор».

**Где используется:**
- Глобально — в AIO SDK Macros Collection (`config`).
- Локально — на лэнде после `{{aio}}` (`trigger`, `conversion`, `form-controls`, `vue-data-object`).

**Пример локального использования:**
```html
{{aio}}
<script>
  window.aioBus.push({
    type: "trigger",
    key: "promo_code",
    value: "WINTER2026"
  });
</script>
```

Так можно вручную заполнять поля визита прямо с лэнда (например, если на лэнде есть форма выбора оффера — её результат шлём в визит-поле).

Локальная переконфигурация формы (`<script>` с `window.aioBus.push({type:"config", config:{form:{…}}})` в HTML конкретного лэнда) **должна стоять после тега `{{aio}}`** — иначе она не перебьёт глобальные настройки формы из `AIO SDK Macros Collection`. Это не «желательно», а условие корректного override: до `{{aio}}` SDK ещё не инициализирован, и локальный конфиг применяется раньше глобального (или теряется). Локальные `<style>`-блоки аналогично перебивают глобальные стили на своём лэнде.

### Значение, заданное в карточке лендинга, пишется в визит без `trigger`

Постоянное значение поля визита — метка лэнда, номер потока, имя оффера — записывать макросом `trigger` не нужно: его вписывают в секцию `User fields` карточки самого лендинга или его `Lander Type`, и в визит оно попадает серверной механикой, когда флоу отдаёт этот лэнд визиту. Скрипт на странице для этого не пишется вообще. Как заполняется секция и как `Lander Type` раздаёт поля своим лэндам — [how-to/user-fields.md](../how-to/user-fields.md).

`window.aioBus.push({type:"trigger"})` остаётся рабочим и нужен там, где значение известно **только в браузере**: выбор посетителя на странице, результат JS-расчёта, параметр из окружения. То есть выбор такой: значение одинаково для всех визитов этого лэнда — карточка лендинга; значение своё у каждого визита — `trigger`.

## SDK-плейсхолдеры — карта

Полная таблица плейсхолдеров (`{{aio}}`, `{{aio:head}}`, `{{form}}`, `{{link}}`, `{{cdn}}`, `{{cdn-lp-folder}}`, `{{aio:macros:<slug>}}`, `{{aio.visit.*}}`, `{{vue.aio.visit.*}}`) с объяснением кто/когда обрабатывает — [reference/placeholders.md](placeholders.md). Не трогать — AIO вставляет сам (кроме `{{aio:macros:<slug>}}` — локальный вызов).

## Что чаще всего меняют в SDK Macros Collection — типичные кастомизации

| Что | Что меняем | Почему |
|---|---|---|
| Расширить Backfix на новый тип лэнда | Добавить UUID типа в `enabledBackFix` | Нужен backFix не только на прилэнде, но и на оффере. |
| Отключить шаринг защиту | `enableStrangeUrlParameters: false` | Ссылка на лэнд, расшаренная коллеге для тестирования, у него не открывается. |
| Кастомный pageView event | Поменять `fbCapi.events.pageView` | Используются кастомные event-name для FB. |
| Добавить `beforeSubmitPromise` | Вставить функцию в массив | Нужно показать модалку подтверждения перед пушем. |
| Подключить web-push / PWA | Включить `features.pwa` (PWA-install + push-подписка, см. §II.15 и [how-to/push-notifications.md](../how-to/push-notifications.md)) | Включить установку приложения и browser push-уведомления. |
| Подключить кастомный пиксель партнёрки | Создать **отдельный** макрос (не править SDK Collection) | Лучше держать SDK Collection чистым. |

## Типичные ошибки в SDK Macros Collection

- **Поставил `features.backFix: false`, но блок `backFix: {...}` оставил** — синтаксис JS не падает, но фича не работает. Это норма, можно так делать (фича выключена, конфиг сохранён на случай возврата).
- **JS-синтаксис сломан** (пропустил запятую, кавычку) — макрос не подгружается, все SDK-фичи отваливаются разом. Проверь в DevTools → Console: будет syntax error.
- **Поменял UUID типа лэнда в `enabledBackFix` на неверный** — Backfix молча не работает на нужных лэндах. UUID проверяй в `Content → Landings`.
- **Дублировал `window.aioBus.push({type: "config", ...})` в кастомном макросе** — может перезаписать настройки из SDK Collection. Если нужна локальная переконфигурация — пиши `{config: {features: {...}}}` только с теми ключами, которые меняешь.

## Смежные темы

- [reference/placeholders.md](placeholders.md) — `{{aio:head}}`, `{{aio}}`, `{{aio:macros:<slug>}}`
- [how-to/landings.md](../how-to/landings.md) — загрузка лэнда (где макросы вставляются автоматически)
- [reference/glossary.md](glossary.md) — определения терминов (Backfix, fbCapi, fbPixel, и т.д.)
- [how-to/forms.md](../how-to/forms.md) — детально кастомизация SDK-формы
- [how-to/push-notifications.md](../how-to/push-notifications.md) — push-уведомления (фича `pwa` / web-push, см. §II.15)

---

## Процедуры

### Как открыть макрос на редактирование

**Путь:** `Content → Macros → <Macro> → Edit Macro`.

Открывается редактор содержимого макроса (JS / конфиг). Слева — атрибуты макроса (Name, Slug, Active, Global, Preferred Placement).

### Как вызвать локальный макрос на лэнде

Когда: макрос с `Global: false` нужно вызвать только на одном конкретном лэнде.

**Синтаксис в HTML лэнда:**

```html
{{aio:macros:<slug>}}
```

Например: `{{aio:macros:aio_nonsdk_form_handler}}`, `{{aio:macros:custom_pixel}}`.

См. [reference/placeholders.md](placeholders.md) → `{{aio:macros:<slug>}}`.

### Как включить enableStrangeUrlParameters

**Путь:** `Content → Macros → AIO SDK Macros Collection → backFix → enableStrangeUrlParameters: true`.

Когда `true` — реальные CUUID / SUUID в URL переписываются на фейковые UTM. Защита от шаринга ссылки.

Минусы:
- Ломает шаринг ссылки (если визит откроет копию URL у себя — данных не будет).
- Отдельным схемам флоу нужно именно `false` — разбор.

См. секцию `backFix` выше.

### Как включить или выключить Backfix — и почему «per-campaign toggle Backfix» это не фича AIO

**Место:** `Content → SDK Macros Collection → Edit AIO SDK Macros Collection → блок backFix` (toggle через `features.backFix: true/false`). Это уровень SDK-конфига, а не тоггл в самой кампании.

Готового «per-campaign toggle Backfix» на стороне AIO нет. Если Backfix нужно включать/выключать по каждой кампании отдельно — это собирается вручную: заводится поле на уровне кампании со своей настройкой (доступные значения в селекте), а в `AIO SDK Macros Collection` пишется JS, который читает значение этого поля и по нему решает, включать Backfix или нет; на само поле ставится placeholder. Что выставлено в кампании — то приезжает в `AIO SDK Macros Collection` и так настраивает Backfix. То есть per-campaign управление возможно, но это ручная сборка через поле + JS-логику, а не отдельный переключатель в UI AIO.

### Как выключить FB Pixel в SDK

Когда: нужно отключить клиентский Pixel-трекинг (например, при инциденте с FB Pixel или при дублировании Lead-событий).

**Путь:** `Content → Macros → AIO SDK Macros Collection → fbPixel → false`.

В JS-конфиге:

```javascript
features: {
  fbPixel: false,
  ...
}
```

После сохранения — Pixel перестаёт грузиться на лэнды.

Если после отключения `fbPixel` события всё равно двоятся, источник дубля не в SDK: их шлёт либо сторонний пиксель на лэнде, либо сам рекламодатель.

### Как настроить SDK-форму только под first-name + phone глобально

Когда: нужна упрощённая форма (только имя + телефон), не дефолтная (имя / фамилия / email / phone).

**Путь:** `Content → Macros → +Macro` (новый кастомный) → редактировать `config.form.controls`.

**Что менять:**

```javascript
config: {
  form: {
    controls: [
      { key: "first-name", ... },  // можно переименовать на "full-name", ключ оставить
      { key: "phone", ... }
      // остальные удалить
    ]
  }
}
```

Ключи (`first_name`, `phone`) **должны совпадать** с AIO-слагами — даже если визуально переименовать на «full-name», ключ оставить как `first_name`.

См. [how-to/forms.md](../how-to/forms.md) → SDK Form настройки.

### Как превратить лэнд в PWA через макрос

Когда: обычный лэнд нужно превратить в PWA (Service Worker + manifest + Install button) без редизайна.

**Путь:** `Content → Macros → +Macro → PWA-макрос` → задать ID кнопки (`Install button`) и div-элементы → добавить на лэнд.

**Шаги:**

1. Создать новый макрос типа PWA-macro.
2. Указать:
   - **Install button ID** — селектор кнопки на лэнде, к которой будет привязан PWA prompt.
   - **Manifest fields** (имя, иконки, цвета).
   - **Div-элементы** — обёртки для PWA UI.
3. Сохранить.
4. На лэнде — `{{aio:macros:<slug>}}` после `{{aio}}`.

См. [mechanics/pwa.md](../mechanics/pwa.md) → PWA-макрос.

### Как настроить глобальный GTAG-макрос

Когда: один GTAG (Google Tag) на кампанию / тенант — хочется не дублировать в каждом лэнде.

**Путь:** `Content → Macros → AIO GTAG` — **отдельный** глобальный макрос, задаётся из шаблона тенанта (в MTK редактор макросов — `Settings → Macros`). Это НЕ блок внутри `AIO SDK Macros Collection` — у gtag свой макрос (`aio_gtag`, см. таблицу макросов выше).

В конфиг макроса добавить ID GTAG и события.

Включать глобально безопасно: макрос срабатывает только для трафика с Google, FB-трафик не затрагивает. Шлёт pageview при попадании визита на страницу и событие конверсии при сабмите формы. `gtag` — отправляется при submit формы, **до** проверки лида (раньше, чем известно, валиден лид или нет).

См. [how-to/campaigns.md](../how-to/campaigns.md) → Ссылка под Google.

### Как вынести GTAG со ссылки на уровень кампании — поля type: campaign

Если GTAG нужно вынести **со ссылки на уровень кампании** (чтобы не светить в URL) — заводи поля с `type: campaign` (см. [how-to/custom-fields.md](../how-to/custom-fields.md) → Создать поле визита), архивируй старые source-поля, поменяй имена полей в конфиге макроса `AIO GTAG`.

Кастомные конверсионные поля (`type: campaign`): если значение **не должно** фигурировать в URL — два пути: (1) поле визита с `type: campaign`, (2) шаг `Fill Fields`. Кейс переноса кастомных параметров (Google Pixel, GTag Conversion ID/Label, токенов) со ссылки на кампанию: создать поля `type: campaign` (или сменить тип через БД — в UI нельзя), архивировать старые source-поля, поменять имена полей в конфиге макроса `AIO GTAG`.

---

# Часть II — SDK изнутри (справочник по коду)

> Этот раздел — устройство SDK изнутри: имена полей визита (slugs), дефолты, эндпоинты агента, протокол `window.aioBus`, lifecycle формы. Для отладки — «что реально делает фича под капотом»; для разработчиков — контракт интеграции.

## II.1 Как SDK грузится и инициализируется

- **Бандл:** собранный SDK отдаётся статикой — `/aio-static/sdk/main.js` (+ стили `/aio-static/sdk/main.css`). Плейсхолдер `{{aio}}` вставляет на лэнд `<script>` с этим бандлом, `{{aio:head}}` — то, что должно грузиться в `<head>`.
- **Порядок инициализации** (по `index.js`):
  1. Ждёт `DOMContentLoaded`.
  2. Собирает итоговый `config`: берёт встроенный `DEFAULT_CONFIG` и **deep-merge** дефолтов каждого макроса.
  3. Читает `window.aioBus`: элементы `type: "config"` мёржатся поверх (так тенантный `AIO SDK Macros Collection` включает/донастраивает фичи).
  4. Поднимает Vue-приложение, **если** на странице есть элемент `config.vue.container` (дефолт `#vue`). Иначе просто опрашивает `aioBus` в интервале.
  5. Запускает все включённые макросы параллельно (`Promise.all`). Ошибка одного макроса **изолирована** — логируется как `<Имя> macros err: ...` в консоль, остальные продолжают работать.
  6. Делает первый сброс (flush) накопленных полей визита.
- **Опрос шины:** `window.aioBus` перечитывается каждые `config.vue.busReadInterval` мс (дефолт **50 мс**). Поэтому команды, добавленные в шину позже (с лэнда, по таймеру), тоже подхватываются.
- **Два глобала:**
  - `window.aio` — объект с серверными данными визита (приезжает на страницу вместе с лэндом, который отдаёт агент).
  - `window.aioBus` — очередь команд от лэнда/макросов к SDK.

## II.2 Объект `window.aio` — данные визита

Лэнд приходит с проставленной структурой вида:

```javascript
window.aio = {
  visit:    { uuid, country_code, fields: {…}, is_push_subscribed, … },
  session:  { uuid },                    // per-visit PWA session-токен (отдельная длинная строка) лежит в visit.session
  landing:  { uuid, name, folder: [{ uuid, name, color }], countries: […], languages: […],
              lander_template_uuid, lander_type_uuid },
  campaign: { uuid, name, country_code, language_code, owner_uuid },
  source:   { uuid, name },
  handle:   { key, value },              // «ручка», по которой сабмит распознаётся как форма
  aio_macros, is_backfix,
  time_on_landings_total, time_on_landings_uuids: {…}, time_on_landing_types_uuids: {…},
  landed_on_landings_uuids: {…}, landed_on_landings_types_uuids: {…},
  scroll_on_landings_uuids: {…}, scroll_on_landings_type_uuids: {…},
  relations                              // служебная строка, SDK отдаёт её обратно в запросе записи сессии
}
```

- `landing` несёт семь ключей лэнда: `uuid`, `name`, `folder` (папки лэнда — `uuid`/`name`/`color`), `countries`, `languages`, `lander_template_uuid`, `lander_type_uuid`. Сам SDK читает из них только `uuid` и `lander_type_uuid` (макросы `landed` / `scrolling` / `timeOnLandings` / `backFix` / `sessionRecords`), остальное — для своего JS на лэнде. Рядом лежат `campaign` (`uuid`, `name`, `country_code`, `language_code`, `owner_uuid`) и `source` (`uuid`, `name`); секрет кампании для Reverse Integration в `window.aio` не попадает.
- `visit.fields` — текущие значения полей визита (SDK сверяется с ними, чтобы не слать дубли).
- `visit.country_code` — гео визита; SDK-форма берёт его как страну телефона по умолчанию.
- `handle.key` / `handle.value` — пара, которую форма подкладывает в сабмит, чтобы запрос распознался как сабмит формы и был обработан как пуш в Destination. Решение по визиту (переход флоу, пуш) вычисляет AIO — агент, принявший запрос, работает по его командам; как устроен агент — [models/server.md](../models/server.md).
- **Имена JS-полей в `window.aio.visit` ≠ написанию плейсхолдеров.** Гео/UA приезжают как `location_city`, `location_postal_code`, `useragent` (одно слово); плюс `location_timezone` (geoIP-таймзона, **≠** `browser_time_zone` браузера) и `destination_push_try_counter`. Важно для MTK, где лэнд читает `aio.visit.<field>` из своего JS.

### `window.aio.landing.name` с `&` приходит как `&amp;` — имя лэнда HTML-экранируется

Имя лэнда и имена его папок кладутся в `window.aio` уже HTML-экранированными: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`, `'` → `&#039;`, табуляция → пробел. Внутри `<script>` браузер эти сущности обратно не разворачивает, поэтому лэнд с именем `Black & White` в `aio.landing.name` увидит `Black &amp; White`. Свой JS на лэнде, который выводит это имя или сравнивает его со строкой, должен ждать экранированный вид (или раскодировать сущности сам). На работу SDK это не влияет — он читает из `landing` только `uuid` и `lander_type_uuid`.

## II.3 `window.aioBus` — полный протокол

Очередь команд. Каждый элемент — объект с `type`; SDK обрабатывает его один раз (ставит внутренний флаг `catched`). Из типов ниже 10 обрабатывает `index.js`/`processAioBus`; **`form-submit-fn` — исключение**: его читает не `index.js`, а сам макрос формы (на него подписываются `fbPixel`/`ttPixel`, чтобы слать lead-событие). Типы:

| `type` | Поля | Что делает |
|---|---|---|
| `config` | `config: {...}` | Мёрж в итоговый конфиг SDK (features + блоки фич). Основной способ настройки. |
| `trigger` | `key`, `value` | Записать значение в поле визита (через exchange, см. II.4). |
| `conversion` | `type_uuid`, `query` | Отправить конверсию по UUID типа (см. II.4). |
| `dom-ready` | `fn` | Выполнить функцию (точка входа для кастомного кода после готовности SDK). |
| `form-controls` | `controls: [...]` | Программно задать набор контролов SDK-формы (+ авто-добавление hidden-handle и submit, если не задан). |
| `form-layout` | `layout: [...]` | Раскладка полей формы по рядам/колонкам. |
| `form-steps` | `steps: [[...],[...]]` | Разбить форму на шаги (массив массивов ключей полей). |
| `form-translations` | `translations: {...}` | Заменить словарь переводов формы **целиком** (присваивание, не слияние — см. «i18n и переводы формы»). |
| `form-submit-fn` | `fn` | Зарегистрировать колбэк, который выполнится на **успешном** сабмите (так FB/TT Pixel шлют lead-событие). Потребляется макросом формы, **не** `index.js`. |
| `intl-instance` | `resolve` | Получить инстанс intl-tel-input из SDK (для собственной обработки телефона). |
| `vue-data-object` | `data: {...}` | Доложить данные в реактивный объект Vue на лэнде. |

## II.4 Exchange: поля визита и конверсии

Ядро обмена с агентом (`aio-exchange.js`). Оба обмена SDK шлёт методом **`POST`** с телом `application/x-www-form-urlencoded` — данных в URL нет, в DevTools этот запрос ищут как `POST` без параметров в ссылке.

- **Поля визита.** `trigger(key, value)` копит значения в буфер. Буфер сбрасывается на сервер:
  - каждые `config.aioExchange.flushInterval` мс (дефолт **10 000 мс = 10 c**);
  - и дополнительно на событие `pagehide` (уход со страницы);
  - Перед отправкой буфер **дедупится** против `aio.visit.fields.*` — шлются только реально изменившиеся значения.
  - Эндпоинт: `POST /api/v1/trigger/field/<visit_uuid>/`, пары `<key>=<value>` — в теле.
- **Конверсии.** `conversion(typeUUID, query)` шлёт **сразу**:
  - Эндпоинт: `POST /api/v1/trigger/conversion/<visit_uuid>/<conversion_type_uuid>/`
  - `query` — произвольные параметры (например, сумма/валюта), уходят в теле запроса.

Ключи и значения тела процентно-кодируются, поэтому ввод с `+`, `&`, `#`, `%`, `=` (телефон вида `+7…`, текст с амперсандом) доезжает целиком; `null` / `undefined` превращаются в пустую строку, массив уходит парами `key[0]=`, `key[1]=`.

### Почему у старых визитов значения полей обрезаны на `#` или `&` — старый SDK слал их в URL без кодирования

До обновления SDK от 2026-09-04 оба обмена уходили запросом `GET` с парами прямо в строке запроса и без процентного кодирования. Поэтому у визитов, записанных до этой даты, значения с спецсимволами лежат в полях испорченными: всё после `#` отрезано (браузер считал это фрагментом), `&` рвал значение и превращал хвост в отдельный «параметр», `+` становился пробелом, `%` с цифрами — раскодировался. Типичная картина — поле формы или значение, которое лэнд своим скриптом отправлял в поле визита (например, имя лэнда из `aio.landing.name`), и у которого в визите остался только кусок до `#`. Поля, приехавшие из tracking-ссылки при регистрации визита, этот путь не затрагивал. Это не текущая поломка: пересчитать старые значения нельзя, а новые визиты записываются целиком.

### Работает ли ссылка `GET` на эти эндпоинты, собранная руками — да

Приёмники полей визита и конверсий метод не проверяют — принимают любой. Поэтому ссылка вида `GET /api/v1/trigger/conversion/<visit_uuid>/<conversion_type_uuid>/?<query>` (ручная проверка, постбэк рекламодателя — форматы постбэк-URL в [models/conversion-model.md](../models/conversion-model.md)) работает штатно. `POST` с телом — это то, что шлёт браузер, а не требование приёмника.

### Почему «поздние» поля визита не доехали — тайминги сброса буфера

То, что известно **сразу на загрузке** (`landed`, размеры окна, таймзона, `fingerprint`, стартовый нулевой скролл), уходит первым сбросом сразу после инициализации макросов — ждать 10 секунд не нужно. А значения, которые копятся **по ходу** (вехи времени на лэнде, рост скролла, поля формы по `blur`, куки `_fbp` / `_fbc` / `_ttp`), появляются с задержкой до 10 секунд или на уходе со страницы.

Сброс на `pagehide` уходит с флагом **`keepalive`** — браузер не отменяет такой запрос на уходе со страницы. Единственное оставшееся условие: сброс выполняется, только если предыдущий уже завершился (пока запрос в полёте, новый не стартует), а сетевой отказ SDK гасит молча. Практический вывод для разбора: отсутствие «поздних» полей у визита, который ушёл со страницы прямо посреди предыдущего сброса, — ожидаемая картина, а не поломка.

## II.5 `features` — что включено по умолчанию в SDK

Важно различать **два слоя дефолтов**:

1. **Встроенный дефолт SDK** (`DEFAULT_CONFIG` в коде) — что работает, если тенант ничего не переопределил.
2. **Шаблон тенанта** — `AIO SDK Macros Collection` обычно **включает** поверх монетизационные фичи (`backFix`, `fbCapi`, `fbPixel`, `form`).

| Feature | Встроенный дефолт SDK | Роль |
|---|---|---|
| `backFix` | **off** | History-trap на «Назад» (II.14). |
| `fbCapi` | **off** | FB: пиксель + PageView + захват кук `_fbp`/`_fbc` для server-CAPI (II.10). |
| `fbPixel` | **off** | FB: пиксель + lead-событие на сабмит (II.10). |
| `ttCapi` | **off** | TikTok: захват куки `_ttp` для server-CAPI (II.11). |
| `ttPixel` | **off** | TikTok: пиксель + page + lead на сабмит (II.11). |
| `form` | **off** | SDK-форма (II.12). |
| `pwa` | **off** | PWA-install + web-push + Android-intent (II.15). |
| `browserTimezone` | **on** | Поле `browser_time_zone`. |
| `landed` | **on** | Метрика Qualified Visits (II.6). |
| `scrolling` | **on** | Скролл-трекинг (Heatmaps). |
| `sessionRecords` | **on** | Запись сессий (rrweb, II.9). |
| `timeOnLandings` | **on** | Время на лэнде. |
| `windowDimensions` | **on** | Размеры окна. |
| `hueChanger` | **on** | Лёгкое смещение оттенка палитры лэнда (II.7). |
| `fingerprint` | **on** | Device-fingerprint → поле `fingerprint` (II.8). |

Модалки (`Pushing modal`, success/reject) не отдельная фича `features` — их рендерит форма/UTILS (II.12) через хуки `beforeSubmitPromise` / `submitPromises`.

## II.6 Макросы трекинга — какие поля визита пишут

Все шлют значения через `trigger` (flush ≤10 c):

- **`landed`** → `landed_on_landings_uuid` = `landing.uuid`, `landed_on_landings_types_uuid` = `lander_type_uuid`. Срабатывает один раз при загрузке. Это сигнал «лэнд реально загрузился» → метрика **Qualified Visits**.
- **`scrolling`** → `scroll_on_landings_uuid` + `scroll_on_landings_scroll` (по лэнду) и `scroll_on_landings_type_uuid` + `scroll_on_landings_type_scroll` (по типу лэнда). На загрузке синхронно (ещё **до** первого скролла, без debounce) шлёт стартовые `scroll_on_landings_scroll=0` / `_type_scroll=0`, дальше — debounce-обновления роста. Процент скролла округляется до кратного `scrolling.round` (дефолт **4**), отправка с debounce **2 c**, значение только **растёт** (откаты вверх не шлются). Для Heatmaps.
- **`timeOnLandings`** → пишет по вехам накопленного времени. Числа **5, 10, 15, 30, 60 c и 2, 3, 5, 10, 15 мин** в коде — это **задержки между отправками**, а не сами вехи; реальные пороги прошедшего времени = их нарастающие суммы: **5 c, 15 c, 30 c, 1 мин, 2 мин, 4 мин, 7 мин, 12 мин, 22 мин, 37 мин**. На каждой вехе пишет `time_on_landings_total`, `time_on_landings_uuid` + `time_on_landings_time`, `time_on_landing_types_uuid` + `time_on_landing_types_time`. Счётчик **продолжается между лэндами** (восстанавливается из `aio.time_on_landings_*`), кап — 1 час.

- **`windowDimensions`** → `browser_window_width` = `innerWidth`, `browser_window_height` = `innerHeight`.
- **`browserTimezone`** → `browser_time_zone` = IANA-таймзона (`Intl.DateTimeFormat().resolvedOptions().timeZone`, напр. `Europe/Berlin`).

## II.7 `hueChanger` — лёгкое смещение оттенка палитры

Вешает на `<html>` стиль `filter: hue-rotate(<random>deg)`, где random — целое из `[hueChanger.min, hueChanger.max]` (дефолт **2–8°**). Слегка смещает оттенок палитры лэнда; эффект незаметен глазу. Включён по умолчанию.

## II.8 `fingerprint` — отпечаток устройства (включён по умолчанию)

Загружает встроенную библиотеку **FingerprintJS** (v4.6.2), берёт `visitorId` и пишет его в поле визита **`fingerprint`**. Используется для идентификации устройства. **On by default** — то есть SDK по умолчанию снимает device-fingerprint каждого визита.

## II.9 `sessionRecords` — запись сессий (rrweb)

- Пишет DOM-события через **rrweb** (встроенная библиотека).
- Sampling по умолчанию: `mousemove: false`, `input: 'last'`, `scroll: 500` (мс), из mouse-interaction включены `MouseUp/MouseDown/Click/ContextMenu/DblClick`, выключены focus/blur/touch-события.
- Каждые **10 c** (если есть события) делает `POST /api/v1/agent/persist-session-record/<visit_uuid>/<session_uuid>` с телом: `events`, `landing_uuid`, `viewport` (w/h + scroll), `useragent`, флаги `is_inside_pwa` / `is_available_for_intent` / `is_available_for_install`, `relations`. Webview-флаг шлётся **дважды** — под `isWebview` (camelCase) и `is_webview` (snake_case) с одним значением.
- Включён по умолчанию. Это движок фичи **Session Records**.

## II.10 FB-трекинг: `fbPixel` vs `fbCapi` — чем отличаются

Pixel-ID обе фичи берут из поля визита `aio.visit.fields.fb_pixel`.

- **`fbCapi`** (server-side):
  - Загрузку пикселя (`fbevents.js`), `fbq('init', pixel)` и `fbq('track', 'PageView')` делает **только если заполнено поле визита `fb_pixel`** (`if(config.fbCapi.pixel)`). Пусто → пиксель не грузится и PageView не уходит — прямой ответ на «PageView не шлётся».
  - Опрос кук `_fbp` / `_fbc` (каждые 100 мс) идёт **без этого условия**: как только появились — пишет их в поля визита (слаги настраиваются: `fbpSlug` дефолт `fbp`, `fbcSlug` дефолт `fbc`). Эти куки нужны серверу для матчинга в Facebook **Conversion API**.
- **`fbPixel`** (client-side lead):
  - Грузит `fbevents.js`, делает `fbq('init', pixel)`.
  - Регистрирует `form-submit-fn`: на **успешном** сабмите формы шлёт `fbq('track', <fb_lead_action>)`, где событие берётся из поля визита `fb_lead_action`.
  - Сам по себе `fbPixel` **PageView не шлёт** — PageView отправляет `fbCapi`. Если включён только `fbPixel`, PageView в FB не уйдёт.
- Если включены обе — пиксель грузится один раз (есть guard `if (f.fbq) return`).

## II.11 TikTok-трекинг: `ttPixel` vs `ttCapi`

Похоже на FB, но **не полностью симметрично**: у TikTok пиксель и page-событие живут в `ttPixel`, а `ttCapi` — только захват куки.

- **`ttPixel`**: pixel из `aio.visit.fields.tt_pixel`; `ttq.load(pixel)`, `ttq.page()` (если `events.pageView`), и на сабмит — `ttq.track(<tt_lead_action>)` (из поля `tt_lead_action`) через `form-submit-fn`.
- **`ttCapi`**: **только** захват куки — опрашивает `_ttp` и пишет её в поле визита (слаг `ttpSlug`, дефолт `ttp`) для серверного TikTok CAPI. В отличие от `fbCapi`, пиксель **не грузит** и PageView **не шлёт** (это делает `ttPixel`).

## II.12 SDK-форма — полный разбор

Всё ниже относится к фиче `form` (`features.form`) и её конфигу `config.form`.

### Как SDK находит и рендерит форму — селектор и сабмит

SDK ищет элементы по `form.selector` (дефолт **`.aio-form`**) и достраивает в них контролы. Сабмит — `GET` на `form.url` (дефолт **`/`**, т.е. на сам лэнд) с query-строкой из данных формы. В данные добавляются `_aio_handle` и пара `handle.key=handle.value` — по ним запрос распознаётся как сабмит формы и обрабатывается как пуш лида в Destination (см. `handle` в §II.2). `_aio_handle` по умолчанию `form`, но если элемент формы несёт атрибут `data-handle`, значение берётся из него (`form.dataset.handle`). Именно этот per-form handle роутит `{{form:N}}` в разные `Handle form N`.

### Контролы формы и валидация — типы и правила

Дефолтный набор: `first_name`, `last_name`, `email`, `phone` (+ скрытые `handle` и `_aio_handle`, + `submit`).

Типы контролов: `text`, `email`, `phone`, `number`, `date`, `textarea`, `select`, `checkbox`, `exclamation`, `hidden`, `submit`.

Атрибуты контрола: `key` (слаг поля; если совпадает со слагом поля визита в AIO — ввод пишется в это поле), `placeholder`, `label`, `value` (дефолтное значение, `null` — без дефолта), `type`, `rules`.

**`select`** — выпадающий список. Опции задаются атрибутом **`values`** = map `значение → отображаемая-подпись` (**ключ** объекта = что уходит в `<option value>` / в поле визита; **значение** объекта = видимый текст). Пример: `{"value1":"Option 1", "value2":"Option 2"}` → `<option value='value1'>Option 1</option>`. Дефолтное значение контрола (`value`) для преселекта должно равняться **КЛЮЧУ**. Только `select` с заданным `values` рендерится как dropdown.

**`checkbox`** — при сабмите сериализуется так: отмечен → поле = строка-литерал `'true'`; не отмечен → поле **вообще не попадает** в пуш (не шлётся `value` из шаблона).

Правила (`rules`) превращаются в HTML-атрибуты: `required`, `minlength:N`, `maxlength:N`. Особое правило `pattern:no-local-emails` подставляет email-regex (отсекает «local»-адреса). Прочие правила: `pattern:<regex>` (произвольный regex), `type:url`, `type:number`. Дефолты: email — `required, minlength:3, maxlength:64, pattern:no-local-emails`; phone — `required, minlength:5, maxlength:20`.

### Прогрессивный сбор данных формы — поля заполняются до сабмита

На событие `form.collectDataEvent` (дефолт **`blur`**) значение каждого поля **сразу** шлётся в визит (`trigger`), ещё **до** сабмита (с дедупом через внутренний кэш).

Blur-сбор работает для всех text/select-полей, **кроме `phone`**: телефон отправляется только на сабмите (через `getNumber()`). Отсюда «имя/почта заполнились в визите, а телефон нет».

Это объясняет частый вопрос: «поля визита (имя/почта) заполнились, хотя юзер форму не отправил». Так и задумано — SDK ловит ввод по `blur`.

### Мультишаг и раскладка формы

`form.steps` — массив массивов ключей (каждый под-массив = шаг). `form.layout` — раскладка по рядам/колонкам. `stepsShowing` — показывать индикатор шагов.

На промежуточном шаге сабмит = отправить поля текущего шага в визит + перейти на следующий шаг (не пуш). Валидационные правила применяются только к полям активного шага.

### Поле phone — intl-tel-input

Поле `phone` инициализирует intl-tel-input. `initialCountry` = `aio.visit.country_code`, а если его нет — `form.defaultCountryCode` (дефолт **`GB`**).

Параметры по умолчанию (`intlParameters`): `nationalMode: true`, `autoPlaceholder: 'aggressive'`.

В кастомных формах-макросах встречается отдельный под-блок `form.phone` с настройками телефонного поля: `defaultCountry` (страна по умолчанию, напр. `"us"`), `showDialCode` (показывать код страны), `separateDialCode` (выносить код в отдельный блок — если поддерживается версией библиотеки).

На сабмите телефон валидируется `isValidNumber()`; в данные кладётся международный формат (`getNumber()` через `intlNumberProcessFn`).

### Lifecycle сабмита SDK-формы — шаги по порядку

1. `preventDefault`, валидация телефона (если невалиден → `submitErrorCb`, дефолт `alert`).
2. `disableForms` (кнопка → состояние loading).
3. Выполняются хуки `form.beforeSubmitPromise` — **после** валидации, **до** пуша.
4. Если заданы `form.submitPromises` — создаётся «прерываемый» promise (`formPromise`), который **резолвится ответом пуша**; через него крутят Pushing/Success-модалки.
5. `fetch(form.url + '?' + params)`:
   - **JSON + `success:true`** → выполняются все `form-submit-fn` (FB/TT lead-события), форма получает класс `aio-sdk-form-success`, идёт countdown длиной `form.successTimeoutSeconds` (дефолт **5 c**, обновляет `%counter%` в кнопке), затем `form.successFn(json.url)` — редирект (дефолт `window.top.location.replace(url)`).
   - **JSON + `success:false`** → `form.rejectedMessage` (дефолт «We cant register you at this time.»).
   - **не-JSON ответ** → `window.top.location.reload()`.

### Хуки формы — имена ключей beforeSubmitPromise и submitPromises

Точные имена ключей в коде:
- `beforeSubmitPromise` — **единственное число** — массив функций `(UTILS, aioExchange, config) => Promise`. После валидации, до пуша.
- `submitPromises` — **множественное** — массив функций `(UTILS, formPromise, aioExchange, config) => Promise`. Вокруг пуша; `formPromise.then(json)` даёт ответ Destination для модалок. Это и есть механизм **«Pushing modal / Advanced Form Sample»**.

### Семантика resolve/reject в хуках формы — два хука ведут себя по-разному

`resolve`/`reject` в двух хуках формы значат разное:
- В `beforeSubmitPromise` (блокирующий пре-сабмит): `resolve()` = шаг прошёл, форма продолжает сабмит; `reject()` = сабмит останавливается, реальный `fetch` **не стартует**, ошибка показывается через `submitErrorCb`.
- В `submitPromises` (неблокирующая обёртка): `resolve()`/`reject()` завершают **только твой собственный** Promise и **НЕ** управляют реальным сабмитом — не запускают/не отменяют `fetch`, не дёргают `submitErrorCb` и не запускают error-flow формы.
- **Оба массива хуков** запускаются через `Promise.all` — все Promise в массиве стартуют **параллельно**, не цепочкой. Если нужна строгая последовательность шагов — объединяй их в **один** Promise с `async`/`await` (а не раскидывай по элементам массива).

### formPromise — Promise результата реального сабмита

`formPromise` — Promise, который резолвится ответом реального пуша:
- `.then(response)` → `{ success: true, url: "..." }` (успех);
- `.catch(reason)` → `reason` = `rejectedMessage` (ошибка сети, невалидный JSON или `success: false`).
- Внутри `submitPromises` через `formPromise` крутят waiting/success/failed-модалки и кастомный редирект (напр. `setTimeout(() => { window.location.href = response.url }, 2000)` в `formPromise.then`).

### Кастомный success-flow через successFn

`successFn` определяет, как форма редиректит после успешного пуша:
- Дефолт: `successFn: (url) => window.top.location.replace(url)` — SDK сам делает редирект на autologin-URL.
- Чтобы сделать редирект вручную — переопредели `successFn: (url) => null`, после чего управляй редиректом сам внутри `submitPromises` (см. `formPromise.then` выше).

### i18n и переводы формы

- Встроен свой пак переводов на **33 языка**. Полный набор ключей (`FIRST_NAME`, `LAST_NAME`, `EMAIL`, `PHONE`, `SUBMIT`, `SUBMIT_LOADING`, `SUBMIT_SUCCESS`, `SUBMIT_STEP`, `HEADER`) есть только у `en`; у остальных языков переведены подписи полей, `SUBMIT` и `HEADER`, а строки состояния кнопки (`SUBMIT_LOADING`, `SUBMIT_SUCCESS`, `SUBMIT_STEP`) — лишь у единиц, остальные падают на английский. Отсюда типичная картина на лэнде, где форма из ERP не подключена: кнопка на языке визитёра, а «отправляется» / «успех» — по-английски.
- Если к лэнду подключена форма из ERP, её словарь замещает встроенный пак целиком, и встроенные переводы остаются фолбэком только для строк `SUBMIT*` — как устроены языки формы, [how-to/forms.md](../how-to/forms.md).
- Язык формы = `form.language` (дефолт — первые 2 буквы `navigator.language`). Фолбэк: нужный язык → `en` → сам ключ.
- Заменить переводы — через `aioBus` тип `form-translations`. Словарь **присваивается целиком**, а не мёржится с текущим: после этого встроенный пак SDK остаётся фолбэком только для четырёх строк `SUBMIT*`, а подписи и плейсхолдеры полей берутся из присвоенного словаря. Тем же способом на лэнд приезжает словарь формы из ERP — что из этого следует для набора языков формы, см. [how-to/forms.md](../how-to/forms.md).

### Кастомизация внешнего вида формы — CSS-переменные

Форме навешивается класс `aio-sdk-form`; статусы — `aio-sdk-form-loading`, `aio-sdk-form-success`.

Тема — через CSS-переменные `--aio-sdk-*`. Переопределять можно глобально (в `AIO SDK Macros Collection`, блок `<style>`) или локально (`<style>` в HTML конкретного лэнда — локальные стили перебивают глобальные на этом лэнде). Переопределяют их в селекторе `.aio-sdk-form { … }`.

### CSS-переменные формы — `--aio-sdk-form-*` и `--aio-sdk-input-*`

Переменные ниже — встроенные дефолты SDK; многие ссылаются друг на друга через `var()`:

| Переменная | Дефолт |
|---|---|
| `--aio-sdk-form-padding` | `30px 30px 20px` |
| `--aio-sdk-form-layout-gap` | `var(--aio-sdk-input-margin)` |
| `--aio-sdk-form-steps-gap` | `60px` |
| `--aio-sdk-form-steps-margin` | `15px` |
| `--aio-sdk-form-step-diameter` | `35px` |
| `--aio-sdk-form-step-font-size` | `var(--aio-sdk-input-font-size)` |
| `--aio-sdk-form-step-line-color` | `var(--aio-sdk-input-bg)` |
| `--aio-sdk-form-step-bg` | `var(--aio-sdk-input-bg)` |
| `--aio-sdk-form-step-border` | `var(--aio-sdk-submit-bg)` |
| `--aio-sdk-form-step-color` | `var(--aio-sdk-input-color)` |
| `--aio-sdk-form-step-active-bg` | `var(--aio-sdk-submit-bg)` |
| `--aio-sdk-form-step-active-border` | `var(--aio-sdk-submit-bg)` |
| `--aio-sdk-form-step-active-color` | `var(--aio-sdk-submit-color)` |
| `--aio-sdk-input-label-color` | `black` |
| `--aio-sdk-input-label-font-size` | `1em` |
| `--aio-sdk-input-label-margin` | `3px` |
| `--aio-sdk-input-margin` | `15px` |
| `--aio-sdk-input-bg` | `white` |
| `--aio-sdk-input-font-size` | `1em` |
| `--aio-sdk-input-border` | `#ced4da` |
| `--aio-sdk-input-border-radius` | `0px` |
| `--aio-sdk-input-padding` | `15px 20px` |
| `--aio-sdk-input-color` | `black` |
| `--aio-sdk-input-textarea-size` | `100px` |
| `--aio-sdk-input-checkbox-gap` | `5px` |

### CSS-переменные формы — `--aio-sdk-submit-*`

| Переменная | Дефолт |
|---|---|
| `--aio-sdk-submit-bg` | `#60359b` |
| `--aio-sdk-submit-padding` | `var(--aio-sdk-input-padding)` |
| `--aio-sdk-submit-border` | `transparent` |
| `--aio-sdk-submit-border-radius` | `var(--aio-sdk-input-border-radius)` |
| `--aio-sdk-submit-color` | `white` |
| `--aio-sdk-submit-font-size` | `var(--aio-sdk-input-font-size)` |

> Таблицы — курируемый субсет встроенных дефолтов, не исчерпывающий список.

### HTML-шаблоны контролов — механизм `templates`

Каждый тип контрола рендерится из своей строки-шаблона в `form.templates`; **любой шаблон переопределяется** в `config.form.templates`.

В шаблонах работает подстановка токенов: `%key%` (слаг поля), `%placeholder%`, `%value%`, плюс структурные `%step%`, `%steps%`, `%layout-rows%`, `%layout-columns%`, `%form-item-key%`; у `select` — `%select-options%`, `%option-value%`, `%option-label%`, `%selected%`; у `submit` — `%loading%`, `%success%`.

Переопределяемы и шаблоны раскладки/шагов, не только полей: `steps`, `step`, `layout`, `layout-row`, `layout-column`, плюс `hidden`, `text`, `number`, `textarea`, `select`, `select-option`, `select-option-placeholder`, `phone`, `email`, `date`, `checkbox`, `submit`.

**Конфигурируемые class-name контейнеров шагов** (в `config.form`, рядом с поведенческими настройками):

| Ключ | Дефолт |
|---|---|
| `activeStepClass` | `aio-sdk-step-container-active` |
| `availableStepClass` | `aio-sdk-step-container-available` |
| `hiddenClass` | `aio-sdk-form-hidden` |

> Ключ хуков пре-сабмита — **`beforeSubmitPromise`, в единственном числе**. Вариант `beforeSubmitPromises` (мн. число) встречается в готовых конфиг-блоках, которые копируют целиком, — SDK его **не читает** (см. §II.20). Дописал туда хуки — они не подхватятся.

### Модалки — UTILS.openModal

`UTILS.openModal({ nativeElement, url, countDown, countDownStyle, countDownCb, resolve, reject })`.

- `countDownStyle`: `number` (дефолт) / `minutes` (`MM:SS`) / `hours` (`HH:MM:SS`).
- `countDownCb` — колбэк по истечении обратного отсчёта; внутри него обычно авто-`resolve()` или `ut.closeModal(modal)`.
- `resolve` / `reject` — пробрасываются из Promise хука, чтобы привязать кнопки модалки к завершению хука (кнопка «подтвердить» → `resolve`, «отмена» → `reject`).
- Это основа двух паттернов: confirm-модалка **перед** сабмитом (в `beforeSubmitPromise` — `resolve`/`reject` управляют тем, пойдёт ли пуш) и waiting-модалка **вокруг** сабмита (в `submitPromises` — `countDownCb` авто-резолвит обёртку, а реальный результат приходит через `formPromise`).
- Разметка: контейнер с классом-якорем + внутри `.aio-sdk-modal-countdown` (тикающий счётчик), `.aio-sdk-modal-redirect-url` (ссылка, в href подставляется `url`), кнопки `.aio-sdk-modal-button-resolve` / `.aio-sdk-modal-button-reject`. Открытие = класс `aio-sdk-modal-opened`.

## II.13 Три способа обработки формы — что выбрать

1. **SDK-форма** (`features.form: true`) — управляемая SDK: `controls`/`layout`/`steps`, intl-tel-input, модалки, переводы из коробки.
2. **Custom / Non-SDK через aioBus** — своя `<form>` на лэнде: через `dom-ready` + `intl-instance` берёшь intl-инстанс из SDK, сам вешаешь submit-листенер, подкладываешь `_aio_handle=form` + `handle.key/value`, шлёшь `GET /?<params>`. **Handle обязателен — без него сабмит не станет переходом `Handle-form`.** По этой же причине кастомную форму нельзя просто повесить на `action="{{link}}"`: GET-сабмит стирает query у `action` вместе с handle. Готовые плейсхолдеры под это — `{{link_key}}` / `{{link_value}}` (скрытым полем в форму) и `{{link-form}}` (сабмит из JS), см. [reference/placeholders.md](placeholders.md). Если в флоу после Destination есть **destination-pushed** transition — юзера редиректит на шаг флоу, **а не** на `url` из ответа.
3. **Legacy jQuery** (`.lander-form`, отдельный standalone-скрипт) — старый обработчик: собирает `.lander-form-send`, шлёт `GET /?<params>`, на успех — задержка ~10 c, потом `window.top.location.replace(url)` и FB-пиксель через `aioDataLayer.destinationPushed`. Применяется на старых лэндах.

## II.14 `backFix` — механика под капотом

- **History-trap.** При старте: `history.replaceState` ставит хэш `#!/<pathName>`, затем `pushState`. На `popstate` (юзер нажал «Назад»), если хэш совпал — через **200 мс** делает `location.replace(config.backFix.link)`.
- `backFix.link` — дефолт `{{link}}&backfix=true`: визит уходит на следующий шаг флоу, а параметр `backfix=true` в URL сервер превращает в поле визита `is_backfix`.
- `enabledBackFix` — **функция-условие** (по умолчанию проверяет `aio.landing.lander_type_uuid`): backFix активен только на нужных типах лэндов.
- **URL переписывается даже когда `enabledBackFix()` = false.** Макрос всё равно срезает query (`url.split('?')[0]`) и дописывает сохранённое `localStorage[localStorageKey]` через `pushState`. То есть маскировка query-параметров идёт на **КАЖДОМ** лэнде с макросом `backFix`, а не только там, где вооружён перехват «Назад».
- `enableStrangeUrlParameters` — если `true`, кладёт в localStorage набор `utm_id` (20 случ. симв.) / `utm_medium` / `utm_campaign` / `utm_content` (по 10) + `utm_visit = aio.visit.uuid` (**реальный uuid визита, не случайный**), маскируя исходную ссылку. Пишется один раз на браузер (guard по наличию ключа) и переиспользуется на последующих визитах. Если `false` — сохраняются реальные query-параметры.
- `localStorageKey`, `pathName` — служебные ключи, обычно не трогают.

## II.15 `pwa` — install + web-push + Android-intent

PWA-макрос превращает лэнд в устанавливаемое приложение и подписывает на web-push. Здесь — что макрос делает и какими ключами конфига это управляется; пошаговая процедура (DOM-скелет лэнда, включение фичи, вшивание PWA во флоу) — в [how-to/pwa.md](../how-to/pwa.md).

Четыре ветки поведения:
- **Доступно для установки** (Android + Chrome, не webview, не внутри PWA): показывает preloader → ловит `beforeinstallprompt` → активирует install-кнопку → по клику `prompt()` → анимация прогресса установки. На `appinstalled` шлёт конверсию `conversions.install`. Service worker при этом регистрируется независимо от установки (см. ниже), состояния кнопки и их таймеры — ниже.
- **Внутри in-app браузера** (FB/IG/TikTok webview на Android): формирует **Android `intent://`** для открытия в Chrome (+ `browser_fallback_url`) и показывает intent-модалку — чтобы вырваться из webview.
- **На iOS**: install-кнопка вместо системного prompt открывает модалку с инструкцией добавить страницу на домашний экран; под шагами инструкции — кнопка, уводящая визит на оффер (разобрана ниже).
- **Внутри уже установленной PWA**: шлёт конверсии открытия и установки, регистрирует service worker и подписывает на web-push (разобрано ниже).

### Откуда PWA-макрос берёт install-UI и manifest

Макрос работает по DOM-контейнерам (id настраиваются): кнопка install (`installButtonId`, дефолт `install`), контейнеры `pwa-install` / `pwa-preloader` / `pwa-inside`, модалки `intent-modal` / `ios-modal` (свои элементы под модалки необязательны — см. ниже).

Иконки install-UI берутся из **серверного manifest-JSON** (не из DOM-слайдера) и заранее префетчатся, чтобы диалог установки Chrome собрался в окне user-activation (`sliderItemsQuerySelector` в текущем бандле не используется). Сам manifest в макросе не пишется — он собирается **на сервере per-visit** и подключается макросом как `<link rel=manifest href=/api/v1/pwa/manifest/<visit_uuid>.json>` (стабильный URL, не blob → install-identity персистит).

### `pwa.session.hess` и `pwa.session.sess` — в каком виде задаются ссылки продолжения флоу

Форму записи SDK разбирает сам и решает, к чему её приклеить:

- **абсолютный URL** (`https://other.com`, `https://other.com?hess=1`) — берётся как есть, адрес лэнда к нему не приписывается;
- **путь от корня** (`/path`) — приклеивается к origin страницы лэнда (схема + домен);
- **всё остальное** — приклеивается к тому же origin через `/`: query-хвост `?hess=x` даёт `https://<домен-лэнда>/?hess=x`.

Что это за ссылки и откуда они приезжают в конфиг макроса — [how-to/pwa.md](../how-to/pwa.md).

### Как SDK узнаёт, что приложение уже установлено

Уже установленное приложение макрос определяет сам, без клика: сразу после префетча manifest-JSON он вызывает `navigator.getInstalledRelatedApps()`. Проверка доступна не везде — нужен Android Chrome, наличие самого метода в браузере и manifest, который перечисляет себя в `related_applications`; вне этих условий SDK опирается на `appinstalled` и таймеры.

Если приложение найдено, кнопка из состояния `loading` или `install` сразу переводится в `open`. Без этой проверки она висела бы в `loading` до таймаута: для уже установленного приложения Chrome не шлёт `beforeinstallprompt` вовсе.

Проверка привязана к визиту. Manifest AIO отдаёт **под каждый визит** своим URL, а установленный WebAPK помнит тот URL, с которого ставился, — совпадение бывает только при возврате **того же** визита. На новом визите URL другой, `getInstalledRelatedApps` вернёт пусто, и об установленном приложении SDK не узнает. Это ожидаемое поведение, а не поломка детекта.

### Когда SDK регистрирует service worker PWA

Service worker `/aio-static/aio-sw.js` макрос регистрирует **уже при своей инициализации на лэнде** — то есть до установки и независимо от платформы и ветки поведения; внутри запущенной установленной PWA регистрация повторяется. Scope в обоих случаях — корень сайта.

### Состояния install-кнопки и ключи конфига, которые ими управляют

Состояние кнопки SDK держит в её атрибуте `data-state`, тексты берёт из `data-install` / `data-open`. Состояний четыре — `loading`, `install`, `progress`, `open`, — и переходы между ними не линейны: часть даёт событие браузера, часть — таймер.

| Переход | Чем вызван | Ключ конфига |
|---|---|---|
| → `loading` | инициализация макроса: браузер ещё не сказал, можно ли предлагать установку | — |
| → `install` | пришёл `beforeinstallprompt` (в том числе ранний, перехваченный в `window.__bip`); на iOS — сразу | — |
| `loading` → `open` | prompt так и не пришёл, сработал таймаут | `pwa.intervalBeforeOpenButton` (деф. **3000 мс**) |
| `install` → `progress` | визит принял системный диалог установки (`outcome === 'accepted'`); клики в этом состоянии игнорируются | `pwa.loadingDuration` (деф. **20 000 мс**) — основная часть анимации |
| `progress` → `open` | приложение готово либо кончилось окно ожидания (разобрано ниже) | окно = `pwa.loadingDuration` + `pwa.installedTimeout` (деф. **15 000 мс**) |
| `open` → `install` | пришёл поздний `beforeinstallprompt` (разобрано ниже) | — |

Прелоадер перед экраном установки держится `pwa.preLoadingDuration` (деф. **2000 мс**). Id самой кнопки — `pwa.installButtonId` (деф. `install`).

Числа выше — встроенные дефолты SDK; шаблоны `PWA Builder` часть таймингов переопределяют ([how-to/pwa.md](../how-to/pwa.md)).

### Почему после подтверждения установки кнопка ещё не `Open`

Конец основной части анимации кнопку не отпускает. Chrome шлёт `appinstalled` в момент, когда визит **подтвердил** диалог установки, а WebAPK в этот момент обычно ещё собирается — отпустить кнопку раньше означает, что клик по `Open` не найдёт приложения и уведёт визит на оффер без установки. Полное окно прогресса = `pwa.loadingDuration` + `pwa.installedTimeout` (деф. 20 000 + 15 000 мс), проценты идут всё это время и упираются в потолок `99%` (100% означало бы `Open`). Сигнал готовности берётся по-разному:

- **готовность проверяема** (условия из «Как SDK узнаёт, что приложение уже установлено») — SDK опрашивает `getInstalledRelatedApps` раз в секунду и переводит кнопку в `open` в момент реальной готовности, в том числе посреди анимации;
- **непроверяема** — сигналом служит `appinstalled`: кнопка отпускается, как только прошла часть `loadingDuration`;
- **ни то, ни другое не наступило** — кнопка отпускается в `open` в конце окна.

Отсюда симптом «прогресс упёрся в 99 % и стоит»: это хвост штатного окна ожидания готовности приложения, а не зависший скрипт.

### Почему кнопка `Open` возвращается в `Install`

Поздний `beforeinstallprompt` переводит кнопку из `open` обратно в `install`. Логика такая: для уже установленного приложения Chrome это событие не шлёт вообще, значит его приход доказывает, что приложение **не** установлено — и надпись `Open` на экране в этот момент ошибочна. Клик по ней ушёл бы в intent, не нашёл бы приложения и увёл бы визит на оффер без установки.

Единственное исключение — состояние `progress`: идущую установку поздний prompt не прерывает.

Тот же приоритет действует и на клике: если у SDK на руках живой prompt-объект, по клику показывается системный диалог установки — независимо от того, что написано на кнопке.

### Установка на iOS — модалка с инструкцией вместо системного prompt

На iOS у макроса своя ветка: install-кнопка создаётся, но по клику открывает не системный install-prompt, а модалку с тремя шагами — нажать `Share` в панели браузера, выбрать `Add to Home Screen`, нажать `Add` и открыть приложение с домашнего экрана. Ветка включается детектом iOS, к которому отнесён и macOS с тач-экраном. Если страница открыта в in-app браузере, модалка сверху показывает подсказку сначала открыть её в `Safari` (меню `···` → `Open in Safari`). Под шагами инструкции модалка показывает кнопку действия, которая уводит визит на оффер без установки. Дополнительно макрос дописывает в `<head>` Apple-специфичные meta-теги (`apple-mobile-web-app-capable` и соседние), если их на лэнде нет.

Заголовок модалки `Install the app` — встроенный дефолт SDK: его перекрывает заголовок, заданный самим лэндом, и все встроенные шаблоны `PWA Builder` ставят свой ([how-to/pwa.md](../how-to/pwa.md)). Дефолтная подпись видна, только если лэнд своей не задал, — искать модалку по тексту `Install the app` не стоит.

### Кнопка под шагами iOS-модалки — уход на оффер без установки

Клик по кнопке действия в iOS-модалке уводит визит по ссылке продолжения флоу (`hess`) с пометкой `_hess_reason=ios_modal`: приложение при этом не ставится и конверсию `Install` этот путь не создаёт. Тап по фону модалки её просто закрывает — шаги инструкции можно пройти и вернуться на лэнд.

Подпись кнопки SDK берёт по цепочке: ключ `pwa.modals.ios.button` (дефолт `null`) → атрибут `data-open` install-кнопки лэнда → литерал `Open`. Встроенные шаблоны `PWA Builder` этот ключ не задают, поэтому на их витринах кнопка подписана тем же словом, что и состояние `open` install-кнопки. Что это меняет для аналитики PWA-визитов с iOS и как переопределить модалку своим элементом — [how-to/pwa.md](../how-to/pwa.md).

### Что делает макрос внутри уже установленной PWA

Шлёт конверсии `conversions.open` **и** `conversions.install` (обе, на каждом открытии); заново регистрирует service worker `/aio-static/aio-sw.js` (scope — корень сайта); запрашивает разрешение на уведомления; если визит ещё не подписан (`aio.visit.is_push_subscribed` ложно) — берёт VAPID-ключ `GET /api/v1/agent/pub-key-push-settings/<visit_uuid>`, делает `pushManager.subscribe`, шлёт подписку `POST /api/v1/agent/persist-push-settings/<visit_uuid>`, затем конверсию `conversions.subscribe`; в конце редиректит на «hess»-URL. `conversions` — это UUID типов конверсий, которые шлются в моменты install / open / subscribe.

Этот же service worker показывает пришедшие push-уведомления, а у пушей remarketing-кампании сообщает об их доставке и открытии (`Delivered` / `Opened`) — что из этого видно в аналитике, см. [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

### Доставка конверсий перед редиректом — `keepalive` и потолок ожидания

Конверсии SDK отправляет с флагом `keepalive`, поэтому запрос переживает немедленный редирект: PWA-ветки шлют конверсию и тут же уводят визит на `hess`, а браузер такой запрос не отменяет на уходе со страницы.

На аварийных ветках внутри установленной PWA — service worker'а в браузере нет или его регистрация упала — добавлено короткое ожидание: перед редиректом макрос ждёт, пока отправка `conversions.open` и `conversions.install` завершится, но **не дольше 2 секунд**. Потолок жёсткий: зависший запрос не может задержать визит на лэнде. На штатной ветке отдельного ожидания нет: редирект там выполняется последним — после запроса разрешения на уведомления и подписки на push.

Отличие от полей визита: конверсия уходит сразу, а поля копятся в буфере, и их сброс на уходе со страницы выполняется, только если предыдущий сброс уже завершился (§II.4). Сам `keepalive` стоит на обоих запросах.

### Модалки intent и iOS — свой элемент на лэнде необязателен

SDK ищет модалки по id (`intentModalId`, дефолт `intent-modal`; `iosModalId`, дефолт `ios-modal`) и, **если элемента с таким id на странице нет, рендерит модалку сам** и дописывает её в `<body>`. Собственная разметка модалки на лэнде — необязательное переопределение, а не обязательный элемент скелета: тексты дефолтных модалок лежат в блоке `modals` конфига PWA-макроса.

### Почему install-запрос виден на каждом открытии PWA

При открытии уже установленной PWA макрос шлёт **обе** конверсии — `open` и `install`, поэтому в сетевых логах лэнда install-запрос виден каждый раз. Появится ли при этом второй `Install` в конверсиях визита, решает `Uniqueness Strategy` у типа конверсии `Install` в тенанте (см. [models/conversion-model.md](../models/conversion-model.md)):

- `Single Conversion` — штатная настройка PWA-шаблона: повторную конверсию того же типа на визите AIO отсекает, в визите остаётся один `Install`.
- `Multiple Infinity` — каждое открытие установленной PWA создаёт новую конверсию `Install`; отсюда симптом «у одного визита несколько инсталлов».

При разборе трафика: расхождение между числом install-запросов в логах лэнда и числом конверсий визита ожидаемо и определяется этой настройкой, а не поломкой лэнда.

## II.17 UTILS — что доступно хукам формы

Хуки (`beforeSubmitPromise`, `submitPromises`) получают объект `UTILS`. Полезное:
- `openModal` / `closeModal` — модалки с countdown (см. II.12).
- `isWebview()` — детект in-app браузеров (Facebook/Instagram/TikTok/Messenger/WhatsApp/Telegram/Discord/Line и др.).
- `isInsidePWA()` — визит открыт в **установленном** приложении. Два пути определения: режим отображения `(display-mode: standalone)` (у iOS Safari — `navigator.standalone`), а для iOS дополнительно UA `iPhone` / `iPad` / `iPod` **без** `Safari`. Второй путь обязательно отсеивает webview: iOS in-app браузеры (Instagram / Facebook / TikTok и др.) тоже убирают `Safari` из UA, но установленной PWA не являются — без этого отсева визит из соцсети ошибочно считался бы «внутри PWA».
- `isAvailableForInstall()` (Android + Chrome, не webview), `isAvailableForIntent()` (Android + webview), `isIOS()` (iPhone/iPad, плюс macOS с тач-экраном).
- `pathGet`, `queryString`, `extractPictures`, `UrlB64ToUint8Array` (для VAPID-ключа push).

## II.18 Эндпоинты агента — сводка

| Назначение | Метод + путь |
|---|---|
| Записать поле визита | `POST /api/v1/trigger/field/<visit_uuid>/` (пары `<k>=<v>` — в теле) |
| Отправить конверсию | `POST /api/v1/trigger/conversion/<visit_uuid>/<type_uuid>/` (`query` — в теле) |
| Сабмит формы (пуш лида) | `GET <form.url>/?<данные>&_aio_handle=form&<handle.key>=<handle.value>` |
| PWA manifest (per-visit) | `GET /api/v1/pwa/manifest/<visit_uuid>.json` |
| Запись сессии | `POST /api/v1/agent/persist-session-record/<visit_uuid>/<session_uuid>` |
| VAPID pub-key для push | `GET /api/v1/agent/pub-key-push-settings/<visit_uuid>` |
| Сохранить push-подписку | `POST /api/v1/agent/persist-push-settings/<visit_uuid>` |
| Статика SDK | `/aio-static/sdk/main.js`, `/aio-static/sdk/main.css`, `/aio-static/aio-sw.js` |

## II.19 Поля визита, которые пишет SDK — сводка слагов

| Поле визита | Источник (макрос) |
|---|---|
| `landed_on_landings_uuid`, `landed_on_landings_types_uuid` | `landed` |
| `scroll_on_landings_uuid` / `_scroll`, `scroll_on_landings_type_uuid` / `_type_scroll` | `scrolling` |
| `time_on_landings_total`, `time_on_landings_uuid` / `_time`, `time_on_landing_types_uuid` / `_time` | `timeOnLandings` |
| `browser_window_width`, `browser_window_height` | `windowDimensions` |
| `browser_time_zone` | `browserTimezone` |
| `fingerprint` | `fingerprint` |
| `fbp`, `fbc` | `fbCapi` |
| `ttp` | `ttCapi` |
| `is_backfix` | пишет **сервер** по параметру `&backfix=true`, который добавляет макрос `backFix` |
| `pwa_hess_reason` | пишет **сервер** по параметру `_hess_reason`, который PWA-макрос дописывает к `hess`-ссылке на каждом уходе на оффер; значения и чтение — [how-to/pwa.md](../how-to/pwa.md) |
| значения полей формы (`first_name`, `email`, …) | `form` (на `blur` + на сабмит) |

> Слаги выше — что **пишет фронт**. Часть значений SDK, наоборот, **читает** из `aio.visit.fields.*` (напр. `fb_pixel`, `tt_pixel`, `fb_lead_action`, `tt_lead_action`) — их значения вычисляет сторона AIO (Fill Fields / Distribution), на лэнд они приезжают в `window.aio`.

## II.20 Ключи, на которых чаще всего спотыкаются

Эти моменты часто сбивают при кастомизации — ориентируйся на **код актуального бандла**:

- **`beforeSubmitPromise` vs `beforeSubmitPromises`.** SDK читает ключ в **единственном** числе (`config.form.beforeSubmitPromise`); написанное во множественном **не подхватится**. (`submitPromises` — наоборот, множественное, это корректно.)
- **`enabledBackFix` vs `backFix`.** Условие включения backFix в коде — `config.backFix.enabledBackFix` (функция). Если в макросе написать просто `backFix: () => true` — не сработает.
- **`autoPlaceholder`.** Дефолт intl-tel-input в коде — `aggressive` (не `polite`).
- **Слаги трекеров.** Имена полей `browser_time_zone`, `browser_window_width/height` **захардкожены** в SDK; настройки вида `browserTimezone.slug` / `windowDimensions.*Slug` на них не влияют.
- **`?v=` не выбирает версию — ни у скрипта, ни у стилей.** На статике лежит по одному файлу, `main.js` и `main.css`, и агент отдаёт `/aio-static/*` как обычную статику: query-параметр в выборе файла не участвует. Разные числа `?v=` на разных лэндах и в превью формы не означают разных версий SDK — единственный эффект параметра — сбить кэш браузера. При расследовании «почему опция не работает» сверяй поведение с кодом актуального бандла, а не с этим параметром.
