---
id: mechanics-pwa
title: PWA в AIO
description: Что такое PWA в AIO — определение, AIO внутренние vs 3rd party, PWA Builder, Service Worker, Install/Open конверсии, 3rd Party PWA Link Generator.
doc_type: mechanic
builds: [erp, mtk]
related: [conversion-model, metric, notifications-flow, visit-lifecycle, how-to-pwa, destinations, glossary]
language: ru
updated: 2026-08-11
---

# PWA в AIO

PWA (Progressive Web App) в AIO — это **лэнд с manifest-ом и Service Worker-ом**, который можно установить на устройство и который умеет принимать push. AIO поддерживает два контура: внутренние PWA (создаются в тенанте) и 3rd party (внешние сервисы, AIO в роли трекера).

---

## Что такое PWA в AIO — определение

PWA-шка = обычный лэнд, у которого:

- В `<head>` есть **manifest** для Chrome (`<link rel="manifest" href="...">`).
- К нему привязан **Service Worker** (JS-скрипт, обрабатывающий install и push).
- **Один и тот же Service Worker** обрабатывает оба сценария — установку и доставку push-уведомлений.

PWA технически работает одинаково у всех команд. Разница — в дизайне и сценарии, не в технике. PWA-функциональность (любой лэнд → PWA через макрос) доступна в **любом** тенанте и любой вертикали.

## Чем AIO внутренние PWA отличаются от 3rd party

| | AIO внутренние PWA | 3rd party PWA |
|---|---|---|
| Где хостится | В тенанте AIO | Внешний сервис |
| Создаётся через | `Content → Landings → +Landing` (PWA Builder) | Не создаётся в AIO |
| Роль AIO | Полный контроль | Трекер: трекает Install в момент возврата визита |
| Можно ли сплитовать | Да | Да |
| Можно ли смешивать в одном Flow | Да | Да |
| Content Splits на элементах | Да | Нет |
| Запись сессий | Да | Нет |

## Как создать PWA — два пути: PWA Builder и макрос

### Создание через PWA Builder

Экшен `Create PWA` доступен под плюсиком в `Content → Landings`. Пвашка создаётся **внутри Landings**, т.к. PWA-шка — это лэнд. Открывает встроенный **PWA Builder**.

### Создание через PWA-макрос (превращение лэнда в PWA)

Альтернативный путь — макрос в `Content → Macros`, который превращает обычный лэнд в PWA. В настройках указываются:

- **ID кнопки-Install** (на лэнде).
- **div-элементы** (что попадает в PWA).

Под капотом макрос подкладывает manifest + Service Worker к лэнду.

## Какие Conversion Types фиксирует PWA — Install, Open PWA, Redep

Сама PWA (через SDK) порождает **две** конверсии:

| Conversion Type | Когда фиксируется (AIO внутренний PWA) | Когда фиксируется (3rd party PWA) |
|---|---|---|
| **Install** | В момент клика «Установить» (согласие, ещё не открыл) | Только когда юзер вернулся в AIO («установил + открыл»). 3rd party PWA не умеют слать postback на факт установки |
| **Open PWA** | Юзер реально открыл cached PWA после установки | (Не применимо отдельно, см. Install) |

Метрика `Open PWA / Install` — процент реально открывших PWA после установки.

> **`Redep` — не конверсия PWA.** Это повторная конверсия (повторное касание того же пользователя после первой оплаченной конверсии), приходит **постбэком от рекламодателя** и существует в воронке независимо от PWA. Заводится как обычный Conversion Type и трекается через постбэк-трекер (см. [models/conversion-model.md](../models/conversion-model.md)). Связанная метрика — `Redep Sum` (см. «Что такое Redep Sum и зачем трекается» ниже).

### Подписочная модель — Install / Trial / Subscription

Под подписочную модель (mobile-apps) `Install` — лишь одно из трёх событий: нужны **три** отдельных `Conversion Type` (`Install` / `Trial` / `Subscription`) и три FB CAPI трекера. Разбор — [models/conversion-model.md](../models/conversion-model.md).

## Что такое Redep Sum и зачем трекается

`Redep Sum` — метрика-сумма всех повторных конверсий (`Redep`), к механике PWA отношения не имеет. Считается **отдельно от Revenue** — механика исключения источника данных описана в [models/metric.md](../models/metric.md).

## Как устроен Third Party PWA Link Generator — зачем нужен и что генерирует

Отдельный Source в AIO, генерирующий универсальные ссылки для подстановки внутрь PWA-шек сторонних сервисов. Особенности:

- Ссылки **без идентификаторов кампании/юзера**.
- Их цель — только вернуть визит в AIO в тот же флоу.
- Возврат визита в нужный Flow обеспечивает параметр `hess` в PWA-ссылке (включён в ссылку, которую генерирует `3rd Party PWA Link Generator`).
- Click ID каждого PWA-сервиса хранится в поле визита `{{aio.visit.fields.pwa_click_id}}` — этот плейсхолдер подставляется в постбэк-URL трекера для PWA-конверсий.

### Как запустить 3rd party PWA — Flow-пресеты и две ссылки

В тенанте уже есть два преднастроенных Flow под 3rd party PWA, выбираются на создании кампании в `Select Flow`:

- **`PWA / OneLink Flow Default`** — дефолтный флоу `AIO → PWA → рекламодатель`.
- **`PWA / OneLink w/ Landing`** — то же, но с прилэндом **до** рекламодателя.

Для запуска генерируются **две** ссылки (`Link Generator`):

- **PWA-ссылка** — Source = `3rd Party PWA Link Generator`; вставляется внутрь PWA-сервиса как его offer-ссылка.
- **Tracking-ссылка** — под реальный Source (для FB — преднастроенный `FB w/ CAPI – DEFAULT`, уже включён в тенант); вставляется в рекламный источник.

### Как трекается Install для 3rd party PWA — Spawn Conversion, не постбэк

Install-конверсия для 3rd party PWA **не приходит постбэком**. Она спавнится внутри флоу, когда визит редиректится с PWA-сервиса на offer-Destination. Настраивается шагом **`Spawn Conversion`** во Flow (`Tracker → Flows`).

### Какие параметры передаёт AIO в 3rd party PWA — только Sub (Visit Session)

3rd party PWA передаём **только** ID сессии визита в sub (нужен для возврата визита тем же визитом). FB-параметры в AIO пишутся **до** пвашки.

Хвостик параметров PWA-сервиса (FB Pixel, FB Click) **уже забит** в `Destination` — в саму PWA-шку добавлять только домен (AIO передаёт только `AIO Visit Session` в Sub). Один и тот же домен можно использовать **и** для FB, **и** внутри PWA-шек — они не привязаны.

## Как AIO Visit Session (Sub 1) обеспечивает возврат из PWA

`AIO Visit Session` — уникальный идентификатор визита, передаваемый AIO в PWA через один из доступных сабов. Нужен, чтобы при возврате из PWA-сервиса визит **«продолжился» в той же сессии**.

## Что происходит при Re-open PWA — возврат в последний State

Если визит уже установил PWA и открывает её снова, он возвращается в **последний State** (а не в начало флоу). Конверсия `Push Subscribe` при этом порождается из JS. Повторное открытие **не дублит `Install`**: `Install` — `Single Conversion`, фиксируется один раз в момент установки, а на каждое открытие идёт `Open PWA` (техника фичи — на стороне AIO SDK).

## FB CAPI vs PWA — что предпочесть для качества лида

FB CAPI через AIO лучше пвашки; FBC/FBP потеря не критична. То есть если стоит выбор между настроенным CAPI через AIO и PWA-сценарием — для качества лида CAPI предпочтительнее.

## PWA как Destination, не Source — в чём парадигма

В флоу PWA стоит **после** оффера/прилэнда как финальный шаг (или промежуточный перед реальной партнёркой), а не как точка входа трафика. Source — это всегда внешняя реклама (FB, Google и т.п.), а PWA — это куда лид доводится. Поэтому PWA = `Destination`, не `Source`.

## PWA State и Offer State — поля визита на Split-шаге

Поля визита, в которые автоматически пишется выбранный вариант на Split-шаге:

- **`PWA State`** — для шага PWA (записывается, какая PWA-шка показалась).
- **`Offer State`** — для шага оффера (записывается, какой оффер показался).

Используется для аналитики «какая PWA / какой оффер показались». Формат — `variant`. Включается через **Linked Field / Link to Field** на уровне Flow (любой вариант, выбранный на этом Split, пишется в указанное поле визита).

## Смежные темы

- [models/conversion-model.md](../models/conversion-model.md) — Conversion Types, Uniqueness Strategy, постбэки.
- [mechanics/notifications-flow.md](notifications-flow.md) — Push, Service Worker для push-уведомлений.
- [mechanics/visit-lifecycle.md](visit-lifecycle.md) — поля визита, Session continuation.
