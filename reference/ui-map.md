---
id: ui-map
title: Карта интерфейса AIO (где что лежит)
description: Полная карта навигации app.aio.tech — топ-меню (Tracker/Content/Marketing/Finance/Tech/Automations/Meta/Analytics/Settings; Dashboard из меню убран, но остаётся стартовой страницей), подразделы каждого, меню профиля (Billing, смена тенанта, Manage tenant). Чтобы быстро находить нужный раздел.
doc_type: reference
builds: [erp, mtk]
related: [postback-generator, landings, analytics, landing, sdk, remarketing-campaigns, auto-rules, meta-ads, google-ads, events-exporter, live-pulse, session-analytics, registration, source-trackers, glossary, permissions-model, debug-with-logs, custom-fields, api, ui-common, profile-settings, placeholders]
language: ru
updated: 2026-09-11
---

# Карта интерфейса AIO (где что лежит)

Навигация рабочего кабинета `app.aio.tech`: топ-меню, подразделы каждого раздела и меню профиля. Нужна, чтобы под вопрос «а где найти X» сразу получить путь: «это в `Content → Macros`» / «это в `Settings → Forms`».

> Карта снята с **ERP-билда** (полная система). В билде MTK набор разделов отличается — различия отмечены по ходу карты. Видимость разделов зависит от **вашей роли и прав** внутри тенанта: часть разделов видна не всем пользователям.

---

## Где смотреть Dashboard и что на нём отображается — `/app/dashboard`

Dashboard — сводка по тенанту, раскладка кастомизируется под вас. **Пункта `Dashboard` в верхнем меню нет** — как и в билдах MTK и AFF. Страница осталась стартовой: корень кабинета ведёт на `/app/dashboard`, туда же попадают сразу после регистрации, и адрес открывается напрямую.

- **Верхняя панель** — ключевые метрики: визиты лэндов, лиды, конверсии, Revenue$, ROI% (набор метрик зависит от настройки тенанта).
- **Средняя панель** — инфографика по тем же показателям: **Performance-чарт** (динамика метрик из верхней панели за выбранный период, можно переключать отображаемую метрику) + боковой чарт с переключателем **Campaigns ↔ Sources**.
- **Map Overview** — зумируемая карта; ховер по стране показывает её метрики по локации. Справа — **Destinations-чарт** (перформанс по каждому Destination).
- Сверху страницы — **фильтры по периоду**; в левом верхнем углу — тумблер **auto-refresh** и кнопка ручного обновления.
- Слева сверху — **синее меню быстрого создания**: Landing / Campaign / Destination / Source из одного места (те же диалоги, что в `Content → Landings` и `Tracker → Campaigns/Destinations/Sources`).

## Где настраивается Tracker и какие подразделы доступны — `/app/tracker`

Tracker — ядро трекинга: кампании, источники, дестинейшны, флоу, визиты, конверсии.

- **Campaigns** `/app/tracker/campaigns` — кампании (дефолтный экран).
- **Sources** `/app/tracker/sources` — источники трафика.
- **Destinations** `/app/tracker/destinations` — точки назначения (рекламодатели).
- **Trackers** `/app/tracker/trackers` — постбэк-трекеры (AIO → Source). В MTK Trackers перенесены в `Settings → Trackers`, см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.
- **Visits** `/app/tracker/visits` — визиты.
- **Conversions** `/app/tracker/conversions` — конверсии. Кнопка в шапке таблицы открывает `Postback generator` — сборку ссылки входящего постбэка для рекламодателя ([how-to/postback-generator.md](../how-to/postback-generator.md)); есть и в ERP, и в MTK.
- **Trash** `/app/tracker/trash` — отсев (Trash-визиты, исключённые из аналитики).
- Брейкдаун-отчёты: **Countries**, **Day part**, **Devices**, **Days**, **OS**, **Browsers**, **Other**.

### Где редактировать Flows (цепочки State'ов) — `/app/tracker/flows`

Flows `/app/tracker/flows` — редактор флоу (цепочки State'ов), доступен только в ERP.

В MTK флоу выбираются как готовые пресеты в настройках кампании (поле Traffic flow) — редактора нет.

## Где находятся лэнды, макросы и файлы — Content `/app/landers-creatives`

Content — раздел для управления лэндами, креативами и переменными контента.

### Где находятся лэнды, PWA-шки и макросы — Landings, PWA и Macros

- **Landings** `/landers` — лэнды в режиме `Editor`. В MTK Landings доступны как отдельная вкладка верхнего меню (`/app/mtk/landers-creatives/landers`). Слева в шапке таблицы — переключатель типов лэндов (по умолчанию `All`, в меню `All types` и список типов): выбранный тип сужает список до лэндов этого типа и переводит колонки метрик на роль-группер `LP: <Тип>` вместо позиции `#N`; выбор запоминается в браузере. Разбор — [how-to/landings.md](../how-to/landings.md), роль-групперы — [how-to/analytics.md](../how-to/analytics.md).
- **PWA** `/pwa` — лэнды в режиме `PWA` (только ERP; пункт закрыт правом `content.landings.view`). Разделение по режиму лэнда навязано: на `Landings` видны только лэнды режима `Editor`, на `PWA` — только режима `PWA`, переключить или снять этот фильтр в интерфейсе нельзя ([models/landing.md](../models/landing.md)).
- **Macros** `/macros` — макросы (в т.ч. `AIO SDK Macros Collection`, см. [reference/sdk.md](sdk.md)). В MTK Macros перенесены в `Settings → Macros`, см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

### Где находятся креативы, Content Library и файлы — Creatives, Files, CDN

- **Creatives** `/creatives` — креативы (баннеры/картинки).
- **Content library** `/lander-placeholders` — Content Library (плейсхолдер-группы, переменные контента).
- **Uploaded files** `/files` — загруженные файлы.
- **CDN files** `/cdn-files` — файлы на CDN.

## Где собираются рассылки пушей — Marketing `/app/remarketing`

Marketing (в главном меню — `Marketing`) — ремаркетинг: рассылки по собранной аудитории визитов, флоу рассылок и шаблоны сообщений. По умолчанию раздел открывается на экране `Campaigns`. Только ERP.

- **Campaigns** `/app/remarketing/campaigns` — ремаркетинг-кампании: аудитория, `Launch trigger`, прогоны ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)).
- **Flows** `/app/remarketing/flows` — флоу рассылок; список сужен до флоу типа `Notifications`, в них живёт узел `Drip schedule`.
- **Template distributions** `/app/remarketing/message-template-distributions` — дистрибуции шаблонов сообщений (тип дистрибуции `Message Templates`); само дерево нод открывается на `/app/remarketing/message-template-distributions/tree`.
- **Messages** `/app/remarketing/messages` — отправленные сообщения.
- **Sender providers** `/app/remarketing/sender-providers` — провайдеры отправки.
- **Message templates** `/app/remarketing/message-templates` — отдельные записи-шаблоны сообщений; тексты ремаркетинг-рассылки лежат не здесь, а в нодах `Template distributions`.
- **Other** `/app/remarketing/other` — служебная страница: сюда открываются групперы `Remarketing`, у которых нет своей страницы (`Channel`, `Send Result`, время).

Пункт `Marketing` в главном меню показывается по правам **других** страниц раздела (`Messages`, `Message templates`, `Sender providers`, `Flows`) — роль, у которой есть только права на ремаркетинг-кампании, пункта не увидит, хотя страница `/app/remarketing/campaigns` ей открывается по прямой ссылке ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)).

## Где смотреть Revenue и Payout Distributions — Finance `/app/finance`

Finance — финансы кампаний: как считается выручка и выплаты по кампаниям/баерам.

- **Payout Settings** `/payout-settings` — правила выплат (Payout Distributions).
- **Revenue Settings** `/revenue-settings` — правила выручки (Revenue Distributions).
- **Distributions** `/distributions` — денежные дистрибуции Finance (те же деревья, что и в `Settings → Distributions`).
- Полный список всех типов Distributions (не только денежных) — в `Settings → Distributions`.

## Где управляются серверы, домены и инфраструктура — Tech `/app/tech`

Tech — инфраструктура: серверы, домены, прокси, провайдеры, чекеры, деплои.

- **Servers** `/servers` — серверы.
- **Domains** `/domains` — домены.
- **Proxies** `/proxies` — прокси.
- **DNS providers** `/dns-providers`, **Domain providers** `/domain-providers`, **Server providers** `/server-providers`, **Proxy providers** `/proxy-providers` — провайдеры.
- **Domain checkers** `/domain-checkers` — чекеры доменов (бан/репутация).
- **Deployments** `/deployments` — деплои.

## Где лежат автоправила над Meta — Automations `/app/automations`

Automations — раздел автоправил над сущностями Meta: движок сам считает метрики кампании, адсета или объявления и сам делает то, что делают руками экшенами `Start`, `Stop`, `Change budget`. Только ERP; в главном меню пункт стоит между `Tech` и `Analytics`. Полный список вердиктов правила и как повесить его на сущности — [how-to/auto-rules.md](../how-to/auto-rules.md).

- **Status** `/app/automations/status` — обзор движка по тенанту: KPI-полоса, счётчик `Pending approvals`, список ассайнов с временем прогонов и кнопкой `Live check`. Дефолтный экран раздела.
- **Assignments** `/app/automations/assignments` — ассайны: какой шаблон правил на каком скоупе Meta-сущностей работает.
- **Rule Templates** `/app/automations/auto-rules` — таблица шаблонов правил.
- **History** `/app/automations/actions` — журнал вердиктов и очередь апрувов; заголовок страницы — `Auto Rules Log`, в подменю она отделена разделителем от трёх настроечных страниц.

### Что открывается по клику на `Automations` и почему пункта может не быть в меню

Клик по пункту `Automations` открывает последнюю посещённую страницу раздела, а при первом заходе — первую доступную по правам, начиная со `Status`. Сам пункт показывается по любому из трёх прав: `automation.templates.view`, `automation.assignments.view`, `automation.actions.view`. Роль, у которой есть только `automation.status.view`, пункта в меню не увидит, хотя страница `/app/automations/status` открывается по прямой ссылке ([how-to/auto-rules.md](../how-to/auto-rules.md)).

### Где правится дерево фаз и правил — `Rule Template Tree` `/app/automations/auto-rules/tree`

Дерево шаблона автоправил правится на отдельной странице `/app/automations/auto-rules/tree` — своего пункта в подменю раздела у неё нет. Открывается она из таблицы `Rule Templates`, требует право `automation.templates.edit`, ссылка возврата подписана `Rule Templates`. Ноды дерева — `Phase` в корне и `Auto Rule` внутри фазы; что из них собирается — [how-to/auto-rules.md](../how-to/auto-rules.md).

## Где находятся Facebook-аккаунты и рекламные кампании Meta — `/app/facebook`

Meta — Facebook/Meta: соцпрофили, рекламные аккаунты, кампании, ad sets, ads.

- **Social profiles** `/social-profiles` — Meta Social Profiles.
- **Ad accounts** `/ad-accounts` — рекламные аккаунты.
- **Campaigns** `/campaigns`, **Ad sets** `/ad-sets`, **Ads** `/ads` — структура FB-рекламы. Управление Meta-модулем целиком — [how-to/meta-ads.md](../how-to/meta-ads.md).
- **Pages** `/pages` — FB-страницы, привязанные к соцпрофилю; пункт закрыт правом `meta.pages.view`.
- **Pixels** `/pixels` — пиксели рекламных аккаунтов.

## Где лежит рекламный кабинет Google — `/app/google`

Google — второй рекламный модуль рядом с Meta: подключённые аккаунты и подтянутая из Google Ads иерархия.

- **Accounts** `/accounts` — подключённые Google-аккаунты.
- **Ad accounts** `/ad-accounts` — рекламные аккаунты.
- **Campaigns** `/campaigns`, **Ad groups** `/ad-groups`, **Ads** `/ads`, **Keywords** `/keywords` — структура рекламы Google.
- **Other** `/other` — прочие сущности модуля.

Разбор модуля целиком (подключение через OAuth, что тянется и как часто) — [how-to/google-ads.md](../how-to/google-ads.md).

## Где смотреть когорты, Roll Up и сравнение — Analytics `/app/analytics`

Analytics — отчёты: когорты, сравнение, Roll Up.

### Где смотреть Roll Up report — `/app/analytics/dd`

- **Roll Up report** (в UI также **Drill Down**) `/dd` — Roll Up (древовидный отчёт через Groupers). В MTK группа Analytics свёрнута в плоскую вкладку **Reports** (`/analytics/dd`) — это только Roll Up, см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.

### Какие отчёты Analytics доступны только в ERP

- **Cohorts report** `/cohorts-report` — когортный отчёт.
- **Comparative analytics** `/compare-analytics` — сравнение периодов. Пункта в меню `Analytics` сейчас нет — страница открывается только прямым адресом.
- **Events Exporter** — асинхронная выгрузка сырых таблиц (`Visits` / `Trash Visits` / `Conversions`) в файлы; открывается диалогом из Analytics ([how-to/events-exporter.md](../how-to/events-exporter.md)).
- **BI Builder** `/bi-builder` — конструктор BI-дашбордов (табы, виджеты).
- **Live Pulse** `/live-pulse` — живой поток событий трафика (`init` / `handle` / `conversion`); не отчёт: счётчики живут только пока страница открыта ([how-to/live-pulse.md](../how-to/live-pulse.md)).
- **Session Analytics** `/session-analytics` — записи сессий с плеером + сравнение scroll- и click-карт до трёх лэндов рядом ([how-to/session-analytics.md](../how-to/session-analytics.md)).

## Где настраиваются права, формы, конверсии и фильтры — Settings `/app/settings`

Settings — настройки тенанта. Состав подразделов зависит от билда: часть доступна в обоих билдах, часть — только в ERP.

### Какие подразделы Settings доступны в ERP и MTK

- **Users** `/users` — пользователи тенанта.
- **Positions** `/positions`, **Teams** `/teams` — роли и команды (права/шеринг, см. [models/permissions-model.md](../models/permissions-model.md)).
- **Advertisers** `/advertisers` — рекламодатели.
- **Servers** `/servers`, **Proxies** `/proxies` — серверы и прокси. В MTK эти разделы перенесены в Settings (в ERP они в Tech).
- **DNS providers** `/dns-providers`, **Domain providers** `/domain-providers`, **Server providers** `/server-providers` — провайдеры.
- **Domain checkers** `/domain-checkers` — чекеры доменов.
- **Tags** `/tags`, **Presets** `/presets` — теги и пресеты.
- **Logs** `/logs` — логи (Monitor / Destination Handler, Loggable UUID). Логи по одному визиту быстрее открыть ПКМ по визиту в `Tracker → Visits` → `Visit logs` (переключи `Severity` на `Debug`, иначе таблица выглядит пустой; [how-to/debug-with-logs.md](../how-to/debug-with-logs.md)).
- **Trackers** `/trackers` — постбэк-трекеры (в MTK перенесены сюда из Tracker).
- **Macros** `/macros` — макросы (в MTK перенесены сюда из Content; см. *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*).
- **Streams** `/money-streams` — механизм периодического обмена revenue/costs с внешними системами; **фича живая, но пока не в активном использовании** (см. [reference/glossary.md](glossary.md) → Streams). Доступен в обоих билдах.
- **Trash** `/trash` — корзина. Только в MTK: в ERP-виде отдельного пункта в `Settings` нет (отсев визитов лежит в `Tracker → Trash`).

### Какие подразделы Settings доступны только в ERP

- **Distributions** `/distributions` — все Distributions (Direct Traffic / Campaign Content / Fill Field / Flow Content / Remarketing Content / Revenue и т.д.). Страница показана раскрывающимися папками по типу дистрибуции — переключателя на плоскую таблицу нет. Шаблоны автоправил лежат здесь же папкой типа `Auto Rules`, но своя страница у них в разделе `Automations` ([how-to/auto-rules.md](../how-to/auto-rules.md)).
- **Fields** `/fields` — поля визита (custom fields). Страница выглядит не плоской таблицей, а раскрывающимися папками по `Type` и `Group`; вернуться к плоскому виду нельзя — переключателя вида на странице нет ([how-to/custom-fields.md](../how-to/custom-fields.md)).
- **Forms** `/forms` — настройки SDK-форм (поля, Required, min/max).
- **Conversion types** `/conversion-types` — типы конверсий.
- **Metrics** `/metrics` — метрики.
- **Traffic filters** `/traffic-filters` — Traffic Filters (антифрод/фильтрация).
- **Generators** `/generators` — справочник записей-интеграций «генераторов»: у каждой записи `Name` + `Integration Type` (список типов приходит с бэка) + settings-JSON под выбранный тип. Права — `settings.generators.*`. Это **не** генератор трекинг-ссылок кампании (`Link Generator`) — отдельная сущность. В MTK раздела нет.
- **Business models** `/business-models` — бизнес-модели.
- **External reports** `/external-reports` — внешние отчёты.
- **Item lists** `/item-lists` — списки значений.

---

## Что доступно в меню профиля (аватар, верхний правый угол)

Меню профиля открывается кликом по аватару в верхнем правом углу.

- **Переключатель тенантов** — поиск + список тенантов, у каждого **бейдж билда** (`ERP` / `MTK`). Так переключаются между тенантами и видно, в каком билде каждый.
- **Billing** — биллинг-модалка тенанта (план, статус, инвойсы, оплата; см. *Билды, тарифы, триал и статусы тенанта*).
- **Manage tenant** — управление тенантом.
- **Profile settings** — настройки профиля пользователя.
- **Support** / **Start chat** — связаться с поддержкой; **Need assistance?** — помощь.
- **Cloud Files** — файловое хранилище тенанта.
- **Switch build** — переключиться между ERP- и MTK-видом того же тенанта.
- **Data API Docs** — спека `Data API` прямо в продукте ([how-to/api.md](../how-to/api.md)).
- **Become affiliate** — реферальная программа AIO (пункт виден не всем тенантам).
- **Changelog**, **Placeholders**, **Public roadmap** — справочные.
- **Appearance** — тема интерфейса: `Light` / `Dark` / `System`.
- **Log out** — выход.

---

## Смежные темы

- [reference/glossary.md](glossary.md) — что значит каждая сущность из меню.
- [reference/ui-common.md](ui-common.md) — общий UI таблиц (тулбар, кнопка `Settings` → `Table settings` и пресеты, кнопка правки прямо в строке, общие right-click экшены, Share).
- [reference/profile-settings.md](profile-settings.md) — детали пункта `Profile settings` (вкладки Profile/Appearance: поля, форматы, опции).
- [reference/sdk.md](sdk.md) — `Content → Macros` → AIO SDK Macros Collection.
- [reference/placeholders.md](placeholders.md) — плейсхолдеры (пункт `Placeholders` в меню профиля).
- *Билды, тарифы, триал и статусы тенанта* — пункт `Billing`, билды, статусы.
- [how-to/registration.md](../how-to/registration.md) — инвайт-коды (`AIO → Invite codes`).
