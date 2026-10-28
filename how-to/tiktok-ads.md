---
id: tiktok-ads
title: Модуль TikTok Ads в AIO — подключение TikTok-аккаунта, кабинеты, кампании и расход TikTok
description: "Модуль TikTok Ads в AIO (ERP, /app/tiktok, пункт TikTok в верхнем меню): подключение аккаунта TikTok for Business через TikTok App (Money Stream Application, OAuth) — Create TikTok account → Connect account; автосинк Auto 30 Minutes (список кабинетов раз в ~30 минут, статистика каждые 10–15 минут), ручной Pull data (Backward for insights, пауза 10 минут); статусы Fine/Issues/Down/Pulling; кабинеты (Account Status, Pull Strategy), кампании (Status, Delivery, Objective, Budget Mode); расход в USD с Commission; права tiktok.*, scopes.tiktok.accounts. Только чтение; расход не раскладывается на визиты и Cost кампаний; не путать с трекером TikTok Events API."
doc_type: how-to
builds: [erp]
related: [google-ads, meta-ads, source-trackers, campaigns, meta-spend-allocation, permissions]
language: ru
updated: 2026-10-28
---

# Модуль TikTok Ads в AIO — подключение TikTok-аккаунта, кабинеты, кампании и расход TikTok

**Модуль TikTok Ads подтягивает в AIO рекламные кабинеты, кампании и статистику TikTok Ads — расход, показы, клики и конверсии TikTok по часам, — чтобы смотреть их, не заходя в TikTok Ads Manager.** Данные идут через подключённый аккаунт TikTok for Business: AIO авторизуется через своё приложение TikTok (`Money Stream Application`) и читает кабинеты, к которым этот аккаунт дал доступ.

Раздел — пункт `TikTok` в верхнем меню ERP рядом с `Meta` и `Google`, путь `/app/tiktok`. Модуль есть только в билде ERP.

**Модуль только читает.** Создавать и останавливать кампании TikTok или менять их бюджеты из AIO нельзя: действия модуля — подключить аккаунт, выбрать интервал обновления и подтянуть данные. Устроен он так же, как Google-модуль ([google-ads.md](google-ads.md)); управление кампаниями из AIO есть в Meta-модуле ([meta-ads.md](meta-ads.md)).

**Модуль TikTok Ads ≠ трекер `TikTok Events API`.** Модуль забирает статистику из TikTok в AIO, а трекер `TikTok Events API` отправляет конверсии AIO в TikTok — он настраивается в `Tracker → Trackers` ([source-trackers.md](source-trackers.md)), и подключать для него модуль не нужно.

## Раздел TikTok: вкладки Accounts, Ad Accounts, Campaigns, Other и права на просмотр

**В разделе `TikTok` четыре вкладки, у каждой своя таблица и своё право на просмотр.** Пункт `TikTok` в верхнем меню виден, если есть хотя бы одно из прав `tiktok.accounts.view`, `tiktok.ad-accounts.view`, `tiktok.campaigns.view`; `/app/tiktok` открывает последнюю открытую вкладку или первую доступную.

- **`Accounts`** (`/app/tiktok/accounts`, `tiktok.accounts.view`) — подключённые аккаунты TikTok for Business: статус, интервал обновления, комиссия, шаринг. Здесь аккаунт подключают и запускают подтяжку.
- **`Ad Accounts`** (`/app/tiktok/ad-accounts`, `tiktok.ad-accounts.view`) — рекламные кабинеты TikTok (advertiser) со статусом в TikTok и метриками.
- **`Campaigns`** (`/app/tiktok/campaigns`, `tiktok.campaigns.view`) — кампании кабинетов со статусом, целью, бюджетом и метриками.
- **`Other`** (`/app/tiktok/other`, тоже `tiktok.campaigns.view`) — статистика TikTok, сгруппированная по времени: день, час, месяц или год.

Кабинеты, кампании и статистика видны только по тем TikTok-аккаунтам, которые видны вам: своим, пошаренным, лежащим в пошаренной папке или всем аккаунтам тенанта при `scopes.tiktok.accounts`.

## Как подключить TikTok-аккаунт: Create TikTok account → Connect account (OAuth TikTok for Business)

**Подключение — два шага на вкладке `Accounts`: завести аккаунт формой `Create TikTok account`, затем пройти авторизацию TikTok for Business через `Connect account`.**

1. Кнопка создания на `Accounts` (право `tiktok.accounts.edit.by-application`) открывает `Create TikTok account` («Create an account via the connected TikTok app.»). Обязательны `Name` и `Money stream application` — приложение TikTok, через которое AIO авторизуется (в списке — `AIO & TikTok`). Необязательные поля: `Description`, теги, `Pulling interval` (по умолчанию `Auto 30 Minutes`), `Commission`. Новый аккаунт получает статус `Issues` с текстом `Waiting for connect.`.
2. ПКМ по строке аккаунта → `Connect account`. В диалоге — поле `Login via TikTok` со ссылкой авторизации и кнопка `Click to Redirect`, которая открывает ссылку в новом окне. Ссылку можно скопировать и открыть в браузере или профиле, где залогинен нужный аккаунт TikTok for Business.
3. В TikTok подтвердите доступ приложению.
4. TikTok возвращает на `/app/tiktok/accounts`: аккаунт получает статус `Fine` с текстом `Connected`, колонка `Connected` включается, и AIO сразу подтягивает список кабинетов, их кампании и статистику за последние три дня и сегодня.

### Commission, Pulling interval, правка и отключение TikTok-аккаунта

**`Pulling interval` и `Commission` задаются в форме аккаунта и правятся пунктом `Edit` в контекстном меню — он открывает ту же форму.**

- **`Pulling interval`** — `Auto 30 Minutes` (AIO обновляет данные сам) или `Manually` (только по `Pull data`). Что и как часто обновляется — секция «Как часто AIO обновляет данные TikTok» этого дока.
- **`Commission`** (`Commission %`) — процент, который AIO прибавляет к расходу аккаунта: `Total Spend` = расход TikTok + расход × `Commission` / 100. Отрицательное значение работает как скидка — в строке аккаунта бейдж с подсказкой `Discount`. Комиссия применяется в момент подтяжки: новое значение действует на дни, подтянутые после правки, а прошлые дни пересчитает `Pull data` с нужной глубиной.

Если в форме сменить `Money stream application`, подключение сбрасывается: статус `Issues`, текст `Waiting for connect.`, нужен повторный `Connect account`. Сохранение формы обнуляет счётчик неудачных подтяжек аккаунта (`Failed Pulls`).

**`Disconnect account`** (ПКМ, виден у подключённого аккаунта) отключает аккаунт: `Connected` выключается, статус `Issues` с текстом `Manually disconnected.`, фоновое обновление останавливается. Уже подтянутые кабинеты, кампании и статистика остаются в таблицах. Вернуть подключение — `Connect account`.

## Статус TikTok-аккаунта и кабинета: Fine, Issues, Down, Pulling — и что делать

**Статус показывает значок в ячейке `TikTok Account` (у кабинета — в ячейке `Ad Account`), текст статуса — в подсказке при наведении: зелёная галочка — `Fine`, жёлтый значок — `Issues`, синяя крутящаяся стрелка — `Pulling`, красный крестик — `Down`.**

- **`Fine`** — `Connected` сразу после авторизации или `No sync issues` после успешной подтяжки.
- **`Issues`** (только у аккаунта) — `Waiting for connect.`: авторизация не пройдена, нужен `Connect account`; `Manually disconnected.`: аккаунт отключили. Если после авторизации вместо AIO открылась страница с `Error: …`, подключение не состоялось; `TikTok did not return auth_code (access denied?)` — доступ в TikTok не подтвердили.
- **`Pulling`** — `Requested to pull...` (ручной `Pull data` принят) или `Pull data...` (подтяжка идёт).
- **`Down`** — подтяжка упала: `Failed: <этап>: <ошибка>` или `Timeout: <ошибка>`. Этап: `Pull Ad Accounts` — список кабинетов, `Pull Campaigns` — кампании, `Pull Spend` — статистика; ответ TikTok приходит как `TikTok API <код>: <сообщение>`. Если при этом выключился `Connected`, TikTok отклонил токен — переподключите аккаунт через `Connect account`.

`Last Sync` — время последней успешной подтяжки; скрытая по умолчанию колонка `Failed Pulls` — неудачи подряд, успешная подтяжка её обнуляет. Не помогает — напишите в чат команде AIO с текстом статуса.

## Как часто AIO обновляет данные TikTok: режим Auto 30 Minutes

**При `Pulling interval` = `Auto 30 Minutes` AIO сам обновляет список кабинетов аккаунта примерно раз в 30 минут, а кампании и статистику каждого кабинета — примерно каждые 10–15 минут.** Название опции относится к списку кабинетов; статистика обновляется чаще.

- **Список кабинетов** — кабинеты, к которым у аккаунта есть доступ в TikTok, с именем, статусом в TikTok, валютой, часовым поясом и компанией. Новый кабинет появляется в `Ad Accounts` с этим обновлением. Кабинет, к которому доступ пропал, из списка сам не удаляется.
- **Кампании** кабинета приходят каждый раз целиком, со статусами и бюджетами.
- **Статистика** — почасовая, за вчера и сегодня по часовому поясу кабинета; эти два дня при каждом обновлении перезаписываются свежими цифрами. Глубже — ручной `Pull data`.

Фоновое обновление пропускает аккаунты с `Manually`, отключённые и архивные аккаунты, объекты в статусе `Pulling` и аккаунт или кабинет с 5 и более неудачами подряд (`Failed Pulls`); у такого аккаунта останавливаются и все его кабинеты. После 5 неудач фон сам не возобновится: счётчик аккаунта обнуляет успешный `Pull data` или сохранение формы аккаунта, счётчик кабинета — успешный `Pull data` по кабинету или по его аккаунту.

## Ручной Pull data в TikTok: глубина Backward for insights и пауза 10 минут

**`Pull data` запускает внеочередную подтяжку: на аккаунте (ПКМ на `Accounts`, виден у подключённого) — список кабинетов, затем кампании и статистика всех его кабинетов; на кабинете (ПКМ на `Ad Accounts`) — кампании и статистика одного кабинета.** Подтяжка идёт в фоне, на это время статус — `Pulling`.

В диалоге два поля:

- **`Backward for insights`** (подсказка `Spend data`) — за сколько дней перетянуть статистику: `1 Day Backward`, `5 Days Backward`, `7 Days Backward`, `1 Month Backward`. Отсчёт — от начала дня по часовому поясу кабинета до сегодня.
- **`Backward for data`** — на подтяжку TikTok не влияет: кампании кабинета и так приходят целиком.

Повтор отбивается двумя ошибками. **`Already pulling`** — объект уже в `Pulling`. **`Should be at least 10 minutes between pull tries`** — с последней успешной подтяжки этого аккаунта или кабинета прошло меньше 10 минут, фоновые обновления тоже считаются. При `Auto 30 Minutes` кабинет и так обновляется каждые 10–15 минут, поэтому ручной `Pull data` нужен в основном, чтобы перетянуть больше двух последних дней (например, после правки `Commission`), или при `Manually`.

## Рекламные кабинеты TikTok (Ad Accounts): Account Status, валюта, Pull Strategy

**`Ad Accounts` — рекламные кабинеты TikTok (advertiser), к которым у подключённого аккаунта есть доступ; AIO заводит их сам при обновлении списка кабинетов, вручную кабинет не добавляется.**

В ячейке `Ad Account` — значок статуса подтяжки, имя (в подсказке компания и часовой пояс), ID и валюта кабинета. Колонки: `TikTok Account`, метрики `TikTok Insights`, `Advertiser ID`, `Account Status`, `Last Sync`; скрытые — `Currency`, `Timezone`, `Company`, `Pull Strategy`, `Pull Status`, `Failed Pulls`.

**`Account Status`** — статус кабинета в самом TikTok, а не статус подтяжки AIO: `Enable` (подсказка `Everything is fine`), `Disable` (`Disabled`), `Limit` (`Limited by TikTok (policy violation)`), `Confirm Fail` и `Confirm Fail End` (`Review failed`), статусы ожидания проверки (`Pending Confirm`, `Pending Verified`, `Wait For Bpm Audit`, `Wait For Public Auth`, `Self Service Unaudited`, `Contract Pending`) и `Unknown`. Неактивный кабинет помечен бейджем прямо в ячейке `Ad Account`.

ПКМ по кабинету — `Pull data` и `Edit`: диалог `Edit TikTok ad account` с полем `Pull strategy` (`Default` или `Light`). Выбор сохраняется в кабинете, на объём подтяжки он не влияет. Архивировать кабинет из интерфейса нельзя.

## Кампании TikTok в AIO: Status, Delivery, Objective, Budget Mode, Budget

**`Campaigns` — кампании всех видимых кабинетов с метриками TikTok; таблица только для просмотра, контекстного меню у кампании нет.**

- **`Status`** — включена ли кампания в TikTok: `Enabled` (подсказка `Active`), `Disabled` (`Paused`) или `Unknown`.
- **`Delivery`** — статус доставки кампании, как его отдаёт TikTok, без префикса `CAMPAIGN_STATUS_` и словами.
- **`Objective`** — цель кампании из TikTok в том же читаемом виде.
- **`Budget Mode`** (скрыта по умолчанию) — режим бюджета: `Day`, `Total` или `Infinite`.
- **`Budget`** — бюджет кампании в долларах: AIO переводит его из валюты кабинета по курсу на момент подтяжки.

Ещё скрыты `TikTok Account` и `Campaign ID`. В ячейке `Campaign` — значок статуса с подсказкой доставки, имя (в подсказке цель и режим бюджета) и ID кампании.

## Метрики TikTok в AIO: Total Spend, Impressions, Clicks, Conversions

**В таблицах `Ad Accounts`, `Campaigns` и `Other` статистика TikTok идёт группой колонок `TikTok Insights`: расход, показы, клики и конверсии по отчёту TikTok за выбранный в таблице период.**

- **`Total Spend`** — расход с комиссией аккаунта (`Total spend with commission`), **`Commission`** — сама комиссия, **`Raw Spend`** (скрыта) — расход без комиссии. Суммы в долларах: AIO переводит расход из валюты кабинета по курсу на момент подтяжки.
- **`Impressions`**, **`Clicks`** — показы и клики из отчёта TikTok; **`CTR`**, **`CPM $`**, **`CPC $`** считаются из них и расхода.
- **`Conversions`** — конверсии, которые насчитал сам TikTok, а не конверсии AIO; **`CPA $`** (скрыта) — расход на такую конверсию.
- **`Conv. Value`**, **`All Conversions`**, **`View-through Conv.`** и **`ROAS`** в списке колонок есть (скрыты по умолчанию), но подтяжка TikTok их не заполняет — там нули.

Самая мелкая единица статистики — час по кампании: разреза по группам объявлений и объявлениям TikTok в модуле нет.

### Разбивка статистики TikTok по дням и часам (Other) и часовой пояс кабинета

**Статистику TikTok раскладывают разбивкой (drill-down) в таблицах раздела: по `Ad Accounts`, `Campaigns` и по времени — `Day`, `Hour`, `Month`, `Year`.** Вкладка `Other` показывает статистику всех видимых кабинетов, сгруппированную по выбранному времени.

TikTok отдаёт почасовую статистику в часовом поясе кабинета (колонка `Timezone` в `Ad Accounts`), и AIO переводит эти часы в абсолютное время. Таблицы AIO считают день в часовом поясе отчёта (по умолчанию — из вашего профиля). Если он не совпадает с поясом кабинета, суммы по дням в AIO и в TikTok Ads Manager расходятся, потому что у суток разные границы. К тому же в AIO расход в долларах и с `Commission`: сверять с TikTok удобнее `Raw Spend` с учётом курса.

## Расход TikTok попадает в Cost кампаний и профит AIO? Нет — он живёт только в разделе TikTok

**Расход TikTok из модуля не раскладывается на визиты и не попадает в `Cost` / `Costs` кампаний, профит и ROI трекера: статистику TikTok показывают только таблицы раздела `TikTok`.** Связки статистики TikTok с визитами AIO нет — ни по макросам ссылки, ни по кампаниям.

- У `Cost Update Strategy` кампании нет TikTok-варианта: там только Meta-уровни и `Manual` ([campaigns.md](campaigns.md)).
- Разнос рекламного расхода по трафику прямо в отчёте (`Meta Spend`) считает только расход Meta ([meta-spend-allocation.md](../mechanics/meta-spend-allocation.md)).
- Разбивки по кабинетам и кампаниям TikTok есть только внутри раздела `TikTok`, в отчётах по визитам их нет.

## Права на модуль TikTok: ключи tiktok.*, scopes.tiktok.accounts, шаринг и папки

**Доступ к модулю раздают в Position ключами `tiktok.*`; все они только для ERP.** Просмотр вкладок — `tiktok.accounts.view`, `tiktok.ad-accounts.view`, `tiktok.campaigns.view`.

- **Аккаунты:** `tiktok.accounts.edit.by-application` — создать и править аккаунт; `tiktok.accounts.edit.connect` / `.disconnect` — `Connect account` / `Disconnect account`; `tiktok.accounts.edit.pull-data` — `Pull data`; `tiktok.accounts.edit.archive` — архив; `tiktok.accounts.edit.assign-folder` — раскладка по папкам; `tiktok.accounts.folders.view` / `.edit` — сами папки; `tiktok.accounts.share` / `tiktok.accounts.share.ownership` — шаринг и смена владельца.
- **Кабинеты:** `tiktok.ad-accounts.edit` — `Edit TikTok ad account`, `tiktok.ad-accounts.edit.pull-data` — `Pull data`.
- **Без действий в интерфейсе:** `tiktok.ad-accounts.edit.archive`, `tiktok.campaigns.edit`, `tiktok.campaigns.edit.pull-data`.
- **`Scopes: TikTok accounts`** (`scopes.tiktok.accounts`) — уровень `View` на все TikTok-аккаунты тенанта; **`Editor Scopes: TikTok accounts`** (`edit-scopes.tiktok.accounts`) — `Own`.

**TikTok-аккаунт шарится поштучно:** в контекстном меню одиночные `Share` и `Change ownership` (нужен доступ `Own`), массовых действий шаринга нет. Несколько аккаунтов сразу — через папку: шаринг папки TikTok-аккаунтов раздаёт доступ её содержимому. Роли шаринга и папки — [permissions.md](permissions.md).

## Аккаунт TikTok в Fine, а свежей статистики или кабинета нет — что проверить

**Если статус `Fine`, а данных нет или они старые, причина обычно в режиме обновления, счётчике неудач или доступе.**

- **`Sync Interval` = `Manually`** — фонового обновления нет, данные приходят только по `Pull data`.
- **`Failed Pulls` 5 и больше** у аккаунта или кабинета (колонка скрыта по умолчанию) — фон их пропускает. Успешный `Pull data` обнуляет счётчик и возвращает автообновление.
- **Нет статистики за дни до подключения** — при подключении AIO берёт последние три дня и сегодня, дальше фон ведёт вчера и сегодня. Глубже, до месяца, — `Pull data` с `Backward for insights` = `1 Month Backward`.
- **Нужного кабинета нет в `Ad Accounts`** — AIO видит только кабинеты, к которым у подключённого аккаунта TikTok есть доступ. Список обновляется раз в ~30 минут или по `Pull data` на аккаунте; кабинет, доступ к которому выдали в TikTok позже, появится после обновления.
- **Вкладки нет или таблица пустая** — проверьте права `tiktok.*.view` в Position и видимость аккаунта: кабинеты и статистика показываются только по своим, пошаренным или открытым через `scopes.tiktok.accounts` аккаунтам.
- **Суммы не совпадают с TikTok Ads Manager** — в AIO расход в долларах и с `Commission`, а день считается в часовом поясе отчёта AIO (секция «Разбивка статистики TikTok по дням и часам» этого дока).
