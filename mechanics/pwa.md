---
id: mechanics-pwa
title: PWA в AIO
description: Что такое PWA в AIO — определение, AIO внутренние vs 3rd party, PWA Builder и раздел Content → PWA (в MTK — страница PWA платного модуля PWA + Push), PWA Skin как заготовка витрины из библиотеки AIO, Service Worker, конверсии Install / Open PWA / Push Subscribe, почему инсталлом считается только настоящая установка и инсталлов закономерно меньше, чем регистраций, как визит с PWA попадает в аудиторию remarketing-кампании, 3rd Party PWA Link Generator.
doc_type: mechanic
builds: [erp, mtk]
related: [notifications-flow, custom-pwa, how-to-pwa, metric, conversion-model, remarketing-campaigns, sdk, visit-lifecycle, destinations, glossary]
language: ru
updated: 2026-10-07
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
| Создаётся через | `Content → PWA` → кнопка `+ PWA` (PWA Builder или `Custom PWA`) | Не создаётся в AIO |
| Роль AIO | Полный контроль | Трекер: трекает Install в момент возврата визита |
| Можно ли сплитовать | Да | Да |
| Можно ли смешивать в одном Flow | Да | Да |
| Content Splits на элементах | Да | Нет |
| Запись сессий | Да (только ERP) | Нет |
| Push-рассылка remarketing-кампанией | Да — нативный web-push AIO | Нативный push AIO не доходит: подписка живёт у внешнего PWA-сервиса |

Исключение — ZM Apps: его подписчикам рассылку отправляют через Sender provider `ZM Apps Push` ([mechanics/notifications-flow.md](notifications-flow.md)).

## Как создать PWA — PWA Builder, Custom PWA или макрос

### Создание через PWA Builder

PWA живёт в **своём разделе `Content → PWA`** — отдельном пункте меню рядом с `Content → Landings`. Кнопка **`+ PWA`** предлагает выбрать, как собирать: `PWA landing` — встроенный **PWA Builder**, `Custom PWA` — свой лэнд из ZIP внутри готового PWA-шелла ([how-to/custom-pwa.md](../how-to/custom-pwa.md)); правка готовой PWA — экшен `Manage PWA` (у `Custom PWA` — `Manage Custom PWA`) в её строке. В `Content → Landings` созданная PWA не показывается: та страница показывает только лэнды режима `Editor`. Технически PWA всё равно остаётся лэндом — во флоу она выбирается на шаге `Content` наравне с обычными лэндами. Процедура — [how-to/pwa.md](../how-to/pwa.md).

В ERP раздел входит в базовый набор. В MTK PWA доступны только с платным модулем `PWA + Push`: страница `PWA` внутри одноимённой вкладки верхнего меню (`/app/mtk/pwa/landers`) — та же кнопка `+ PWA` с выбором PWA Builder или `Custom PWA`, правка по клику на строку. Без модуля вкладки `PWA` в навигации MTK нет, а таблица лэндов там жёстко ограничена режимом `Editor` — PWA-строки в её выборку не попадают вовсе. Как включить модуль — *Билды, тарифы, триал и статусы тенанта*, что ещё он открывает — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

### Создание через PWA-макрос (превращение лэнда в PWA)

Альтернативный путь — макрос в `Content → Macros`, который превращает обычный лэнд в PWA. В настройках указываются:

- **ID кнопки-Install** (на лэнде).
- **div-элементы** (что попадает в PWA).

Под капотом макрос подкладывает manifest + Service Worker к лэнду.

## Какие Conversion Types фиксирует PWA — Install, Open PWA, Push Subscribe

Сама PWA (через SDK) порождает **три** конверсии — `Install`, `Open PWA` и `Push Subscribe`. Какой `Conversion Type` спавнится на каждое событие, для PWA из Builder-а и `Custom PWA` задаётся на типе лэнда (`Lander Type` с `Is PWA?`), для PWA-макроса — в его конфиге ([how-to/pwa.md](../how-to/pwa.md)):

| Conversion Type | Когда фиксируется (AIO внутренний PWA) | Когда фиксируется (3rd party PWA) |
|---|---|---|
| **Install** | Android + Chrome: когда визит **подтвердил браузерный диалог установки** (событие `appinstalled`) — согласие есть, приложение ещё не открывали; отказ в диалоге конверсию не создаёт. iOS: при **первом запуске** установленного приложения с домашнего экрана — само добавление на домашний экран AIO не видит | Только когда юзер вернулся в AIO («установил + открыл»). 3rd party PWA не умеют слать postback на факт установки |
| **Open PWA** | Юзер реально открыл cached PWA после установки | (Не применимо отдельно, см. Install) |
| **Push Subscribe** | При **первой успешной подписке** на push внутри установленной PWA — один раз; на последующих открытиях не повторяется | (Не применимо) |

Клики по кнопке установки и конверсии `Install` — разные числа: сам клик по кнопке конверсию не создаёт, на Android + Chrome её даёт только подтверждение браузерного диалога установки. Настройка push внутри PWA — [how-to/pwa.md](../how-to/pwa.md).

Процент реально открывших PWA после установки считает метрика тенанта: в iGaming-шаблоне (ERP и MTK) она заведена из коробки как `PWA Open Rate %` = `PWA Opens / Installs` — рядом с `Install Rate %` = `Installs / Visits` и `Push Subscription Rate %` = `Push Subscriptions / PWA Opens` (счётчики `Installs`, `PWA Opens`, `Push Subscriptions` — метрики типа `Conversions By Type Count` по этим трём типам). В шаблонах других вертикалей PWA-метрик нет — их собирают своей `Computable metric` ([models/metric.md](../models/metric.md)).

### Чем размечен уход визита с PWA-лэнда на оффер

Полем визита `pwa_hess_reason` (колонка `PWA Hess Reason`): по нему видно, ушёл визит на оффер из запущенного установленного приложения или мимо установки. Этим и объясняется расхождение «до оффера дошло больше визитов, чем зафиксировано `Install`». Поле хранит **последний** путь — оно перезаписывается на каждом уходе визита на оффер. Значения и как их читать — [how-to/pwa.md](../how-to/pwa.md).

### Почему инсталлов меньше, чем регистраций — что AIO считает установкой

Инсталлом AIO считает **только настоящую установку приложения**, поэтому конверсий `Install` закономерно меньше, чем визитов, дошедших до оффера, и вполне может оказаться меньше, чем регистраций.

Конверсию `Install` создаёт подтверждённая установка: на Android + Chrome — подтверждённый браузерный диалог, на iOS — первый запуск приложения с домашнего экрана. Визит, который установить не смог (iOS без ручного добавления, десктоп, другой браузер, встроенный браузер соцсети, окно инкогнито — там Chrome установку не предлагает) или не захотел, уходит на оффер **фолбэком** — и это не инсталл: такой переход размечается не конверсией, а полем `PWA Hess Reason`.

Отсюда нормальная картина «инсталлов 3, регистраций 10»: регистрация приходит постбэком от рекламодателя за визит, который до оффера дошёл любым путём, а `Install` — только за того, кто реально поставил приложение. Потерей данных это не является — счёт установок намеренно строгий.

Встроенной (системной) метрики «install rate» в AIO нет — это метрика тенанта поверх счётчиков: в iGaming-шаблоне она уже заведена (`Install Rate %` = `Installs / Visits`), в шаблонах других вертикалей её собирают своей `Computable metric` ([models/metric.md](../models/metric.md)).

### Подписочная модель — Install / Trial / Subscription

Под подписочную модель (mobile-apps) `Install` — лишь одно из трёх событий: нужны **три** отдельных `Conversion Type` (`Install` / `Trial` / `Subscription`) и три FB CAPI трекера. Разбор — [models/conversion-model.md](../models/conversion-model.md).

## Как визит с PWA попадает в рассылку ремаркетинга

Через привязку remarketing-кампании к `Landing Type` этой PWA: визит, прошедший PWA, помечается кампанией **на проходе** и дальше получает её рассылку. Это штатный способ добивать установивших пушами — и в ERP, и в MTK: сбор аудитории серверный.

Гейта «подписан на push» тут нет — в аудиторию попадает любой нетрэшевый визит, прошедший объект с привязкой; дойдёт ли до него сам push, решается уже на отправке. Пометка накопительная: колонка визита `RMK Audience` показывает историю попаданий, а не текущий состав аудитории. Как устроена рассылка — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md); где у PWA ставится привязка — [how-to/pwa.md](../how-to/pwa.md).

В MTK с модулем `PWA + Push` секция привязки есть в формах трафиковой кампании, источника, дестинейшена, лендинга и PWA, но форм типа лендинга нет — поэтому визит со всех PWA сразу попадает в аудиторию привязкой remarketing-кампаний к типу лендинга `PWA`: новому тенанту вертикали iGaming она приходит из шаблона вместе с готовыми кампаниями (в шаблонах MTK Finance / Sweeps набора рассылки и типа `PWA` нет), тенанту без набора или созданному раньше, чем шаблон стал включать рассылку, её ставит команда AIO ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)).

## Как устроен Third Party PWA Link Generator — зачем нужен и что генерирует

Отдельный Source в AIO, генерирующий универсальные ссылки для подстановки внутрь PWA-шек сторонних сервисов. Особенности:

- Ссылки **без идентификаторов кампании/юзера**.
- Их цель — только вернуть визит в AIO в тот же флоу.
- Возврат визита в нужный Flow обеспечивает параметр `hess` в PWA-ссылке (включён в ссылку, которую генерирует `3rd Party PWA Link Generator`).
- Click ID каждого PWA-сервиса хранится в поле визита `{{aio.visit.fields.pwa_click_id}}` — этот плейсхолдер подставляется в постбэк-URL трекера для PWA-конверсий. В iGaming-шаблоне так собран трекер-пример `Example - 3rd Party PWA Postback` (тип `HTTP Get`, на конверсию `Registration`, приезжает выключенным): URL вида `https://some-3rd-party?…={{aio.visit.fields.pwa_click_id}}` (адрес-заглушка под подстановку реального постбэка партнёрки).

### Что уже заведено в источнике `3rd Party PWA Link Generator` — ссылка на каждую партнёрку

Источник `3rd Party PWA Link Generator` есть из коробки только в iGaming-шаблоне (ERP и MTK); в шаблонах других вертикалей его нет. Внутри — не одна ссылка, а **по ссылке на партнёрскую PWA-платформу** (`<Партнёрка> Link inside PWA`): PWA.Group, SkakApp, AppsHeroes, WWApps, iRent Market, ZMApps, PWA.Partners, LiteApp, TSL Apps. Каждая ссылка мапит макросы партнёрки на два параметра AIO — `hess` (служебный параметр возврата визита, не поле) и поле `pwa_click_id`; у AppsHeroes дополнительно `app_id`. У ZMApps в ссылке только `pwa_click_id` — параметр `hess` эта партнёрка прокидывает сама. Тултип каждой ссылки: `DO NOT use Encrypted. Insert inside PWA as tracking link` — партнёрке отдаётся обычный URL с видимыми плейсхолдерами, а не короткая ссылка.

Поля под это — группа `PWA` того же шаблона: `pwa_click_id` («Unique ID from PWA Service»), `app_id` («Unique ID of Application»), `free_param_1…4`. В тенанте другой вертикали источник и поле `pwa_click_id` заводятся вручную.

### Какие флоу под PWA заведены в тенанте — три флоу iGaming-шаблона

Готовые PWA-флоу есть только в iGaming-шаблоне (ERP и MTK) — выбираются на создании кампании в `Select Flow`, у каждого есть кампания-пример с меткой `AIO EXAMPLE`. В шаблонах Finance / Nutra / Sweeps PWA-флоу нет.

- **`PWA / OneLink`** — шаг `PWA / AIO PWA / Onelink` → `Offer`. Кампания-пример `Default Flow - PWA Campaign Example`: на PWA-шаге в одном сплите стоят лэнд `AIO PWA Example w/ Splits` и дестинейшены партнёрских PWA — `Example AppsHeroes PWA`, `Example ZMApps PWA`.
- **`PWA / OneLink w/ Landing`** — то же, но после PWA-шага стоят `Spawn Install` (шаг `Spawn Conversion`) и прилэнд `Landing` **до** оффера. Кампания-пример `PWA w/ Landing Flow - Campaign Example`.
- **`Advanced - PWA / OneLink`** — разводка по операционной системе: Android и iOS получают PWA, остальной трафик уходит на `Fallback Offer`. Кампания-пример `Advanced Flow - Campaign Example`; разбор веток — [how-to/pwa.md](../how-to/pwa.md).

### Как запустить 3rd party PWA — две ссылки

Для запуска генерируются **две** ссылки (`Link Generator`):

- **PWA-ссылка** — Source = `3rd Party PWA Link Generator` (в iGaming-шаблоне он заведён со ссылкой под каждую партнёрку — см. «Что уже заведено в источнике `3rd Party PWA Link Generator`»); вставляется внутрь PWA-сервиса как его offer-ссылка.
- **Tracking-ссылка** — под реальный Source (для FB — `FB w/ CAPI`, этот источник есть во всех шаблонах тенанта); вставляется в рекламный источник.

### Как трекается Install для 3rd party PWA — Spawn Conversion, не постбэк

Install-конверсия для 3rd party PWA **не приходит постбэком**. Она спавнится внутри флоу, когда визит редиректится с PWA-сервиса на offer-Destination. В iGaming-шаблоне так собран `PWA / OneLink w/ Landing`: шаг `Spawn Install` (тип `Spawn Conversion`, конверсия `Install`) стоит сразу после PWA-шага. В `PWA / OneLink` и `Advanced - PWA / OneLink` шага спавна нет и спавн конверсии на шаге `Offer` не задан — там `Install` создаёт только собственная AIO PWA через SDK, а визит, вернувшийся из партнёрской PWA, инсталлом не засчитывается, пока не добавлен шаг `Spawn Conversion` или спавн `Install` на шаге `Offer` ([how-to/pwa.md](../how-to/pwa.md)).

Настраивается шагом **`Spawn Conversion`** во Flow (`Tracker → Flows`).

### Какие параметры передаёт AIO в 3rd party PWA — только Sub (Visit Session)

3rd party PWA передаём **только** ID сессии визита в sub (нужен для возврата визита тем же визитом). FB-параметры в AIO пишутся **до** пвашки.

Хвостик параметров PWA-сервиса (FB Pixel, FB Click) **уже забит** в `Destination` — в саму PWA-шку добавлять только домен (AIO передаёт только `AIO Visit Session` в Sub). Один и тот же домен можно использовать **и** для FB, **и** внутри PWA-шек — они не привязаны.

## Как AIO Visit Session (Sub 1) обеспечивает возврат из PWA

`AIO Visit Session` — уникальный идентификатор визита, передаваемый AIO в PWA через один из доступных сабов. Нужен, чтобы при возврате из PWA-сервиса визит **«продолжился» в той же сессии**.

## Что происходит при Re-open PWA — возврат в последний State

Если визит уже установил PWA и открывает её снова, он возвращается в **последний State** (а не в начало флоу), и на каждое открытие идёт `Open PWA`.

`Push Subscribe` на повторном открытии не порождается: SDK видит, что визит уже подписан, и не повторяет ни подписку, ни конверсию. Если разрешение на уведомления не выдано, подписки и конверсии нет вовсе — она может возникнуть позже, на том открытии, где визит наконец согласился.

Повторный `Install` отсекается настройкой самого типа конверсии: в iGaming-шаблоне (ERP и MTK) у `Install`, как и у `Open PWA` и `Push Subscribe`, выставлен `Uniqueness Strategy = Single Conversion`, и вторая конверсия того же типа на визите не создаётся (механика — [models/conversion-model.md](../models/conversion-model.md)). Если тип заведён как `Multiple Infinity`, каждое открытие установленной PWA даёт новую конверсию `Install` — отсюда симптом «у одного визита несколько инсталлов». В сетевых логах лэнда install-запрос при открытии виден в любом случае (см. [reference/sdk.md](../reference/sdk.md)).

## FB CAPI vs PWA — что предпочесть для качества лида

FB CAPI через AIO лучше пвашки; FBC/FBP потеря не критична. То есть если стоит выбор между настроенным CAPI через AIO и PWA-сценарием — для качества лида CAPI предпочтительнее.

## PWA как Destination, не Source — в чём парадигма

В флоу PWA стоит **после** оффера/прилэнда как финальный шаг (или промежуточный перед реальной партнёркой), а не как точка входа трафика. Source — это всегда внешняя реклама (FB, Google и т.п.), а PWA — это куда лид доводится. Поэтому PWA = `Destination`, не `Source`.

## PWA State и Offer State — поля визита на Split-шаге

Поля визита, в которые автоматически пишется выбранный вариант на Split-шаге:

- **`PWA State`** (`pwa_state`) — для шага PWA: «PWA the visit went through - own AIO PWA or a 3rd-party one».
- **`Offer State`** (`offer_state`) — для шага оффера: «Offer the visit reached, whatever destination position it holds».

Используется для аналитики «какая PWA / какой оффер показались». Формат — `Variant`, группа `Tracking`. Оба поля заведены в iGaming-шаблоне (ERP и MTK), и в его PWA-флоу привязка уже стоит: шаг `PWA / AIO PWA / Onelink` пишет в `PWA State`, шаг `Offer` — в `Offer State`; на них же построены пресеты отчётов `Campaign → PWA`, `Campaign → Offer` и `PWA → Geo → Offer`. В шаблонах других вертикалей этих полей нет.

Включается через **Linked Field / Link to Field** на уровне Flow (любой вариант, выбранный на этом Split, пишется в указанное поле визита).

## Смежные темы

- [models/conversion-model.md](../models/conversion-model.md) — Conversion Types, Uniqueness Strategy, постбэки.
- [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md) — рассылка пушей по собранной аудитории визитов (дефолтный способ рассылки).
- [mechanics/notifications-flow.md](notifications-flow.md) — Push, Service Worker для push-уведомлений.
- [mechanics/visit-lifecycle.md](visit-lifecycle.md) — поля визита, Session continuation.
