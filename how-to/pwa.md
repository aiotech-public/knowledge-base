---
id: how-to-pwa
title: Как создать и настроить AIO PWA (внутреннюю)
description: Создание внутренней PWA в AIO (Create PWA / PWA Builder и PWA-макрос), вшивание во флоу и кампанию, push с PWA (подписка + follow-up), разделение Android / iOS / Desktop.
doc_type: how-to
builds: [erp]
related: [mechanics-pwa, destinations, push-notifications, conversion-ai-testing, campaigns, domains, landings, user-fields, placeholders, sdk, flow-model, conversion-model, notifications-flow, glossary]
language: ru
updated: 2026-08-11
---

# Как создать и настроить AIO PWA (внутреннюю)

Как собрать **внутреннюю** PWA-шку (которая создаётся и хостится в самом AIO), вшить её в флоу/кампанию и включить push. Концепт-уровень (что такое PWA, AIO внутренние vs 3rd party, конверсии PWA Install/Open PWA) — в [mechanics/pwa.md](../mechanics/pwa.md), его здесь **не переобъясняем**.

> **Внутренняя PWA ≠ 3rd party PWA.** Этот файл — про PWA, собранную в AIO. Если PWA лежит у стороннего сервиса и AIO работает трекером — см. [how-to/destinations.md](destinations.md) → «PWA-Destination через 3-сервис» и [mechanics/pwa.md](../mechanics/pwa.md).

---

## TL;DR

- **PWA в AIO = обычный лэнд** + manifest для Chrome + Service Worker. Поэтому она создаётся **внутри `Content → Landings`**, а не в отдельном разделе.
- **Два пути собрать:** (A) `Create PWA` → встроенный **PWA Builder** (новая PWA с нуля); (B) **PWA-макрос** — превратить уже загруженный лэнд в PWA (указываешь ID кнопки Install + div-элементы).
- **Source для внутренней PWA — это FB** (обычная трекинговая ссылка), **не** `Third Party PWA Link Generator` (тот нужен только для сторонних сервисов).
- **Push с PWA** = в AIO PWA подписка на push работает **базово, по умолчанию** (отдельный `Push Subscribe Toggle` не нужен) + доставка follow-up через `Notifications Flow`. Полная сборка follow-up пушей — в [how-to/push-notifications.md](push-notifications.md).
- **iOS:** на уровне флоу есть готовое разделение **Android / iOS / Desktop**, но задокументированная install + web-push механика AIO рассчитана на **Android + Chrome**. Подробности iOS — см. раздел «iOS / Android / Desktop» (есть нюанс, см. ниже).

---

## Когда это нужно

- «Хочу свою PWA-шку внутри AIO, не через сторонний сервис».
- «Хочу превратить готовый лэнд в устанавливаемое приложение».
- «Хочу сравнить свою AIO-PWA со сторонней в одном сплите» (валидно только на одинаковой аудитории — см. [heuristics/conversion-ai-testing.md](../heuristics/conversion-ai-testing.md)).
- **PWA-функциональность доступна в любом тенанте и любой вертикали** — любой лэнд можно превратить в PWA: устанавливаемое веб-приложение (иконка на домашнем экране, полноэкранный режим, пуши) без публикации в сторе.

## Что должно быть готово до начала

- Тенант с доступом к `Content → Landings` и `Content → Macros`.
- Подключённый Source (FB) и домен под кампанию — как для обычного лэнда (см. [how-to/campaigns.md](campaigns.md), [how-to/domains.md](domains.md)).
- Если планируешь push — заранее посмотри [how-to/push-notifications.md](push-notifications.md) (Push Template → Notifications Flow → Push Distribution).
- Понимание, что внутренняя PWA — это **лэнд**: в флоу она ставится как обычный лэндинг на шаг **`Content`**, а не отдельным «PWA»-шагом или Destination-ом (Destination-ом PWA бывает только у 3rd party). После PWA визит идёт дальше по флоу — на оффер. См. [mechanics/pwa.md](../mechanics/pwa.md).
- Нужные типы конверсий заведены в тенанте: `Install` / `Open PWA` (их фиксирует сама PWA) и `Push Subscribe`. Если их нет — создать в `Settings → Conversion Types` (могут отсутствовать, если тенант не под PWA-шаблон). *(Конверсии воронки рекламодателя — `Registration`, `Purchase`, повторное касание и т.п. — приходят постбэком от рекламодателя, это **не** PWA-конверсии.)*

---

## Путь A. Create PWA / PWA Builder (новая PWA с нуля)

Когда: нужна PWA-шка, собранная в AIO «с нуля», без своего готового лэнда.

1. Open `Content → Landings → +Landing`.
2. Выбери экшен **`Create a PWA`** (опция в модалке `Add Landing`). Открывается встроенный **PWA Builder**. PWA создаётся именно внутри Landings, потому что **PWA-шка — это лэнд**.
3. Настрой элементы PWA в Builder-е (три блока: `PWA Settings`, `Design Settings`, `Comments`; + опц. `Content Splits`, см. ниже). Так как это лэнд, к нему применимы все возможности лэндов: можно завести **Content Splits** на любой элемент (название, описание, иконка, рейтинг, комментарии, дизайн) — тип `Text` / `Image` / `HTML`. См. [how-to/landings.md](landings.md) → Content Splits.
4. Сохрани. Дальше PWA используется как **обычный лэнд** на шаге **`Content`** Campaign Flow (см. «Вшить PWA во флоу»).

### Что настраивается в PWA Builder — три блока настроек

Builder делится на три блока: `PWA Settings`, `Design Settings`, `Comments`. Любой их элемент можно тестировать через **Content Splits** (см. ниже).

Отдельно от этих трёх блоков в левой колонке Builder-а живёт секция **`User fields`** — значения полей визита, которые проставятся визиту, когда флоу отдаст ему эту PWA. Поля добавляются кнопкой `Add field`, часть их может прийти от `Lander Type` этой PWA. Это не элемент витрины и в Content Splits не участвует; как заполняется секция и что делают режимы родительских полей — [how-to/user-fields.md](user-fields.md).

#### PWA Settings — конфиг PWA и привязка конверсий

**`PWA Settings`** — как PWA выглядит внутри AIO + привязка конверсий. Поля: `PWA name`, `Lander type` (= PWA), `Countries`, `Description`, `Languages`, `Tags`, плюс **три отдельных поля выбора конверсии**:

- **`Conversion type on install`** — конверсия при клике по кнопке *Install* внутри PWA.
- **`Conversion type on open`** — конверсия при тапе по иконке PWA на домашнем экране смартфона.
- **`Conversion type on subscribe`** — конверсия при подписке на push.

Это те самые `Install` / `Open PWA` / `Push Subscribe`, что должны существовать в тенанте (см. «Вшить PWA во флоу» и `Settings → Conversion Types`).

#### Design Settings — витрина приложения

**`Design Settings`** — витрина приложения (имитация страницы магазина приложений), как PWA видит визит: `Language`, `App Name`, `App Avatar` (иконка), `Developer`, `Ratings` (звёзды), `Age`, `Installs` (число установок), `Images` (скриншоты), `App description`, `Tags for the description`, `Last Update`, `Number of Reviews`.

#### Comments — отзывы под приложением

**`Comments`** — отзывы под приложением: `Author Avatar`, `Author Name`, `Rating`, `Comment Date`, `Comment`, `Developer Answer`.

В поле `Comment Date` можно подставить **date-макросы** вместо фиксированной даты — тогда дата отзыва генерится автоматически относительно текущего дня показа: `{date}` — сегодня, `{date-1d}` — вчера (минус 1 день), `{date+3d}` — плюс 3 дня. Формат — `{date}` со сдвигом `±Nd` (UI-подсказка: «Use {date}, {date-1d}, {date+3d} for auto-generated dates»). Так отзывы всегда выглядят свежими, а не с зашитой прошлой датой.

### Как добавить Content Split в PWA Builder (A/B тест элементов PWA)

1. В Builder-е открой секцию `Content Splits` (справа) → **`+Split`**.
2. Выбери `text content` или `image content`. Поля добавляются кнопками **`+Add Text Field`** / **`+Add Image Field`**. Для image-сплита можно загрузить картинки или дать прямую ссылку через **`Enter picture URL`** на файл из Uploaded Files.
3. Включи стратегию оптимизации: вкладка **`Control` → `Conversions AI`**; в поле `Conversion` выбери конкретную конверсию или оставь **`Any Conversion by business value`**.
4. Скопируй placeholder сплита и вставь его в нужный элемент (например, `App name`). Сохрани — варианты начнут тестироваться и оптимизироваться AI.

Связывать элементы между сплитами можно через Color Groups (см. [how-to/landings.md](landings.md) → Content Splits).

## Путь B. PWA-макрос (превратить любой лэнд в PWA)

Любой уже загруженный лэнд превращается в устанавливаемое PWA-приложение без переверстки: включаешь в SDK фичу `pwa`, даёшь лэнду минимальный DOM-скелет (кнопка + контейнеры-состояния) и указываешь их id в конфиге. Дальше всё делает SDK — подключает manifest, регистрирует Service Worker, показывает prompt установки, подписывает на push и уводит визит на оффер.

Под капотом нужно ровно три вещи: **manifest** (для Chrome), **Service Worker** и **кнопка установки**. Service Worker (`/aio-static/aio-sw.js`) регистрирует сам SDK; manifest AIO отдаёт **на сервере под каждый визит** (SDK подключает его как `<link rel=manifest href=/api/v1/pwa/manifest/<visit_uuid>.json>`) по данным, которые ты задал в конфиге макроса (`pwa.manifest` — имя, иконки, цвета). «Магазинную» витрину (иконка, скриншоты, рейтинг, отзывы) при кастомной PWA ты **верстаешь прямо в HTML лэнда**.

Этим кастомная PWA и отличается от `PWA Builder` (Путь A) — отдельной фичи с интерфейсом, которая и витрину, и конфиг макроса собирает за тебя, настраивать вручную нечего. Здесь наоборот: берёшь готовый лэнд и **сам** добавляешь макрос, чтобы превратить его в PWA. Именно этот сценарий — про этот раздел.

### Что должно быть в HTML лэнда — DOM-скелет PWA

Фича `pwa` работает по элементам лэнда, которые находит по id (id настраиваемые — укажешь свои в конфиге). Нужны:

- **Кнопка установки** — любой элемент с id (дефолт `install`). SDK сам меняет её текст и состояние: подписи берутся из атрибутов `data-install` / `data-open` (напр. `<a id="install" data-install="Install" data-open="Open"></a>`), текущее состояние — из `data-state` (`loading` → `install` → `progress` → `open`).
- **Три контейнера-состояния** (дефолт id `pwa-install` / `pwa-preloader` / `pwa-inside`) — SDK показывает ровно один за раз:
  - `pwa-install` — видимый экран-витрина с кнопкой установки (когда установка доступна);
  - `pwa-preloader` — короткий лоадер перед экраном установки;
  - `pwa-inside` — лоадер, видимый когда лэнд открыт **уже установленной** PWA (пока идёт подписка на push и редирект на оффер).
- **Intent-модалка** (дефолт id `intent-modal`) со ссылкой класса `.aio-sdk-modal-redirect-url` — показывается, если лэнд открыт во **встроенном браузере** соцсети (in-app webview: FB / IG / TikTok и т.п.), где установка PWA технически невозможна: предлагает открыть страницу в Chrome.

Готового лэнда-примера «из коробки» под этот сценарий может не быть — тогда берётся эталонный PWA-скрипт и адаптируется под конкретный лэнд: меняются id кнопки и контейнеров под разметку лэнда.

### Как включить фичу `pwa` и задать конфиг

1. Убедись, что на лэнде есть DOM-скелет выше и загружен SDK (плейсхолдер `{{aio}}`).
2. Включи фичу и задай конфиг — либо отдельным макросом (`Content → Macros → +Macro`, `Global: false`), либо инлайн-скриптом на лэнде **после `{{aio}}`**:

```javascript
window.aioBus.push({ type: "config", config: {
  features: { pwa: true },
  pwa: {
    installButtonId: "install",            // id кнопки на твоём лэнде
    installPWAContainerId: "pwa-install",
    preloaderPWAContainerId: "pwa-preloader",
    insidePWAContainerId: "pwa-inside",
    intentModalId: "intent-modal",
    serviceWorkers: { pwa: "/aio-static/aio-sw.js" },
    conversions: { install: "<uuid>", open: "<uuid>", subscribe: "<uuid>" }
  }
}});
```

3. Если это отдельный макрос — вызови его на лэнде после `{{aio}}`: `{{aio:macros:<slug>}}`.
4. Заведи в тенанте нужные `Conversion Types` (`Install` / `Open PWA` / `Push Subscribe`) и подставь их UUID в `conversions`.

Отдельно приходят от AIO (в отдаче лэнда, руками не сочиняются) параметры `pwa.session.hess` — URL, куда SDK уводит визит **дальше по флоу** (на оффер) после установки/открытия, и `pwa.session.sess` — стартовый URL самой PWA (заново вводит визит в его текущий шаг). Это постоянные ссылки: визит в них опознаётся по сессии, поэтому они работают и после того, как исходная страница закрыта, — механика и плейсхолдеры `{{link:s:permanent}}` / `{{link:h:permanent}}` в [reference/placeholders.md](../reference/placeholders.md). Полный разбор конфига и эндпоинтов — [reference/sdk.md](../reference/sdk.md) §II.15.

### Что происходит на лэнде — три ветки поведения

SDK определяет окружение визита (Android / Chrome / webview / уже-в-PWA) и идёт по одной из веток:

- **Установка доступна** (Android + Chrome, не webview, не внутри PWA): короткий preloader → SDK ловит `beforeinstallprompt` → активирует кнопку установки → по клику показывает браузерный prompt «Установить» → анимация прогресса. На событие `appinstalled` фиксируется конверсия `Install`.
- **Встроенный браузер соцсети** (in-app webview на Android): установить PWA нельзя — SDK формирует Android `intent://`-ссылку и показывает intent-модалку, предлагая открыть страницу в Chrome (с fallback-URL, если приложение не открылось).
- **Уже внутри установленной PWA**: SDK фиксирует `Open PWA`, регистрирует Service Worker, запрашивает разрешение на уведомления, подписывает визит на push (если ещё не подписан) и уводит его на оффер (`hess`). `Install` при повторном открытии не задваивается — это `Single Conversion`.

Опционально в `<head>` лэнда кладут микро-сниппет, который ловит `beforeinstallprompt` в `window.__bip` ещё до загрузки SDK — тогда prompt, сработавший рано, не теряется.

### Push внутри PWA — тот же Service Worker

Подписка на push происходит **внутри установленной PWA** (не на обычном лэнде): тот же Service Worker, что отвечает за установку, регистрирует push-подписку. Когда визит соглашается на запрос браузера, SDK подписывает его через push-эндпоинты агента и фиксирует конверсию `Push Subscribe` (эндпоинты — [reference/sdk.md](../reference/sdk.md) §II.15). Отдельный `Push Subscribe Toggle` для AIO-PWA не нужен. Дальше — доставка follow-up пушей через `Notifications Flow` (см. «Push с PWA» ниже и [how-to/push-notifications.md](push-notifications.md)).

---

## Вшить PWA во флоу и кампанию

Внутренняя PWA — это лэнд, поэтому в Campaign Flow она ставится как **обычный лэндинг, на шаг `Content`** (не отдельным «PWA»-шагом и не как Destination — Destination-ом PWA бывает только у 3rd party). Логика флоу: `Source (FB) → Filter → [Domain Change] → Content (PWA) → Offer`. На шаге `Content` выбираешь свою PWA как лэнд (можно в сплите с другими лэндами/PWA).

### Есть ли готовые флоу-шаблоны под PWA в тенанте?

Преднастроенные PWA-флоу и шаблоны (включая **Advanced Flow** с разделением по платформам) есть **только если тенант создавался под соответствующий шаблон**. Если их в тенанте нет — собери флоу сам в `Tracker → Flows`: PWA подставляется как лэнд на шаг `Content` внутри обычной воронки. В PWA-шаблоне на месте PWA можно оставить PWA-лэнд либо поставить лэнд/скипы. *(Имена конкретных шаблонов бывают тенант-специфичны — ориентируйся на структуру, не на имя.)* Полный разбор шаблонов, нод и платформенного разделения — в [models/flow-model.md](../models/flow-model.md) (раздел «Advanced Flow и APK-шаблоны»).

### Source — это FB, не Link Generator

Для **внутренней** PWA ссылка генерируется как для обычной кампании: Source = **FB**, обычная трекинговая ссылка AIO. `Third Party PWA Link Generator` здесь **не нужен** — он только для возврата визита из **стороннего** PWA-сервиса (см. [how-to/destinations.md](destinations.md)).

### Domain Change — зачем он PWA

В Advanced Flow после фильтра обычно стоит **Domain Change**, чтобы PWA-шка открывалась под **другим** доменом, не тем, на котором крутится FB-трафик — это разносит «рекламный» домен и домен, под которым живёт PWA. Общая механика и мотивация Domain Change (замена домена без переподключения FB) — в [how-to/domains.md](domains.md) и [models/flow-model.md](../models/flow-model.md).

### Conversion Spawn на шаге Offer (важно: внутренняя vs 3rd party)

На шаге Offer есть тоггл **Conversion Spawn** (Pushed conversion spawn) — «когда визит попал в этот шаг, автоматически заспавнить конверсию».

- **Сторонние PWA-сервисы** не умеют слать постбэк на факт установки, поэтому `Install` спавнят при переходе визита в Offer («дошёл до оффера → значит установил и открыл»).
- **Внутренняя AIO-PWA** спавнит `Install` сама — **в момент клика «Установить»**. Поэтому для аиошной PWA спавн `Install` на уровне Offer **убирают** (иначе задвоение).

Соответственно `Install` у внутренней PWA = «нажал Установить», у 3rd party = «установил и открыл». Подробнее про конверсии PWA (`Install` / `Open PWA`) — в [mechanics/pwa.md](../mechanics/pwa.md). *(Конверсии воронки рекламодателя — `Registration`, `Purchase`, повторное касание и т.п. — приходят постбэком от рекламодателя; это **не** PWA-конверсии, см. [models/conversion-model.md](../models/conversion-model.md).)*

Типы `Install` / `Open PWA` могут **отсутствовать** в тенанте (заводятся под PWA-шаблон). Если их нет — создай в `Settings → Conversion Types`, иначе спавнить и считать будет нечего.

---

## Push с PWA (как пушить)

Push с PWA — это две независимые части: **подписка** (происходит на лэнде/PWA) и **доставка** (follow-up пуши через Notifications Flow). Полная пошаговая сборка доставки уже описана — см. [how-to/push-notifications.md](push-notifications.md). Ниже — то, что специфично для PWA-лэнда.

### Как включается подписка на push в AIO PWA

В **AIO PWA подписка на push работает базово, по умолчанию** — отдельно включать ничего не нужно, **`Push Subscribe Toggle` для этого не требуется**. Тот же Service Worker, что отвечает за установку, обрабатывает и пуши (фича `features.pwa`, технический разбор — [reference/sdk.md](../reference/sdk.md) §II.15 «`pwa` — install + web-push + Android-intent»). Когда визит соглашается на запрос браузера, AIO автоматически фиксирует конверсию **`Push Subscribe`** из JS-кода PWA.

Подписаться визит может **только на странице** (на PWA/лэнде); после ухода со страницы — нельзя.

> **Конверсия `Push Subscribe` должна существовать в тенанте**, иначе подписки не считаются. Нет — заведи в `Settings → Conversion Types` (см. [models/conversion-model.md](../models/conversion-model.md)).

> **Если Service Worker подкладывается вручную** (не через Builder/макрос, а отдельным файлом для обычного лэнда): он должен лежать **в корне домена**, иначе подписки не регистрируются. Исторический способ — sub-page с путём `push-service-worker/push-client-worker.js`. См. *Пуш / PWA не работает*.

### Как настроить доставку follow-up пушей с PWA

Дальше — стандартный путь, **не дублируем**: `Push Template` → `Notifications Flow` (ноды `Wait` / `Check Conversion` / `Push` / `Repeat`, `Wait` минимум 60 сек) → `Push Distribution` (Remarketing Content) → подключить `Notifications Flow node` в Campaign Flow (обычно после шага с PWA). Вся процедура — в [how-to/push-notifications.md](push-notifications.md); концепт — в [mechanics/notifications-flow.md](../mechanics/notifications-flow.md).

### Ограничения и особенности пушей с PWA: домен, State, Chrome, аналитика

- **Подписка привязана к домену.** Визит подписывается от лица текущего домена, и пуши шлются ему с того же домена. Подписать на одном домене, а слать с другого — **технически невозможно** (ограничение браузеров).
- **Клик по пушу возвращает визита в его текущий State** (не в начало флоу), на тот же домен → он попадает в ту же кампанию. Если визит уже ушёл в оффер — по клику откроется оффер. Поэтому при re-open установленной PWA показывается последний State.
- **Канал push — Chrome.** Manifest у PWA — «для Chrome»; задокументированная подписка/доставка — браузерный (Chrome) web-push.
- **Бан домена обычно не блокирует отправку** пушей подписавшимся, но при клике браузер может показать warning-страницу про небезопасный/фишинговый домен.
- **Аналитика по пушам** сейчас — только логи в `Marketing → Messages`. CTR / open-rate / конверсия после пуша не трекаются.
- **Дожать тех, кто не нажал «дальше»:** ноду `Notifications Flow` ставят **до** лэнда и задают большой стартовый `Wait` (≈ среднее время на лэнде).

---

## iOS / Android / Desktop — есть ли iOS-версия?

Коротко: **на уровне флоу iOS обрабатывается** (есть готовое разделение Android / iOS / Desktop), но **install + web-push механика AIO задокументирована под Android + Chrome**; отдельного iOS-сценария установки/пушей нет.

### Что по iOS / Android / Desktop точно есть в системе

- **Advanced Flow разделяет трафик по платформам.** После фильтра идёт Domain Change, затем: если `Android` → своя ветка с PWA; иначе если `iOS` → отдельная iOS-ветка; всё, что не Android и не iOS (десктоп и пр.) — по дефолту сразу в оффер. Это готовый шаблон (см. [models/flow-model.md](../models/flow-model.md)).
- **Android / iOS / Windows** доступны как UA/OS-фильтры фильтра (быстрые селекторы устройств) и как групперы аналитики (`ua_os_name`).
- **Установка PWA заточена под Android + Chrome.** Детали поведения по платформам/браузерам (включая выход из in-app webview на Android) — [reference/sdk.md](../reference/sdk.md) §II.15 «`pwa` — install + web-push + Android-intent».

### Что по iOS-установке и iOS-push гарантировать нельзя

- Установка PWA и web-push «как на Android+Chrome» на **iOS / Safari** не гарантируются: в AIO iOS фигурирует как ветка device-split во флоу и как UA-селектор, а не как поддерживаемый install/push-сценарий.
- На практике iOS-ветку часто ведут на **отдельный сервис** (например, iOS One-Link), а не на ту же устанавливаемую PWA. По iOS «запускаться нужно чуть по-другому».

## Как проверить, что получилось

1. **PWA создалась как лэнд** — она видна в `Content → Landings` и доступна для выбора на шаге `Content` флоу (в сплите фигурирует как обычный лэнд-вариант).
2. **Установка работает** — на Android+Chrome по клику на кнопку появляется prompt «Установить»; после установки в визитах появляется конверсия `Install`.
3. **Подписка работает** — после согласия на push появляется конверсия `Push Subscribe`; в карточке визита видно подписку.
4. **Пуши уходят** — в `Marketing → Messages` видно отправленные сообщения (Requested → Processing → Done).
5. **Сессии PWA пишутся** — как для любого лэнда доступны Session Visit / Heatmaps (видно, как юзер скроллил, нажал Install).

## Типичные ошибки при настройке PWA

- **Service Worker не в корне домена** → подписки не регистрируются. Клади `push-client-worker.js` в корень (или через sub-page `push-service-worker/push-client-worker.js`), не на CDN как обычный файл.
- **Нужных Conversion Types нет в тенанте** (`Install` / `Open PWA` / `Push Subscribe`) → установки, открытия и подписки не считаются. Заведи их в `Settings → Conversion Types`.
- **Для внутренней PWA оставили Conversion Spawn `Install` на шаге Offer** → задвоение install. Спавн на оффере нужен только сторонним сервисам; внутренняя PWA трекает install сама в момент клика.
- **Взяли `Third Party PWA Link Generator` для внутренней PWA** — он только для возврата визита из стороннего сервиса. Внутренняя PWA запускается обычной FB-ссылкой.
- **Рассчитывали на iOS-install/iOS-push «как на Android»** — на iOS такой сценарий не гарантируется; iOS-ветка часто идёт через отдельный сервис.
- **Ждут CTR/open-rate пушей** — сейчас только логи в `Marketing → Messages`.

## Смежные темы

- [mechanics/pwa.md](../mechanics/pwa.md) — концепт PWA: AIO внутренние vs 3rd party, конверсии PWA (Install / Open PWA), PWA = Destination, PWA State / Offer State.
- [how-to/destinations.md](destinations.md) — PWA-Destination через 3-сервис и через PWA Builder, By Advertiser, капы.
- [how-to/push-notifications.md](push-notifications.md) — полная сборка follow-up пушей (Push Template → Notifications Flow → Push Distribution), Telegram.
- [mechanics/notifications-flow.md](../mechanics/notifications-flow.md) — концепт push-флоу, AIO Push Subscribe, типы пушей, мультиканальность.
- *Пуш / PWA не работает* — диагностика «push не приходит» / «превратить лэнд в PWA».
- [reference/sdk.md](../reference/sdk.md) — §II.15 «`pwa` — install + web-push + Android-intent» и «Как превратить лэнд в PWA через макрос».
- [models/flow-model.md](../models/flow-model.md) — «Advanced Flow и APK-шаблоны», разделение Android / iOS / Desktop, Conversion Spawn.
- [how-to/landings.md](landings.md) — загрузка лэндов, Content Splits, sub-pages, Sessions.
- [reference/glossary.md](../reference/glossary.md) — `Create PWA / PWA Builder`, `PWA-макрос`, `Push Subscribe`, `Install`, `Open PWA`.
