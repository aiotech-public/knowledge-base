---
id: destinations
title: Destinations / Advertisers / Caps (How-to)
description: Создание Advertiser-ов, Destination через By Advertiser / By Integration / Simple Redirect / Telegram, своей CRM нет в списке интеграций (`Search by parameters`, запрос новой), поле `Proxy to communicate`, капы, PWA-destinations; реферер (Referrer-Policy same-origin), `#` и списковые плейсхолдеры в URL Parameters; колонки списка Destinations (`Rate` и другие метрики тенанта, `Destination logs`, позиция `#1`).
doc_type: how-to
builds: [erp, mtk]
related: [conversion-model, placeholders, flow-model, visit-lifecycle, how-to-pwa, push-notifications, glossary, architecture, permissions-model, ui-common, destination, metric, user-fields, virtual-metrics, distributions-model, api, mcp-server, flow-editor, notification-center, landings, user, mechanics-pwa, source-trackers, custom-pwa, advertiser]
language: ru
updated: 2026-10-28
---

# Destinations / Advertisers / Caps (How-to)

Создание Advertiser-ов, Destination-ов разных типов, настройка кап, специфика PWA. Концепт-уровень — [models/conversion-model.md](../models/conversion-model.md).

Пути с `Tracker → …` ниже относятся к ERP. В MTK `Campaigns`, `Sources` и `Destinations` открываются сразу из основного меню, а `Trackers` — через `Settings`.

---

## TL;DR

- **Advertiser** = рекламодатель / партнёрка. Создаётся в `Settings → Advertisers`.
- **Destination** = точка выхода визита: редирект или API-пуш. Создаётся на странице `Destinations`.
- **`+Destination` — 2 метода:** `By Advertiser` (через сохранённого Advertiser; разбивка по нему в аналитике) и `By Integration` (готовый шаблон интеграции без Advertiser). Для редирект-офферов рекомендуется `By Advertiser`, для прямого API — `By Integration`.
- **В модалке предусмотрены 4 варианта**, доступность зависит от прав:
  - `By advertiser` — через сохранённого Advertiser.
  - `By integration` — выбор готовой интеграции.
  - `Simple redirect` — пресет `By Integration` с типом `Simple Redirect`: редирект без Advertiser и API-пуша, без автологина. Подходит и для перехода в TG-бота или разового URL.
  - `Telegram` — пресет `By Integration` с типом `Telegram Destination`: переход в бот или канал; нужен Telegram Sender Provider и `Behaviour`. Конверсии на старт бота и вступление в канал настраиваются в Sender Provider.
- **Cap** — лимит конверсий выбранного типа у дестинейшена (`Conversion Cap`). В ERP: `Destination → ПКМ → Caps → Edit conversion cap`. Четыре типа: `Unlimited`, `Daily`, `Monthly`, `Lifetime`.
- **PWA** — два пути: сторонний PWA-сервис как Destination или AIO Builder как Landing.

---

## Создать Advertiser

**Путь:** `Settings → Advertisers → +Advertiser`.

**`+Advertiser` сразу открывает форму создания.** В поле `Integration type` по умолчанию стоит `Simple Redirect`; другую интеграцию выбирают в самой форме.

**Шаги:**

1. **Name** — имя (обязательно).
2. **Advertiser type** — тип Advertiser (опционально, для группировки в UI; см. «Как сгруппировать Advertiser-ов по типу» ниже). Для адвертайзеров под обычные редирект-URL обычно `Offer`.
3. **Integration type** (обяз.) — конкретная партнёрка/сеть из **большого каталога интеграций** (`AdCombo`, `Adbrt`, `Adpulse`, `Adw`, `Aff1`, `AffBay`, `AffGenius`, `AffScale`, … — десятки CPA/affiliate-сетей, каждая = свой постбэк/API-шаблон). Выбирается в форме. Помимо каталога есть значение **`Simple Redirect`** — для адвертайзеров под обычные редирект-URL без API. Для raw-шаблонов вроде `Bridge`/`IREV Lite` см. Destination → `By Integration`.
4. **URL Parameters** — параметры адвертайзера. Поле лежит в блоке `Settings` и есть только у `Integration type` = `Simple Redirect`; заданные в нём параметры **автоматически дописываются** к URL каждого Destination, созданного `By Advertiser` для этого адвертайзера (список AIO-плейсхолдеров — по иконке-шестиграннику).
5. **Image** — опционально (для UI). Картинку можно загрузить **с компьютера**, указать её URL или выбрать из недавних; **Content Library** доступна в ERP.
6. `Save` / `Confirm`.

### Как выглядит готовый Advertiser под редирект-офферы — `Example Advertiser` из шаблона Sweeps

В шаблонах Sweeps (ERP и MTK) адвертайзер под редирект-офферы уже заведён: `Example Advertiser` с `Advertiser type` = `Offer`, `Integration type` = `Simple Redirect` и `URL Parameters` = `click_id={{aio.visit.uuid}}`. Его три дестинейшена созданы `By Advertiser` (`SweepNet #10442 | iPhone 17 | US | $2.20 SOI`, `SweepNet #10517 | $1000 Gift Card | DE | $1.80 SOI`, `DirectBrand #sample-box | Sample Box | GB | $3.40 DOI`) — у каждого свой `Redirect URL`, а `click_id` к нему дописывает адвертайзер. В iGaming два адвертайзера типа `Offer` — `Partners Platform Example` и `Example Direct Brand` — собраны иначе: `URL Parameters` у них пустые, а `clickid={{aio.visit.uuid}}` стоит прямо в `Redirect URL` каждого из шести офферов-примеров (`Example Partners | Alpha Casino | BR | slots | CPA 25$` и подобные); девять PWA-сервисов там же — см. «PWA-Destination через сторонний сервис» ниже. В Nutra под партнёрки заведены адвертайзеры с интеграциями API (`AdCombo Example`, `AlterCpa Example`, `LeadBit Example`, `LemonAd Example`), в Finance примеров адвертайзеров нет.

### Параметр из `URL Parameters` не подставился в ссылку оффера

Параметр из `URL Parameters` адвертайзера **не перезаписывает** одноимённый параметр, у которого в `Redirect URL` дестинейшена уже есть значение: то, что задано прямо в ссылке, сильнее. Правило склейки у адвертайзера с `Integration type` = `Simple Redirect`:

- **Параметра с таким именем в ссылке нет** — пара дописывается в хвост URL.

- **Параметр есть, но пустой** (`?click_id=` или `?click_id` вообще без знака равенства) — значение из `URL Parameters` его заполняет. Рабочий приём: оставить параметр в `Redirect URL` пустым, чтобы его подставил адвертайзер.

- **Параметр есть и заполнен** — значение из `URL Parameters` пропускается, к рекламодателю уходит то, что стояло в ссылке.

Если один и тот же ключ встречается в `Redirect URL` дважды, решение принимается по первому вхождению, а оба дубликата в ссылке остаются.

Ведущие `?` и `&` и пробелы вокруг имени параметра в `URL Parameters` срезаются — вычищать их руками не нужно. Всё, что написано в `URL Parameters` после `#`, отбрасывается; собственный `#fragment` ссылки `Redirect URL` при этом сохраняется и остаётся в самом конце готового URL.

### Часть параметров не доехала до рекламодателя — `#` внутри подставленного значения режет хвост `URL Parameters`

Обрезка по `#` в `URL Parameters` происходит **после** подстановки плейсхолдеров, поэтому `#` внутри подставленного значения обрубает строку так же, как `#`, написанный руками: решётка в значении поля визита (ответ юзера в поле формы вроде `Promo #3`, значение, записанное в поле шагом `Fill Fields`) — и все пары `URL Parameters`, стоящие после этой, к рекламодателю не уезжают. Симптом: у части визитов в `destination_url` не хватает хвоста параметров, у остальных всё на месте.

Лечится модификатором `{{...:urlencode}}` на плейсхолдере, в значение которого может попасть `#`: он уходит как `%23` и хвост остаётся целым. Синтаксис и полный список модификаторов — [reference/placeholders.md](../reference/placeholders.md).

### Какие символы можно писать в именах параметров `URL Parameters`

AIO сохраняет символы имени параметра в готовом URL: точка идёт как есть (`s2s.req_id=...` так и уходит `s2s.req_id=...`), пробел и квадратные скобки — в URL-кодировке (`%20`, `%5B`).

Значения существующих параметров, которых склейка не касалась, не перекодируются: `+`, готовые `%XX`-последовательности и неразвёрнутые `{{плейсхолдеры}}` сохраняются. Буквальные пробелы вокруг имени параметра срезаются и в исходном `Redirect URL`. Дописанные и заполненные из `URL Parameters` пары собираются заново с URL-кодированием — поэтому `+` в значении, написанном в `URL Parameters`, уедет как `%20`.

### Плейсхолдеры дестинейшена в `URL Parameters`

Помимо обычных `{{aio.visit.*}}` в `URL Parameters` работают три плейсхолдера уровня самого дестинейшена — `{{destination_domain}}`, `{{destination_human_id}}` и `{{destination_name}}`; их же перечисляет тултип поля. Где именно они разворачиваются, что подставляют и полный список плейсхолдеров — [reference/placeholders.md](../reference/placeholders.md).

Списковые плейсхолдеры визита — `{{aio.visit.landing_human_ids}}`, `{{aio.visit.landing_uuids}}`, `{{aio.visit.landing_type_uuids}}`, `{{aio.visit.flow_uuids}}` — подставляют списки значений. Без модификатора числовой список выглядит как `[309889]` (в URL — `%5B309889%5D`), но кавычки строковых значений удаляются. Для сохранения JSON-массива нужен `:raw`; для JSON-объекта query — `{{aio.visit.query:raw}}`. Один элемент отдаёт `{{aio.visit.landing_human_ids:json_array_first}}` (первый лэнд; есть `json_array_second` … `json_array_ninth`). Разбор — [reference/placeholders.md](../reference/placeholders.md).

### Почему у разных Advertiser-ов разные поля (settings)

**Общие поля Advertiser-а — `Name`, `Description`, `Tags`, `Advertiser type`, `Integration type`, `Image`; для `Simple Redirect` в `Settings` добавляется `URL Parameters`.** Схема `Settings` приходит от выбранной интеграции. У API-интеграций, в том числе `AdCombo` и `IREV Lite`, эта схема Advertiser-а пустая: API-ключи, эндпоинты и маппинг заполняют в карточке Destination, а не Advertiser-а. Наборы полей Destination — секция «Обязательные поля популярных интеграций» ниже.

### Как сгруппировать Advertiser-ов по типу (Advertiser Types)

Advertiser Types — группировка Advertiser-ов под одной категорией. Из коробки в каждом шаблоне тенанта два типа: `Offer` (партнёрки и прямые рекламодатели, `Metrics Event Group` = `Offer`) и `PWA` (сторонние PWA-сервисы, `Metrics Event Group` = `PWA`); свои типы добавляются рядом (`Manage Types`, см. «Завести свой PWA-сервис как Advertiser» ниже).

В ERP типы создают через `Settings → Advertisers` → стрелка у `+Advertiser` → `Manage Types` → `+Advertiser Type`. Затем тип выбирают в поле `Advertiser type` формы Advertiser-а. В MTK пункта `Manage Types` на странице нет.

## Destination через By Advertiser

**Для редирект-офферов рекомендуется By Advertiser.** Привязка даёт разбивку по Advertiser; `Offer Visits` требует типа Advertiser с `Metrics Event Group` = `Offer`. При `Integration type` = `Simple Redirect` параметры Advertiser-а автоматически добавляются к URL.

**Путь:** `Tracker → Destinations → +Destination → By Advertiser → выбрать Advertiser`.

**Атрибуты:**

- **Name** — имя оффера.
- **Countries** — гео категоризация (**не фильтр**).
- **Languages** — языки категоризация.
- **Redirect URL** — основной URL у интеграции `Simple Redirect`; у API-интеграций вместо него свои поля.
- При `Simple Redirect` хвостик из `URL Parameters` Advertiser-а добавляется **автоматически**.

### Почему именно By Advertiser

Полная эвристика — [models/conversion-model.md](../models/conversion-model.md).

## Destination типа Simple Redirect

Голый редирект-URL без адвертайзера, без API-интеграции, без автологина. Без Advertiser нет разбивки по нему и событий его `Metrics Event Group`, по которым считается `Offer Visits`. Для редирект-офферов рекомендуется `By Advertiser`: так удобнее группировать офферы и анализировать их вместе. `Simple Redirect` подходит и для быстрого перехода в TG-бота или разового URL.

**`Simple Redirect` создаётся БЕЗ выбора Advertiser** — это его отличие от `By Advertiser`: Advertiser не назначается, поэтому нет автоматического «хвостика» параметров и привязки к Advertiser в аналитике. Задаётся только сам redirect-URL (с плейсхолдерами визита).

**Путь:** `+Destination → Simple Redirect → задать Redirect URL` (без выбора Advertiser).

Для редирект-оффера рекомендуется `By Advertiser`, для прямой API-интеграции — `By Integration` (см. эвристику в [models/conversion-model.md](../models/conversion-model.md)). Для `Offer Visits` у Advertiser-а нужен тип с `Metrics Event Group` = `Offer`.

## Какой реферер уезжает рекламодателю — `Referrer-Policy: same-origin` на редиректах и лэндах AIO

**AIO ставит `Referrer-Policy: same-origin` на редиректах через Destination (`301` по умолчанию и `302`) и на HTML лэнда, прослойки и `Field verification`: на другой домен — оффер, партнёрку, рекламодателя — реферер не уходит, а запросы в пределах того же домена (подстраницы, файлы лэнда, SDK) идут с реферером.** Прежнее значение заголовка — `no-referrer`: тогда реферер не уходил и внутри домена, для рекламодателя на другом домене смена ничего не поменяла. Это относится к редиректам `Simple Redirect`, `By Advertiser` и `By Integration`. На обычном лэнде политику задаёт документ, включая переходы по `{{link}}` и `location.href`; настройки отключения заголовка нет. Ограничение Safari на прямых 30x-редиректах описано ниже. Для внешнего лэнда через `Reflect` / `Proxy Reflect` политика зависит от отражаемого сайта.

На sub-параметры это не влияет: они уезжают параметрами URL / полями payload, и если передавать их не нужно — убираются из ссылки или из настроек Destination.

### Safari игнорирует `Referrer-Policy` на 30x — когда нужна стратегия `Meta Refresh No Referer`

Safari/WebKit не учитывает `Referrer-Policy` на 30x-ответах и на прямом редиректе без лэнда (визит уходит на оффер сразу) может дослать реферер входящего запроса. Если срез реферера нужен во всех браузерах, на шаге ставится `Redirect strategy` = `Meta Refresh No Referer` — вместо 30x визитёру отдаётся HTML-прослойка с политикой документа. Стратегии редиректа и на каких узлах они есть — [models/flow-model.md](../models/flow-model.md).

### Рекламодателю нужен реферер — отдать его параметром `{{aio.visit.fields.referer}}` или `{{aio.visit.domain}}`

Колонка `Referer` хранит заголовок **первого захода**, которым создался визит. Для нетрэшевого визита непустой заголовок также записывается в системное поле `Referer` (slug `referer`); это поле можно менять отдельно от колонки — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Рекламодатель же под «реферером» обычно ждёт адрес страницы, с которой пришёл лид, — то есть домен лэнда. Отдаётся параметром `URL Parameters` / полем payload:

- `{{aio.visit.fields.referer}}` — реферер захода (площадка). Отдельного `{{aio.visit.referer}}` нет — только через поле.

- `{{aio.visit.domain}}` — домен визита; в примерах дестинейшенов шаблона Nutra (`AdCombo Example | Diabetes Support | RO | 9 EUR CPA`, `LeadBit Example | Prostatitis Care | PL | 11 EUR CPA`) поле интеграции `referrer` заполнено именно им, у `LemonAd Example | Slim Detox | ES | 12 EUR CPA` тем же плейсхолдером заполнено поле `domain`.

## Destination типа Telegram (Telegram Destination)

`Telegram Destination` ведёт трафик в бот или канал. Типы конверсий на старт бота и вступление в канал задаются в Sender Provider.

**Путь:** `Tracker → Destinations → +Destination` → плитка `Telegram` (у созданного Destination `Integration type` = `Telegram Destination`). Два обязательных поля — `Sender Provider` и `Behaviour`.

- **Sender Provider** — заранее созданный провайдер типа `Telegram` (бот). Тот же Telegram Sender Provider обслуживает и маркетинг-рассылки, и Telegram Destination. Поведение бота (приветствие, приглашение в канал, типы конверсий) настраивается **внутри Sender Provider** — см. [how-to/push-notifications.md](push-notifications.md) → Telegram Sender Provider.

### Как визит попадает в Telegram (Behaviour)

Поле **`Behaviour`** задаёт, куда ведёт визит:

| Behaviour | Что происходит |
|---|---|
| **`Bot -> Channel`** | Визит сначала попадает в **бота**. Приветствие отправляется, если оно задано в Sender Provider; приглашение с кнопкой — если настроен канал и Telegram выдал ссылку. Текст и кнопка настраиваются в Sender Provider. |
| **`Channel`** | Визит **редиректится сразу в канал**, без промежуточного бота. |

**Бот должен быть админом канала.** Иначе Telegram не отдаёт боту событие вступления, и **конверсия подписки на канал не зафиксируется** — обратная петля конверсии ломается. Это требование Telegram к правам бота, не специфика AIO.

### Лёгкий путь в TG-бота — через Simple Redirect

Если нужно просто загнать трафик в Telegram-бота (без приглашения в канал и фиксации подписки), полный `Telegram Destination` с Sender Provider **не нужен**. Достаточно обычного `Simple Redirect` Destination (или `By Advertiser`, если завёл отдельный Advertiser под Telegram). В поле `Redirect URL` — deep-link бота со start-параметром, передающим UUID визита:

```
https://t.me/<bot_username>?start={{aio.visit.uuid}}
```

UUID в `start` позволяет сопоставить визит, но бот должен отправлять конверсии в AIO постбэком: один редирект их не создаёт. `Telegram Destination` через Sender Provider (`Behaviour` `Bot -> Channel` / `Channel`) даёт встроенное приглашение в **канал** и фиксацию подписки; провайдер также может создавать конверсию на `/start` — см. «Destination типа Telegram (Telegram Destination)» выше и [how-to/push-notifications.md](push-notifications.md).

## Destination типа By Integration

`By Integration` — создание Destination на основе готовой интеграции, встроенной в AIO, без привязки к Advertiser. Помимо API-шаблонов в списке есть редиректные и тестовые типы.

**Путь:** `+Destination → By Integration → выбрать тип интеграции из списка`. После выбора открывается окно **Create new destination** с базовыми полями (Name, Description, Tags, Countries, Languages, `Cap monitoring user`).

**Остальные поля payload зависят от выбранной платформы** — у каждой интеграции свой набор. Что и как заполнять — смотреть в **API-документации платформы**, с которой интегрируешься. Список AIO-плейсхолдеров для маппинга — по иконке-шестиграннику. Click ID в AIO = Visit UUID, плейсхолдер `{{aio.visit.uuid}}`.

Если нужной интеграции в списке не видно — поищите её по имени и кнопкой `Search by parameters`, а если её нет, запросите новую (секция «Своей CRM нет в списке интеграций» ниже).

### Какие примеры `By Integration` уже есть в шаблоне тенанта

Из коробки дестинейшены без адвертайзера (`By Integration`) лежат в шаблонах Finance (ERP и MTK) и Nutra: `Example Bridge Integration` (`Bridge`), `Example IREV Integration` (`IREV Lite`) и тестовый дестинейшн на интеграции `DummyApi` (в Finance — `Dummy API [ DEBUG ]`, в Nutra — `Dummy API - Flow Testing`, в Sweeps — `Dummy API`; это тот самый `Dummy Destination` для прогона формы без отправки лидов в партнёрку — [reference/glossary.md](../reference/glossary.md)). Примеры под партнёрские API `AdCombo`, `AlterCpa`, `LeadBit`, `LemonAd` в Nutra (`AdCombo Example | Diabetes Support | RO | 9 EUR CPA` и подобные, с заполненным маппингом полей) привязаны к своим адвертайзерам — см. «Как выглядит готовый Advertiser под редирект-офферы». В iGaming все примеры дестинейшенов — редиректы (`Simple Redirect`), примеров API-интеграций там нет. Что клонируется из шаблона — [context/architecture.md](../context/architecture.md).

### Обязательные поля популярных интеграций

Набор обязательных полей зависит от интеграции. Конкретные наборы для частых получателей:

| Платформа | Обязательные поля |
|---|---|
| **IREV** (тип интеграции подписан `IREV Lite`) | `Api URL`, `Api Key`, `IREV Affiliate ID`, `IREV Offer ID` |
| **Getlinked** | `Api URL`, `Api Key`; `Offer Name` и `Comment field` необязательны |
| **Trackbox** | `TB Api URL`, `TB Api Key`, `TB Api Username`, `TB Api Password`, `TB Ai Field`, `TB Ci Field`, `TB Gi Field`; также `Lead First Name`, `Lead Last Name`, `Lead Email`, `Lead Phone`, `Lead IP`, `Lead Password` |
| **Voralis** | `API Token`, `Affiliate`, `First Name`, `Phone`, `Country`, `Quantity` — плюс `Product ID` **или** `Product Name` (заполнить нужно хотя бы одно; при обоих заполненных приоритет у `Product ID`) |
| **Cryptahl** | `Token`, `First Name`, `Phone`, `Email`, `Country Code`, `Sub ID` |

У **`AlterCpa`** и **`AlterCpa Moe`** есть необязательное поле `Item` (плейсхолдер `Ex: 3`) — незаполненным оно в запрос к платформе не уходит.

У **`IREV Lite`** и **`IREV Lite (Mapped)`** страна лида передаётся только из необязательного поля `Country Code` (ISO2, плейсхолдер `Ex: DE`, поддерживает плейсхолдеры — например `{{aio.visit.country_code}}` — страна визита, [reference/placeholders.md](../reference/placeholders.md)): незаполненным поле `country_code` в запрос к IREV не уходит вовсе. Если партнёр ждёт страну лида — заполните поле.

(Полный каталог интеграций постоянно расширяется — [reference/glossary.md](../reference/glossary.md) → «Destination Integrations».)

### Поле `Proxy to communicate` — пуш уходит через ваш прокси

**Поле `Proxy to communicate` (у части интеграций — `Proxy`) отправляет запросы пуша этого дестинейшена через указанный прокси — получатель видит IP прокси, а не адреса AIO.** Формат — строка `user:pass@ip:port` (плейсхолдер `Ex: user:pass@ip:port`, подсказка `Proxy to communicate with API`); пустое поле — запрос идёт напрямую. Поле есть почти у всех API-интеграций; его нет у редиректов (`Simple Redirect`, `Telegram Destination`), у тестовых `DummyApi` / `DummyRedirect` и у части интеграций (например `Bridge`, `Trackbox`, `LeadGreed`). У нескольких интеграций поле обязательное — форма без него не сохранится.

Строку прокси AIO не проверяет ни при сохранении, ни кнопкой: теста прокси у дестинейшена нет, а записи `Proxies` (`Tech → Proxies`, в MTK — `Settings`) к дестинейшенам не привязываются (их проверка). Первый сигнал неработающего прокси — отказ пуша с `cURL error <N>: …` в причине (*уточните у поддержки* → «Ошибка прокси при пуше»).

## Своей CRM нет в списке интеграций — как отправлять в неё лиды

**Серверная отправка лида (API-пуш) в AIO идёт только через шаблон интеграции: универсального «кастомного» дестинейшена, где вы сами задаёте метод, путь, тело запроса и заголовки, нет.** API-типы в списке `By integration` шлют запрос в формате своего API. Адрес получателя у большинства интеграций задаётся настройкой (`Push URL`, `API URL` и т.п.) и может указывать на ваш сервер, а формат запроса и то, какой ответ считается успехом, зашиты в шаблоне. Поэтому путь такой: найти готовую интеграцию под формат вашей CRM, а если её нет — запросить новую.

### Как найти готовую интеграцию под вашу CRM — поиск по имени и `Search by parameters`

**В окне `Create destination by integration` список ищется по имени или UUID типа интеграции, а кнопка `Search by parameters` подбирает тип по адресу и параметрам API.** В списке все типы интеграций, имя в нём — значение `Integration type`; если CRM построена на известной платформе, ищите по имени платформы.

`Search by parameters`: в `API URL` вставьте полный HTTPS-адрес эндпоинта (для подбора берётся только путь), в `Parameters` — имена параметров, которые принимает API. Совпасть должны **все** введённые параметры и путь; у шаблонов с вариативным путём проверяются только параметры. Для точности добавьте минимум три уникальных. `Nothing found` — совпадений нет, `More than two integrations matched…` — добавьте параметров. Подбор знает не все интеграции: перед запросом новой поищите по имени платформы.

### Готовой интеграции нет — запросить новую; редирект API-пуш не заменяет

**Новую интеграцию добавляет команда AIO: передайте в поддержку API-документацию CRM и тестовые доступы для пробного пуша.** Кнопки запроса в окне создания нет — запрос идёт через чат поддержки (*уточните у поддержки* → «Payload, который мы шлём»).

Без интеграции остаётся только `Simple redirect`: он уводит браузер визитёра на ваш URL, данные визита подставляются в адрес плейсхолдерами (`{{aio.visit.fields.phone}}` и т.п.). Это переход, а не серверный пуш: запрос шлёт браузер, ответа CRM AIO не читает, пуш всегда считается принятым. `Bridge` в списке — тоже не универсальный HTTP: это интеграция одного сервиса с фиксированными путём и форматом (адрес по умолчанию — `lbridge.app`).

## Редактор Destination (Edit destination)

Созданный Destination виден в таблице `Tracker → Destinations`, а также доступен через `Tracker → Campaigns` (внутри кампаний, где он используется).

В таблице `Tracker → Destinations` дестинейшн ищется по **имени, номеру или UUID**, либо быстрыми фильтрами **`Advertiser`** и **`Geo`** (в MTK тоже). Фильтр Advertiser требует права `tracker.destinations.view.advertiser`. `Geo` смотрит на `Countries`, а при пустом списке стран — на гео, определённое по трафику.

`Tracker → Destinations` → правый клик → **Edit destination**. Поля:

- Name, Description, Tags, Countries, Languages.
- **Cap monitoring user** — кому летят алерты по капам.
- **Integration type** — шаблон интеграции получателя (`Bridge`, `IREV Lite` и др.). От него зависят поля payload ниже. У дестинейшена без адвертайзера в строке таблицы стоит бейдж: `Simple Redirect` у редиректа, `API integration` у остальных — это пометка, а не тип интеграции.
- **Поля payload** (зависят от Integration type) — маппинг данных визита в запрос к получателю через плейсхолдеры: `Token`, `Buyer UTM` = `{{aio.visit.fields.buyer_utm}}`, `Country Code` = `{{aio.visit.country_code}}`, `Language Code` = `{{aio.visit.language_code}}` и т.д.

**Экшены (правый клик):** `Edit destination`, `Copy destination`, `Destination logs`, `Switch archive`, `Show visits`, `Show conversions`. В ERP также есть `Change domains`, `Build Roll Up report`, в группе `Caps` — `Edit conversion cap` / `Reset cap`, в `Folders` — `Assign to folder` / `Unassign from folder`; в MTK этих пунктов нет.

В группе `Teams & permissions` — `Share` и `Change ownership`; для нескольких строк — `Share also`, `Force share`, `Unshare`, `Mass change ownership`. Нужны права Position и доступ к строке: для `Share` — `Own` или `Can share further`, для `Change ownership` — `Own`. Свой уровень — `Your access: <уровень>` внизу меню ([models/permissions-model.md](../models/permissions-model.md)).

`Destination logs` — записи о самом дестинейшене, прежде всего кто и когда менял его настройки. Попыток пуша там нет: они пишутся в логи визита (*уточните у поддержки* → «В `Destination logs` только сохранения настроек»).

### Кнопка `#1` над таблицей Destinations — у оффера нули, хотя лиды на него идут

Метрики строки в `Tracker → Destinations` считаются **по позиции пуша**: жёлтая кнопка `#1` задаёт позицию, клик переключает `#1` → `#2` → `#1`. Позиция — порядковый номер **успешного пуша в новый Destination**; реджект и повтор в тот же Destination её не добавляют. Второй успешно принявший Destination учитывается на `#2`; если первый отказал, следующий успешный останется на `#1`. Значение хранится в браузере и общее для Landings и Destinations — [reference/ui-common.md](../reference/ui-common.md), [models/destination.md](../models/destination.md).

### Колонка `Rate` в `Tracker → Destinations` — это метрика или поле вашего тенанта, а не ставка дестинейшена

**Встроенной колонки `Rate` нет: колонку с таким именем добавляет настройка тенанта.** Общей ставки Destination нет, но отдельные интеграции имеют свои поля выплаты, например `Payout` у `AlterCpa Moe`. Кроме системных колонок, `Tracker → Destinations` показывает:

- **метрики тенанта** — считаются по визитам, у которых дестинейшн стоит на позиции, выбранной кнопкой `#1` / `#2`; в подсказке заголовка — описание метрики и `By Visit Time` (или `By Conversion Time`). Сами метрики — в `Settings → Metrics` ([models/metric.md](../models/metric.md));
- **поля типа `Destination`** — значение из секции `User fields` карточки дестинейшена ([how-to/user-fields.md](user-fields.md));
- **группы тегов** — теги дестинейшена из одной группы;
- **`Custom formulas`** — формула, собранная в браузере; в подсказке заголовка — сама формула ([how-to/virtual-metrics.md](virtual-metrics.md)).

Revenue / Payout в аналитике считаются дистрибуциями типа конверсии ([models/distributions-model.md](../models/distributions-model.md)); это отдельная механика от полей payload конкретной интеграции.

### Как получить такую колонку и имя дестинейшена через API и в AI-чате

**Имя дестинейшена отдают все пути; значение колонки — только те, что считают метрики.**

- **`Data API`** (`GET /api/v1/data/destinations`) — атрибуты дестинейшена: имя, `human_id`, адвертайзер, гео, языки; метрик и значений колонок там нет ([how-to/api.md](api.md)).
- **`Tables Data`** (`POST /api/v1/tables/data`, `table` = `Tracker\Destinations`) — строки таблицы как в интерфейсе, со значением каждой колонки: метрика — под ключом `metric_<uuid>`, поле — под UUID поля. `Custom formulas` сервер не знает — формулу пересчитайте сами.
- **AI-чат и MCP-сервер** — `traffic_analytics` с `group_by` = `destination-position-1` (`Destination #1`) возвращает имя и `human_id` дестинейшена и запрошенные метрики; метрика передаётся по точному имени (список — `list_metrics`). Поля дестинейшена и формулы эти инструменты не отдают ([how-to/mcp-server.md](mcp-server.md)).

### Где в карточке дестинейшена задаются User fields

В формах создания и правки есть секция `User fields` с полями типа `Destination`. При пуше они добавляются в поля визита до подстановки плейсхолдеров. У `By Advertiser` есть также поля родителя; в `By Integration` — только свои. Механика — [how-to/user-fields.md](user-fields.md).

### `Change domains` — замена host в Redirect URL дестинейшена

`Change domains` (правый клик) работает только у `Simple Redirect`: задаются `From domain` → `To domain`. Если host `Redirect URL` точно совпадает с `From domain`, заменяется только host; совпадения в пути или query не меняются.

Не путать с кампанийным `Replace domain`: там домен выбирается из дропдауна, здесь вводят два имени домена.

## Как ограничить число конверсий на дестинейшн (Conversion Cap)

`Conversion Cap` ограничивает число конверсий выбранного типа; исходящий трафик блокируется при стратегии `Reject traffic`. Настройка кап на странице Destinations есть в ERP; в MTK её нет.

**Путь в ERP:** `Tracker → Destinations → <D>` → правый клик → `Caps` → **`Edit conversion cap`**. Соседний **`Reset cap`** обнуляет выбранный счётчик.

Капа гейтит **исходящую доставку**: при заполнении и стратегии `Reject traffic` (дефолт) новый пуш не уходит. Событие `Destination Full` выбирается, если такой переход есть; иначе действует цепочка fallback ниже. При `Do nothing` эта капа только считает. Приём постбэков она не блокирует: созданная конверсия выбранного типа увеличивает счётчик. Симптом «конверсия по офферу не растёт» проверяется со стороны исходящего пуша и получателя, а не блокировки постбэка капой.

### Экшен `Reset cap` — что именно он обнуляет

`Reset cap` обнуляет счётчик капы и отклонённых по ней, а дату начала периода ставит текущую. При положительном лимите капа перестаёт считаться заполненной. Обязательное поле — `Cap type` (тултип `Type of cap to reset`): счётчик конверсий обнуляет **только** `Conversion Cap`.

В селекте по умолчанию подставлено другое значение — `Push Cap`. Сохранение с ним конверсионный счётчик не трогает: дестинейшн остаётся полным, хотя сброс формально прошёл.

### Как читать колонку `Conversions Cap` в таблице

Колонка `Conversions Cap` показывает тип капы (календарь — `Daily`, флажок — `Lifetime`, бесконечность — `Infinity`; у `Monthly` своей иконки нет), шкалу, `использовано / лимит` (`Used` / `Total cap`), иконку стратегии и красный бейдж `+N` (`Rejected`) при ненулевом счётчике отклонённых. Тултип типа — `Daily cap` / `Lifetime cap` / `Infinity cap` / `Monthly`; к нему могут добавляться зона и дата сброса. У стратегии — `Cap rejects traffic when full` или `Cap does not reject traffic`.

У безлимитной капы числа расхода и иконка стратегии скрыты; остаются тип, нейтрально-полная шкала и бейдж `+N`, если его счётчик ненулевой.

### Бейджа `+N` нет, хотя лиды отбиваются по капе — счётчик отклонённых не растёт

Счётчик отклонённых по капе, который стоит за бейджем `+N`, при реджекте по капе не увеличивается: пуш пишет реджект на визит, но счётчик дестинейшена не трогает. Поэтому у переполненного дестинейшена бейджа, как правило, нет, а `Reset cap` и полуночный сброс `Daily` обнуляют и так нулевое значение. Сколько лидов отбила капа, считается по визитам: в `Tracker → Visits` у них `Last Rejection Type` = `Cap` и `Last Rejection Reason` = `Conversion Cap is Full` ([models/destination.md](../models/destination.md)).

### Ограничения Cap — что не поддерживается и как считается

- **Периоды капы — сутки (`Daily`), календарный месяц (`Monthly`) или всё время жизни дестинейшена (`Lifetime`)**; недельного и произвольного периода нет.
- **Под-кап на один оффер под разных байеров — нет**.
- Механика подсчёта и блокирующая логика кап — [models/conversion-model.md](../models/conversion-model.md).
- При переполнении капы со стратегией `Reject traffic` — алерт в Telegram назначенному `Cap monitoring user` (доставка требует привязки аккаунта к Telegram, как у любого Monitoring User; 2FA через Google Authenticator привязку не даёт).

### Fallback при переполнении капы (Destination Full)

Fallback при переполнении со стратегией `Reject traffic` — переход `Destination Full` во флоу ([how-to/flow-editor.md](flow-editor.md)). **Его надо нарисовать явно:** без него выбирается `Destination-rejected`, затем `Destination-interacted`, если соответствующий переход есть. Если нет ни одного, используется `state-back-content` — возврат на последний Content в истории, а не обязательно на предыдущий оффер. Разбор — [models/destination.md](../models/destination.md).

При `Reject traffic` переполненные `Daily`, `Monthly` и `Lifetime` блокируют трафик одинаково. `Daily` сбрасывается после ближайшей полуночи выбранной зоны, `Monthly` — после начала месяца, а `Lifetime` остаётся полным, пока лимит не поднимут или счётчик не сбросят.

## Какой `Cap type` выбрать: `Unlimited`, `Daily`, `Monthly` или `Lifetime`

Тип капы выбирается в диалоге `Edit conversion cap` четырьмя кнопками — `Unlimited`, `Daily`, `Monthly`, `Lifetime`; под кнопками показывается подсказка про выбранный тип. Отличаются типы ровно одним — когда обнуляется счётчик. Блокировка при `Reject traffic` у `Daily`, `Monthly` и `Lifetime` одинаковая.

`Unlimited` и `Infinity` — один и тот же тип: в диалоге он подписан `Unlimited`, в колонке таблицы и в API — `Infinity`.

### `Unlimited` (`Infinity`) — лимита нет

`Unlimited` означает, что капы у дестинейшена нет: поле `Limit` в диалоге скрыто, лиды уходят без ограничения. Это состояние нового дестинейшена — капа задаётся уже после создания, отдельным экшеном.

Счётчик при `Unlimited` не заморожен: если в капе выбран `Conversion type`, конверсии этого типа его увеличивают, а полуночный пересчёт обнуляет. Поэтому дестинейшн, переведённый с `Unlimited` на ограниченный тип среди дня, стартует не с нуля, а с расходом, накопленным с прошлой полуночи.

### `Daily` — счётчик обнуляется в полночь зоны `Reset timezone` (по умолчанию UTC)

`Daily` — суточная капа: счётчик обнуляется раз в сутки в полночь той таймзоны, что выбрана в поле `Reset timezone` диалога `Edit conversion cap` (подсказка в диалоге — `The counter resets every day at midnight.`, тултип поля — `Daily and monthly caps reset at local midnight of this timezone. Empty — UTC.`). Поле появляется только при `Cap type` = `Daily` или `Monthly`, значение выбирается из списка таймзон, пустое (плейсхолдер `UTC (default)`) = полночь по UTC. Обнуление делает фоновый пересчёт AIO — с лагом до нескольких минут после полуночи; механика подсчёта в [models/destination.md](../models/destination.md).

Вместе со счётчиком обнуляется и число отклонённых по капе, и двигается дата начала периода.

### `Monthly` — счётчик обнуляется в полночь первого числа месяца

`Monthly` — месячная капа: счётчик обнуляется раз в месяц, в полночь первого числа по таймзоне из поля `Reset timezone` (пусто — UTC). Подсказка в диалоге — `The counter resets at midnight on the 1st of each month.` Месяц календарный: капа, заведённая 20-го, первый раз обнулится 1-го числа следующего месяца, а не через 30 дней. Вместе со счётчиком обнуляется число отклонённых по капе, а дата начала периода обновляется; сброс, как и у `Daily`, приходит с лагом до нескольких минут после полуночи.

### `Lifetime` — счётчик не обнуляется никогда

`Lifetime` — капа на всё время жизни дестинейшена: счётчик копится и полуночным пересчётом не трогается (подсказка в диалоге — `The counter never resets — the limit applies to the whole lifetime.`).

Обнулить счётчик можно только руками — кнопкой `Reset counter` в диалоге `Edit conversion cap` или экшеном `Reset cap`. Поднятый `Limit` счётчик не трогает: он отодвигает потолок, накопленный расход остаётся. При `Reject traffic`, пока не поднят лимит и не сброшен счётчик, заполненная `Lifetime`-капа держит дестинейшн закрытым и на следующие сутки.

## Диалог `Edit conversion cap` — какие поля заполнять

Диалог `Edit conversion cap` состоит из шести блоков:

- **`Conversion type`** — обязательное поле: тип конверсии, который капа считает. Счётчик растёт **только** на конверсиях этого типа, конверсии других типов капу не расходуют.
- **`Cap type`** — четыре кнопки выбора типа (`Unlimited` / `Daily` / `Monthly` / `Lifetime`) с подсказкой под ними.
- **`Limit`** (`Лимит`, плейсхолдер `Conversions allowed`) — числовой лимит.
- **`Reset timezone`** — только при `Daily` и `Monthly`: таймзона, по чьей полуночи обнуляется счётчик; пусто = UTC (секции «`Daily`» и «`Monthly`» выше).
- **`When the cap is full`** — две кнопки `Reject traffic` / `Do nothing` с подсказкой (секция ниже).
- **`Current usage`** (`Текущий расход`) — шкала с числами «использовано / лимит» и кнопка сброса счётчика.

При `Cap type = Unlimited` видны только первые два блока. Поля лимита, стратегии и расхода скрыты, но счётчик конверсий продолжает работать.

### `When the cap is full` — `Reject traffic` (дефолт) или `Do nothing`: капа только считает

`When the cap is full` — что делать с трафиком, когда капа заполнена; дефолт и историческое поведение — `Reject traffic`. С ним при полной капе пуш в получателя не уходит, визит получает реджект с `Last Rejection Type` = `Cap`. При наличии перехода выбирается `Destination Full`, иначе — цепочка fallback из секции выше (подсказка под кнопкой: `Traffic is not pushed to the advertiser: a reject conversion is spawned and the flow gets a Destination Full event — it can be routed to a fallback.`).

`Do nothing` делает **Conversion Cap** информативной: она не блокирует пуш, счётчик растёт дальше (подсказка: `Traffic keeps going through — the cap only counts. Set up a notification rule for alerts.`). Заполнение видно в таблице и в уведомлениях по правилу на `Tenant alerts` ([mechanics/notification-center.md](../mechanics/notification-center.md)). Эта капа не шлёт inline Telegram-алерт Monitoring User и не вызывает `Destination Full` или реджект. Отказы получателя и другие ограничения действуют отдельно.

### Что делает кнопка `Reset counter`

`Reset counter` (`Сбросить счётчик`) сразу ничего не меняет — она лишь помечает счётчик к обнулению, а применяется вместе с `Save`. Помеченный сброс виден прямо в `Current usage`: текущее число со стрелкой на `0`. До сохранения решение отменяется повторным нажатием, кнопка при этом подписана `Cancel reset` (`Отменить сброс`).

Ручное уменьшение счётчика считается началом нового периода капы: двигается дата сброса, и пороги правила уведомлений по капе разрешаются заново ([mechanics/notification-center.md](../mechanics/notification-center.md)). Счётчик отклонённых по капе кнопка не трогает — его обнуляет экшен `Reset cap`.

### Смена `Cap type` счётчик не обнуляет — расход переносится

Перевод капы с одного типа на другой (например с `Daily` на `Lifetime`) сохраняет текущий расход: смена типа сама по себе счётчик не трогает. Чтобы начать счёт с нуля, в том же диалоге нажимается `Reset counter` перед сохранением — либо отдельно выполняется экшен `Reset cap`.

Типовая ловушка: дестинейшн, переведённый в `Lifetime` среди дня, уже несёт накопленный за этот день расход — и после полуночи его больше не потеряет.

## PWA-Destination через сторонний сервис

Когда: ПВА-оффер у стороннего PWA-сервиса.

**Шаги:**

1. **Получить ссылку из AIO:** открыть `Link Generator` у кампании (`Campaigns → ПКМ → Link Generator`), выбрать домен и Source `3rd Party PWA Link Generator`, нажать `Generate Link` и скопировать ссылку нужного сервиса. В шаблонах iGaming source `3rd Party PWA Link Generator` уже заведён (по ссылке на каждый из девяти сервисов); в шаблонах других вертикалей его нет — источник добавляется отдельно через `Sources → +Source`.
2. **В сервисе:** создать пвашку → вставить ссылку из AIO → получить домен пвашки от сервиса.
3. **Создать Destination в AIO:** `Tracker → Destinations → +Destination → By Advertiser → выбрать PWA-сервис → вставить домен пвашки`.

**Поля PWA-Destination** (`By Advertiser`): Name, Description, Tags, Countries, Languages, `Cap monitoring user`, сам `Advertiser` и поля его интеграции — среди них `Redirect URL`, куда вставляется домен пвашки (параметры advertiser-а допишутся к URL автоматически). Капа в форме создания не задаётся. Настройка капы в ERP — в секции «Капы у дестинейшена» выше.

Также см. [mechanics/pwa.md](../mechanics/pwa.md) → 3rd Party PWA.

Для возврата в тот же визит PWA-сервису нужна ссылка с параметром `hess`. Параметры исходного Source AIO записывает при входе визита, до перехода на PWA.

### Завести свой PWA-сервис как Advertiser

Если нужного PWA-сервиса нет среди заведённых в шаблоне advertiser-ов (или вертикаль не iGaming), его можно добавить вручную.

1. **Проверить Advertiser Type `PWA`.** Тип `PWA` с `Metrics Event Group` = `PWA` заведён в каждом шаблоне тенанта.

Если типа нет, создайте его: `Settings → Advertisers → стрелка у +Advertiser → Manage Types → +Advertiser Type`. Поля: `Name` (можно `PWA`), `Color`, `Icon`, `Description` и **`Metrics Event Group = PWA`** — эта группа нужна для событий, по которым считаются PWA-метрики.

2. **Создать Advertiser:** `+Advertiser` с `Advertiser type` = `PWA`, `Integration type` = `Simple Redirect`. Заполнить `URL Parameters` (параметры, что допишутся к Destination URL; список плейсхолдеров — по иконке-шестиграннику), `Image`/`Description`/`Tags`.
3. **Создать Destination:** через `By Advertiser`, выбрав этот Advertiser (поля — как в списке выше).

### Как настроить постбэк-трекер для стороннего PWA-сервиса

Чтобы конверсии PWA-сервиса доходили в AIO, ставится постбэк-трекер. Цепочка постбэков: **рекламодатель → AIO → PWA-сервис**.

**Путь:** `Tracker → Trackers → +Tracker` типа `HTTP Get`. Поля:

- **`Sources`** — source, по ссылке которого визит **пришёл** в AIO (для FB — `FB w/ CAPI`), а не `3rd Party PWA Link Generator`: ссылка из генератора возвращает по `hess` тот же визит, source у него остаётся исходный. Если `Sources` заполнено, трекер срабатывает на конверсиях визитов из указанных источников; пустое поле разрешает все Source.
- **`Conversion type`** — тип конверсии, по которому срабатывает постбэк.
- **`Url`** — постбэк-URL сервиса; макросы сервиса заменяются на AIO-плейсхолдеры.

В шаблонах iGaming такой трекер уже есть как заготовка — `Example - 3rd Party PWA Postback` (`HTTP Get`, тип `Registration`, привязан к `FB w/ CAPI`), выключенный: подставьте постбэк-URL своего сервиса и включите. Click ID PWA-сервиса доступен плейсхолдером `{{aio.visit.fields.pwa_click_id}}`. Постбэки AIO для рекламодателя берутся в [how-to/source-trackers.md](source-trackers.md).

### Как собрать кампанию и ссылки под сторонний PWA-сервис

После того как Destination-ы (PWA + Offer) и постбэк-трекер готовы — собирается кампания. В шаблонах iGaming (ERP и MTK) под PWA уже есть три флоу, выбираются в `Select flow` при создании кампании, у каждого — кампания-пример с меткой `AIO EXAMPLE`: `PWA / OneLink` (`AIO Filter` → шаг `PWA / AIO PWA / Onelink` → `Offer`; пример `Default Flow - PWA Campaign Example`), `PWA / OneLink w/ Landing` (после PWA-шага — `Spawn Install` и прелэнд `Landing`, потом `Offer`; пример `PWA w/ Landing Flow - Campaign Example`) и `Advanced - PWA / OneLink` (разводка Android / iOS, всё остальное — на `Fallback Offer`; пример `Advanced Flow - Campaign Example`). Состав флоу и кампаний-примеров — [mechanics/pwa.md](../mechanics/pwa.md); в шаблонах Finance, Nutra и Sweeps PWA-флоу нет.

**Campaign steps:** на PWA-шаге в примерах стоит сплит из лэнда `AIO PWA Example w/ Splits` и партнёрских PWA-дестинейшенов (`Example ZMApps PWA`, `Example AppsHeroes PWA`) — сюда ставится ваш PWA-Destination; на `Offer` — Offer-Destination либо `Offer Distribution`.

**В готовых PWA-сценариях Install-конверсия** создаётся во флоу, когда визит уходит с PWA-сервиса дальше по флоу. В шаблонах iGaming это сделано двумя способами: в `PWA / OneLink` и `Advanced - PWA / OneLink` у ноды `Offer` стоит `Pushed conversion spawn` = `Install` — конверсия создаётся при редиректе визита на Offer-Destination; в `PWA / OneLink w/ Landing` за это отвечает отдельный шаг `Spawn Install` (`Spawn Conversion`) сразу после PWA-шага. Настройки ноды — [models/flow-model.md](../models/flow-model.md); сравнение со спавном у внутренней PWA — [how-to/pwa.md](pwa.md).

Переставить спавн или сменить его тип конверсии можно в редакторе флоу (`Tracker → Flows`): либо `Pushed conversion spawn` на ноде `Offer`, либо отдельный шаг `Spawn Conversion` в нужном месте.

В MTK нет редактора флоу. В шаблонах iGaming спавн `Install` уже стоит во всех трёх PWA-флоу; изменение места или типа спавна запросите у команды AIO.

### Какие две ссылки генерятся под сторонний PWA-сервис

Под сторонний PWA-сервис генерируются **две разные ссылки** (обе через `Link Generator`):

1. **PWA-ссылка** — Source = `3rd Party PWA Link Generator`; вставляется в сам PWA-сервис как offer link.
2. **Трекинговая ссылка** под твой реальный Source (например, FB) — вставляется в рекламный источник. Если FB — выбирается source **`FB w/ CAPI`** (заведён в каждом шаблоне тенанта; в iGaming его replace-поля включают и событие установки — `fb_action_i` → `fb_install_action`).

## AIO PWA (PWA Builder) — Destination как лэнд

Внутренняя PWA AIO создаётся как лэнд через PWA Builder и ставится во флоу как Landing. В окне `+ PWA` есть `PWA landing` (встроенный Builder) и `Custom PWA` — свой ZIP в PWA-шелле ([how-to/custom-pwa.md](custom-pwa.md)). Доступ к созданию зависит от прав.

**Путь в ERP:** `Content → PWA → + PWA`.

**Путь в MTK:** `PWA → PWA → + PWA`. Раздел виден, когда модуль PWA включён у тенанта и есть права на него.

В обычном списке `Landings` PWA не создаётся и не показывается. Процедура целиком — [how-to/pwa.md](pwa.md).

**Шаги:**

1. Настроить название, описание и иконку PWA — по ним Builder формирует manifest.
2. Опционально — `Content Splits` для тестирования.
3. Привязать к флоу как Landing.

Также см. [mechanics/pwa.md](../mechanics/pwa.md), [how-to/landings.md](landings.md).

## Типичные ошибки

- **Destination сделан через Simple Redirect без Advertiser.** Разбивки по Advertiser нет; чтобы группировать офферы по нему, создайте Destination через `By Advertiser`.
- **Капа в `Daily`, ждали сброс по локальному времени.** Сброс — в полночь зоны `Reset timezone`; если поле пустое — в **0:00 UTC** (секция про `Daily` выше).
- **Месячная капа обнулилась не через 30 дней.** `Monthly` сбрасывается в полночь 1-го числа календарного месяца (по `Reset timezone`), а не через месяц от создания.
- **Перевели капу в `Lifetime`, а она сразу полная.** Смена `Cap type` счётчик не обнуляет — расход переносится. В ERP обнулить можно через `Reset counter` в диалоге или экшен `Reset cap`.
- **Капа не растёт, хотя конверсии идут.** Счётчик считает только конверсии того `Conversion type`, что выбран в самой капе.
- **Archive Destination → продолжает распределять трафик.** Архивирование не удаляет Destination из кампаний. Для нового выбора деактивируйте вариант в кампании или дистрибуции. Уже закреплённый визит может продолжить использовать Destination после деактивации варианта.
- **Сторонний PWA-сервис: лид не возвращается на визит.** Не передан Sub-параметр с ID сессии — проверить настройки в PWA-сервисе.

## Смежные темы

- [models/destination.md](../models/destination.md) — концепт Destination (выход флоу, push≠conversion, капы).
- [models/advertiser.md](../models/advertiser.md) — концепт Advertiser (контейнер Destination-ов, Revenue, offer name for_advertiser).
- [models/conversion-model.md](../models/conversion-model.md) — модель конверсий, Reward, Business Models, эвристики (Destination = By Advertiser, Uniqueness Strategy).
- [mechanics/pwa.md](../mechanics/pwa.md) — PWA в AIO, AIO внутренние vs сторонние.
- [models/flow-model.md](../models/flow-model.md) — Destination как Flow State.
- [how-to/flow-editor.md](flow-editor.md) — Destination Full fallback transition.
- [how-to/source-trackers.md](source-trackers.md) — постбэк-трекеры для Destination.
- [how-to/push-notifications.md](push-notifications.md) — Telegram Sender Provider (бот для Telegram Destination и для рассылок).
- [reference/glossary.md](../reference/glossary.md) — Destination, Destination Integrations, Replace Destination, Cap / Caps, Destination Full, Offer Visits.
