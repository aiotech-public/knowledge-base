---
id: destinations
title: Destinations / Advertisers / Caps (How-to)
description: Создание Advertiser-ов, Destination через By Advertiser / By Integration / Simple Redirect / Telegram, капы, PWA-destinations (AIO и сторонние сервисы); что уезжает рекламодателю — реферер (Referrer-Policy no-referrer на любом ответе AIO, как отдать реферер параметром), `#` и списковые плейсхолдеры в URL Parameters; позиция пуша `#1` в списке Destinations.
doc_type: how-to
builds: [erp, mtk]
related: [conversion-model, placeholders, flow-model, visit-lifecycle, how-to-pwa, push-notifications, glossary, ui-common, destination, user-fields, flow-editor, notification-center, landings, user, mechanics-pwa, source-trackers, advertiser]
language: ru
updated: 2026-09-11
---

# Destinations / Advertisers / Caps (How-to)

Создание Advertiser-ов, Destination-ов разных типов, настройка кап, специфика PWA. Концепт-уровень — [models/conversion-model.md](../models/conversion-model.md).

---

## TL;DR

- **Advertiser** = рекламодатель / партнёрка. Создаётся в `Settings → Advertisers`.
- **Destination** = точка пуша (оффер). Создаётся в `Tracker → Destinations`.
- **`+Destination` — 2 метода:** `By Advertiser` (через сохранённого Advertiser; аналитика по адвертайзеру + единый Replace) и `By Integration` (raw API-интеграция, **без адвертайзера**). **Штатно: ссылки → `By Advertiser`, API-интеграции → `By Integration`.** `Simple Redirect` — голый редирект без адвертайзера, **не штатный**; для оффера бери `By Advertiser`.
- **4 плитки в модалке `+Destination`** — два метода плюс два ускоренных пресета `By Integration`:
  - `By advertiser` — через сохранённого Advertiser.
  - `By integration` — raw-интеграция по API-шаблону.
  - `Simple redirect` — пресет `By Integration` с типом `Simple Redirect`: голый редирект-URL **без адвертайзера/интеграции** (автологина нет). **Не штатный** — для оффера бери `By Advertiser`; узкие случаи (редирект в TG-бота, разовый URL).
  - `Telegram` — пресет `By Integration` с типом `Telegram Destination` (трафик в TG-бот/канал, целевое действие подписка; нужен Telegram Sender Provider + `Behaviour`).
- **Cap** — лимит конверсий у дестинейшена (`Conversion Cap`). `Destination → ПКМ → Edit conversion cap`. Три типа: `Unlimited`, `Daily`, `Lifetime`; **месячных кап нет**.
- **PWA-Destination** — два пути: сторонний PWA-сервис или AIO Builder.

---

## Создать Advertiser

**Путь:** `Settings → Advertisers → +Advertiser`.

**Перед формой — выбор `Destination Type`.** По `+Advertiser` сначала открывается grid-выбор «Select a destination type to create advertiser»: плитки типов дестинейшна. Выбор плитки предвыбирает `Integration type` будущего адвертайзера (значение из плитки подставляется в поле `Integration type` формы, шаг 3) — поэтому создать адвертайзер «в вакууме» без типа нельзя. После выбора плитки открывается собственно форма.

**Шаги:**

1. **Name** — имя (обязательно).
2. **Advertiser type** — тип Advertiser (опционально, для группировки в UI; см. «Как сгруппировать Advertiser-ов по типу» ниже). Для адвертайзеров под обычные редирект-URL обычно `Offer`.
3. **Integration type** (обяз.) — конкретная партнёрка/сеть из **большого каталога интеграций** (`AdCombo`, `Adbrt`, `Adpulse`, `Adw`, `Aff1`, `AffBay`, `AffGenius`, `AffScale`, … — десятки CPA/affiliate-сетей, каждая = свой постбэк/API-шаблон). Предвыбирается плиткой `Destination Type` (см. выше), в форме остаётся доступным. Помимо каталога есть значение **`Simple Redirect`** — для адвертайзеров под обычные редирект-URL без API. *(В UI 2026-06-06 поле называется `Integration type`; для raw-шаблонов вроде `Bridge`/`IREV` см. Destination → `By Integration`.)*
4. **URL Parameters** — параметры адвертайзера. Поле лежит в блоке `Settings` и есть только у `Integration type` = `Simple Redirect`; заданные в нём параметры **автоматически дописываются** к URL каждого Destination, созданного `By Advertiser` для этого адвертайзера (список AIO-плейсхолдеров — по иконке-шестиграннику).
5. **Image** / **Logo** — опционально (для UI). Картинку можно загрузить **с компьютера** или выбрать из **Content Library**.
6. `Save` / `Confirm`.

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

Имя параметра доезжает к рекламодателю тем же, каким его написали: точка идёт в URL как есть (`s2s.req_id=...` так и уходит `s2s.req_id=...`), пробел и квадратные скобки — в URL-кодировке (`%20`, `%5B`), которая на стороне получателя разворачивается обратно в исходное имя.

Пары, которые уже стояли в `Redirect URL` и которых склейка не касалась, уходят байт-в-байт: `+`, готовые `%XX`-последовательности и неразвёрнутые `{{плейсхолдеры}}` не перекодируются. Дописанные и заполненные из `URL Parameters` пары, наоборот, собираются заново с URL-кодированием — поэтому `+` в значении, написанном в `URL Parameters`, уедет к рекламодателю как `%20`.

### Плейсхолдеры дестинейшена в `URL Parameters`

Помимо обычных `{{aio.visit.*}}` в `URL Parameters` работают три плейсхолдера уровня самого дестинейшена — `{{destination_domain}}`, `{{destination_human_id}}` и `{{destination_name}}`; их же перечисляет тултип поля. Где именно они разворачиваются, что подставляют и полный список плейсхолдеров — [reference/placeholders.md](../reference/placeholders.md).

Списковые плейсхолдеры визита — `{{aio.visit.landing_human_ids}}`, `{{aio.visit.landing_uuids}}`, `{{aio.visit.landing_type_uuids}}`, `{{aio.visit.flow_uuids}}` — без модификатора уезжают целым JSON-массивом: `[309889]`, в URL это `%5B309889%5D` (а `{{aio.visit.query}}` — целым JSON-объектом). Чтобы отдать один элемент, ставится модификатор `{{aio.visit.landing_human_ids:json_array_first}}` (первый лэнд; есть `json_array_second` … `json_array_ninth`). Разбор модификаторов — [reference/placeholders.md](../reference/placeholders.md).

### Почему у разных Advertiser-ов разные поля (settings)

**Набор полей формы Advertiser-а меняется под выбранную интеграцию** (`Integration type` / плитку `Destination Type`) — это не баг и не разные версии UI. У Advertiser-а есть блок настроек интеграции `settings` (json-схема): её поля **приходят от выбранной интеграции**, поэтому под `AdCombo`, `IREV`, `Simple Redirect` форма показывает разные поля. Общие поля (`Name`, `Description`, `Tags`, `Advertiser type`, `Integration type`, `Image`) есть всегда; блок `Settings` — свой у каждой платформы, и `URL Parameters` — как раз его поле, объявленное интеграцией `Simple Redirect`.

Конкретный набор полей под платформу — из её API-документации (как и у Destination `By Integration` — см. «Обязательные поля популярных интеграций» ниже).

Поля `settings` под конкретную интеграцию — это её креды и параметры подключения (эндпоинты, API-ключи/токены, ID партнёра/оффера). Набор дословных ключей `settings` под каждую платформу берётся из её интеграционного шаблона, не хардкодится в UI.

### Как сгруппировать Advertiser-ов по типу (Advertiser Types)

Advertiser Types — группировка Advertiser-ов под одной категорией (по вертикали или направлению — например `Finance`, `Nutra`, `Gambling`).

**Путь:** `Advertisers → создать Type → внутри Type добавлять Advertiser-ов`.

## Destination через By Advertiser

**Рекомендуемый путь.** Привязка к Advertiser даёт автоматический «хвостик» URL, корректную аналитику по `Offer Visits` и `Advertiser`, единый Replace URL при смене Advertiser.

**Путь:** `Tracker → Destinations → +Destination → By Advertiser → выбрать Advertiser`.

**Атрибуты:**

- **Name** — имя оффера.
- **Countries** — гео категоризация (**не фильтр**).
- **Languages** — языки категоризация.
- **URL оффера** — основной URL.
- Хвостик от Advertiser добавляется **автоматически**.

### Почему именно By Advertiser

Полная эвристика — [models/conversion-model.md](../models/conversion-model.md).

## Destination типа Simple Redirect

Голый редирект-URL без адвертайзера, без API-интеграции, без автологина. **Это не штатный путь для реальных офферов:** раз нет Advertiser, ломается аналитика по адвертайзеру (`Offer Visits`, разбивка по Advertiser), а замена URL идёт в каждый Destination руками. Штатно: редирект/ссылка-оффер → `By Advertiser`, API-интеграция → `By Integration`. `Simple Redirect` остаётся под узкие случаи — быстрый редирект в TG-бота, разовый/временный URL.

**`Simple Redirect` создаётся БЕЗ выбора Advertiser** — это его отличие от `By Advertiser`: Advertiser не назначается, поэтому нет автоматического «хвостика» параметров и привязки к Advertiser в аналитике. Задаётся только сам redirect-URL (с плейсхолдерами визита).

**Путь:** `+Destination → Simple Redirect → задать Redirect URL` (без выбора Advertiser).

Для реального оффера — `By Advertiser` (редирект/ссылка) или `By Integration` (API) (см. эвристику в [models/conversion-model.md](../models/conversion-model.md)): аналитика по Advertiser и `Offer Visits` работают именно потому, что у `By Advertiser` есть Advertiser, которого у `Simple Redirect` нет.

## Какой реферер уезжает рекламодателю — `Referrer-Policy: no-referrer` на любом ответе AIO

**Реферер (referer / referrer, «откуда пришёл юзер») до рекламодателя не доезжает.** На любом редиректе через `Destination` (`301` по умолчанию и `302`) и на любом HTML, который отдаёт AIO, — лэнд, страница-прослойка, страница `Field verification` — стоит заголовок `Referrer-Policy: no-referrer`. Это в силе для всех типов Destination (`Simple Redirect`, `By Advertiser` с редирект-URL, `By Integration`) и для переходов с лэнда, включая переходы по JS: политику задаёт сам документ лэнда, поэтому браузер не шлёт `Referer` ни на клик по `{{link}}`, ни на `location.href` из скрипта. Настройки, которая это выключает, нет — заголовок ставится всегда.

На sub-параметры это не влияет: они уезжают параметрами URL / полями payload, и если передавать их не нужно — убираются из ссылки или из настроек Destination.

### Safari игнорирует `Referrer-Policy` на 30x — когда нужна стратегия `Meta Refresh No Referer`

Safari/WebKit не учитывает `Referrer-Policy` на 30x-ответах и на прямом редиректе без лэнда (визит уходит на оффер сразу) может дослать реферер входящего запроса. Если срез реферера нужен во всех браузерах, на шаге ставится `Redirect strategy` = `Meta Refresh No Referer` — вместо 30x визитёру отдаётся HTML-прослойка с политикой документа. Стратегии редиректа и на каких узлах они есть — [models/flow-model.md](../models/flow-model.md).

### Рекламодателю нужен реферер — отдать его параметром `{{aio.visit.fields.referer}}` или `{{aio.visit.domain}}`

Реферер, который AIO хранит на визите, — это реферер **первого захода**: заголовок `Referer` запроса, которым визит создался (площадка или страница, с которой пришёл клик). Он пишется один раз и по пути визита не меняется; на визите лежит дважды — колонкой `Referer` и системным полем `Referer` (slug `referer`) — разбор в [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Рекламодатель же под «реферером» обычно ждёт адрес страницы, с которой пришёл лид, — то есть домен лэнда. Отдаётся параметром `URL Parameters` / полем payload:

- `{{aio.visit.fields.referer}}` — реферер захода (площадка). Отдельного `{{aio.visit.referer}}` нет — только через поле.

- `{{aio.visit.domain}}` — домен визита; в примерах дестинейшенов из шаблона тенанта поле интеграции `referrer` (у интеграций, где оно есть) заполнено именно им.

## Destination типа Telegram (Telegram Destination)

Когда: нужно загнать трафик не на форму рекламодателя, а в **Telegram-бот / канал**. Целевое действие здесь — **подписка на канал**, а не сабмит формы.

**Путь:** `Tracker → Destinations → +Destination` → плитка `Telegram` (у созданного Destination `Integration type` = `Telegram Destination`). Два обязательных поля — `Sender Provider` и `Behaviour`.

- **Sender Provider** — заранее созданный провайдер типа `Telegram` (бот). Тот же Telegram Sender Provider обслуживает и маркетинг-рассылки, и Telegram Destination. Поведение бота (приветствие, приглашение в канал, типы конверсий) настраивается **внутри Sender Provider** — см. [how-to/push-notifications.md](push-notifications.md) → Telegram Sender Provider.

### Как визит попадает в Telegram (Behaviour)

Поле **`Behaviour`** задаёт, куда ведёт визит:

| Behaviour | Что происходит |
|---|---|
| **`Bot -> Channel`** | Визит сначала попадает в **бота**: бот шлёт приветственное сообщение и кнопку-приглашение вступить в канал (текст и кнопка настраиваются в Sender Provider). Юзер вступает из бота. |
| **`Channel`** | Визит **редиректится сразу в канал**, без промежуточного бота. |

**Бот должен быть админом канала.** Иначе Telegram не отдаёт боту событие вступления, и **конверсия подписки на канал не зафиксируется** — обратная петля конверсии ломается. Это требование Telegram к правам бота, не специфика AIO.

### Лёгкий путь в TG-бота — через Simple Redirect

Если нужно просто загнать трафик в Telegram-бота (без приглашения в канал и фиксации подписки), полный `Telegram Destination` с Sender Provider **не нужен**. Достаточно обычного `Simple Redirect` Destination (или `By Advertiser`, если завёл отдельный Advertiser под Telegram). В поле `Redirect URL` — deep-link бота со start-параметром, передающим UUID визита:

```
https://t.me/<bot_username>?start={{aio.visit.uuid}}
```

Этого хватает, чтобы AIO смэтчил конверсию по визиту. Полный `Telegram Destination` через Sender Provider (`Behaviour` `Bot -> Channel` / `Channel`) нужен только когда требуется приглашение в **канал** и фиксация подписки — см. «Destination типа Telegram (Telegram Destination)» выше и [how-to/push-notifications.md](push-notifications.md).

## Destination типа By Integration

`By Integration` — создание Destination на основе **готового API-шаблона**, уже встроенного в AIO (raw-интеграция напрямую с API внешней платформы, в отличие от `By Advertiser`, где интеграция привязана к сохранённому Advertiser).

**Путь:** `+Destination → By Integration → выбрать API-шаблон из списка`. После выбора шаблона открывается окно **Create New Destination** с базовыми полями (Name, Description, Tags, Countries, Languages, `Cap Monitoring User`).

**Остальные поля payload зависят от выбранной платформы** — у каждой интеграции свой набор. Что и как заполнять — смотреть в **API-документации платформы**, с которой интегрируешься. Список AIO-плейсхолдеров для маппинга — по иконке-шестиграннику. Click ID в AIO = Visit UUID, плейсхолдер `{{aio.visit.uuid}}`.

Если нужной интеграции в списке нет — можно **запросить новую**: открывается поп-ап с контактами интеграционного партнёра (процесс заведения новой интеграции — [reference/glossary.md](../reference/glossary.md) → «Запрос новой интеграции»).

### Обязательные поля популярных интеграций

Базовый паттерн любой интеграции — `API URL` + `API Key` + специфичные для платформы поля. Конкретные наборы для частых получателей:

| Платформа | Обязательные поля |
|---|---|
| **IREV** | `API URL`, `API Key`, `IREV Affiliate ID`, `IREV Offer ID` |
| **Getlinked** | `API URL`, `API Key`, `Offer Name`, `Comment` |
| **Trackbox** | `API URL`, `API Key`, `API Username`, `API Password`, `Ai`, `Ci`, `Gi` |
| **Voralis** | `API Token`, `Affiliate`, `First Name`, `Phone`, `Country`, `Quantity` — плюс `Product ID` **или** `Product Name` (заполнить нужно хотя бы одно; при обоих заполненных приоритет у `Product ID`) |
| **Cryptahl** | `Token`, `First Name`, `Phone`, `Email`, `Country Code`, `Sub ID` |

У **`AlterCpa`** и **`AlterCpa Moe`** есть необязательное поле `Item` (плейсхолдер `Ex: 3`) — незаполненным оно в запрос к платформе не уходит.

(Полный каталог интеграций постоянно расширяется — [reference/glossary.md](../reference/glossary.md) → «Destination Integrations».)

## Редактор Destination (Edit destination)

Созданный Destination виден в таблице `Tracker → Destinations`, а также доступен через `Tracker → Campaigns` (внутри кампаний, где он используется).

В таблице `Tracker → Destinations` нужный дестинейшн ищется строкой поиска по **имени, номеру или UUID**, либо отбирается кнопкой-фильтром **`Advertiser`**.

### Кнопка `#1` над таблицей Destinations — у оффера нули, хотя лиды на него идут

Метрики строки в `Tracker → Destinations` считаются **по позиции пуша**: жёлтая кнопка `#1` в блоке быстрых фильтров задаёт, за какую позицию в визите дестинейшн получает цифры, клик переключает `#1` → `#2` → `#1`. Позиция — это порядковый номер **успешного** пуша в визите (реджекты и повторный пуш в тот же дестинейшн позицию не добавляют), а не место ноды во флоу. Поэтому оффер, в который визиты уходят вторым пушем (после реджекта первого или вторым шагом `Destination`), при `#1` показывает нули — переключите на `#2`. Значение хранится в браузере и общее для списков Landings и Destinations; разбор кнопки и тулбара — [reference/ui-common.md](../reference/ui-common.md), как визит нумерует офферы — [models/destination.md](../models/destination.md).

`Tracker → Destinations` → правый клик → **Edit destination**. Поля:

- Name, Description, Tags, Countries, Languages.
- **Cap monitoring user** — кому летят алерты по капам.
- **Integration type** — шаблон интеграции получателя (`Bridge`, `IREV`, `API integration` и др. пребилты). От него зависят поля payload ниже.
- **Поля payload** (зависят от Integration type) — маппинг данных визита в запрос к получателю через плейсхолдеры: `Token`, `Buyer UTM` = `{{aio.visit.fields.buyer_utm}}`, `Country Code` = `{{aio.visit.country_code}}`, `Language Code` = `{{aio.visit.language_code}}` и т.д.

**Экшены (правый клик):** `Edit destination`, `Copy destination`, `Destination logs`, `Switch archive`, `Change domains`, `Assign to folder`, `Edit conversion cap`, `Reset cap`, `Show visits`, `Show conversions`, `Build Roll Up report`, `Share`, `Change ownership`.

### Где в карточке дестинейшена задаются поля визита

В формах создания и правки дестинейшена (все способы) есть секция `User fields`: поля визита заполняются прямо в карточке. В формах, где выбирается `Advertiser`, к ним добавляются поля, унаследованные от него; в `By Integration` родителя нет — только свои. Механика — [how-to/user-fields.md](user-fields.md).

### `Change domains` — точечная замена домена в URL дестинейшена

`Change domains` (правый клик по дестинейшену) — текстовый свап домена в URL: задаются `From domain` → `To domain`, и подстрока домена заменяется по всему URL дестинейшена.

Не путать с кампанийным `Replace domain`: там домен выбирается из дропдауна, здесь — find/replace по строке.

## Как ограничить число конверсий на дестинейшн (Conversion Cap)

`Conversion Cap` — капа дестинейшена: лимит конверсий, после которого лиды на этот дестинейшн перестают уходить. **Путь:** `Tracker → Destinations → <D>` → правый клик → **`Edit conversion cap`**. Соседний экшен **`Reset cap`** обнуляет счётчик. На странице дестинейшенов MTK капов нет.

Капа гейтит **исходящую доставку лида в получателя**: когда счётчик конверсий достигает лимита, новый пуш лида в дестинейшн не уходит — визит отправляется в fallback `Destination Full` (разбор — секция «Fallback при переполнении капы» этого дока). Так работает стратегия `When the cap is full` = `Reject traffic` — дефолт; при `Do nothing` капа только считает и трафик не режет (секция «`When the cap is full`» ниже). Приём входящих постбэков капа **не** блокирует — постбэк создаёт конверсию и инкрементит счётчик капы. Симптом «конверсия по офферу не растёт» смотри со стороны исходящего пуша (`Destination Full` / реджект получателя), а не приёма постбэка.

### Экшен `Reset cap` — что именно он обнуляет

`Reset cap` (правый клик по дестинейшену) обнуляет счётчик капы, счётчик отклонённых по ней и дату начала периода — после этого дестинейшн перестаёт считаться заполненным. Обязательное поле диалога — `Cap type` (тултип `Type of cap to reset`): счётчик конверсий обнуляет **только** значение `Conversion Cap`.

В селекте по умолчанию подставлено другое значение — `Push Cap`. Сохранение с ним конверсионный счётчик не трогает: дестинейшн остаётся полным, хотя сброс формально прошёл.

### Как читать колонку `Conversions Cap` в таблице

Колонка `Conversions Cap` в `Tracker → Destinations` собрана из четырёх элементов: иконка типа капы (календарь — `Daily`, флажок — `Lifetime`, знак бесконечности — `Infinity`; тултип на иконке — `Daily cap` / `Lifetime cap` / `Infinity cap`), шкала заполнения, числа `использовано / лимит` (тултипы `Used` и `Total cap`) и красный бейдж `+N` (тултип `Rejected`) — он рисуется только при ненулевом счётчике отклонённых по капе.

У безлимитной капы числа расхода не выводятся вовсе — только иконка и нейтрально-полная шкала.

### Бейджа `+N` нет, хотя лиды отбиваются по капе — счётчик отклонённых не растёт

Счётчик отклонённых по капе, который стоит за бейджем `+N`, при реджекте по капе не увеличивается: пуш пишет реджект на визит, но счётчик дестинейшена не трогает. Поэтому у переполненного дестинейшена бейджа, как правило, нет, а `Reset cap` и полуночный сброс `Daily` обнуляют и так нулевое значение. Сколько лидов отбила капа, считается по визитам: в `Tracker → Visits` у них `Last Rejection Type` = `Cap` и `Last Rejection Reason` = `Conversion Cap is Full` ([models/destination.md](../models/destination.md)).

### Ограничения Cap — что не поддерживается и как считается

- **Месячных кап нет** — период капы либо сутки (`Daily`), либо всё время жизни дестинейшена (`Lifetime`).
- **Под-кап на один оффер под разных байеров — нет**.
- Механика подсчёта и блокирующая логика кап — [models/conversion-model.md](../models/conversion-model.md).
- При переполнении капы со стратегией `Reject traffic` — алерт в Telegram назначенному `Cap Monitoring User` (доставка требует привязки аккаунта к Telegram, как у любого Monitoring User; 2FA через Google Authenticator привязку не даёт).

### Fallback при переполнении капы (Destination Full)

Fallback при переполнении — через `Destination Full` transition во флоу ([how-to/flow-editor.md](flow-editor.md)). **Этот переход надо нарисовать явно:** если ветки `Destination Full` нет, переполненная капа откатывает визит на предыдущий оффер так же, как обычный реджект (деградация в `Destination-rejected` → `Destination-interacted` → неявный возврат) — разбор цепочки в [models/destination.md](../models/destination.md) → «Капа полна, а переход `Destination Full` не нарисован».

Переполненные `Daily` и `Lifetime` блокируют трафик одинаково, но `Lifetime` сам по себе не отпустит: после полуночи он остаётся полным, пока лимит не поднимут или счётчик не сбросят.

## Какой `Cap Type` выбрать: `Unlimited`, `Daily` или `Lifetime`

Тип капы выбирается в диалоге `Edit conversion cap` тремя кнопками — `Unlimited`, `Daily`, `Lifetime`; под кнопками показывается подсказка про выбранный тип. Отличаются типы ровно одним — когда обнуляется счётчик. Блокировка трафика у `Daily` и `Lifetime` одинаковая, месячного периода нет ни у одного.

`Unlimited` и `Infinity` — один и тот же тип: в диалоге он подписан `Unlimited`, в колонке таблицы и в API — `Infinity`.

### `Unlimited` (`Infinity`) — лимита нет

`Unlimited` означает, что капы у дестинейшена нет: поле `Limit` в диалоге скрыто, лиды уходят без ограничения. Это состояние нового дестинейшена — капа задаётся уже после создания, отдельным экшеном.

Счётчик при `Unlimited` не заморожен: если в капе выбран `Conversion type`, конверсии этого типа его увеличивают, а полуночный пересчёт обнуляет. Поэтому дестинейшн, переведённый с `Unlimited` на `Daily` или `Lifetime` среди дня, стартует не с нуля, а с расходом, накопленным с прошлой полуночи.

### `Daily` — счётчик обнуляется в полночь зоны `Reset timezone` (по умолчанию UTC)

`Daily` — суточная капа: счётчик обнуляется раз в сутки в полночь той таймзоны, что выбрана в поле `Reset timezone` диалога `Edit conversion cap` (подсказка в диалоге — `The counter resets every day at midnight.`, тултип поля — `Daily cap resets at local midnight of this timezone. Empty — UTC.`). Поле появляется только при `Cap type` = `Daily`, значение выбирается из списка таймзон, пустое (плейсхолдер `UTC (default)`) = полночь по UTC. Обнуление делает фоновый пересчёт, который идёт раз в пять минут, — то есть с лагом до нескольких минут после полуночи; механика подсчёта в [models/destination.md](../models/destination.md).

Вместе со счётчиком обнуляется и число отклонённых по капе, и двигается дата начала периода.

### `Lifetime` — счётчик не обнуляется никогда

`Lifetime` — капа на всё время жизни дестинейшена: счётчик копится и полуночным пересчётом не трогается (подсказка в диалоге — `The counter never resets — the limit applies to the whole lifetime.`).

Обнулить счётчик можно только руками — кнопкой `Reset counter` в диалоге `Edit conversion cap` или экшеном `Reset cap`. Поднятый `Limit` счётчик не трогает: он отодвигает потолок, накопленный расход остаётся. Пока не сделано ни того ни другого, заполненная `Lifetime`-капа держит дестинейшн закрытым и на следующие сутки.

## Диалог `Edit conversion cap` — какие поля заполнять

Диалог `Edit conversion cap` состоит из шести блоков:

- **`Conversion type`** — обязательное поле: тип конверсии, который капа считает. Счётчик растёт **только** на конверсиях этого типа, конверсии других типов капу не расходуют.
- **`Cap type`** — три кнопки выбора типа (`Unlimited` / `Daily` / `Lifetime`) с подсказкой под ними.
- **`Limit`** (`Лимит`, плейсхолдер `Conversions allowed`) — числовой лимит.
- **`Reset timezone`** — только при `Daily`: таймзона, по чьей полуночи обнуляется счётчик; пусто = UTC (секция «`Daily`» выше).
- **`When the cap is full`** — две кнопки `Reject traffic` / `Do nothing` с подсказкой (секция ниже).
- **`Current usage`** (`Текущий расход`) — шкала с числами «использовано / лимит» и кнопка сброса счётчика.

При `Cap type = Unlimited` остаются только первые два блока: ограничивать, считать и выбирать стратегию там нечего.

### `When the cap is full` — `Reject traffic` (дефолт) или `Do nothing`: капа только считает

`When the cap is full` — что делать с трафиком, когда капа заполнена; дефолт и историческое поведение — `Reject traffic`. С ним при полной капе пуш в получателя не уходит, визит получает реджект с `Last Rejection Type` = `Cap` и уходит во флоу по переходу `Destination Full` (подсказка под кнопкой: `Traffic is not pushed to the advertiser: a reject conversion is spawned and the flow gets a Destination Full event — it can be routed to a fallback.`).

`Do nothing` делает капу информативной: лиды продолжают уходить в получателя, счётчик просто растёт дальше (подсказка: `Traffic keeps going through — the cap only counts. Set up a notification rule for alerts.`). Про заполнение тогда узнают только из уведомления — правило по капе дестинейшена на вкладке `Tenant alerts` ([mechanics/notification-center.md](../mechanics/notification-center.md)); Telegram-алерт `Cap Monitoring User` о заполненной капе при `Do nothing` не шлётся, переход `Destination Full` не срабатывает, реджектов по капе на визитах нет.

### Что делает кнопка `Reset counter`

`Reset counter` (`Сбросить счётчик`) сразу ничего не меняет — она лишь помечает счётчик к обнулению, а применяется вместе с `Save`. Помеченный сброс виден прямо в `Current usage`: текущее число со стрелкой на `0`. До сохранения решение отменяется повторным нажатием, кнопка при этом подписана `Cancel reset` (`Отменить сброс`).

Ручное уменьшение счётчика считается началом нового периода капы: двигается дата сброса, и пороги правила уведомлений по капе разрешаются заново ([mechanics/notification-center.md](../mechanics/notification-center.md)). Счётчик отклонённых по капе кнопка не трогает — его обнуляет экшен `Reset cap`.

### Смена `Cap Type` счётчик не обнуляет — расход переносится

Перевод капы с одного типа на другой (например с `Daily` на `Lifetime`) сохраняет текущий расход: смена типа сама по себе счётчик не трогает. Чтобы начать счёт с нуля, в том же диалоге нажимается `Reset counter` перед сохранением — либо отдельно выполняется экшен `Reset cap`.

Типовая ловушка: дестинейшн, переведённый в `Lifetime` среди дня, уже несёт накопленный за этот день расход — и после полуночи его больше не потеряет.

## PWA-Destination через сторонний сервис

Когда: ПВА-оффер у стороннего PWA-сервиса.

**Шаги:**

1. **Получить ссылку из AIO:** `Sources → +Source → 3rd Party PWA Link Generator → выбрать домен → выбрать сервис → скопировать ссылку`.
2. **В сервисе:** создать пвашку → вставить ссылку из AIO → получить домен пвашки от сервиса.
3. **Создать Destination в AIO:** `Tracker → Destinations → +Destination → By Advertiser → выбрать PWA-сервис → вставить домен пвашки`.

**Поля PWA-Destination** (`By Advertiser`): Name, Description, Tags, Countries, Languages, `Cap Monitoring User`, сам `Advertiser` и поля его интеграции — среди них `Redirect URL`, куда вставляется домен пвашки (параметры advertiser-а допишутся к URL автоматически). Капа в форме создания не задаётся: её ставят уже созданному дестинейшену экшеном `Edit conversion cap`.

Также см. [mechanics/pwa.md](../mechanics/pwa.md) → 3rd Party PWA.

Через сторонний сервис передаётся **только Sub-параметр** с ID сессии визита — этого достаточно для возврата визита. Параметры источника AIO пишет **до** пвашки.

### Завести свой PWA-сервис как Advertiser

Если нужного PWA-сервиса нет среди дефолтных advertiser-ов, его можно добавить вручную.

1. **Создать Advertiser Type** (если такого типа ещё нет): `Settings → Advertisers → стрелка у +Advertiser → Manage Types → +Advertiser Type`. Поля: `Name` (можно `PWA`), `Color`, `Icon`, `Description` и обязательно **`Metrics Event Group = PWA`** — без него метрики по PWA-полям не работают.
2. **Создать Advertiser:** `+Advertiser` с `Advertiser type` = `PWA`, `Integration type` = `Simple Redirect`. Заполнить `URL Parameters` (параметры, что допишутся к Destination URL; список плейсхолдеров — по иконке-шестиграннику), `Logo`/`Description`/`Tags`.
3. **Создать Destination:** через `By Advertiser`, выбрав этот Advertiser (поля — как в списке выше).

### Как настроить постбэк-трекер для стороннего PWA-сервиса

Чтобы конверсии PWA-сервиса доходили в AIO, ставится постбэк-трекер. Цепочка постбэков: **рекламодатель → AIO → PWA-сервис**.

**Путь:** `Tracker → Trackers → +Tracker` типа `HTTP GET`. Поля:

- **`Sources`** — `3rd Party PWA Link Generator` (рекомендуется по умолчанию — этот source покрывает все сторонние PWA-сервисы).
- **`Conversion Type`** — тип конверсии, по которому срабатывает постбэк.
- **`URL`** — постбэк-URL сервиса; макросы сервиса заменяются на AIO-плейсхолдеры.

Click ID PWA-сервиса доступен плейсхолдером `{{aio.visit.fields.pwa_click_id}}`. Постбэки AIO для рекламодателя берутся в [how-to/source-trackers.md](source-trackers.md).

### Как собрать кампанию и ссылки под сторонний PWA-сервис

После того как Destination-ы (PWA + Offer) и постбэк-трекер готовы — собирается кампания.

- **Flow.** В тенанте под PWA-шаблон уже есть два дефолтных флоу, выбираются в `Select Flow` при создании кампании:
  - **`PWA / OneLink Flow Default`** — дефолт `AIO → PWA → оффер`.
  - **`PWA / OneLink w/ Landing`** — добавляет лэндинг перед редиректом на оффер.
- **Campaign steps:** `Destination` = ранее созданный PWA-Destination, `Offer` = ранее созданный Offer-Destination.
- **Install-конверсия** для стороннего PWA-сервиса настраивается на шаге `Spawn Conversion` флоу (`Tracker → Flows`), **не** постбэком: AIO спавнит `Install` при редиректе визита с PWA-сервиса на Offer-Destination. (Сравнение со спавном у внутренней PWA — [how-to/pwa.md](pwa.md).)

### Какие две ссылки генерятся под сторонний PWA-сервис

Под сторонний PWA-сервис генерируются **две разные ссылки** (обе через `Link Generator`):

1. **PWA-ссылка** — Source = `3rd Party PWA Link Generator`; вставляется в сам PWA-сервис как offer link.
2. **Трекинговая ссылка** под твой реальный Source (например, FB) — вставляется в рекламный источник. Если FB — выбирается преднастроенный source **`FB w/ CAPI – DEFAULT`** (уже в тенанте).

## AIO PWA (PWA Builder) — Destination как лэнд

Когда: внутренняя ПВА AIO (без стороннего сервиса). Создаётся как лэнд через PWA Builder, дальше работает как обычный лэнд во флоу.

**Путь:** `Content → PWA` → кнопка `+ PWA` — премодалки с выбором способа здесь нет, сразу открывается PWA Builder. В `Content → Landings` PWA не создаётся и не показывается. Процедура целиком — [how-to/pwa.md](pwa.md).

**Шаги:**

1. Настроить элементы PWA (manifest, иконки, splash).
2. Опционально — `Content Splits` для тестирования.
3. Привязать к флоу как обычный лэнд.

Также см. [mechanics/pwa.md](../mechanics/pwa.md), [how-to/landings.md](landings.md).

## Типичные ошибки

- **Destination сделан через Simple Redirect.** Аналитика по Advertiser ломается, замена URL — в каждом Destination. Переделать через `By Advertiser`.
- **Капа в `Daily`, ждали сброс по локальному времени.** Сброс — в **0:00 UTC**.
- **Хочу месячную капу.** Месячной нет: либо `Daily` (сброс каждые сутки), либо `Lifetime` (без сброса вообще).
- **Перевели капу в `Lifetime`, а она сразу полная.** Смена `Cap Type` счётчик не обнуляет — расход переносится. Обнулить: `Reset counter` в диалоге или экшен `Reset cap`.
- **Капа не растёт, хотя конверсии идут.** Счётчик считает только конверсии того `Conversion type`, что выбран в самой капе.
- **Archive Destination → продолжает распределять трафик.** Archive Destination **не убирает** его из кампаний, где он уже используется. Чтобы отключить — деактивировать **в самой кампании** (или через дистрибуцию).
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
