---
id: permissions-model
title: Permissions Model AIO
description: Модель доступа AIO — уровни доступа к объекту Use / View / Edit / Own и откуда они берутся (владение, команды с числовыми уровнями, Peers / Below, шаринг, Access type, Scopes / Editor Scopes, роли Owner / Admin / Employee), права Positions и Imperatives с серверной проверкой ключей, Your access, Change ownership, копирование, Log in as user и дословные тексты отказов.
doc_type: model
builds: [erp, mtk]
related: [architecture, generations, analytics, limits, postback-generator, session-analytics, comments, remarketing-campaigns, permissions, auto-rules, meta-ads, auto-rules-engine, notification-center, api, campaigns, registration, custom-fields, tenant, user, glossary, how-to-pwa]
language: ru
updated: 2026-10-16
---

# Permissions Model AIO

Доступ в AIO проверяется по двум независимым осям. **Права позиции** — роль плюс Positions с Imperatives — решают, какие разделы и какие действия юзеру вообще открыты. **Уровень доступа к объекту** — `Use`, `View`, `Edit` или `Own` — решает, что юзер может сделать с конкретной кампанией, лэндом или доменом; он складывается из владения, команды, шаринга, `Access type`, `Scopes` / `Editor Scopes` и роли. Для Employee действие, закрытое ключом Position и уровнем объекта, требует обоих разрешений; Owner / Admin проходят эти проверки по роли. Когда объект не виден, не открывается или не сохраняется, первым смотрят строку `Your access` в контекстном меню объекта и точный текст отказа.

## Какие уровни доступа к объекту бывают — `Use`, `View`, `Edit`, `Own`

**Для сущностей с объектным доступом итоговый уровень — `Use` < `View` < `Edit` < `Own` либо `none`, если доступа нет; каждый следующий уровень включает предыдущие. Для Employee действия с настроенной проверкой ключа Position или Feature-права дополнительно требуют этого разрешения; Owner / Admin проходят эти проверки по роли.**

| Уровень | Что даёт |
|---|---|
| `Use` | Объект виден в списках и селектах, его можно выбрать в своей кампании или форме. Карточка не открывается, настройки закрыты. |
| `View` | Плюс открыть карточку только на чтение, превью и скачивание лэнда, записи сессий, копирование (при праве копирования в Position). |
| `Edit` | Плюс сохранять настройки, теги, цвет, папку и запускать операционные действия. |
| `Own` | Плюс `Change ownership` и смена `Access type`, архив, раздача доступа другим. |

Отдельно от уровня есть флаг «может раздавать» — `Can share further` в диалоге `Share`. При уровне `Own` он есть всегда, при шаринге выдаётся отдельной галкой. Без `Own` и без этого флага пункта `Share` на объекте нет.

### Откуда складывается уровень доступа — владение, команда, шаринг, `Access type`, позиция

**Итоговый уровень — максимум из всех источников, которые есть у юзера на этот объект.** Источник подписан в подсказке `Your access` (подпись — в скобках):

- владелец объекта — `Own` (`owner`);
- участник команды владельца с более высоким уровнем — `Own` на объекты нижестоящих (`subordinate’s object`);
- участник того же уровня — что выставлено уровню кнопкой `Peers:`, по умолчанию ничего (`peer in your level`);
- `Access type` объекта — `Use` или `View` для всех в тенанте (`for everyone`);
- шаринг лично юзеру — уровень роли шары: `Use`, `View` (роль `Viewer`) или `Edit` (роль `Editor`) (`shared with you`);
- шаринг любой команде, где юзер состоит, — уровень роли шары, уровень юзера в команде не важен (`shared with your team`);
- шаринг нижестоящему или его команде — только если уровню выбран режим `Below:` с шарами (`subordinate’s object` / `shared with your subordinate’s team`);
- `Scopes: <раздел>` в Position — `View` на все объекты раздела, `Editor Scopes: <раздел>` — `Own` (`position`);
- роль `Owner` или `Admin` — `Own` на всё (`full access`).

Шаринг не даёт больше `Edit`, `Access type` — больше `View`. Истёкшая шара (`Expires` в прошлом) не даёт ничего.

## `Your access` — как узнать свой уровень доступа к объекту и откуда он

**В таблицах с колонкой объектного доступа внизу контекстного меню строки (правый клик) показано `Your access: <уровень>` — `Use`, `View`, `Edit`, `Own` или `none`; при наведении видно, откуда доступ и что он разрешает.** Разрешения перечислены с отметками: `open the card`, `edit`, `share`, `ownership & archive`. Источник подписан одним из вариантов: `owner`, `subordinate’s object` (вы выше владельца в его команде), `peer in your level`, `for everyone` (`Access type`), `shared with you`, `shared with your team`, `shared with your subordinate’s team`, `position` (`Scopes` / `Editor Scopes`), `full access`. Пункты меню, на которые уровня не хватает, скрыты. RU-подпись — `Ваш доступ: <уровень>`. Это первое, что проверяют, когда объект «не открывается», «нет кнопки `Share`» или «не сохраняется».

### Иконка строки, `Read-only` и тост `Sorry, you do not have access`

**Иконка действия в строке показывает, что будет по клику: карандаш `Edit` — карточка откроется на правку, глаз `Open` — только на чтение, серый перечёркнутый глаз `No access to open` — открыть нельзя.** В карточке, открытой на чтение, вместо кнопки сохранения стоит `Read-only` с подсказкой `You have no edit access to this object`.

Тост `Sorry, you do not have access` значит, что карточку объекта открыть не удалось: интерфейс сам не открывает карточку ниже нужного уровня, а на любой отказ `403` при загрузке формы — нехватку уровня на объект или ключа в Position — показывает этот тост вместо текста сервера. Дословный текст сервера (`Action access denied (…)`, `No access to this object (…)`) виден, только когда отказ приходит на сохранение или запуск действия. Страница-заглушка `Sorry, you do not have access to this section` — другое: у Position нет права на весь раздел.

## Как Roles задают базовый доступ — Owner, Admin, Employee

| Роль | Доступ |
|---|---|
| **Owner** | Полный доступ: `Own` на все объекты тенанта и все права Position без назначения позиций. По дефолту — тот, кто создал тенант. |
| **Admin** | Полный доступ, как у Owner. Управляет юзерами в `Settings → Users`: приглашает, меняет роль, отключает. |
| **Employee** | Только то, что дают его Positions и лестница уровней на объекты. |

**Что даёт полный доступ (Owner / Admin в текущем тенанте):**
- уровень `Own` на все объекты тенанта независимо от автора — в `Your access` источник `full access`;
- все права Position — серверная проверка ключей на действия к этим ролям не применяется;
- приглашение юзеров, смена ролей, назначение Positions, создание команд, биллинг, `Log in as user` (в ERP);
- пункт `Manage tenant` в меню профиля в правом верхнем углу — настройки самого тенанта.

### Что видит Employee — свои объекты, команда, шаринг, `Access type`, `Scopes`

**Employee видит объект, только если у него есть на объект хотя бы уровень `Use` из какого-то источника.** Это:
- свои объекты;
- объекты участников ниже него по уровню в его командах;
- объекты равных по уровню — если уровню так выставлено `Peers:`;
- всё с `Access type` `Use` (в списках) или `View` (на чтение);
- объекты, пошаренные лично ему или любой его команде;
- шары его нижестоящих — только при режиме `Below:` с шарами;
- все объекты разделов, где в Position есть `Scopes` / `Editor Scopes` (сам раздел открывает только его ключ страницы — секция «Как дать доступ ко всем объектам раздела без шаринга»).

Объект с `Access type = Private` без шаринга видят владелец, участники выше него по уровню в его командах, равные ему по уровню (если уровню так выставлено `Peers:`), позиции со `Scopes` / `Editor Scopes` на раздел и Owner / Admin.

## `Access type` — доступ для всех в тенанте: `Private`, `Use`, `View`

**`Access type` задаёт уровень, который получают на объект все юзеры тенанта: `Private` — никакого, `Use` — объект виден в списках и селектах, `View` — все могут открыть карточку на чтение.** Правку `Access type` не даёт никогда: для правки нужны владение, шара `Editor`, более высокий уровень в команде владельца, `Peers:`, `Editor Scopes` или роль Owner / Admin.

В таблицах и в селекте диалога `Change ownership` значения подписаны `Private` / `Use` / `View` (RU `Приватный` / `Использование` / `Просмотр`), в данных и API — `By Share` / `Everyone` / `View`. Подсказки в интерфейсе: `Private` — «Only the owner, shares and superiors by team», `Use` — «Everyone sees it in lists and selects, settings are closed», `View` — «Everyone can open it read-only». Меняется `Access type` в диалоге `Change ownership`, поэтому для смены нужен уровень `Own`. Когда нужно, чтобы все видели объект, но никто не менял, — это `View`.

### У каких сущностей есть `Access type` и какое значение по умолчанию

**`Access type` есть у всех сущностей, которые шарятся, и правило для всех одно: `Use` и `View` значат одно и то же, исключений по сущностям нет.** Это кампании, флоу, источники, дестинейшены, лэнды, Content Library, пресеты, поля, метрики, дистрибуции, Money Streams, внешние отчёты, списки, шаблоны сообщений, Meta-профили, Meta-шаблоны, Google-аккаунты, ассайны автоправил, домены, сервера, прокси, провайдеры доменов, серверов и DNS, а также папки, макросы, формы и адвертайзеры, а в ERP — ещё генерации и генераторы.

Значение по умолчанию у большинства — `Private`; у папок, макросов, форм, адвертайзеров и списков (`Item lists`) — `Use`, поэтому их видят в списках и селектах все. `Use` получают и поля, которые создаются сами, когда источник создают из шаблона. Генераторы, заведённые до того, как у них появились владелец и `Access type`, принадлежат Owner-у тенанта и открыты всем с `Use`; новые генераторы и генерации создаются `Private`. У элемента Content Library `Access type` выбирают ещё в форме создания (по умолчанию `Private`). У кампаний (трафиковых и remarketing), источников, лэндов, макросов, форм, адвертайзеров, прокси, Meta-профилей, Meta-шаблонов и Google-аккаунтов колонка `Access type` в таблице по умолчанию скрыта и включается в настройках колонок.

## Как Positions кастомизируют доступ Employee

Positions — это то, чем тонко настраивается доступ Employee'ов (Owner/Admin Positions не нужны — у них и так всё открыто).

Создаются в `Settings → Positions`. Каждая Position — это набор **Imperatives** (см. ниже) с маркерами **Allow** или **Deny**. Position решает, какие разделы и действия юзеру открыты; с какими объектами он может их выполнить, решает уровень доступа к объекту.

### Какие Positions уже есть в новом тенанте — набор из шаблона

Новый тенант получает Positions из шаблона тенанта, базовый набор заводить с нуля не нужно. В шаблонах ERP их семь, по возрастанию `Priority`: `Full Access` (10), `Team Lead` (20), `Buyer` (30), `Tech` (40), `Creative` (50), `Analyst` (60), `Finance` (70); в шаблонах MTK — пять, без `Creative` и `Finance`. Набор одинаков у всех вертикалей одного билда. У каждой позиции в `Description` одной строкой сказано, для кого она: `Buyer` — «Launches and runs traffic. Sees only their own campaigns, no team data.», `Analyst` — «Reads everything across the tenant and changes nothing.», `Tech` — «Domains, servers, proxies and landing setup. No ad accounts, no money.», `Full Access` — «Everything the product can do. Owner and Admin already have it by role.» (позиция для сотрудника с ролью `Employee`, которому нужно всё). Это обычные Positions: их правят, переименовывают и удаляют как свои. Что ещё копируется из шаблона — [context/architecture.md](../context/architecture.md).

### Как назначить Position юзеру

В инвайте выбирается одна Position; назначить юзеру позицию после регистрации — вкладка `Assigned users` самой Position (`Settings → Positions`). Юзер может иметь **несколько Positions**.

### Как Priority разрешает конфликт между Positions

**При конфликте разных императивов между Positions выигрывает более высокий `Priority`: позиции применяются по возрастанию. Это правило предсказуемо, когда сами ключи императивов не дублируются.** Пример: позиция с `Deny tech` (`Priority: 20`) и позиция с `Allow tech.domains` (`Priority: 40`) — у юзера с обеими ключи `tech.domains.*` разрешены, потому что разрешающая применяется позже.

**Диапазон `Priority`-слайдера:** min `10`, max `500`, шаг `10` (слайдер стартует с `10`). Форм-дефолт значения — `1`, но сам слайдер начинается с `10`, поэтому практический минимум при перетаскивании — `10`.

## Какие Imperative Types существуют и чем отличаются по масштабу

Imperative — это «единица разрешения» внутри Position. Они отличаются **масштабом** (от крупного к мелкому); за каждой подписью в матрице стоит ключ, который сервер называет в тексте отказа:

| Тип | Что | Пример подписи (ключ) |
|---|---|---|
| Sections | Целая секция со всеми подсекциями и действиями | `Analytics section`, `Content section`, `Meta section` |
| Subsections | Подсекция со всеми её страницами и действиями | `Content: landings`, `Content: macros`, `Settings: distributions` |
| Pages | Только просмотр конкретной страницы | `Content: CDN files page` (`content.cdn-files.view`), `Tracker: campaigns page` |
| Actions | Конкретное действие | `Content: landings edit`, `Content: landings archive`, `Tracker: campaigns update costs` |
| Features | Спец-фича, закрытая отдельно от раздела | `Features: heatmaps`, `Features: table export` |
| Sharing & Ownership Change | Шаринг (`Share`, `Share also`, `Force share`, `Unshare`) и `Change ownership`; «пошарить на команду», «расшарить сервер» | `Content: landings share`, `Tech: servers ownership change` |
| Scopes | `View` на все объекты раздела без шаринга («view-only доступ», «смотреть стату баеров без edit»); сам раздел не открывает — только вместе с его ключами | `Scopes: tracker campaigns` |
| Editor Scopes | `Own` на все объекты раздела без владения и шаринга; действия раздела по-прежнему требуют своих ключей | `Editor Scopes: content landings` |

**Allow / Deny toggle:** в одной Position можно скомбинировать `Allow Section X` + `Deny Subsection Y внутри X` — тогда юзер получает всю секцию X, кроме Y. Так у позиции `Team Lead` из шаблона тенанта (в обоих билдах) закрыты юзеры и `Streams`: Allow на всю секцию `Settings`, а ниже Deny на `settings.users` (`Settings: users`) и `settings.streams` (`Settings: streams`) — тимлид ходит по настройкам, но `Settings → Users` и `Streams` не видит.

### Какой ключ Position проверяет каждое действие — серверная проверка

**Для действий, закрытых ключом Position, сервер берёт ключ из общей карты, а интерфейс использует эту карту для пунктов меню; Owner / Admin проходят проверку ключа по роли.** Ключом действия становится самый узкий существующий ключ раздела, поэтому Allow на раздел или подраздел покрывает все его действия; ветки `Scopes` / `Editor Scopes` ни за какой ключ действия не засчитываются. Открыть форму существующего объекта можно с `<раздел>.view`, сохранить — только с ключом `.edit…`. Режим `Read-only` в карточке задаёт уровень на объект, а не ключ позиции: при уровне `Edit` / `Own` без ключа правки карточка открывается на правку, а сохранение отказывает. Нет ключа — `403` `Action access denied (<ключ> required)`, ключ назван в тексте.

Неочевидные ключи:
- правка и создание тегов — `settings.tags.edit` (`Settings: tags edit`), одного просмотра тегов мало;
- повтор регистрации домена — `tech.domains.edit.buy-domain` (`Tech: domains buy domain`);
- копирование — `<раздел>.edit.copy` (например, `Tracker: campaigns copy`), в разделе без такого ключа — `<раздел>.edit`;
- смена владельца — `<раздел>.share.ownership` (например, `Tech: servers ownership change`), в разделе без такого ключа — `<раздел>.share`, а без него — `<раздел>.edit`;
- приглашение юзера и смена роли — только Owner / Admin, никаким ключом Position не выдаются.

### Какие ключи прав появились и какие убраны — логи, комментарии (бывшие заметки), PWA, Suppliers

**Отдельных прав `<раздел>.view.show-logs` больше нет. Страница `Settings → Logs` и обычные меню логов объектов требуют `settings.logs.view` (`Settings: logs page`); в `Settings → Users` меню логов проверяет `settings.users.view`.** Императив на удалённый `….view.show-logs` больше ничего не открывает. Новые ключи:

- `content.landings.view.pwa` (`Content: PWA page`) — страница `Content → PWA` (в MTK — `PWA → PWA`). Ключ вложен в `content.landings.view`, поэтому открывается вместе с лэндами, но отнимается отдельно (`Deny content.landings.view.pwa`) или выдаётся без `Landings`.
- `content.landings.edit.custom-pwa` (`Content: landings custom PWA`) — создание PWA из ZIP (`Custom PWA`); конструктор PWA — `content.landings.edit.pwa`.
- `meta.suppliers.view` / `meta.suppliers.edit` (`Meta: suppliers page` / `Meta: suppliers edit`) — поставщики рекламных аккаунтов (`Manage suppliers`).
- `tech.servers.edit.place-monitor` (`Tech: servers place monitor`) — размещение мониторинга на сервере.
- `settings.forms.share` / `settings.forms.share.ownership` и `settings.advertisers.share` / `settings.advertisers.share.ownership` — шаринг и смена владельца форм и адвертайзеров (только ERP).
- `features.notes` (`Features: notes`) — теперь комментарии к кампаниям и лэндам, в ERP и MTK; `features.notes.edit` (`Features: notes edit`) остался от заметок и на комментарии не влияет.
- `content.generations.*` и новые `settings.generators.*` — генерации и генераторы (секции «Какими ключами закрыты генерации» и «Какими ключами закрыты генераторы»); ключ `content.landings.edit.generate` удалён.

Позиция с Allow на раздел получает новые ключи этого раздела автоматически, по префиксу.

### Какими ключами закрыты генерации — `content.generations.*`

**Страница `Content → Generations` открывается ключом `content.generations.view` (`Content: generations page`); запуск, повтор, отмена и проверка генерации — `content.generations.edit` (`Content: generations create & edit`), архив — `content.generations.edit.archive`, папка — `content.generations.edit.assign-folder`, шаринг — `content.generations.share`, смена владельца — `content.generations.share.ownership`, список папок — `content.generations.folders.view` / `content.generations.folders.edit`.** Страница есть только в ERP; в наборе прав MTK ветка тоже объявлена, но страницы генераций там нет.

Лэнд из готовой генерации создаётся обычной загрузкой архива в `Landings`, поэтому для этого пункта нужно ещё `content.landings.edit.upload` (`Content: landings upload`). Ключа `content.landings.edit.generate` больше нет — генерации лэнда в разделе лэндов тоже нет. Уровень на все генерации тенанта дают ветки `Scopes: generations` (`View`) и `Editor Scopes: generations` (`Own`), только ERP и только вместе с ключами раздела. Как устроены сами генерации — [how-to/generations.md](../how-to/generations.md).

### Какими ключами закрыты генераторы — шаринг и владение `settings.generators.*` (ERP)

**Генераторы (`Settings → Generators`) — сущность с владельцем, `Access type` и шарингом: к `settings.generators.view` (`Settings: generators page`) и `settings.generators.edit` добавились `settings.generators.edit.archive` (`Settings: generators archive`), `settings.generators.share` (`Settings: generators share`) и `settings.generators.share.ownership` (`Settings: generators ownership change`).** На странице генераторов есть `Share`, массовые `Share also` / `Force share` / `Unshare`, `Change ownership` и архив — с теми же правилами уровней, что у остальных сущностей. Уровень на все генераторы тенанта дают ветки `Scopes: generators` (`View`) и `Editor Scopes: generators` (`Own`) — вместе с ключами раздела.

### Выгрузка таблицы в CSV — отдельное право, а не следствие доступа к таблице

Экспорт таблицы гейтится собственным Feature-императивом: `features.table-export`, в матрице `Settings → Positions` — `Features: table export` (RU `Функции: экспорт таблиц`). **Право видеть данные на экране не даёт права забрать их файлом** — это сделано отдельно специально. Императив есть в обоих билдах, ERP и MTK; Owner и Admin проходят автоматически.

Симптом: «кнопки `Export as CSV` в таблице нет» или «выгрузка ругается на роль». Без права интерфейс кнопку не показывает вовсе ([how-to/analytics.md](../how-to/analytics.md)), а запрос, ушедший мимо интерфейса, отвечает `403` с текстом `Table export is not allowed for your role`. Проверять надо не доступ к таблице, а наличие этого императива в Position. Сам потолок выгрузки — [reference/limits.md](../reference/limits.md).

### Генератор постбэков — отдельное право, а не следствие доступа к конверсиям

Экран `Postback generator` на странице `Tracker → Conversions` закрыт собственным Feature-императивом: `features.conversion-postback-generator`, в матрице `Settings → Positions` — `Features: postback generator` (RU `Функции: генератор постбеков`). Императив заведён в обоих билдах, ERP и MTK; Owner и Admin проходят автоматически.

Разделение то же, что у выгрузки таблицы: видеть конверсии и уметь выдать наружу ссылку, которой конверсии создаются, — разные права.

Симптом со стороны прав: кнопки `Postback generator` на странице конверсий нет или она не нажимается, а запрос, ушедший мимо интерфейса, отвечает `403` с текстом `Postback generator is not allowed for your role`. Причина — отсутствующий (или перекрытый `Deny`) императив в Position. Как ведёт себя кнопка и что именно собирает генератор — [how-to/postback-generator.md](../how-to/postback-generator.md).

### Какие фичи закрыты отдельными Feature-императивами — полный список

Feature-императивов восемь, каждый закрывает свою фичу отдельно от доступа к разделу, где она живёт:

- `Features: sessions` (`features.sessions`) — записи сессий ([how-to/session-analytics.md](../how-to/session-analytics.md)); только в билде ERP.
- `Features: heatmaps` (`features.heatmaps`) — тепловые карты лэнда: вкладка `Heatmap` в `Manage landing` и в окне `Landing quick preview`; только в билде ERP.
- `Features: notes` (`features.notes`) — комментарии к кампаниям и лэндам и лента `All comments` ([how-to/comments.md](../how-to/comments.md)); в ERP и MTK.
- `Features: notes edit` (`features.notes.edit`) — остался в матрице от заметок, которые заменены комментариями, и в интерфейсе ни на что не влияет; только в билде ERP.
- `Features: conversions revenue` (`features.conversions-revenue`) — колонки `Revenue` и `Revenue Business Model` в `Tracker → Conversions`. Без права этих колонок в таблице нет, хотя сама таблица открывается.
- `Features: conversions payout` (`features.conversions-payout`) — колонки `Payout` и `Payout Business Model` там же, на тех же условиях.
- `Features: table export` (`features.table-export`) — выгрузка таблицы в CSV ([how-to/analytics.md](../how-to/analytics.md)).
- `Features: postback generator` (`features.conversion-postback-generator`) — экран сборки ссылки входящего постбэка ([how-to/postback-generator.md](../how-to/postback-generator.md)).

Права раздаются по префиксу, поэтому `Allow features` или `Allow features.notes` включает и `features.notes.edit`. Подпись `Features: notes` осталась от заметок: теперь это право на комментарии, и комментарии видны юзеру только к тем кампаниям и лэндам, к которым у него есть доступ (уровень `Use` и выше). Правка и удаление комментария от `Features: notes edit` не зависят: править может только автор, удалять — автор, владелец объекта и Owner / Admin. Вкладку `Map comparison` в `Analytics → Session Analytics` право `Features: heatmaps` не закрывает — её открывает право страницы `Analytics: sessions`.

В обоих билдах, ERP и MTK, заведены комментарии, колонки `Revenue` / `Payout`, выгрузка таблицы и генератор постбэков; записи сессий, хитмапы и `Features: notes edit` — только в ERP, и в MTK редактор позиции в `Settings → Positions` их не предлагает.

### Ремаркетинг закрыт своими ветками прав — доступ к трафиковым кампаниям его не даёт

Раздел `Remarketing` гейтится собственными ветками прав, независимыми от трафиковых `tracker.*`: `marketing.campaigns.*` — remarketing-кампании, `marketing.template-distributions.*` — контент рассылок. В матрице `Settings → Positions` обе лежат в секции `Marketing section`: у `marketing.campaigns.*` подписи вида `Marketing: campaigns view`, `Marketing: campaigns edit`, `Marketing: campaigns share`, `Marketing: campaigns ownership change`, у `marketing.template-distributions.*` — `Marketing: template distributions page`, `Marketing: template distributions edit`, `Marketing: template distributions share`, `Marketing: template distributions ownership change`. Отдельно от них заведены права на шаринг флоу рассылок — `marketing.flows.share` (`Marketing: flows share`) и `marketing.flows.share.ownership` (`Marketing: flows ownership change`).

Страница аналитики рассылок `Other` закрыта правом `marketing.campaigns.view.analytics` (лейбл `Remarketing: analytics pages`). Права раздаются по префиксу, поэтому `Allow marketing.campaigns.view` (как и `Allow marketing.campaigns` или `Allow marketing`) включает и его — страница `Other` открывается вместе с кампаниями. Отдельным ключом страницу можно выдать без страницы кампаний или отнять: `Deny marketing.campaigns.view.analytics` ниже Allow. Пункт `Other` пропадает, если стоит такой Deny или если в Position выдан не view-ключ (например, только `marketing.campaigns.edit`).

Ветки `marketing.campaigns.*` и `marketing.template-distributions.*` объявлены в обоих билдах — в MTK они работают внутри вкладки `PWA` платного модуля `PWA + Push`; остальные ветки раздела (`marketing.flows.*`, `marketing.messages.*`, `marketing.message-templates.*`, `marketing.sender-providers.*`) есть только в ERP. Роли полного доступа (Owner / Admin) проходят по всем автоматически, как и по остальным императивам.

Отдельных `Scopes` для remarketing-кампаний и флоу рассылок нет: на уровне объектов remarketing-кампании относятся к разделу `tracker.campaigns`, а флоу рассылок — к `tracker.flows`. Поэтому `Scopes` / `Editor Scopes` на трафиковые кампании и флоу открывают и объекты ремаркетинга (вместе с их аналитикой), но кнопки действий в разделе `Remarketing` всё равно требуют ключей `marketing.*`. У шаблонов сообщений свои ветки — `scopes.marketing.message-templates` / `edit-scopes.marketing.message-templates`.

Наборы шаблонов сообщений технически — дистрибуции типа `Message Templates`. Страницу наборов и их дерево открывают ключи `marketing.template-distributions.*`, а действия с самими наборами — создание, правку узлов дерева, дублирование, архив, шаринг и смену владельца — сервер проверяет ключами дистрибуций `settings.distributions.*`, а `marketing.template-distributions.share*` ни одного действия не открывают. По объектам наборы открывает `Scopes` / `Editor Scopes` на `settings.distributions`. Что делает сам раздел и как выглядит каждая его страница — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md); процедура шаринга и скрытия флоу рассылок — [how-to/permissions.md](../how-to/permissions.md).

### Automations закрыт своей веткой прав `automation.*` — доступ к Meta его не даёт

Раздел `Automations` (автоправила над сущностями Meta) гейтится собственной веткой прав `automation.*`, независимой и от трафиковых `tracker.*`, и от `settings.distributions.*`, и от `meta.*`. Права на рекламные кабинеты и страницы Meta автоправил не открывают, и наоборот — право на автоправила само по себе доступа к Meta-страницам не даёт. Ветка объявлена только в ERP-наборе, в MTK её нет.

Отдельного тумблера доступности у раздела нет: включать его тенанту не нужно, видимость решают только права. Роли полного доступа (Owner / Admin) проходят по ветке автоматически, как и по остальным императивам. Симптом «у сотрудника нет раздела `Automations`» разбирается его Position, а не настройкой тенанта. Что делает сам раздел — [how-to/auto-rules.md](../how-to/auto-rules.md).

### Какие императивы `automation.*` бывают — четыре группы

Императивов шестнадцать. В матрице `Settings → Positions` они собраны в секцию `Automation section` (RU `Раздел «Автоматизации»`), подписи гранулярных — вида `Automation: view templates`, `Automation: edit assignments`, `Automation: approve verdicts`.

- Шаблоны правил — `automation.templates.view`, `automation.templates.edit`, `automation.templates.edit.duplicate`, `automation.templates.edit.archive`, `automation.templates.share`, `automation.templates.share.ownership`.
- Ассайны — `automation.assignments.view`, `automation.assignments.edit`, `automation.assignments.edit.archive`, `automation.assignments.edit.test-run`, `automation.assignments.share`, `automation.assignments.share.ownership`.
- Журнал вердиктов — `automation.actions.view`, `automation.actions.edit.approve`, `automation.actions.edit.decline`. Права `.approve` / `.decline` гейтят кнопки на странице `History`.
- Страница статуса — `automation.status.view`.

Правка дерева шаблона (фазы и правила) идёт по `automation.templates.edit` — отдельного права на узлы дерева нет. Не все ключи ветки что-то проверяют: `automation.templates.share*` и сам `automation.assignments.share` не открывают ни одного действия, а шаринг, владение, дублирование и архив шаблонов проверяются ключами дистрибуций — секция «Шаблоны автоправил шарятся, ассайны — нет». Какие императивы собрать в Position под типовые роли — [how-to/permissions.md](../how-to/permissions.md).

### Шаблоны автоправил шарятся, ассайны — нет: какими ключами проверяются шаринг и владение

**Страница `Rule Templates` — это таблица дистрибуций с зашитым типом `Auto Rules`, поэтому действия владения с шаблоном проверяются ключами дистрибуций, а не `automation.templates.share*`.** `Share`, `Share also`, `Force share`, `Unshare` требуют `settings.distributions.share` (`Settings: distributions share`), `Change ownership` и массовый `Mass change ownership` — `settings.distributions.share.ownership` (`Settings: distributions ownership change`). Ключи `automation.templates.share` и `automation.templates.share.ownership` объявлены в матрице, но ни одного действия не открывают. При запуске из интерфейса дублирование и архив шаблона требуют двух ключей: `automation.templates.edit.duplicate` / `automation.templates.edit.archive` на странице и `settings.distributions.edit.duplicate` / `settings.distributions.edit.archive` в общей проверке действия.

У ассайнов из этого набора есть только `Change ownership` (право `automation.assignments.share.ownership`; пункт виден тому, у кого на ассайн уровень `Own`): в диалоге меняются владелец, `Access type` и галка `Keep Editor access for the previous owner`. Действий `Share` / `Share also` / `Force share` / `Unshare` на странице ассайнов нет — адресно пошарить ассайн конкретному юзеру или команде из интерфейса нельзя. Сам ключ `automation.assignments.share` ни одно действие не проверяет, но `Allow automation.assignments.share` по префиксу включает и `automation.assignments.share.ownership` — то есть открывает `Change ownership`.

### Кто видит ассайн автоправила и как открыть сотруднику все ассайны тенанта

**Ассайн видят по общей лестнице уровней: владелец, участники с более высоким уровнем в его команде, равные по уровню (если так выставлено `Peers:`), позиции со `Scopes` / `Editor Scopes` на ассайны, Owner / Admin и все в тенанте при `Access type` `Use` (в списке) или `View` (карточка на чтение).** Новый ассайн создаётся с `Access type = Private`. Правку чужого ассайна `Access type` не даёт.

Чтобы сотрудник видел все ассайны тенанта и их вердикты в `History`, в его Position вдобавок к правам на страницы (`automation.assignments.view`, для `History` — `automation.actions.view`) добавляют `scopes.automation.assignments` (`Scopes: automation assignments`); чтобы ещё и правил, передавал и архивировал любые — `edit-scopes.automation.assignments` (`Editor Scopes: automation assignments`) вместе с ключами этих действий: `automation.assignments.edit`, `automation.assignments.share.ownership`, `automation.assignments.edit.archive`. Ветка сама ни одного действия не открывает. Разбор по страницам раздела — [how-to/auto-rules.md](../how-to/auto-rules.md).

### Какая страница раздела Automations каким правом открывается

Каждая из четырёх страниц закрыта своим view-императивом: `/app/automations/status` — `automation.status.view`, `/app/automations/assignments` — `automation.assignments.view`, `/app/automations/auto-rules` (`Rule Templates`) — `automation.templates.view`, `/app/automations/actions` (`History`) — `automation.actions.view`. Без нужного права страница не открывается. Заход на `/app/automations` без указания страницы ведёт на последнюю открытую страницу раздела (она запоминается в браузере), а если её нет или права на неё нет — на первую доступную по правам в порядке `Status` → `Assignments` → `Rule Templates` → `History`.

Сам пункт меню `Automations` появляется по любому из трёх прав — `automation.templates.view`, `automation.assignments.view`, `automation.actions.view`. Одного `automation.status.view` для пункта меню мало, хотя страница статуса по прямой ссылке при нём открывается ([how-to/auto-rules.md](../how-to/auto-rules.md)).

В матрице право страницы статуса подписано `Automation: status page` (`automation.status.view`), вся ветка страницы — `Automation: status`.

### Автоправило исполняет действие правами владельца ассайна, а не апрувера

Действие, которое автоправило отправляет в Meta, уходит с правами **владельца ассайна**, а не того, кто нажал `Approve`. Упирается оно в те же императивы `meta.*`, что и ручной экшен на странице Meta — `meta.campaigns.edit.stop` и соседние ([how-to/meta-ads.md](../how-to/meta-ads.md)); нет такого права у владельца — действие не исполняется. Что при этом попадает в журнал и почему владелец заморожен на момент вердикта — [mechanics/auto-rules-engine.md](../mechanics/auto-rules-engine.md).

### Как императивы применяются в UI (Settings → Positions)

Модалка `Settings → Positions → + Position`: `Position name` + `Description` + `Assign color` + **`Priority`** (слайдер) + две вкладки — **`Imperatives`** и **`Assigned users`** (кому назначена).

Императивы — **упорядоченный список** (`#1`, `#2`, …); у каждого ряда тоггл **`Deny` / `Allow`** + дропдаун выбора правила (`+ Add imperative` добавляет ряд, ряды перетаскиваются). Правила в дропдауне сгруппированы по **секциям-папкам**: можно взять целую секцию (`Analytics section`) или гранулярно `<Секция>: <подсекция>` (`Analytics: Roll Up report`, `Content: CDN files`…). Подпись есть у каждого ряда, включая все ветки `Scopes: <раздел>` и `Editor Scopes: <раздел>`. В **дропдауне** императивы перечислены по алфавиту — это порядок выбора, **не** путать с порядком применения (сверху вниз в самом списке Position).

**Порядок важен:** интерфейс применяет императивы **сверху вниз** (`Allow` добавляет, `Deny` убирает). Сервер следует этому порядку для разных ключей императивов. Для предсказуемого результата не повторяйте один ключ в нескольких строках. Между Positions с разными ключами конфликт решает `Priority`.

### Как проверить итоговые права позиции — панель `Granted permissions`

**Справа в редакторе позиции панель `Granted permissions` показывает итоговый набор ключей, который дают императивы этой позиции: счётчик, поиск (`Search permissions…`) и тип каждого ключа — `folder`, `page`, `scope`, `feature`, `view`, `action` или `billing`.** Клик по строке добавляет на этот ключ `Deny`-императив (подсказка `Click to add deny imperative to this`); пустая панель — `No effective permissions yet`. Панель учитывает только эту позицию: другие позиции юзера в ней не видны. Отдельной матрицы «просмотр / правка» с чекбоксами в редакторе нет — как собрать позицию «только просмотр», разобрано в секции «Дать баеру смотреть стату без права редактировать».

### Почему императив в Position есть, но не применяется — `is_active`

У каждого императива в списке есть per-императивный тоггл **`is_active`** (вкл/выкл, подсказка `Are we using item or not`). Деактивированный императив **пропускается при резолве прав** — он не удаляется из Position, но при вычислении итогового набора не участвует, будто его нет. То есть правило может физически стоять в списке (и `Allow`, и `Deny`), но быть просто выключенным.

Диагностика «правило в Position есть, а доступ ведёт себя не по нему»: проверь, что нужный императив **включён** (`is_active`), а не деактивирован. Выключенный `Deny` перестаёт резать; выключенный `Allow` перестаёт давать. Это отдельная причина помимо порядка (сверху вниз) и `Priority` между Positions.

## Как дать доступ ко всем объектам раздела без шаринга — `Scopes` и `Editor Scopes`

**`Scopes: <раздел>` (ключ `scopes.<раздел>`) даёт уровень `View` на все объекты раздела, `Editor Scopes: <раздел>` (`edit-scopes.<раздел>`) — уровень `Own`, независимо от владельца, команды и шаринга; сам раздел ветки не открывают.** Для Employee доступ к странице и действиям, закрытым ключом Position, задают права раздела: страницу — `<раздел>.view`, правку — `<раздел>.edit…`, архив — `<раздел>.edit.archive`, шаринг — `<раздел>.share`, смену владельца — `<раздел>.share.ownership`. Owner / Admin проходят эти проверки по роли. Поэтому ветку выдают вместе с ключами раздела: `Content: landings page` + `Scopes: content landings` — все лэнды тенанта видны в таблице и открываются на чтение; `Allow content.landings` + `Editor Scopes: content landings` — ещё правка, шаринг с правом раздавать, `Change ownership`, смена `Access type` и архив любого лэнда. Добавлять `Scopes` к `Editor Scopes` не нужно — `Own` включает `View`. В подсказке `Your access` такой доступ подписан `position`.

### Позиция со `Scopes` / `Editor Scopes` не видит раздел или получает `Action access denied`

**Без ключа страницы раздела ветка раздел не открывает: пункта в меню нет, прямая ссылка ведёт на `Sorry, you do not have access to this section`; без ключа действия сервер отвечает `Action access denied (<ключ> required)` — например, `Action access denied (content.landings.edit required)` при сохранении лэнда с одним `Editor Scopes: content landings`.** `Editor Scopes` без `Content: landings page` раздел не открывает — добавьте ключ раздела или Allow на весь раздел.

Вне страниц раздела уровень от ветки действует и без его ключей: объекты раздела видны в выпадающих списках других форм (выбор лэнда, домена, источника и т.п.), а ветка на кампании (`scopes.tracker.campaigns` или `edit-scopes.tracker.campaigns`) снимает отбор аналитики по доступу к кампаниям — отчёты считаются по всем кампаниям тенанта; ветка на источники так же снимает отбор по источникам.

### Для каких разделов есть `Scopes` и `Editor Scopes`

**Обе ветки заведены для одних и тех же 29 разделов: 10 есть в обоих билдах, 19 — только в ERP.**

- ERP и MTK: `tracker.campaigns`, `tracker.sources`, `tracker.destinations`, `content.landings`, `settings.presets`, `settings.streams`, `settings.teams`, `meta.social-profiles`, `tech.domains`, `tech.proxies`.
- Только ERP: `tracker.flows`, `content.content-library`, `content.macros`, `content.generations`, `settings.fields`, `settings.metrics`, `settings.distributions` (включая шаблоны автоправил `Rule Templates`), `settings.external-reports`, `settings.item-lists`, `settings.forms`, `settings.advertisers`, `settings.generators`, `marketing.message-templates`, `google.accounts`, `automation.assignments`, `tech.servers`, `tech.domain-providers`, `tech.dns-providers`, `tech.server-providers`.

Групповые узлы по префиксу включают все разделы внутри: `Scopes section` / `Editor Scopes section` (ключи `scopes` / `edit-scopes`) — все разделы, `Scopes: tech`, `Editor Scopes: tracker` и т.п. — разделы своей секции. Каждый раздел подписан в матрице: `Scopes: tracker campaigns`, `Editor Scopes: content landings`, `Scopes: automation assignments`, `Editor Scopes: generations` и т.п. У папок и Meta-шаблонов таких веток нет — их открывают только владение, команда, шаринг, `Access type` и полный доступ; у трекеров и креативов объектного доступа нет вовсе — их видят все, у кого есть право раздела.

## Как Teams дают доступ к объектам — числовые уровни участников

**Команда (`Settings → Teams`) даёт доступ через числовой уровень участника: кто стоит выше по уровню в той же команде, получает `Own` на объекты всех, кто ниже.** Команд в тенанте может быть несколько (по вертикалям, по группам баеров, по гео), юзер может состоять в нескольких. Пошарить объект можно на любую команду тенанта — шару получают все её участники, независимо от уровня.

Три команды приезжают из шаблона тенанта во всех семи шаблонах — `Tech Department`, `Media Buying Team A` (описание: «Buyers see their own traffic; the team head sees everyone's.») и `Media Buying Team B` («A second team - proof that two teams do not see each other.»). Участников в них нет: юзеры шаблоном не копируются, поэтому состав и уровни в каждой назначаются самостоятельно.

### Уровень участника команды — кто выше, владеет объектами тех, кто ниже

**У каждого участника команды — целый уровень от 0 до 100 (`Level N`); участник с более высоким уровнем получает на объекты, которыми владеют участники ниже него в этой же команде, уровень `Own`: видит, правит, шарит, передаёт владение и архивирует.** Равные по уровню по умолчанию объектов друг друга не видят. Наследование считается внутри каждой команды отдельно и через чужие команды не переходит: если A выше B в одной команде, а B выше C в другой, где A нет, объекты C юзеру A не достаются.

Ролей `Team Member` / `Team Leader` / `Team Head` в интерфейсе больше нет — они переведены в уровни 0, 1 и 2. По инвайту юзер попадает в команду на уровень 0. Практически: тимлиду ставят уровень выше, чем баерам (например, 1 и 0) — каждый баер видит свои кампании, тимлид владеет кампаниями всех баеров команды.

### Что участники одного уровня получают друг на друга — настройка `Peers:`

**Что равные по уровню получают на объекты друг друга, задаётся в диалоге команды кнопкой `Peers:` (RU `Равные:`) у каждого уровня отдельно: `No access` (по умолчанию), `Use`, `View`, `Edit` или `Own`.** `Use` — объекты коллег видны в списках и селектах, настройки закрыты; `View` — карточки открываются на чтение; `Edit` — можно менять настройки, но не владеть и не шарить; `Own` — полный доступ, включая владение, шаринг и архив. Настройка действует только на объекты, которыми коллега владеет, — пошаренное коллеге через `Peers:` не передаётся.

### Что поднимается от нижестоящих — настройка `Below:`

**По умолчанию участник с более высоким уровнем наследует только объекты, которыми нижестоящие владеют; пошаренное нижестоящим поднимается вверх, только если уровню в диалоге команды выбран другой режим `Below:` (RU `Снизу:`).**

| Режим `Below:` | Что поднимается сверх своих объектов нижестоящего |
|---|---|
| `Owned only` (по умолчанию, RU `Только свои`) | Ничего |
| `Owned and direct shares` (RU `Свои и личные шары`) | Объекты, пошаренные нижестоящему лично |
| `Owned and all shares` (RU `Свои и все шары`) | Плюс объекты, пошаренные командам нижестоящего, включая команды, где вас нет |

Наследованная шара даёт уровень самой шары, а не `Own`. При любом режиме всегда видны свои объекты, шары лично вам и шары вашим командам. Для режима `Owned and all shares` диалог предупреждает: уровень увидит всё, что видят его нижестоящие.

### Кто может создать и править команду

**Создать команду может Owner / Admin или позиция, где есть и `Settings: teams edit` (`settings.teams.edit`), и `Editor Scopes: teams` (`edit-scopes.settings.teams`); править существующую — они же или участник с самым высоким уровнем в этой команде, если в его Position есть `Settings: teams edit`.** Одна ветка `Editor Scopes: teams` без `Settings: teams edit` команды не правит: сервер отвечает `Action access denied (settings.teams.edit required)`. В диалоге создания текущий юзер заранее стоит в составе с уровнем 2; менять в диалоге собственный уровень и участие может только полный доступ (Owner / Admin). Участник, который правит команду только как старший по уровню (без полного доступа и без `Editor Scopes` на команды), не может выдать уровень выше своего, менять собственный уровень и убирать себя из команды. Отказы сервера:

- `You don't have permission to create teams`;
- `You don't have permission to edit this team (the highest level in the team is required)`;
- `Level <N> is above your level (<M>)`;
- `You can't change your own level`;
- `You can't remove yourself from the team`;
- `Level for user <uuid> must be between 0 and 100`.

### Старший в команде не может её править, хотя выше него никого не видно

**Проверка «самый высокий уровень» учитывает всех, кто записан в состав команды, — и деактивированных в тенанте, и тех, кого в тенанте уже нет.** Деактивированного участника в колонках `Managers` / `Regular Members` таблицы `Settings → Teams` не видно; ушедший из тенанта в этих колонках остаётся, хотя в `Settings → Users` его уже нет. Поэтому оставшийся старший видит диалог команды только на чтение с пометкой `You can’t edit this team: only full access, the teams edit scope or the member with the highest level can`, а запрос мимо интерфейса получает `You don't have permission to edit this team (the highest level in the team is required)`.

Лечится из-под Owner / Admin (или позиции с `Settings: teams edit` и `Editor Scopes: teams`): открыть команду и сохранить. Участники, которых в тенанте уже нет, при сохранении сами выпадают из состава — сохранение на них не падает; деактивированного участника с более высоким уровнем убирают из команды или понижают вручную.

Состав команды заменяется присланным списком участников целиком. Сохранение без списка — например, правка только имени, описания или аватарки через API — состав не меняет.

### Какие команды видно в `Settings → Teams` и где посмотреть уровень юзера

**В `Settings → Teams` сотрудник видит только команды, в которых состоит; все команды тенанта видят Owner / Admin и позиции, где к странице `Settings: teams page` (`settings.teams.view`) добавлены `Scopes` / `Editor Scopes` на команды (`scopes.settings.teams` / `edit-scopes.settings.teams`, оба билда).** В таблице участники разбиты на колонки `Managers` (все выше нижнего уровня команды) и `Regular Members` (нижний уровень команды — не обязательно 0). В ERP в `Settings → Users` колонка `Teams` показывает команды юзера, а в подсказке к команде — его уровень в ней: `<команда> · Level N`. В диалоге шаринга доступны все команды тенанта, не только свои.

### Не сохраняется имя команды / ошибка на символах — валидация имени Team

Имя Team валидируется на **длину и набор символов**: максимум `30` символов, на сервере допустимы только латиница, цифры, пробел и `- _` (дефис, подчёркивание). При превышении длины форма показывает ошибку, где фраза задвоена — **`Value must be less than Value must be less than 30`** (так её собирает интерфейс; читать надо как «не длиннее 30»). Симптом «не сохраняется имя команды» или «ругается на символы» — это валидация имени, а не баг: убери запрещённые символы (кириллицу, спецсимволы) и уложись в 30 символов.

Форма в UI визуально допускает ещё бэктик `` ` `` и апостроф `'`, но сервер их **отклоняет** — имя вида `O'Brien` пройдёт клиентскую проверку, но отвалится при сохранении. Действующее правило = серверный набор (латиница/цифры/пробел/дефис/подчёркивание).

У `Position name` набор символов не ограничен — можно кириллицу и спецсимволы; длина ограничена, но гораздо мягче — до `255` символов. Ограничение «только латиница/цифры/пробел/дефис/подчёркивание и ≤30» действует только для Team.

### Юзеры друг от друга не скрыты (by design)

AIO намеренно не скрывает юзеров друг от друга — Slack/Discord-модель, by design. Ни юзера целиком, ни его отдельные поля (аватар, Telegram, имя/позиция) скрыть нельзя — такой настройки в системе нет.

## Как работает Sharing — доступ к конкретному объекту

**Шаринг выдаёт юзеру или команде доступ к одному объекту (лэнду, кампании, домену, папке) с одной из трёх ролей — `Use`, `Viewer` или `Editor` — плюс, по желанию, правом раздавать дальше (`Can share further`) и сроком (`Expires`).** Роль `Use` даёт уровень `Use` (объект виден в списках и селектах, карточка закрыта), `Viewer` — `View` (карточка на чтение), `Editor` — `Edit` (правка настроек, тегов, цвета, папки и операционные действия). Владения шаринг не даёт: смена владельца, `Access type` и архив остаются за уровнем `Own`.

Срок по умолчанию — `No expiry`, задать можно только дату в будущем; истёкшая шара остаётся в списке с пометкой `Expired: access is no longer active` и доступа не даёт. Кто выдал шару, видно в колонке `Granted by` диалога; кому объект пошарен — в колонке **`Shares`** в списке сущностей. Пошаговая процедура, диалог `Share` и папки — [how-to/permissions.md](../how-to/permissions.md).

### Кто может шарить объект и какую роль можно выдать

**Шарить может тот, у кого на объект уровень `Own` (владелец, участник выше по уровню в команде владельца, равный с `Peers: Own`, позиция с `Editor Scopes` на раздел, Owner / Admin), или тот, кому объект пошарили с `Can share further`; для Employee при настроенной проверке действия дополнительно нужен ключ `<раздел>.share` (в разделе без такого ключа — `<раздел>.edit`). Owner / Admin проходят проверку ключа по роли.** Выдать можно роль не выше собственного уровня на объект — более высокие кнопки ролей в диалоге не показываются. Галку `Can share further` включает только уровень `Own`, поэтому получивший её раздаёт дальше, но без права раздачи. Пункта `Share` нет на строке, где у вас `Use`, `View` или `Edit` без права раздавать. Старые шары с ролями `Share` и `Owner` теперь означают `Editor` + `Can share further`: владельцем объекта они не делают.

### Чужую выдачу не изменить — замок в колонке `Granted by`

**Без уровня `Own` менять и снимать можно только шары, выданные вами или участниками ниже вас по уровню в команде; чужие выдачи в диалоге `Share` стоят под замком.** Подсказка замка — `Granted by someone else: you can only change or revoke shares granted by you or your subordinates`. Шары, выданные до появления колонки `Granted by`, считаются выданными владельцем (`Granted by the owner`) — их меняет только уровень `Own`. Запрос мимо интерфейса получает `You can only change or revoke shares granted by you or your team members below you (this one was granted by the owner or another user)`.

### Шаринг кампании тянет её лэнды и дестинейшены с ролью `Use`

**Когда кампанию шарят, её Landings и Destinations автоматически получают роль `Use`: они видны получателю в списках и в карточке кампании, но их собственные карточки не открываются и не правятся.** Список того, что уйдёт вместе с кампанией, показывает вкладка `Details` (кнопка `Next` в диалоге `Share`) с подписью `Linked landers and destinations get the Use role, no choice here`. Если получателю нужен просмотр или правка лэнда, лэнд шарят отдельно с `Viewer` или `Editor`; уже существующая более высокая шара на зависимость не понижается. `Can share further` на зависимости не переходит, срок — как у шары кампании. `Unshare` кампании снимает и выданные вместе с ней `Use`-шары, если у получателя нет к ним другого пути (другой пошаренной кампании или папки с ними).

### Откуда у владельца кампании `Use` на чужие лэнды — автошаринг при сохранении кампании

**При каждом сохранении кампании её владелец автоматически получает `Use` на её лэнды, дестинейшены, источник и домен — те, что не его, не открыты всем через `Access type` и не пошарены ему лично; открыть или править эти объекты такой доступ не даёт.** Условия выдачи и что происходит при смене владельца кампании — [how-to/campaigns.md](../how-to/campaigns.md).

### Шаринг папки — что получает её содержимое (ERP)

**Шаринг папки выдаёт доступ её содержимому: каждый объект в папке получает у получателя роль папки (прежняя шара этого получателя на объект перезаписывается), но без `Can share further`; зависимости этих объектов (лэнды и дестинейшены кампаний) — роль `Use`.** Объект, положенный в уже пошаренную папку позже, получает у каждого получателя роль, `Can share further` и срок его шары на папку. Шарить папки могут Owner и Admin тенанта, а также те, кому они пошарили папку с `Can share further`. У трекеров и креативов объектного доступа нет — шаринг их папки на видимость не влияет. Путь к меню папки, изъятие из папки и что при этом снимается — [how-to/permissions.md](../how-to/permissions.md). Создавать и шарить папки можно только в ERP: в MTK списка папок в таблицах нет.

### Массовые действия шаринга — `Share also`, `Force share`, `Unshare`, `Mass change ownership`

Как это называют: «пошарить на команду», «расшарить сервер», «массово пошарить». Массовые пункты появляются только при выделении строк, в группе `Teams & permissions`.

- **`Share also`** — добавить доступ тем, у кого его нет; существующую шару не понижает (роль только повышается, срок не меняется).
- **`Force share`** — справа заранее стоят получатели прямых шар хотя бы одного выбранного объекта, кроме вас; в этот список входят и истёкшие шары. При сохранении отправляются редактируемые получатели, видимые справа с текущим поиском: им роль, `Can share further` и срок перезаписываются на выбранные (по умолчанию `Viewer`, без раздачи и без срока). Изменение применяется к объектам, на которые хватает доступа; шару самому владельцу объекта сервер пропускает. Убранные влево отправляются на снятие шары. Пункт помечен как опасный.
- **`Unshare`** — снять доступ у выбранных.
- **`Mass change ownership`** — сменить владельца у всех выделенных (секция «Как передать владение — `Change ownership`»).

### Массовое действие применилось не ко всем — причины пропуска

**Массовое действие применяется к тем выделенным объектам, где хватает доступа, остальные пропускает с причиной; тост показывает `Applied: N` · `Skipped: M` и по строке на каждую причину.**

| Причина (RU) | Что значит |
|---|---|
| `no access` (`нет доступа`) | Нет нужного уровня на объект; для шаринга — нет `Own` и `Can share further` |
| `role above your level` (`роль выше вашего уровня`) | Выдаваемая роль выше вашего уровня на объект |
| `sharing rights require owner level` (`право раздавать выдаёт только владелец`) | `Can share further` выдаётся только с уровнем `Own` |
| `not granted by you` (`выдано не вами`) | Без `Own` затрагивается чужая выдача |
| `not found` (`не найдено`) | Объекта уже нет |

Ошибка на всю пачку — только когда применить не к чему: `No access to the selected objects (<level> level required): '<имена>'` (`403`) или, если не нашлось ни одного выделенного объекта, `Some UUIDs were not found in this model. No access to: '<…>'.` (`404`). В обычном диалоге массового действия (кроме шаринга) до подтверждения видно `Will apply to <N> of <M>: <K> skipped, no access`.

## Как Ownership определяет владельца сущности

У сущностей с объектным доступом (campaign, landing, domain и т.п.) есть **Owner** — обычно тот, кто её создал. Владелец всегда имеет на неё уровень `Own`.

### Как передать владение — `Change ownership`

**Владелец объекта меняется одним диалогом `Change ownership` (массово — `Mass change ownership`) на всех сущностях: в нём выбирается новый владелец (`Tenant users`), `Access type` и галка `Keep Editor access for the previous owner`.** Галка включена по умолчанию: прежнему владельцу добавляется шара `Editor`. Уже существующая более высокая роль, право раздавать и срок шары сохраняются; остальные источники доступа тоже продолжают действовать. Новая шара `Editor` сама по себе не даёт смену владельца и архив. Если галку снять, прежний владелец видит объект только по общим правилам (команда, шаринг, `Access type`). Отдельного пункта `Change owner` больше нет — у кампаний, лэндов и прокси тот же `Change ownership`.

### Кто может сменить владельца объекта

**Сменить владельца может тот, у кого на объект уровень `Own`: владелец, участник выше по уровню в команде владельца, равный с `Peers: Own`, позиция с `Editor Scopes` на раздел, Owner / Admin; для Employee при настроенной проверке действия дополнительно нужен ключ `<раздел>.share.ownership` (например, `Tech: servers ownership change`; в разделе без такого ключа — `<раздел>.share`, а без него — `<раздел>.edit`). Owner / Admin проходят проверку ключа по роли.** Пункт показывается только на строках, где у вас `Own`. Отказы: `No access to this object (own level required)` или `No access to change ownership of this object (own level required)` — не хватает уровня; `Action access denied (<раздел>.share.ownership required)` — нет ключа в Position.

### Копирование объекта — нужен `View`, копия ваша и `Private`

**В интерфейсе копирование доступно при уровне хотя бы `View`; для Employee действие также требует права копирования раздела, если оно проверяется (например, `Tracker: campaigns copy`). Owner / Admin проходят проверку ключа по роли.** Копия принадлежит тому, кто копировал, получает `Access type = Private`; в диалоге копирования к предложенному имени добавляется ` copy`, и имя можно изменить перед сохранением; шары оригинала на неё не переносятся, теги — переносятся.

### Удаление vs деактивация юзера

Чтобы было видно авторство прошлых заливок — деактивировать (`Settings → Users` → `Switch activity`), не удалять. Процедура и предупреждение «откатить нельзя» — [how-to/permissions.md](../how-to/permissions.md).

### Tenant Owner — особый случай

**НЕ путать** `Owner сущности` и `Tenant Owner`. Tenant Owner — это владелец **самого тенанта**, имеет full access ко всем сущностям тенанта **независимо** от owner-ства отдельных сущностей.

**Tenant Owner — юзер с ролью `Owner` в этом тенанте.** Отдельного действия «сменить владельца тенанта» нет: `Change ownership` работает только на сущностях (кампания / лэнд / домен) и на роль тенанта не влияет. В `Settings → Users → Edit tenant role` назначаются только `Admin` и `Employee`; роль `Owner` в этом действии недоступна.

## Как добавить юзера в тенант (Invite User)

**В ERP и MTK `Settings → Users → +User` открывает короткий инвайт для `Employee`: выбираются Position и Team, код и остальные параметры создаются автоматически.** Приглашать юзеров в тенант могут Owner и Admin — правом Position это не выдаётся.

В ERP отдельный пункт `Invite user` в выпадающем меню рядом с кнопкой открывает полную форму с выбором `Admin` / `Employee` и полями ниже.

| Поле | Что |
|---|---|
| Name | название инвайта |
| Code | кастомное **или** случайное значение — это код, который шаришь юзеру; от 4 до 255 символов. **Регистр не важен ни при создании, ни при вводе:** код, отличающийся от уже существующего только регистром, создать нельзя — форма подсветит поле ошибкой `The code has already been taken.`, а при регистрации `free14` и `FREE14` откроют один и тот же инвайт ([how-to/registration.md](../how-to/registration.md)) |
| Usages | сколько раз можно использовать инвайт-код (например, `1` — одноразовый) |
| Status | active/inactive |
| Type | `User` (обычный) |
| Till date | срок действия |
| Role | в этой форме доступны **`Admin` / `Employee`**; `Owner` не предлагается. В `Edit tenant role` тоже назначаются только `Admin` и `Employee`. |
| Position | одна; остальные назначают во вкладке `Assigned users` нужной Position |
| Team | в какую team зачислить; юзер попадает в неё на уровень 0, поднять уровень можно в диалоге команды |

Кнопка `Generate invite` создаёт инвайт-код и регистрационную ссылку. Юзер регистрируется по ссылке (вводит Name, Email, Telegram, Password, опционально включает 2FA).

### Как самопроверить доступы новой Position

**Проверить, что видит сотрудник с позицией, проще всего через `Log in as user`: Owner или Admin в ERP входит под юзера с нужной позицией и видит интерфейс его глазами** (секция «Как посмотреть интерфейс глазами сотрудника — `Log in as user`»). В MTK такого пункта нет — там регистрируют тестового юзера на позицию (например, `Buyer` из шаблона тенанта), открывают его в инкогнито и смотрят, что реально видно, а что скрыто.

### Без 2FA — поля скрыты

При включённой защите данных чувствительные значения скрыты до подтверждения 2FA. Подтверждение включает временный `Unsecured Mode`, в котором они открываются. В `Settings → Logs` защищены `Message`, `IP Address` и `Useragent`; у полей визита защита определяется атрибутом `2FA Protected`. 2FA настраивается через Telegram-бота (`AIO-tech-bot`) или Google Authenticator. «Не вижу телефон/email юзера» чаще всего означает, что у смотрящего не включена 2FA.

**Доступ сам скрывается снова по таймауту.** После ввода 2FA-кода чувствительные поля (`phone`, `email`, логи) видны не бессрочно — по истечении `Unsecured Mode` они закрываются опять. Длительность задаётся **на каждого пользователя отдельно** (чтобы изменить её, напишите в чат команде AIO), по умолчанию — **10 минут**. Подпись в интерфейсе `Data will be hidden after 30 minutes for security reasons.` — статичный текст, он не подстраивается под реальное значение, поэтому ориентироваться на него нельзя. Симптом «только что видел телефон, а теперь снова скрыто» — это истёкший таймаут, а не баг: нужно повторно подтвердить 2FA.

### Как скрыть чувствительные поля лида (`first_name`, `phone`) от джуна

**Когда:** «скрыть контакты лида от джуна», «чтобы баер видел стату, но не видел телефон/имя в визите».

Видимость **значения** чувствительного поля визита регулируется **на уровне самого поля** — атрибутом `2FA Protected` (`Settings → Fields → Edit Field`): при `2FA Protected = Yes` и включённой у смотрящего защите данных значение скрыто до подтверждения 2FA (типовой кейс — `phone`, `email`). Атрибут поля — SSOT в [how-to/custom-fields.md](../how-to/custom-fields.md) (раздел атрибутов поля). Защита действует для смотрящих с включённой защитой данных, а не адресно для одной Position.

`Deny` на `Page` / `Subsection` закрывает соответствующую страницу; это не настройка скрытия одного значения в визите. `Access type` определяет доступ к объекту Field (`Private` / `Use` / `View` — секция «`Access type` — доступ для всех в тенанте»). Для чувствительного значения используется `2FA Protected`.

## Как посмотреть интерфейс глазами сотрудника — `Log in as user`

**Owner и Admin тенанта могут войти под другого активного юзера своего тенанта (Admin — не под Owner): `Settings → Users` → правый клик по юзеру → `Log in as user`.** Перед входом показывается подтверждение `You will work as <имя> for up to 2 hours. Every action will be logged as made on their behalf by you.` Сессия длится до 2 часов и действует только в этом тенанте — переключиться под чужим именем в другой тенант нельзя (`Tenant switching is not allowed while impersonating`). Вверху висит плашка `You are logged in as <юзер> (opened by <вы>)` с кнопкой `Exit`; тот же выход — `Exit impersonation` в меню профиля. Через 2 часа вход под юзером завершается автоматически.

### Что нельзя делать под чужим аккаунтом и как это видно в логах

**Под `Log in as user` нельзя править профиль юзера, его 2FA и часовой пояс: такие действия отвечают `Profile editing is not allowed while impersonating`, а пункт `Profile settings` в меню недоступен (`Unavailable while logged in as another user`).** Нельзя войти под себя, под неактивного юзера и под юзера не из этого тенанта; Admin не может войти под Owner (`Only the tenant Owner can impersonate the Owner`); войти под кого-то, уже работая под чужим аккаунтом, нельзя (`Impersonation chains are not allowed`). В логах выполненных объектных Actions к записи от имени юзера добавляется `Impersonated by <имя> (<uuid>)`; у самого юзера в логах остаётся запись о том, кто и до какого времени под ним работал.

## Какие бывают отказы доступа — дословные ошибки прав

**В объектных Actions отказ может назвать отсутствующий ключ Position (`Action access denied (<ключ> required)`) или недостаточный уровень на объект (`No access to this object (<level> level required)`).** Когда действие закрыто ключом, сервер проверяет его перед загрузкой объекта с нужным уровнем; Owner / Admin проходят эти проверки по роли. Точный текст = точка входа: по нему находится причина. Если отказ пришёл при открытии карточки, интерфейс показывает тост `Sorry, you do not have access`; дословный текст сервера виден при сохранении или запуске действия.

### `Action access denied (<ключ> required)` — не хватает права в Position

**`403` с текстом `Action access denied (<ключ> required)` значит, что в Position юзера нет ключа, которым сервер закрывает это действие; ключ назван прямо в тексте — например, `Action access denied (tech.domains.edit.repoint required)`.** Лечится Allow на этот ключ или на его раздел-префикс (`tech.domains`, `tech`) — и проверкой, что его не перекрывает `Deny` ниже в списке или в позиции с большим `Priority` и что императив включён (`is_active`).

`401` `Auth Middleware: Unauthorized` — отказ аутентификации, а не недостаток ключа Position. Для UI проверьте сессию и перелогиньтесь; для API — валидность токена и доступ к выбранному тенанту.

### `No access to this object (<level> level required)` — не хватает уровня на объект

**Текст `No access to this object (use|view|edit|own level required)` значит, что проверка действия пройдена, но уровня доступа к этому конкретному объекту мало.** `view` — нужно открыть карточку или скопировать, `edit` — сохранить или запустить операционное действие, `own` — архив, `Change ownership`, `Access type`. Лечится повышением уровня: шара `Viewer` / `Editor`, владение, более высокий уровень в команде владельца, `Peers:`, `Editor Scopes` на раздел. Родственные тексты:

- `No access to the selected objects (<level> level required): '<имена>'` — в массовом действии ни к одному объекту не хватает уровня;
- `No access to share this object (own level or "can share" is required)` — шаринг без `Own` и без `Can share further`;
- `No access to change ownership of this object (own level required)` — смена владельца без `Own`;
- `Some UUIDs were not found in this model. No access to: '<…>'.` — в массовом действии не нашлось ни одного выделенного объекта.

### Отказы при шаринге — роль выше уровня, право раздавать, чужая выдача

**Диалог `Share` валидирует выдачу и отвечает дословными ошибками, когда роль или флаг выдать нельзя.**

- `Role <роль> is above your access level (<уровень>)` — выдаваемая роль выше вашего уровня на объект.
- `Only the owner can grant the right to share` (и вариант `Only the owner can grant the right to share (role <роль> means Editor + can share)`) — `Can share further` или старые роли `Share` / `Owner` выдаёт только уровень `Own`.
- `You can only change or revoke shares granted by you or your team members below you (this one was granted by the owner or another user)` — без `Own` чужую выдачу не изменить и не снять.

### Отказы по ролям, тенанту и профилю

- **`Visit timeline requires full access`** — таймлайн визита открывается только под Owner / Admin, не точечным императивом.
- **`You cannot assign this role`** — при смене роли выдаются только `Admin` / `Employee`; `Owner` так не назначается (см. «Tenant Owner — особый случай»).
- **`You cannot edit yourself`** — свою собственную роль сменить нельзя, нужен другой Owner/Admin.
- **`landing not found by permissions`** — правка файла лэнда отклонена: лэнда нет среди доступных юзеру (в его списках лэнда нет); приходит как ошибка `500`, а не `403`. Для шаблона лэнда тот же отказ звучит `landing template not found by permissions` — шаблон не найден в текущем тенанте.
- **`You don't have permission to edit this tenant`** / **`You don't have permission assign subscriptions`** — нет прав на редактирование тенанта / назначение подписок (действия уровня Owner/Admin).
- **`Read access denied`** / **`Edit access denied`** — попытка открыть или сохранить чужой профиль юзера; свой профиль правит только сам юзер. К правам Position эти тексты отношения не имеют.
- **`Profile editing is not allowed while impersonating`** — правка профиля, 2FA или часового пояса под `Log in as user`.

## Что НЕ делает Permissions Model

- **API-токен наследует права юзера.** Токен использует контекст того же юзера. Если у юзера полный доступ — API-токен тоже. Рекомендуется создавать отдельного юзера под API-автоматизацию.
- **Не работает между тенантами.** Permissions замкнуты внутри тенанта. Если тенантов несколько — права назначаются отдельно в каждом.
- **Не управляет billing.** Биллинг/финансы — отдельный механизм (см. *Билды, тарифы, триал и статусы тенанта*).

## Как логируются действия юзеров (Show Logs) — право `Settings: logs page`

Логи выполненных объектных Actions содержат действие, время, юзера, IP и User-Agent запроса. Страница `Settings → Logs` и обычные меню `Logs` / `Show Logs` на объектах требуют `Settings: logs page` (`settings.logs.view`); меню логов в `Settings → Users` проверяет `settings.users.view`. Отдельных прав `<раздел>.view.show-logs` нет. В логах Actions, выполненных через `Log in as user`, добавляется пометка об инициаторе входа. Для расследования «кто это сделал» ищите нужную запись; каждый запрос или отказ не гарантирует отдельный лог.

## Смежные темы

- [context/architecture.md](../context/architecture.md) — тенант как единица, multi-tenancy
- [how-to/permissions.md](../how-to/permissions.md) — приглашение/деактивация юзеров, Position, Team, Share, папки, Change ownership
- [models/tenant.md](tenant.md) — доптенант под другую вертикаль (мульти-тенант)
- [models/user.md](user.md) — сущность-actor (аккаунт), чьи права описывает эта модель
- [reference/glossary.md](../reference/glossary.md) — определения (Imperatives, Positions, Teams, etc.)

## Эвристики — настройка прав на практике

### «Обрезать всё, потом раздавать» — как настраивать права для новой команды

**Когда:** настройка permissions-модели для новой команды баеров (или новой роли).

**Делай:** **сначала режь до минимума**, потом раздавай по запросу. Базовый набор байера:

- `Analytics` (видеть статистику).
- `Campaigns` (свои кампании создавать и запускать).
- **НЕТ** `Tech` (сервера, домены не нужны байеру) — и без `Scopes` / `Editor Scopes` на `tech`.
- **НЕТ** доступа к доменам (репойнт, чекеры, защита от DDoS — задача тимлида / администратора тенанта).

Если кому-то конкретно понадобилось что-то ещё — обсуди и добавь. Не раздавай **на всякий случай**.

Так и собрана позиция `Buyer` из шаблона тенанта — готовая отправная точка для «обрезать всё, потом раздавать». Каркас общий для обоих билдов: Allow на `Tracker` (с Deny на `Tracker: destinations edit` и на `Tracker: trackers`), лэнды, загруженные файлы и Content Library, `Analytics`, `Features`, `Meta`, пресеты и теги.

В шаблонах ERP к этому добавлены креативы, `Google` и рассылки (`Marketing: campaigns` и `Marketing: flows page`); секции `Tech` нет вовсе.

В шаблонах MTK креативов, `Google` и рассылок у `Buyer` нет, зато открыт просмотр доменов — `Tech: domains page` (`tech.domains.view`): байер видит список доменов, но не управляет ими.

**Почему:**

- Доступ к Tech без понимания механики = case «байер случайно нажал Repoint, домен лёг на 10 минут».
- Принцип least privilege — стандартная индустрия.
- Расширить доступ потом легко; обратно ограничить — тяжело (политически).

### «Дать баеру смотреть стату без права редактировать» — view-only через Position

**Когда:** нужен view-only доступ — «пусть баер видит статистику кампаний, но ничего не редактирует», «дать смотреть, но не трогать».

**Делай:** отдельного «read-only» права в Position **нет** — view-only собирается позицией, где у разделов оставлены view-ключи (`Tracker: campaigns page`, нужные `Section` / `Subsection` / `Page`), а edit-ключи сняты (`Deny` на `Content: landings edit`, `Content: landings archive`, `Tracker: campaigns update costs` и т.п.). Чтобы видеть на чтение и чужие объекты раздела, добавь `Scopes: <раздел>` (например, `Scopes: tracker campaigns`). Для отдельных объектов «только чтение» даёт шаринг с ролью `Viewer` или `Access type = View`; в карточке такого объекта вместо сохранения стоит `Read-only`.

Карточку на чтение интерфейс открывает по уровню доступа к объекту, а не по ключам позиции: **свои объекты юзера (уровень `Own`) при view-only позиции всё равно открываются с кнопкой сохранения**. Для Employee без полного доступа сохранение, закрытое снятым edit-ключом, отклоняется с `Action access denied (<ключ> required)`.

**Почему:** доступ в AIO складывается из прав на действие и уровня на объект (см. «Как императивы применяются в UI» и «Какие уровни доступа к объекту бывают»), а видеть сущность ≠ исполнять над ней Action. View-only Position ограничивает операции, закрытые соответствующими ключами; сам уровень `Own` не заменяет такой ключ.

Для API-автоматизации создайте отдельного Employee с нужными view-ключами в Position и используйте его токен. Токен наследует пользовательский контекст, а снятые edit-ключи ограничивают действия, которые ими закрыты. Разбор со стороны API — [how-to/api.md](../how-to/api.md) → «Дайте read-only токен».

### «Байеры одной группы должны видеть кампании друг друга» — настройка `Peers:`

**Когда:** нужно, чтобы байеры одной команды видели (или правили) кампании друг друга.

**Делай:** в диалоге команды у уровня байеров выставь `Peers:` — `View` (видеть и открывать на чтение) или `Edit` (ещё и править настройки). По умолчанию (`Peers: No access`) байеры одного уровня кампании друг друга не видят, а участник с более высоким уровнем владеет кампаниями всех ниже.

Альтернатива без изменения команды — шаринг кампаний на команду (`Share` / массовый `Share also` → команда): шара на команду действует на всех её участников независимо от уровня. Разница: `Peers:` работает для всех будущих кампаний сразу, а шаринг — только для выделенных.

**Почему:** by design AIO даёт baseline-изоляцию байеров одного уровня — это нормальная корпоративная модель (junior не видит чужие тесты). Если хочется иначе — это **явный choice**, включается настройкой команды, а не дефолтом.

### Как байер меняет заголовок через сплиты (ERP)

Если заголовок уже вынесен в `Content Split` лэнда, байер может задать свои варианты в кампании через `LP splits → Use campaign splits → Edit splits`. Для этого достаточно уровня `View` на лэнд; для Employee без полного доступа сохранение требует ключа `content.landings.edit` в Position. Уровень `Edit` на сам лэнд для этой операции не нужен.

### Как ротировать варианты лэндинга (MTK)

В MTK обычный редактор лэнда не показывает `Content Splits`, а в кампании нет вкладки `LP splits`. Для ротации целых лэндингов выберите разные лэндинги в кампании и настройте их веса (`Weights`). В `PWA Builder` настройки `Content Splits` доступны и в MTK ([how-to/pwa.md](../how-to/pwa.md)). Подробнее о методах A/B тестирования в MTK — *Как устроен MTK-вид AIO: навигация, пресеты из шаблона и методы под типовые задачи*.
