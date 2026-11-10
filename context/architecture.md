---
id: architecture
title: Архитектура AIO
description: SaaS-приложение + AIO-агент на серверах клиента, CDN для лэндов, тенанты, репойнт при падении сервера, мониторинг.
doc_type: context
builds: [erp, mtk]
related: [ui-map, api, server, landing, registration, business-model, permissions-model, domains, remarketing-campaigns, tenant, notification-center, profile-settings, domain, forms, destinations, meta-spend-allocation, campaigns, visit-lifecycle, glossary]
language: ru
updated: 2026-11-10
---

# Архитектура AIO

Как физически устроен AIO — какие есть слои, что где живёт, почему лэнды на CDN, почему сервера агностичны, и что реально происходит, когда сервер падает. Без этой картинки половина вопросов выглядит как магия («куда делся мой лэнд?», «почему трафик не идёт?», «почему нотификации не приходят?»).

## TL;DR

- AIO = **SaaS-приложение** на `app.aio.tech` + **AIO-агент** на серверах клиента + **CDN** для контента лэндов.
- Лэнды живут **на CDN**, а не на серверах клиента. Сервера агностичны — на них только агент. Поэтому замена сервера не требует миграции лэндов.
- Клиент = **Tenant** в AIO. У одного клиента может быть несколько тенантов (под разные вертикали или команды).
- Сервера / Домены / DNS подключаются через интеграции (`Server Provider`, `Domain Provider`, `DNS Provider`) в `Tech` (в MTK — в `Settings`).
- При падении сервера работает **Repoint** — автоматическое перенаправление его доменов на **случайный** живой сервер в той же `Repoint Group` (гео/latency при выборе не учитываются — географию задаёт сама группа). Даунтайм — время, пока мониторинг заметит падение, плюс DNS-пропагация.
- Мониторинг — отдельная выделенная подсистема: AIO сам следит за доступностью серверов и доменов и переводит упавший объект в `Danger`. Алерты мониторинга — сгруппированные отчёты только в Telegram через `AIO-tech-bot`: сразу при смене статуса, по висящим инцидентам — напоминание раз в час; без привязки аккаунта Monitoring User к Telegram бот не пишет.

## Две части системы

### 1. SaaS-приложение — `app.aio.tech`

Веб-приложение, в котором живут все пользователи AIO (баеры, тех-команды, админы). Здесь:

- UI всех разделов системы (`Tracker`, `Content`, `Marketing`, `Finance`, `Tech`, `Automations`, `Analytics`, `Settings`; состав меню зависит от билда — [reference/ui-map.md](../reference/ui-map.md)).
- Конфигурация тенантов: флоу, кампании, дистрибуции, права, метрики.
- Метаданные визитов, лидов, постбэков.
- Аналитическое хранилище. **Trash-визиты** (визиты с пометкой `is_trash` — они исключены из основной аналитики, причина отсева пишется в поле визита `Trash reason`) хранятся в этом же хранилище.
- Внутренний и публичный API: всё, что можно нажать кнопкой в UI, можно сделать через API. Для внешних систем есть узкий документированный `Data API` и MCP-сервер ([how-to/api.md](../how-to/api.md)).

Важно: **визит не попадает в SaaS**. Клик по трекерной ссылке не ведёт на `app.aio.tech` — он ведёт на домен клиента, который смотрит на сервер клиента, где работает AIO-агент.

### 2. AIO-агент

Серверный компонент, который ставится на серверы клиента через `Deploy`-скрипт. Это конечный автомат (FSM), работающий **по командам AIO**: агент принимает запрос визитёра, на каждом шаге обращается к AIO и исполняет полученную команду — что отдать визитёру, куда вести визит дальше. Решения флоу — переходы, фильтрацию, выбор сплит-варианта, пуш в Destination — вычисляет AIO; агент исполняет их на краю и сообщает AIO о событиях флоу (submit формы, переход по ссылке-шагу). Полная модель агента и протокол команд — [models/server.md](../models/server.md).

Что делает агент:

- Принимает входящий запрос на доменах, которые смотрят на этот сервер.
- Распознаёт визит по `cuuid` (UUID кампании) и `suuid` (UUID Source) в URL; сам визит идентифицируется отдельным `visit_uuid`.
- Снимает с визита трекинговые параметры (IP, user-agent и прочие заголовки, гео, язык браузера и т.п.).
- Исполняет команды AIO: отдаёт визиту HTML лэнда, который пришёл ему в команде от AIO, запускает Backfix, принимает submit формы — а решение, куда вести визит дальше (фильтр пропустил/срезал, какой сплит-вариант, какой переход), приходит от AIO. Страницу агент не рендерит и не собирает: подресурсы лэнда (картинки, css, js, видео) браузер визитёра тянет с CDN через CDN-путь агента, а единственная правка HTML на агенте — правка вида этого пути, если у типа лэнда включён флаг `Temp CDN used`: файлы отдаются те же ([models/landing.md](../models/landing.md)).
- Сообщает AIO о событиях флоу (submit формы, переход по ссылке-шагу) — на их основе AIO ведёт визит дальше и считает аналитику. Поведение посетителя на странице (клики, скроллы для heatmap'ов и записи сессий) снимает отдельно клиентский SDK на самом лэнде, не агент.

Рядом с агентом стандартный `Deploy` ставит на тот же сервер Squid Proxy; прокси он работает, если в скрипте заданы логин и пароль Squid ([models/server.md](../models/server.md)).

## Почему серверы клиента — агностичны

Это ключевое архитектурное решение, от которого зависят масштабирование, репойнт и работа с лэндами:

- Файлы лэндов (HTML, JS, CSS, картинки, видео) **не лежат** на серверах клиента. Они лежат на **CDN**.
- На сервере живёт только AIO-агент и его служебные контейнеры (по дефолту — со Squid Proxy).
- Серверы на стандартном `Deploy` внутри идентичны: один и тот же агент и один и тот же набор контейнеров.

Отсюда вытекает практическая суть:

- **Сервер можно мгновенно заменить.** Не нужно переносить файлы, как в классических трекерах, где лэнды лежат на самих серверах. На серверах клиента файлов лэндов нет — они на CDN.
- **Репойнт работает быстро.** Достаточно перекинуть DNS-рекорды — все лэнды уже доступны на CDN отовсюду.
- **Сервера взаимозаменяемы.** Можно строить пул серверов разных гео и распределять трафик по регионам (`Repoint Group` держит регионы раздельно). На скорость открытия гео сервера при этом влияет слабо — динамические шаги упираются в контрол-плейн, статику кэширует Cloudflare ([models/server.md](../models/server.md)).

## Tenant как единица

### Что такое tenant

`Tenant` — это **тенант** (workspace по терминологии других SaaS) в AIO. У каждого тенанта есть:

- `Name`, `Description`, `Billing email`, `Country code`.
- Свой набор серверов, доменов, флоу, кампаний, лэндов, дистрибуций, юзеров, ролей.
- Свои метрики, конверсии и аналитика.

Понятие `Workspace` **не используется** — используется только `Tenant`. Если кто-то говорит «воркспейс», имеется в виду тенант.

### Шаблон тенанта — откуда берётся новый тенант

**Новый тенант — одноразовая копия тенанта-донора, привязанного к шаблону; шаблон выбирается карточкой на шаге «Тенант» регистрации.** Публичные шаблоны — по вертикали и билду: `ERP Finance`, `ERP iGaming`, `ERP Nutra`, `ERP Sweeps` и `MTK Finance`, `MTK iGaming`, `MTK Nutra`, `MTK Sweeps`. На карточке — логотип, имя и, если оно задано, короткое описание вертикали (`Trading, loans, investments.`, `Casino, slots, poker funnel`, `Health, weight-loss products.`, `Prize giveaways and contests.`); список подставляется под билд тенанта ([how-to/registration.md](../how-to/registration.md)).

Копия снимается один раз, в момент создания: правки в доноре в уже созданный тенант не попадают, сменить шаблон у существующего тенанта нельзя — тенант не хранит, из какого шаблона создан. Сущности-примеры узнаются по имени: у трафиковых кампаний в имени — `Campaign Example`, у большинства лэндов и дестинейшенов — `Example`, у кампаний рассылок — хвост `| Drip` / `| Cron` (у части — с префиксом `Example - `), плюс превью-кампании (`Preview Campaign`, `Global Preview Campaign`, `1Step Preview Campaign`); метка-тег на них не приезжает. Всё приехавшее — обычные сущности тенанта: их правят, архивируют и заменяют своими. Шаблон — стартовая точка, а не ограничение: команды запускаются по-разному, и состав подстраивают под себя.

### Что приезжает из шаблона Finance — флоу, лэнды, формы

**`ERP Finance` и `MTK Finance` строятся вокруг двухшаговой воронки прилэнд → оффер с SDK-формой; у каждого флоу есть кампания-пример.** Флоу ERP: `1/2Step | Manual Offer Name` (и варианты `+ TY`, `+ TY + Domain Change`), `1/2Step | Offer Name Distribution` (оффер-нейм из дистрибуции), `1Step | Form Landing`, `Preview Flow`; в MTK те же сценарии названы `1/2Step | Preland to Offer…`, флоу с дистрибуцией оффер-нейма нет. Кампании-примеры повторяют имена флоу (`Manual Offer Name - Campaign Example`, `Form Landing - Campaign Example` …).

Лэнды: `Preland Example #1 w/ AI Splits & Dynamic Offer Name`, `Example Preland w/ Form — Starter Plan Quiz`, `Offer Example #1 w/ Dynamic Offer Name` (SDK-форма), `Example Offer Page — Custom HTML Form` (своя HTML-форма через `AIO Non-SDK Form Handler`), `Example White Page — Budget`, `AIO Thank You Page`; в MTK вместо двух лэндов с динамическим оффер-неймом — `Example Preland w/ AI Splits — News Story` и `Example Offer Page — Immediate Connect`. Формы: `Default Leads Form`, `Leads Form w/ Consent`, `2-Step Leads Form`, `Qualifying Form w/ Investment Range`, `Short Form - Name & Phone`. Дестинейшены: `Dummy API [ DEBUG ]` (принимает лид и всегда отвечает успехом — для проверки воронки без реальной партнёрки), `Example Bridge Integration`, `Example IREV Integration`.

### Что приезжает из шаблона Nutra и Sweeps — флоу, лэнды, формы

**`ERP Nutra` / `MTK Nutra` — воронка под заказ товара, в том числе через видео (VSL); `ERP Sweeps` / `MTK Sweeps` — короткие one-step-воронки под подарок или розыгрыш.**

Nutra: флоу `1/2Step | Preland -> Order Page`, `1Step | Order Page`, `VSL | Video -> Offer`, `VSL | Video -> Package Checkout`, `Preview Flow`; кампании-примеры — `Preland -> Order`, `Order Page`, `Multigeo IT-ES-PL`, `Landings AI by Leads`, `Package Upsell`, `Quiz Order`, `Custom HTML Form`, `VSL -> Offer`, `VSL Network Test AI`, `VSL Package Checkout` (все с суффиксом `- Campaign Example`, у части задано гео) и `Global Preview Campaign`. Лэнды: `Preland Example #1 - Health Quiz`, `Preland Example #2 - News Story`, `Offer Example #1 - Order Form (SDK)`, `Offer Example #2 - Package Upsell`, `Offer Example #3 - Custom HTML Form`, `VSL Example #1 - Video Sales Letter`, `VSL Example #2 - Package Checkout`, `Example White Page - Wellbeing Guide`, `AIO Thank You Page`. Формы заказа: `Order Form - Name & Phone`, `Order Form - Package & Address`, `Order Form - Full Address`, `Quiz Form - Age & Contacts`. Дестинейшены — примеры под четыре партнёрские интеграции, `Simple Redirect`-примеры под партнёрки с оплатой за продажу плюс `Dummy API - Flow Testing`, `Example Bridge Integration`, `Example IREV Integration`. В `MTK Nutra` те же флоу, лэнды, формы и дестинейшены, но без кампаний рассылок.

Sweeps: флоу `Default 1Step Flow` (и `w/ Domain Change`; в ERP ещё `w/ Dynamic Product`), `1/2Step | Preland -> Offer w/ Form` (в MTK ещё вариант `+ Antifraud`), `1step preview`; кампании-примеры `Prize Wheel -> SDK Form`, `Quiz Boxes -> Custom Form`, `Multigeo US-DE-FR`, `Domain Change`, `Landings AI by Registrations` (в ERP ещё `Dynamic Product`). Лэнды: `Preland Example #1 - Prize Wheel`, `Preland Example #2 - Gift Boxes w/ Quiz Form`, `Offer Example #1 - AIO SDK Form`, `Offer Example #2 - Custom HTML Form`, `Example White Page - Weeknight Recipes`, `AIO Thank You Page`. Формы: `Email Only Form`, `Default SOI Form`, `Quiz Form - Prize & Age`, `Shipping Form - Physical Prize`. Дестинейшены: `Dummy API` и три `Simple Redirect`-примера.

### Что приезжает из шаблона iGaming — PWA-флоу без форм

**`ERP iGaming` и `MTK iGaming` собраны вокруг PWA и редиректов на оффер — SDK-форм в них нет.** Флоу: `Web - Preland / Offer`, `PWA / OneLink`, `PWA / OneLink w/ Landing`, `Advanced - PWA / OneLink` (ветвление по `Android` / `iOS` шагами `Condition`), `Deeplink Android`, `Preview Flow`; на каждый — кампания-пример (`Web Flow - Campaign Example`, `Default Flow - PWA Campaign Example`, `PWA w/ Landing Flow - Campaign Example`, `Advanced Flow - Campaign Example`, `Android Deeplink - Campaign Example`). Лэнды: `AIO PWA Example w/ Splits` (тип `PWA`), `Example Preland — Bonus Chests`, `Example Preland — Wheel of Fortune`, `Example Offer Page — LuckySpin`, `Example White Page`. Дестинейшены — `Simple Redirect`-примеры с адвертайзерами: сторонние PWA-сервисы (`Example … PWA`) и партнёрские офферы (`Example Partners | …`, `Example Direct Brand | …`). Источников больше, чем в других вертикалях: кроме `FB w/ CAPI`, `Google` и `Preview Link` — генератор ссылок под сторонние PWA и два источника под Android-приложения.

### Что приезжает из шаблона — общий слой всех вертикалей

**Под флоу в каждом шаблоне лежит общий слой: источники, трекеры, типы конверсий, бизнес-модели, дистрибуции, фильтры, поля, метрики, пресеты, макросы, позиции и команды.**

- **Источники:** `FB w/ CAPI`, `Google`, `Preview Link` (в Sweeps — `1Step Preview Link`). **Трекеры** типа `Facebook Conversion API`: в Finance, Nutra и Sweeps — `FB Lead` плюс трекер под целевое событие вертикали (`FB Registration` в Sweeps, `FB Approved` в Nutra); в iGaming — `FB AIO Install`, `FB AIO Registration`, трекер под целевое событие вертикали и пример HTTP-постбэка `Example - 3rd Party PWA Postback`.

- **Типы конверсий:** Finance — `Lead`, `Lead Rejected`, `Fraud Lead`, `Push Subscribe`, `Sale Status Update` и тип под целевое действие вертикали; Nutra — `Lead`, `Approved`, `Rejected`, `Trash`, `Upsell`, `Push Subscribe` (с десятками slug-синонимов под статусы партнёрок); Sweeps — `Lead`, `Registration`, `Fraud Lead`, `Lead Rejected`, `Push Subscribe`; iGaming — `Install`, `Open PWA`, `Registration`, `Push Subscribe`, `Correction` и два типа под целевые события вертикали.

- **Бизнес-модели** — восемь во всех шаблонах (`Arrived` / `Static` / `Zero` / `Cancel` × `Revenue` / `Payout`); `Revenue Distribution` стоит на `Arrived Revenue`, `Payout Distribution` — на `Zero Payout` ([models/business-model.md](../models/business-model.md)).

- **Дистрибуции:** `Payout Distribution` и `Revenue Distribution` везде; в ERP ещё `Buyer UTM Distribution` (шаг `Fill Buyer UTM` в начале флоу-примеров, кроме превью- и deeplink-флоу), в ERP Finance — `Offer Names Distribution` и `Celebrity Distribution`, в iGaming — `Offer Distribution` и `Direct Traffic Distribution`; деревья `Message Templates` с примерами пушей (`Example - … pushes`) — в ERP-шаблонах и MTK iGaming.

- **Фильтры (`Settings → Traffic filters`):** `AIO Filter Service` везде — шаг `AIO Filter` (в части iGaming-флоу подписан `Filter`) в флоу-примерах работает через него; `AIO Antifraud` — в Finance, Nutra и Sweeps: после отправки формы в флоу стоят шаги `AIO Antifraud` → `Check Antifraud Points` (`Condition`, в шаблоне условие `fraud_score < 999` — пропускает всё, порог задаётся под себя) → при отказе `Spawn Fraud Lead` (в Nutra — `Spawn Trash Lead`). В Finance эта связка стоит во всех флоу-примерах, в Nutra — в `1/2Step | Preland -> Order Page` и `1Step | Order Page` (в VSL-флоу её нет), в ERP Sweeps — в `Default 1Step Flow` и `1/2Step | Preland -> Offer w/ Form`, в MTK Sweeps — только в `1/2Step | Preland -> Offer w/ Form + Antifraud`.

- **Поля визита:** лиды (`first_name`, `email`, `phone` …), трекинг (`buyer_utm`, `creative`; в ERP Finance ещё `offer_name`), параметры Facebook и Google, сплиты лэнда (`lp_header`, `lp_subheader`, `lp_header_image`; в iGaming — `lp_header`, `lp_image`, `lp_description`); в Finance и iGaming — кастомные `campaign_field_1..3` / `landing_field_1..3` / `source_field_1..3` / `destination_field_1..3`; в MTK ещё `sub1`…`sub15`.

- **Метрики и пресеты:** набор метрик вертикали (воронка, деньги; `[Meta] …` — в ERP-шаблонах и MTK Nutra; счётчики пушей — в ERP-шаблонах и MTK iGaming) и пресеты таблиц и Roll Up (`Unit Economics`, `Landing Quality`, `By Buyer [Owner → Campaign]`, `FB Structure [Campaign → Adset]` …); одноимённые метрики в разных вертикалях могут считаться по-разному — проверяйте формулу в `Settings → Metrics`.

- **Макросы:** глобальные `AIO SDK Macros Collection` (везде) и `AIO Push Subscription` (ERP Finance / Nutra / Sweeps, приезжает выключенным); `AIO GTAG` (кроме iGaming) — не глобальный, ставится на лэнд тегом `{{aio:macros:aio_gtag}}`; локальные под лэнды-примеры — `AIO Form Loader`, `AIO Non-SDK Form Handler`, `AIO Static Language Country for Form` (в Finance ещё `AIO Autologin Redirect` и глобальный `AIO Save Landing Params to Visit`, в Sweeps — `AIO Pre-Fill Form Handler`, в Nutra — `AIO VSL Kit` для VSL-лэндов); в iGaming, кроме коллекции, только `AIO Custom PWA`.

- **Типы лэндов:** `Preland`, `Offer`, `White` везде; `Preland w/ Form` и `Thank-You Page` — Finance и Nutra, `VSL` — Nutra, `Thank-You Page` — Sweeps, `PWA` — iGaming; шаблон лэнда один — `Simple`. **Типы адвертайзеров:** `Offer`, `PWA`.

- **Позиции и команды:** `Full Access`, `Team Lead`, `Buyer`, `Tech`, `Analyst` (в ERP ещё `Creative`, `Finance`) и команды `Tech Department`, `Media Buying Team A`, `Media Buying Team B` — владелец нового тенанта при создании входит в каждую из них с уровнем 1 (`Level 1`); базовые роли `Owner` / `Admin` / `Employee` создаёт сам продукт, не шаблон ([models/permissions-model.md](../models/permissions-model.md)).

### Что из шаблона в новый тенант НЕ приезжает — архив, домены и сервера

**Архивные сущности шаблоном не переносятся: в новый тенант едет только живое.** Правило действует почти на всё, что копируется: теги, поля визита, формы, метрики, типы конверсий, пресеты, источники, дестинейшны, трекеры, макросы, креативы, трафик-фильтры, дистрибуции, команды, флоу с их шагами, кампании, адвертайзеры, группы плейсхолдеров, ленды и позиции. Производные сущности подрезаются по живым родителям: языки форм берутся только у неархивных форм, ноды — только у неархивных дистрибуций, переходы флоу — только между неархивными шагами.

**Рабочие домены и сервера донора шаблоном не копируются: для своего трафика сервер и домен заводятся самостоятельно** ([models/server.md](../models/server.md), [how-to/domains.md](../how-to/domains.md)). В кампаниях-примерах домен не выбран — перед запуском его выбирают вручную. Не копируются также папки, шаринги и пользователи донора. Записей `Message templates` и правил уведомлений в публичных шаблонах нет — рассылки-примеры живут в деревьях `Message Templates` и кампаниях типа `Notifications` ([how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md)).

### Multi-tenancy

- Один **User** может иметь доступ к **нескольким** тенантам — через дроп-даун в правом верхнем углу переключается между ними.
- Один **Tenant** может быть пошарен между **несколькими** юзерами.
- Тенанты изолированы: метрики, поля визита, конверсии, команды одного тенанта не пересекаются с другим.
- Под разные вертикали обычно заводят **отдельные тенанты** (доптенант — см. [models/tenant.md](../models/tenant.md)).

## Сервера и домены — связка

### Как добавляется сервер

Два пути (`Buy a server` / `Add manually`) и полная таблица атрибутов — [models/server.md](../models/server.md). Deploy-скрипт ставит Docker, Squid, AIO-агент + сервис обновления (самообновление); кастомный Deploy — например, чтобы сервер стал прокси. При `Buy a server` скрипт выполняет провайдер при первом запуске машины, при `Add manually` — владелец сервера по SSH; AIO в обоих случаях лишь ждёт, пока установленный агент начнёт отвечать, — что делать, если сервер завис в `Processing` или упал в `Danger`,.

### Как добавляется домен

Через `Tech → Domains → +Domain` (в MTK — `Domains → +Domain`). Два пути (`Buy` / `Add manually`) и атрибуты домена (`Description`, `Tags`, `Monitoring User`, `Allow indexing by search engines`, `Default query Code`) — [how-to/domains.md](../how-to/domains.md).

### Связка домен → сервер

Домен указывает на IP сервера через A-record в DNS-провайдере. Один домен обычно смотрит на **один сервер** в моменте, но связку можно поменять — это и есть механика репойнта.

## Repoint — что это и зачем

### Сценарий

Сервер падает, провайдер блокирует аккаунт, или не прошла оплата — сервер становится недоступен. Как только мониторинг переводит сервер из `Running` в `Danger` (секция «Мониторинг и нотификации» в этом документе), AIO **один раз на этот переход** делает следующее:

1. Берёт все домены (кроме архивных), у которых этот сервер основной. Домены, где упавший сервер стоит вторым в списке `Servers`, авто-репойнт не трогает.
2. Для каждого домена выбирает **случайный живой сервер** (`Running`) в той же `Repoint Group` (выбор случайный — гео/latency при выборе не учитываются; географию задаёт сама группа; сервер без группы ищет замену среди серверов без группы), поэтому домены одного упавшего сервера могут разъехаться по разным серверам группы. Домену Cloudflare в режиме `Only IPv6` подходят только серверы с IPv6.
3. Через DNS-провайдера меняет A-record каждого домена на IP нового сервера (в режиме `Only IPv6` — AAAA-запись) — домены переезжают по одному; пока идёт перенос, домен в `Processing` со строкой `Repoint requested - Current Incident: <причина>`.
4. Трафик продолжает идти, как только DNS-пропагация дойдёт до юзеров.

Когда упавший сервер снова отвечает, он возвращается в `Running`, но переехавшие домены остаются на новых серверах — назад их AIO не переносит.

Это работает **именно** потому что сервера агностичны: на новом сервере уже стоит тот же агент, готовый принимать визиты, а лэнды лежат на CDN — их переносить не надо.

### Repoint повторяется сам при временной ошибке DNS-провайдера — до 6 попыток

**Репойнт домена переживает временные сбои DNS-провайдера: до 6 попыток, домен всё это время в `Processing` со строкой `Repoint attempt <n> of 6 failed: <ошибка>. Retrying in <N> min`.** Пауза до следующей попытки — `<N>` в этой строке; на лимите запросов (Cloudflare `429` / код `1015`, текст `rate limit`) она от попытки к попытке растёт. Так ведут себя и авто-репойнт, и ручной `Repoint to a new server`, одиночный и массовый.

Если попытки кончились, домен уходит в `Danger` со строкой `All 6 repoint attempts failed: <ошибка>`. Ошибки, которые повтор не исправит, — у домена нет DNS-провайдера (`there is no dns provider`) или провайдер типа `Unknown`, целевой сервер удалён или в архиве (`Server for repoint not found`, `Server for repoint is archived`), — дают `Danger` сразу, строкой `Repoint failed: <ошибка>`. Провал репойнта не тихий: домен попадает в ближайший Telegram-отчёт мониторинга, если у него включён мониторинг, а у Monitoring User есть привязка к Telegram. Новый репойнт того же домена (например, ручной поверх автоматического) отменяет предыдущий — выполняется последний.

Домен без DNS-провайдера (или с провайдером типа `Unknown`) при падении сервера не репойнтится вовсе — он сразу уходит в `Danger` с текстом `Current Incident: <причина>`, а в отчёте по серверу появляется строка `<N> left in Danger: no DNS provider (not under AIO control)`. Разбор строк → «Домен ушёл в репойнт».

### Repoint Group

Атрибут сервера = строка (например, `eu`, `us`, `tier1`). Репойнт работает **только внутри группы**.

- Если в группе есть живой сервер — репойнтит на него.
- Если в группе **нет живого** сервера — переезжать некуда: репойнт НЕ выходит за пределы группы, трафик с этих доменов останавливается.

Поэтому правило: **в одной репойнт-группе нужно держать минимум 2 сервера**. Если в группе всего один — репойнт фактически нечем отрабатывать, и единственная защита — держать в группе минимум 2 живых сервера.

Также используется для разделения трафика по гео / вертикалям / командам — чтобы в момент репойнта Сан-Франциско не оказался в Амстердаме.

### Рекомендация на практике

- Держать аккаунты у **нескольких** Server Providers. Если падает один провайдер целиком — трафик уйдёт на другой.
- Серверы в одной репойнт-группе — желательно от **разных** провайдеров.
- Минимум 2 сервера в группе.

## Мониторинг и нотификации

### Как мониторинг следит за серверами и доменами — статусы `Danger` и `Fine`

**AIO сам следит за доступностью серверов и доменов, у которых включён мониторинг: упавший объект получает `Danger` со строкой причины, восстановившийся — снова `Running` со строкой `Fine`.** Объект в `Processing` (установка, выпуск SSL, репойнт, удаление) и объект в архиве мониторинг не трогает.

Строки причины у сервера — `Unreachable: Connection refused`, `Unreachable: Check timed out`, `Unhealthy: Disk exhausted (free …, … inodes)` (на томе агента кончилось место) и т. п.; у домена — `Unreachable: …`, `Wrong server: Probe did not answer 200 "ok"` (по адресу ответил не агент AIO), плюс строки про DNS, сертификат и срок регистрации (секция «Что отслеживается»). Переход сервера в `Danger` запускает авто-репойнт его доменов (секция «Repoint — что это и зачем»). Мониторинг фиксирует **факт** недоступности и текст ошибки проверки — почему именно сервер или домен лёг, он не определяет.

### Объект завис в `Processing` дольше 3 часов — мониторинг переводит его в `Danger`

**Домен или сервер, который висит в `Processing` без изменений больше 3 часов, AIO сам переводит в `Danger` со строкой `Stuck in processing for more than 3 h with no progress — check manually`.** Идущая операция (покупка, DNS, выпуск SSL, репойнт, проверка установки) запись обновляет, поэтому так помечаются только операции, которые оборвались. В `Processing` мониторинг объект не проверяет — после пометки `Danger` он снова под мониторингом.

Такой объект попадает в Telegram-отчёт, если у него включён мониторинг и у Monitoring User есть привязка к Telegram. Если он на самом деле работает (и у сервера есть IP), мониторинг сам вернёт его в `Running`.

### Объект в `Danger` дольше 30 дней — мониторинг выключается сам, строка `Monitoring paused`

**Домен или сервер, который пролежал в `Danger` больше 30 дней и так и не поднялся, AIO сам снимает с мониторинга: строка статуса — `Monitoring paused: down for 30+ days — re-enable monitoring to check again`.** Отсчёт идёт от последней смены статуса. Статус остаётся `Danger`, но объект больше не проверяется и пропадает из Telegram-отчётов и ежечасных напоминаний; отдельного уведомления о паузе нет. В логах объекта (namespace `Monitoring`) остаётся запись `Monitoring paused: down for 30+ days (no reminders, not checked until re-enabled)`. Архивные объекты и объекты с уже выключенным мониторингом пауза не трогает.

Вернуть проверки — снова включить мониторинг: у домена `Place monitoring` (по одному или массово), у сервера `Place monitor`. Строка статуса меняется на `Monitoring resumed`, в логе — `Monitoring re-enabled: checks resume, the 30-day pause counter restarts`. Поднимется объект — мониторинг сам вернёт `Running`, и восстановление придёт в отчёт; не поднимется — повторного алерта не будет, а через 30 дней объект снова встанет на паузу. Как ещё спрашивают: «пропали алерты по старым доменам», «мониторинг сам выключился», «что значит `Monitoring paused`».

### Алерты мониторинга в Telegram — отчёты `AIO Monitoring · Domains` и `AIO Monitoring · Servers`

**Алерты мониторинга приходят Monitoring User-у в Telegram двумя сгруппированными отчётами — `🛰 AIO Monitoring · Domains` и `🛰 AIO Monitoring · Servers`.** Отчёт уходит сразу, как только что-то изменилось: новый инцидент или восстановление (блоки `Ongoing incidents` и `Recovered`). Пока инцидент висит, а нового ничего нет, приходит напоминание раз в час — так и написано в подвале: `To mute Ongoing alerts, archive the domain or remove it from monitoring. Otherwise a reminder is sent every hour.`

Внутри — счётчики `Ongoing` / `Resolved` и списки по тенантам, до 5 позиций на тенант (свежие изменения первыми), остальное — строкой `⋯ and N more` со ссылкой на `Tech → Domains` или `Tech → Servers`. У домена под именем — IP сервера и причина; у сервера — IP, имя и строки с причиной и итогом авто-репойнта: `Repointing N domains → group <группа>`, `N left in Danger: no running server to repoint in group <группа>`, `N left in Danger: no DNS provider (not under AIO control)` (сервер без группы подписан группой `default`). Отдельных сообщений мониторинга на каждый сервер или домен нет; вердикт домен-чекера (`Google Threat` / VirusTotal) бот присылает отдельным сообщением `🔴 Domain Incident (…)` по каждому домену.

Заглушить отчёты по объекту — заархивировать его или выключить ему мониторинг; объект, который лежит дольше 30 дней, AIO снимает с мониторинга сам (секция «Объект в `Danger` дольше 30 дней — мониторинг выключается сам»). Как отобрать алертящие домены фильтром `Alerting Now` — [how-to/domains.md](../how-to/domains.md) → «Как работает мониторинг доменов».

### Статусы в интерфейсе — что означает иконка у сервера и домена

Статусов три, и они **одни и те же у сервера и домена**: `Running`, `Processing`, `Danger`. Иконка в строке — это статус записи, а не «здоровье прямо сейчас»: `Danger` держится, пока инцидент не закрыт или операция не повторена.

- **Иконки нет** — `Running`: всё ок.
- **Синий вращающийся значок** — `Processing`: идёт операция (создание, установка, выпуск SSL, репойнт, удаление).
- **Красный крестик** — `Danger`: открытый инцидент недоступности **или** провалившаяся операция. Причина — в подсказке у иконки (`status_text`): у сервера `Unreachable: …` / `Wrong server: …` от мониторинга или `All install checks failed: …`, у домена `All 6 repoint attempts failed: …`, `All checks failed: …`, `Domain expiring: …`, `<имя чекера> Incident: <вердикт>` и другие строки.

Вердикт домен-чекера (`Google Threat` / VirusTotal) **тоже кладёт домен в `Danger`** — со строкой `<имя чекера> Incident: <вердикт>` (например, `1 Suspicious & 2 Malicious`), отдельного «жёлтого» статуса под чекеры нет. Жёлтый значок-предупреждение в строке домена означает включённый `Under Attack`; значок глаза / перечёркнутого глаза — включён или выключен мониторинг домена.

### Что отслеживается

- Доступность сервера (отвечает ли агент AIO на сервере).
- Доступность домена (отвечает ли через него агент AIO), а заодно его DNS и сертификат: `Domain not found: …` — имя не резолвится, `TLS invalid: Certificate has expired` / `TLS invalid: Certificate signed by unknown CA` — проблема с сертификатом.
- Место на диске сервера: агент сообщает о нехватке, сервер уходит в `Danger` со строкой `Unhealthy: Disk exhausted (…)`.
- Нагрузка агента (опционально): при высокой загрузке CPU несколько минут подряд алерт приходит в отдельный чат, куда добавлен второй бот AIO, — [how-to/domains.md](../how-to/domains.md) → «Алерты по нагрузке (опционально)».
- Срок регистрации домена: за 7 дней до окончания домен уходит в `Danger` со строкой `Domain expiring: <дата> (N days left)`, в последний день — `(expires today)`, после — `Domain expired: <дата> (N days ago)`. Число дней в строке может отставать от реального — ориентир дата.

### Monitoring User

`Monitoring User` — это юзер в AIO, которому `AIO-tech-bot` шлёт нотификации в Telegram, если статус сервера/домена меняется. Назначается отдельно для сервера и для домена в момент добавления (`Tech → Servers / Domains → Edit`); у Destination есть своё поле `Cap monitoring user` — ему приходит алерт о заполненной капе.

Можно использовать не отдельного юзера, а **групповой чат** — тогда создаётся «технический юзер-чат» (бот добавляется в чат, в системе заводится юзер, привязанный к этому чату), и все алерты падают в общий чат, где их видит вся команда.

### AIO-tech-bot и 2FA: что это и что гейтит

`AIO-tech-bot` — это Telegram-бот, который шлёт нотификации и может использоваться для включения 2FA. 2FA настраивается **через Telegram (`AIO-tech-bot`) или через Google Authenticator** — привязка к Telegram для 2FA больше не обязательна (её сделали необязательной, когда добавили 2FA через Google Authenticator). Других способов нет: получить код 2FA на email или по SMS нельзя. Алерты мониторинга серверов и доменов и другие прямые сообщения бота доставляются только в Telegram; уведомления Notification Center (колокольчик) бот проксирует в Telegram наравне со Slack — [mechanics/notification-center.md](../mechanics/notification-center.md).

Когда 2FA включена, она гейтит **сам вход**: при каждом логине (после логаута) AIO требует код подтверждения (verification-код от `AIO-tech-bot` в Telegram либо TOTP-код из Google Authenticator). Второе, что гейтит 2FA, — защищённые колонки: поля визита с флагом `2FA Protected`, `Email` и `Telegram` юзеров, IP и user-agent в логах, строки прокси. При включённом у аккаунта `2FA Data Protection` (по умолчанию включён) они показаны как `Protected`, пока не ввести код 2FA: код открывает их на ограниченное время (`Unsecured Mode`), а без настроенной 2FA их не открыть. Доставка же алертов бота в Telegram зависит не от 2FA, а от привязки аккаунта к Telegram.

### Почему не приходят алерты: нет привязки к Telegram

**Алерты уходят через `AIO-tech-bot` в Telegram, поэтому без привязки аккаунта к Telegram доставить их некуда.** AIO не может связать аккаунт с TG — и нотификация не уходит. Проверять надо именно наличие TG-привязки у Monitoring User'а: в `Profile settings` на карточке Telegram видно `Telegram is linked to your account`, а без привязки — кнопка `Link Telegram`. Привязка сбрасывается и при смене Telegram-ника в `Profile settings` — вернуть её кнопкой `Link Telegram` ([reference/profile-settings.md](../reference/profile-settings.md)).

2FA через Google Authenticator привязку к TG **не даёт** — с ним 2FA есть, а алерты всё равно не идут, пока нет привязки к Telegram. Раньше привязка к TG была обязательной частью 2FA, поэтому симптом совпадал с «2FA не включена»; теперь эти две вещи разошлись.

### На каком языке приходит алерт

**Отчёты мониторинга `AIO Monitoring · Domains` / `AIO Monitoring · Servers` приходят на английском при любом языке интерфейса.** Сообщение бота о вердикте домен-чекера (`🔴 Domain Incident …`) и уведомления Notification Center рендерятся на языке получателя: язык хранится на его аккаунте и следует за переключателем языка интерфейса, в самом правиле он не задаётся. Что переводится, а что остаётся английским, и почему интерфейс уже на русском, а уведомления продолжают приходить английскими — [mechanics/notification-center.md](../mechanics/notification-center.md).

### Как включить 2FA

Включить 2FA (Telegram):
1. Открыть `Profile settings` (меню профиля, правый верхний угол) → вкладка `Profile` → блок `Select a 2FA authentication method` → карточка `Telegram`.
2. Если Telegram ещё не привязан, нажать `Link Telegram` (или отсканировать QR телефоном) и в открывшемся чате с ботом нажать `Start`. Бот отвечает `Telegram linked to your AIO account.`, а карточка через несколько секунд показывает `Telegram is linked to your account`. Ссылка одноразовая и живёт 15 минут; форма меняет её сама раз в 14 минут, а новую сразу даёт повторное открытие `Profile settings`.
3. Нажать `Enable 2FA` и ввести 6-значный код из бота. Код живёт 5 минут; правила кода — [how-to/registration.md](../how-to/registration.md) → «Код 2FA из Telegram».

Пока Telegram не привязан, `Enable 2FA` для способа Telegram недоступна. Бот отказывает в привязке с `This Telegram account is already linked to another AIO account.`, если этот Telegram уже стоит на другом аккаунте AIO: один Telegram — один аккаунт.

Включить 2FA (Google Authenticator): `Profile settings` → вкладка `Profile` → блок `Select a 2FA authentication method` → карточка `Google Authenticator`. Пока 2FA выключена, под карточкой видны QR-код и строка `Secret`: отсканировать QR или ввести секрет в приложении-аутентификаторе, нажать `Enable 2FA` и ввести 6-значный код в окне `Two factor authentication`. Пока 2FA не включена, каждое открытие `Profile settings` выпускает новый секрет: если окно закрыли до ввода кода, QR сканируют заново.

### Как выключить 2FA или сменить способ (Telegram ↔ Google Authenticator)

Способ 2FA меняется только через выключение текущего. `Disable 2FA` просит код именно включённого способа — 6 цифр от бота в привязанный Telegram-чат или 6 цифр из приложения, — даже если выбрана карточка другого способа; после выключения можно включить другой. Без доступа к привязанному чату бота Telegram-2FA самостоятельно не выключить, а значит и не сменить способ.

При включённой 2FA QR и секрет Google Authenticator не показываются: при включённом Google на карточке видно `Google Authenticator 2FA is enabled`, при включённом Telegram на карточке Google — `Another 2FA method is enabled. Disable it first to set up Google Authenticator`. После выключения Google старая запись в приложении не подойдёт — для повторного включения нужен новый QR. Тексты окна `Two factor authentication`: `Success!` / `2FA has been enabled` или `2FA has been disabled`; при ошибке — `Oops!` / `The entered code is incorrect` и `Try again`.

### Код из Google Authenticator не подходит: `The entered code is incorrect` при включении 2FA

Окно `Two factor authentication` отвечает `Oops!` / `The entered code is incorrect`, если код набран с ошибкой, уже сменился или взят из записи по старому QR. Пока 2FA у аккаунта выключена, каждое открытие `Profile settings` (в любой вкладке, на любом устройстве) и каждое выключение 2FA выпускают новый секрет; пока окно открыто, секрет не меняется — его не перевыпускают ни окно ввода кода, ни `Try again`, ни `Cancel`.

Что сделать: оставить открытым одно окно `Profile settings`, закрыть окно кода кнопкой `Cancel`, удалить в приложении старые записи AIO (подпись — AIO и ваш email); затем, не закрывая `Profile settings`, отсканировать QR, который виден сейчас (или ввести `Secret`), нажать `Enable 2FA` и ввести свежий 6-значный код. Одно нажатие `Try again` при старой записи не поможет. Как ещё спрашивают: «код из Google Authenticator не подходит», «неверный код 2FA». На шаге 2FA при регистрации секрет выпускается один раз — там этот текст значит только неверный или сменившийся код ([how-to/registration.md](../how-to/registration.md)).

## CDN — куда едут лэнды

Когда лэнд загружается ZIP-архивом через `Content → Landings → +Landing`, AIO процессит архив (переписывает пути на CDN, вставляет `{{aio:head}}`/`{{aio}}`, если это разрешает тип лэнда) и выкладывает все файлы на CDN. Детали пайплайна — [models/landing.md](../models/landing.md).

Практические последствия:

- **Лэнд доступен мгновенно отовсюду** — CDN решает географию.
- **Замена сервера не требует переноса лэндов** — они уже на CDN, не на серверах клиента.
- **Через вкладку `CDN files`** можно править отдельные файлы (CSS / HTML / JS) без перезаливки всего архива. Это полезно для мелких правок.

## Защита и устойчивость доменов

### Устойчивость к массовым жалобам на домен

Домен может стать недоступен по внешним причинам, и от этого не должна встать вся связка. Основной инструмент — **шаг `Domain change` во флоу**: он разделяет трекинговый и публичный домен ([models/domain.md](../models/domain.md)), поэтому публичный домен меняется массовым `Replace domain` без переподключения рекламных кампаний. Вторая сторона — настройки шага `Filter` во флоу, они разобраны отдельно.

### Assign under attack (Cloudflare) — Under Attack на домене

`Assign under attack` — пункт контекстного меню домена с DNS-провайдером Cloudflare (в ERP; в меню MTK его нет). Переключает Under Attack mode в Cloudflare: включает, повторный вызов снимает. Во включённом режиме Cloudflare показывает каждому визиту проверку браузера перед открытием страницы.

- Один и тот же пункт — для одного домена и для выделенной пачки.
- Это **первая ступень** защиты от DDoS, рубильник.
- Использовать с осторожностью: captcha рубит обычным юзерам конверсию.

### AS-Net Block-листы

Списки IP-диапазонов прямо в AIO (не в Cloudflare): в ERP это списки типа IP в `Settings → Item lists`, которые подставляют в условие по IP в правилах фильтрации. Поддерживают CIDR (`x.x.x.x/24`) и через тире (`x.x.x.0-x.x.x.255`). Отсечение идёт на стороне AIO, в фильтре флоу, — сетевые запросы до агента оно не останавливает ([how-to/domains.md](../how-to/domains.md)).

### Direct без Cloudflare

Штатный способ подключить домен: при ручном добавлении или покупке оставить `DNS Provider` пустым. SSL-сертификат можно выпустить кнопкой прямо из AIO либо добавить вручную. Запросы идут напрямую на сервер; авто-репойнт и `Assign under attack` у такого домена не работают — сервер меняют пунктом `Change server in AIO`, а A-запись переставляют у себя.

Когда уместен: где Cloudflare-прослойка не нужна, например при низком бан-рейте серверов. Риски: без CF сервер уязвимее к DDoS, а при бан-атаке может лечь весь сервер, а не отдельные домены.

## Domains: trackingDomain vs publicDomain

Если во флоу есть шаг `Domain change`, визит начинается на трекинговом домене, а дальше по флоу переходит на публичный. Это разделяет роли: трекинговый домен зашит в рекламную ссылку, публичный домен открывает лэнд. Публичный домен можно сменить массовым `Replace domain`, не переподключая рекламные кампании. Смена трекингового домена требует перезапуска рекламной кампании (ссылка на стороне площадки поменяется). Один визит может пройти через несколько доменов — это by design. Полная механика — [models/domain.md](../models/domain.md).

## Чекеры доменов

Отдельные интеграции для мониторинга бана доменов (`Tech → Domain checkers`, в MTK — `Settings → Domain checkers`): **`Google Threat`** (API-проверка и браузерная дают разные результаты — дополнительный сигнал, не главный) и **VirusTotal** (лимит по IP запроса, не по ключу; ~100-150 доменов на аккаунт). Подробнее — [how-to/domains.md](../how-to/domains.md).

Рекомендация: **добавить сразу** — обе интеграции бесплатные, и после подключения мониторинг бана работает без дополнительной настройки (из шаблона тенанта чекеры и провайдеры `Tech` не приезжают).

## Что из этого следует на практике

### Серверы агностичны и лэнды на CDN — что это значит при замене сервера

- **Сервера агностичны → можно безопасно пересобирать инфру.** Замена/добавление сервера ничего не ломает в работе уже загруженных лэндов.
- **Repoint Group обязательна → если в группе 1 сервер, защиты нет.** При вопросе «почему репойнт не сработал» — первое, что смотрим: сколько серверов в группе.
- **Файлы лэндов — на CDN (доступны отовсюду), поэтому при смене сервера их не переносят.** Контент визитёру отдаёт агент на сервере (по командам AIO), но гео сервера на скорость открытия влияет слабо: динамические шаги упираются в контрол-плейн AIO, статику кэширует Cloudflare. Разбор и что реально ускоряет открытие — [models/server.md](../models/server.md).

### Алерты не приходят или нужно сменить публичный домен — что делать

- **Нет привязки к Telegram → алерты некуда доставить.** Если «не приходят алерты» — проверяем, привязан ли аккаунт Monitoring User'а к Telegram (алерты идут только в TG). 2FA через Google Authenticator привязку к TG не даёт — с ним алерты не пойдут. Механика — § «AIO-tech-bot и 2FA».
- **Публичный домен меняется массовым `Replace domain`.** Рекламную кампанию перезапускать не надо.

### Интеграции, API и аналитика — практические выводы

- **Все интеграции (Server / Domain / DNS Providers) добавляются по запросу.** Если нужно что-то, чего нет в дефолтном списке, — напишите в чат с командой AIO.
- **API = всё, что есть в UI.** Автоматизировать можно что угодно: интерфейс работает через тот же API, поэтому любое действие из UI воспроизводится тем же запросом ([how-to/api.md](../how-to/api.md)).
- **Аналитика на больших объёмах.** Тормоза при разбивке по `fb_ad_id` за месяц — это природа аналитического хранилища на больших объёмах.

## Дополнительные факты архитектуры

### Multi-tenancy ограничения и tab-sessions

- **Домен может быть только в одном тенанте одновременно**: имя домена уникально на всю платформу, и тот же домен во втором тенанте не завести. Остальные сущности кросс-тенантно изолированы стенами.
- Для одновременной работы в нескольких тенантах в браузере — механизм **локальных tab-session**: выбранный тенант хранится в сессии вкладки, и запросы каждой вкладки идут в её тенант.

### Серверы — нагрузка и uptime

- Статусы мониторинга AIO ставит по доступности агента и месту на диске; нагрузку сервера AIO **видит** (опциональный алерт по загрузке CPU в отдельный чат), но **не управляет** ею — реакция на нагрузку остаётся зоной владельца сервера.
- Страница кампаний постранична — 1000-3000 кампаний не «убивают» AIO. Дистрибуцию для direct-трафика выбирают на домене (`Edit domain` → `Direct traffic distribution`); одна дистрибуция может стоять на любом числе доменов.

### Безопасность входа

- **Self-whitelist по IP есть** — тенант сам ведёт список доверенных IP (`Manage tenant` → многострочное поле `IP address`, по одному IP на строку; правит роль `Admin`/`Owner`). Это не ограничение входа в аккаунт: вайтлист нужен, когда перед AIO стоит собственный прокси-сервер клиента — без него в поле визита `ip_address` попадает IP прокси, а не визитёра. Механика, что при этом ломается и как настроить — [models/server.md](../models/server.md) §«У всех визитов один и тот же IP».
- Вход в UI `app.aio.tech` — логин/пароль + 2FA (Telegram или Google Authenticator).
- AIO **не отдаёт** клиенту токены / API-ключи подключённой Meta Application — скрыты. Альтернатива — тянуть FB-данные из AIO через AIO-API.

### Своя инфраструктура и CDN

- Подключение **своего CDN** сейчас недоступно.
- Свой проксирующий сервер перед AIO-агентом — возможен, такая схема встречается в бою. Сам прокси собирается самостоятельно; что настроить в AIO, чтобы в визит попадал IP визитёра, а не прокси, — [models/server.md](../models/server.md) §«У всех визитов один и тот же IP».

### Лэнды на агенте: pseudo-paths и re-upload

- Лэнд на сервере отдаётся **одним HTTP-response'ом**; пути внутри лэнда — pseudo-path, локальные для этого лэнда. `style.css` разных лэндов на одном домене **не конфликтуют**.
- Каждая загрузка лэнда — **полный re-upload** на CDN + переписывание ссылок (не инкрементально).
- SDK-формы (`Settings → Forms`) подставляются на лэнд **из SDK**, а не из HTML самого лэнда (рендерится HTML-форма, но из SDK). Благодаря этому SDK-форму выбирают и сплитуют на уровне кампании (шаг `Fill form`). Своя HTML-форма лэнда тоже возможна — её сабмит принимает макрос `AIO Non-SDK Form Handler` ([how-to/forms.md](../how-to/forms.md)).

### Антибот-защита агента

Не каждый входящий запрос доходит до флоу: часть агент отбрасывает на входе, ещё **до шага `Filter`**. Настраиваемая фильтрация во флоу — отдельный слой.

### Режимы обработки запроса на агенте — не только отдача лэнда

- Отдать лэнд с CDN — не единственный режим обработки входящего запроса на агенте: какой режим применить на шаге, решает AIO и присылает агенту командой.

### AIO записывает клик, который дошёл до него через агент

- Клик, который агент передал в AIO, записывается визитом, даже если фильтр его срезал (такой визит — trash). Визит не создаётся, если домена нет в AIO или он в архиве, тенант заблокирован, приостановлен или исчерпал триал, AIO не смог определить кампанию и источник, либо агент отбросил запрос на входе. Гипотеза «AIO не принял клик» при дебаге обычно неверна — сначала проверьте эти условия.
- AIO **не whitelist-ит** свои IP в брокерских интеграциях — пул IP большой и динамичный. Альтернатива — поле `Proxy to communicate` (у части интеграций — `Proxy`) в настройках Destination: пуш уходит через ваш прокси со статическим IP ([how-to/destinations.md](../how-to/destinations.md)).

### Косты и профит по FB — через метрику `Meta Spend`, уровень матчинга переключаемый

**Косты и профит по FB — основной способ через метрику `Meta Spend`.** Она считается на лету в момент отчёта, матчит трафик с расходом Meta по идентификаторам кампании / адсета / объявления; спенд, который привязать не к чему, не пропадает, а показывается отдельно — строкой `Out of scope` в расшифровке итога. Рядом стоит revenue — профит и ROI по FB читаются прямо в отчёте по любому разрезу. Уровень матчинга выбирается пунктом `Attribution level` в меню `Table tools` над таблицей (`Campaign` / `Adset` / `Ad`), дефолт — `Campaign`, выбор хранится в браузере — поэтому у двух людей одна таблица может показывать разный `Meta Spend`. Полная механика — [mechanics/meta-spend-allocation.md](../mechanics/meta-spend-allocation.md).

Старый перелив костов **на визиты** (`Costs update strategy` = `AIO Meta`; уровень у него свой и на `Meta Spend` не влияет, разбор — [how-to/campaigns.md](../how-to/campaigns.md)) — устаревший путь: он наполняет метрику `Costs` на визитах для классического ROI в трекере и теряет часть расхода, если по ключу нет визита; ещё работает, но по костам/профиту FB ориентируйся на `Meta Spend`.

### Косты на уровне визита для других источников

Косты для источников, кроме Meta, проставляют вручную через `Update costs` (за период / по полю / Costs или CPC) либо они приходят прямо с кликом, если источник передаёт цену клика в URL-параметрах. ROI по таким костам считается на визитах. Как работает ручной `Update costs` и как введённая сумма разливается по визитам — [how-to/campaigns.md](../how-to/campaigns.md).

### Собственная защита IP серверов

На уровне самих машин IP сервера ничем не прикрыт: с доменом за Cloudflare в режиме `Proxied` = `Yes` (дефолт настройки Cloudflare DNS Provider) реальный IP сервера снаружи не виден; при `Proxied` = `No` и без Cloudflare сервер доступен по IP напрямую.

### Постбэки обрабатываются не мгновенно — задержки при пиковой нагрузке

Постбэк AIO обрабатывает не в момент приёма, а следом за ним — поэтому при пиковой нагрузке возможны задержки.

### Что отсекает трафик автоматически, а что только обогащает визит

- Обогащение (IPInfo, Emailable и проверка WhatsApp в режиме `Manual`) само визит не отсекает — оно пишет данные в поля визита, а решение принимает шаг `Condition` дальше по флоу. Отсекают фильтры шага `Filter` (`AIO Filter Service`, сторонние серверные фильтры) и режим `Auto Reject` у Emailable / проверки WhatsApp.

### Геобалансировка

- Геобалансировка трафика через Cloudflare (multi-A-record) возможна.
- Схема «tracking-домен под рекламу + второй домен под лэнды» (шаг `Domain change`) повышает устойчивость: два домена с разными ролями изнашиваются независимо, и замена публичного не трогает рекламную ссылку.

### Monitoring User через групповой чат

Алерты можно слать не отдельному юзеру, а в общий TG-чат («технический юзер-чат»). Полная пошаговая процедура — [how-to/domains.md](../how-to/domains.md) → «Monitoring User через групповой чат TG».

## Смежные концепты

- *Медиабаинг и место AIO в нём* — что такое медиабаинг и как AIO в него встроен.
- [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) — путь визита от клика до конверсии: `cuuid`/`suuid`, флоу-движок, finger­print.
- [models/permissions-model.md](../models/permissions-model.md) — Tenants, Teams, Positions, Sharing.
- [reference/glossary.md](../reference/glossary.md) — полная терминология (разделы «Инфраструктура», «Трекинг и флоу»).

## Где это видно в работе

- [how-to/domains.md](../how-to/domains.md) — добавление сервера и домена (Buy / Manually), репойнт, чекеры, Replace.
- [models/server.md](../models/server.md) — сервер как сущность (агент, Repoint Group, мониторинг/2FA).
- [models/domain.md](../models/domain.md) — Domain как сущность (репойнт/Repoint Group, `Domain change`, чекеры).
- [how-to/domains.md](../how-to/domains.md) → «Monitoring User» — почему не приходят алерты (привязка Monitoring User к Telegram).
