---
id: generations
title: Генерации — как сгенерировать лэндинг или креатив через AI-провайдера (страница Generations)
description: "Страница Content → Generations (только ERP): генерация лэндинга (сайт ZIP-архивом) через AIO Site Generator — готовый сайт, сайт под заказ или большой сайт по брифу (поле Mode) — или внешний генератор сайтов и креатива (статичные картинки) через Higgsfield; подключения в Settings → Generators, поля формы каждого провайдера (язык, страна, логотип, WordPress-вид, контакты), статусы Pending / Processing / Fine / Issues / Canceled, прогресс в процентах, карточка и Embed preview, Create landing из архива, Retry / Refresh status / Cancel, уведомление и ссылки по умолчанию на 7 дней, колонка Cost, лимиты сайтов, доступ и права content.generations.*, почему генерация висит или упала."
doc_type: how-to
builds: [erp]
related: [placeholders, landings, notification-center, permissions-model]
language: ru
updated: 2026-10-16
---

# Генерации — как сгенерировать лэндинг или креатив через AI-провайдера (страница Generations)

**Генерация (`Generation`) — запуск генератора прямо из AIO: тип `Landing` отдаёт готовый сайт ZIP-архивом, тип `Creative` — статичные картинки для рекламы.** Генерации собраны на странице `Content → Generations` (только в полной версии, ERP), запускаются через подключения к провайдерам из `Settings → Generators` и идут асинхронно: у каждой есть статус, уведомление о готовности и результат, который скачивается или превращается в лэнд. Так же спрашивают: «сгенерировать лэндинг», «AI-генерация лендинга», «нагенерить креатив», «генератор сайтов в AIO», «сделать сайт через AI».

## Где страница `Generations` и что на ней видно

**Страница — `Content → Generations` (`/app/landers-creatives/generations`), открывается с правом `Content: generations page` (`content.generations.view`); в MTK её нет.** По умолчанию генерации показаны карточками с превью, таблица включается переключателем вида. Колонки таблицы: `Generation` (Human ID, иконка типа, глаз превью, значок статуса и имя), `Type`, `Cost`, `Generator` (подключение), `Settings` (параметры запуска, до трёх значений и «+N»), `Owner`, `Created`, `Finished`.

Статус отдельной колонкой не выводится, он в ячейке `Generation`: у `Pending` и `Processing` крутится значок, у `Issues` — красный крестик, у `Canceled` — серый минус, у `Fine` значка нет. Подсказка на значке — текст стадии или причины.

Быстрые фильтры — `Archived`, `Type`, `Status`, `Owner`, `Tags`; в полном списке ещё `Provider`, `Generator`, `Team` и папки. Поиск — по имени, UUID и Human ID. Архивные генерации скрыты, пока не включён фильтр `Archived`.

## Как запустить генерацию лэндинга или креатива — `+ Generation`

**Генерация запускается кнопкой `+ Generation` на странице `Content → Generations`: выберите `Landing` или `Creative`, подключение провайдера и параметры, затем нажмите `Generate`.** Право — `Content: generations create & edit` (`content.generations.edit`).

1. `+ Generation` → карточка `Landing` («A ready-made website as a ZIP archive — upload it as a landing or download it.») или `Creative` («Static ad images in the sizes you need — download them or use them as creatives.»).
2. `Provider` — подключение из `Settings → Generators`, которое умеет выбранный тип. Единственное подходящее подключение подставляется само; если подходящих нет, в форме текст `No providers available. Add a connection in Settings → Generators.`
3. Блок `Settings` — параметры провайдера. Смена подключения сбрасывает заполненные параметры.
4. Свёрнутый блок `Name, description & tags`. Имя можно оставить пустым («Leave empty to name it automatically»): до ответа провайдера генерация называется `Landing generation` / `Creative generation`, потом AIO Site Generator подставляет имя вида `<компания сайта> — <вид сайта>`, а Higgsfield — начало промпта.
5. `Generate` — всплывает `Generation started — you'll get a notification when it's ready`, генерация появляется в списке в статусе `Pending`.

## Как подключить генератор — `Settings → Generators`

**Генерация идёт только через подключение (`Generator`) из `Settings → Generators` — запись с именем, типом интеграции и её настройками; без подключения нужного типа выбрать провайдера в форме генерации нельзя.** Раздел есть только в ERP: страница открывается правом `Settings: generators page`, создание и правка — `Settings: generators edit`. Кнопка `+ Generator` → поля `Generator name`, `Integration type` и блок `Settings` под выбранный тип.

| `Integration type` | Что генерирует | Что вписать в `Settings` |
|---|---|---|
| `AIO Site Generator` | `Landing` — готовый сайт, сайт под заказ или большой сайт по брифу | ничего: доступ к сервису на стороне AIO, но подключение всё равно нужно создать |
| `Higgsfield` | `Creative` — картинки | `API Key` из консоли Higgsfield (`console.higgsfield.ai → API keys`) |
| внешний генератор сайтов | `Landing` — готовый сайт | `API Key` своего аккаунта в этом сервисе |

В поле `Provider` формы генерации попадают только неархивные подключения, к которым у вас есть уровень доступа не ниже `Use`. Одно подключение удобно завести на команду и раздать доступ — секция «Кто видит генерации и подключения — владелец, `Access type`, шаринг» этого документа.

## Какие параметры у `AIO Site Generator` — режим, вид сайта, стиль, тема, язык, страна, размер, WordPress-вид

**`AIO Site Generator` делает сайт ZIP-архивом в одном из трёх режимов поля `Mode`: готовый сайт, сайт под заказ или большой сайт по брифу. Обязательны `Mode`, `Site type` и `Language`, в режиме `Huge site by brief` — ещё бриф `Who the brand is`.** Когда в поле `Provider` выбрано такое подключение, под ним появляется пометка `Free for now — we're testing AIO Site Generator before the paid launch.`

- `Mode` — режим; от него зависит набор остальных полей.
- `Site type` — `Random` (по умолчанию, подсказка `Random — the type with the most ready sites`) или вид из списка генератора: сайт компании, лендинг услуги, портфолио, личный сайт, блог, журнал, новостной сайт, витрина продукта или приложения, событие, курс, сообщество, заведение.
- `Style` — `Random` (по умолчанию), `Corporate`, `Minimal`, `Bold`, `Warm`, `Elegant`, `Tech`, `Fresh`, `Classic`.
- `Topic` — `Random` (по умолчанию) или тема из списка генератора.
- `Site structure` — `Type default` (по умолчанию), `One page with sections (to order)`, `Multiple pages`; одна страница — только под заказ (секция «Одностраничный сайт — только под заказ»).
- `Language` и `Country` — язык сайта и страна, в которой он «живёт».
- `Size` — `Small (4-6 pages)`, `Medium (6-9 pages)` (по умолчанию), `Large (9-14 pages)`.
- `Look` — `Regular` (по умолчанию) или `WordPress`: тот же сайт, пересобранный как статический WordPress-сайт (WordPress-версия сайта).
- `Company name`, `Logo`, `Brand colour`, `Contacts on the site`, `Quality` и бриф — поля бренда и режимов, у каждого своя секция в этом документе.

Форма разбита на блоки: режим и его поля, `Brief` (только в `Huge site by brief`), `Site type, style and content`, `Language and country` и два свёрнутых — `Your brand` (`Optional: your name, logo and colour instead of invented ones`) и `Contacts` (`Optional: whose contacts the site shows`).

### Какие режимы у AIO Site Generator — `Ready site`, `Generate to order`, `Huge site by brief` (сайт под заказ, большой сайт по брифу)

**Первое поле формы AIO Site Generator — `Mode`, по умолчанию `Ready site`; режим решает, выдать готовый сайт или сгенерировать новый с нуля.** Режимы стоят карточками с описанием, у двух под заказ — пометка `paid`. Подсказка поля — `Ready site — instant from the pool (multi-page only); to order — a new site in minutes; huge — 32–38 pages by your brief`.

- `Ready site` (`Instant · multi-page sites from the pool`) — готовый многостраничный сайт из пула, приходит сразу. Если точно подходящего готового сайта нет, исход решает поле `If no exact match` (секция «Что будет, если точного готового сайта нет»).
- `Generate to order` (`A new site in 2–4 min`, `paid`) — сайт под заказ: если подходящего готового сайта нет, за несколько минут генерируется новый.
- `Huge site by brief` (`32–38 pages by your brief · up to 60 min`, `paid`) — всегда новый большой сайт на 32–38 страниц по брифу бренда, до 60 минут.

Набор полей зависит от режима. `Quality` есть только в `Generate to order`, `If no exact match` — только в `Ready site`, бриф `Who the brand is` и `Wishes for the site` — только в `Huge site by brief`. `Topic`, `Site structure` и `Size` есть везде, кроме `Huge site by brief`. `Site type`, `Style`, `Language`, `Country`, `Look`, блок `Your brand` (`Company name`, `Logo`, `Brand colour`) и `Contacts on the site` — во всех режимах. Поля другого режима форма прячет и не сохраняет.

### Что будет, если точного готового сайта нет — `If no exact match` и метка `closest match`

**В режиме `Ready site` без точного совпадения исход решает поле `If no exact match`: по умолчанию стоит `Generate to order (paid)` — делается новый сайт под заказ, значение `Closest ready site` даёт ближайший готовый сайт.** Поэтому генерация в `Ready site` иногда идёт не секунды, а минуты — статус `Processing` с текстом `Generating site…` (секция «Генерация висит в `Processing`»). `Style`, `Topic` и `Size` — пожелания, подсказка у них — `Preference: the closest site is served if there is no exact match`.

В карточке готовой генерации блок результата показывает, что выдано на самом деле: имя сайта, `Pages: N` со списком страниц и выданные вид, структуру, тему, стиль, язык, `Quality` и `Look`. У значения, которое не совпало с запросом, стоит метка `closest match` («The provider had no exact match for this request and picked the closest option.»); у `Size` метки не бывает, а при `Site structure` = `Type default` выданная структура считается совпадением.

Со значением `Closest ready site` генерация, для которой нет готового сайта нужного вида и языка, ждёт в `Pending` с текстом `No ready sites of this kind yet, retrying…` и повторяет запрос сама — секция «Генерация висит в `Pending`».

### Как работает `Random` у AIO Site Generator — кто выбирает вид, тему и стиль

**При `Random` у `Topic` и `Style` тему и стиль подбирает сам генератор; `Random` у `Site type` в режимах `Ready site` и `Huge site by brief` тоже оставляет выбор генератору, а в `Generate to order` вид выбирает AIO при запуске.** Если при `Random`-виде выбрана `Country`, AIO в любом режиме берёт вид из тех, что принимают эту страну.

Фактически выданное видно в блоке результата карточки, а в параметрах карточки остаётся `Random`. В колонке `Settings` таблицы на месте `Random` стоит выданное значение, а до результата — отправленное генератору; если значение генератору не уходило (тема, стиль, вид в `Ready site` или `Huge site by brief` без страны), до результата этой строки в колонке нет. `Retry` — это новая попытка, и `Random` в ней выбирается заново.

### Одностраничный сайт — только под заказ (`One page with sections (to order)`)

**Готовые сайты в пуле только многостраничные, поэтому `One page with sections (to order)` в `Site structure` — вариант под заказ.** В `Generate to order` он есть всегда, в `Ready site` — только пока `If no exact match` = `Generate to order (paid)`; со значением `Closest ready site` его в списке нет. В `Huge site by brief` поля `Site structure` нет.

Если одностраничная структура всё же ушла в `Ready site` без заказа, генерация получает `Issues` с текстом `One page with sections is made to order only — choose «Generate to order if no exact match»` — в тексте прежнее имя варианта, сейчас это `Generate to order (paid)`. Запустите новую генерацию с этим значением или в `Generate to order`.

### Какие темы `Topic` есть у AIO Site Generator

**Темы `Topic` отдаёт генератор; в списке они сгруппированы по разделам — название раздела стоит заголовком группы, а не в подписи темы (`Law firm` в группе `Services`); по умолчанию стоит `Random`, в режиме `Huge site by brief` поля нет.** Список может пополняться.

- `Health` — фитнес-студия, клиника, салон красоты.
- `Lifestyle` — ресторан, кофейня, турагентство, спортклуб, музыкальная школа, ивент-агентство, активный отдых.
- `Retail` — бутик одежды, интернет-магазин.
- `Services` — автосалон, агентство недвижимости, бухгалтерия, юридическая фирма, IT-консалтинг, студия разработки, языковая школа, строительство, клининг, ветклиника.
- `Creative` — фотостудия, дизайн-агентство.
- Ещё есть магазин настольных игр и брачное агентство.

Своей темы вписать нельзя: поле — выбор из списка.

### Какие языки есть у AIO Site Generator — языки готовых сайтов и `(to order)`

**Язык у AIO Site Generator бывает двух видов: язык готовых сайтов и язык только под заказ — второй подписан в списке суффиксом ` (to order)` и стоит после языков готовых сайтов; по умолчанию — английский.** Подсказка у `Language` — `Languages marked «to order» are generated only (no ready sites)`. Список языков и пометку отдаёт генератор, состав может меняться.

- Готовые сайты: английский, немецкий, французский, испанский, итальянский, португальский (Бразилия и Португалия), нидерландский, польский, турецкий, русский, индонезийский, хорватский, чешский, финский, литовский, словацкий, датский, словенский, японский.
- Только под заказ: румынский, венгерский, болгарский, греческий, латышский, сербский (латиница), малайский, бенгальский, арабский.

Язык с пометкой `(to order)` в режиме `Ready site` не работает: генерация уходит в `Issues` (секция «`Issues` у AIO Site Generator — язык без готовых сайтов или недоступная страна в `Ready site`»). Для такого языка нужен `Generate to order` или `Huge site by brief`.

### Как выбрать страну сайта — поле `Country` (сайт для Канады, Швейцарии, Бельгии)

**Необязательное поле `Country` рядом с `Language` задаёт страну, в которой сайт «живёт»: город, телефон, домен, местное написание языка и правовые страницы; пустое поле — домашняя страна языка.** Подсказка — `The site is set in this country: town, phone, domain, spelling, legal pages`. Так же спрашивают: «сайт под другую страну», «страна сайта». Как устроено поле — зависит от режима:

- **`Ready site`** — поле видно для пары `Language` × `Site type`, для которой в каталоге генератора есть страны; при `Random` оно объединяет страны выбранного языка. В списке — только страны, которые генератор принимает для этой пары язык × вид сайта (placeholder `Home country of the language`); сам список отдаёт генератор:
  - немецкий — Австрия, Швейцария;
  - английский — Великобритания, Канада, Австралия, Сингапур, Кения;
  - французский — Бельгия, Швейцария, Канада;
  - испанский — Аргентина, Уругвай, Чили, Мексика;
  - нидерландский — Бельгия.
  Наличие поля у других языков зависит от каталога генератора. При `Site type` = `Random` в поле все страны языка, а вид сайта при запуске выбирается из тех, что принимают выбранную страну.
- **`Generate to order` и `Huge site by brief`** — одно поле `Country` на все языки и виды сайта: любая страна из каталога (placeholder `Home country of the language or from the brief`), привязки к паре язык × тип нет, допустимость проверяет уже сам генератор при сборке.

### Как задать имя компании, логотип и цвет сайта — блок `Your brand`, `Contacts on the site`, `Quality`

**Свёрнутый блок `Your brand` есть во всех режимах: имя компании, логотип и основной цвет сайта; в `Ready site` готовый сайт пересобирается под них, в режимах под заказ генерируется новый.** Всё необязательно — пустые поля генератор заполнит сам. `Quality` есть только в `Generate to order`.

- `Company name` — имя бренда везде на сайте: тексты, логотип, заголовки, правовые страницы, домен и почта (подсказка `The brand name everywhere: texts, logo, headings, legal pages, domain and email`); пустое поле (placeholder `Invented by the generator`) — имя придумает генератор. До 60 символов.
- `Logo` — картинка PNG, WebP, JPEG, GIF или SVG, 32–4096 px, до 2 МБ: впишите ссылку (placeholder `Enter picture URL`) или выберите картинку из библиотеки кнопкой-папкой справа. Логотип идёт в шапку сайта и в favicon, цвета сайта подстраиваются под него; если загрузить его не удалось, остаётся нарисованный генератором. Генератор получает логотип, только если ссылка начинается с `https://`; другая ссылка молча не учитывается.
- `Brand colour` — основной цвет сайта в формате `#RRGGBB` (подсказка `Main colour of the site, hex #RRGGBB`). Другой формат форма не пропускает: `Brand colour must be a hex colour like #1A73E8`.
- `Quality` — `Standard` (по умолчанию), `High (best sites only)` или `Any`; подсказка `High — only the best-reviewed sites`.

`Contacts on the site` (в блоке `Contacts`, во всех режимах) — `Site domain` (по умолчанию), `Invented` (контакты, придуманные генератором) или `My contacts` (свои контакты — секция ниже). С `Site domain` AIO передаёт генератору домен плейсхолдером `{{aio.visit.domain}}` ([reference/placeholders.md](../reference/placeholders.md)): реальный домен подставляется, когда сайт работает лэндом (`Create landing`), а в `Embed preview` и скачанном архиве плейсхолдеры не обрабатываются.

### Как поставить на сгенерированный сайт свои почту, телефон, юрлицо и адрес — `My contacts`

**Вариант `My contacts` в поле `Contacts on the site` открывает пять полей: `Email`, `Phone`, `Legal entity`, `Address`, `Footer text`; заполненные генератор ставит на сайт вместо придуманных, пустые не передаются.** Домен, на котором открыт сайт, AIO передаёт и в этом варианте: пустой `Email` значит почту на этом домене.

| Поле | Что делает | Ограничение |
|---|---|---|
| `Email` | почта в контактах сайта | до 190 символов, можно плейсхолдер AIO |
| `Phone` | телефон | до 60 символов, можно плейсхолдер AIO |
| `Legal entity` | юрлицо (placeholder `Ex: Media Clicks Ltd`): строка ©, правовые страницы, разметка; имя бренда, логотип и тексты остаются (бренд — поле `Company name`) | до 120 символов, одной строкой, только текст |
| `Address` | адрес вместо придуманного на всём сайте | до 250 символов, только текст |
| `Footer text` | мелкий текст под строкой © на каждой странице: лицензия, регистрационный номер, VAT | до 1000 символов, только текст |

«Только текст» — без фигурных и угловых скобок `{ } < >`, то есть без плейсхолдеров и HTML. Лишнее форма не пропускает: при `Generate` под полем ошибка `<Поле> is too long (max N characters)` или `Plain text only: no { } < >`, и генерация не создаётся — исправьте поле и нажмите `Generate` ещё раз.

### Что вписать в бриф большого сайта — `Who the brand is` и `Wishes for the site`

**В режиме `Huge site by brief` сайт строится по брифу (блок `Brief`): обязательное поле `Who the brand is` описывает бренд, необязательное `Wishes for the site` — пожелания к содержанию и виду.** Подсказки в полях — `Trade, city, history, audience, tone` и `Content and look you want`. Описание бренда — до 1500 символов, пожелания — до 2000; длиннее форма не пропускает (`Who the brand is is too long (max 1500 characters)`). `Topic`, `Site structure` и `Size` в этом режиме скрыты.

Пока `Who the brand is` пустое, кнопка `Generate` неактивна. Бриф из одних пробелов форма пропускает, но генерация падает в `Issues` с текстом `Describe who the brand is for a huge site`.

## Какие параметры у внешнего генератора сайтов со своим `API Key`

**Внешний генератор сайтов (подключение со своим `API Key` в `Settings → Generators`) делает `Landing`; в форме три поля — `Target`, `Keywords` и `Language`, обязательны `Target` и `Language`.**

- `Target` — тематика сайта (подсказка `Select subject`): `Tech`, `Sport`, `Beauty`, `Finance`, `Dating`, `Games`, `Cars`, `Technologies`, `Fashion`, `Music`, `Activities`, `Entertainment`, `Health`, `Food`, `Education`, `Estate`, `Fitness`, `Commerce`.
- `Keywords` — ключевые слова для размещения на странице («Keywords for placement on the page.»), через запятую.
- `Language` — язык сайта, выбор из 46 языков.

Результат — архив сайта. Имени сайта этот генератор не возвращает, поэтому генерация без заданного имени так и остаётся `Landing generation`, а в карточке нет блока с именем сайта и списком страниц.

## Какие параметры у `Higgsfield` — модель, промпт, формат, качество

**`Higgsfield` делает `Creative` — статичные картинки по текстовому промпту; в форме `Model`, `Prompt`, `Aspect ratio` и `Quality`, обязательны `Model` и `Prompt`.**

- `Model` — `Soul 2` (по умолчанию), `Soul Standard`, `Qwen Image 3`, `Grok Imagine 2.0`, `Z-Image Turbo`.
- `Prompt` — описание креатива («Describe the creative»).
- `Aspect ratio` — `1:1 (Square)` (по умолчанию), `4:3`, `3:4 (Feed portrait)`, `2:3`, `3:2`, `16:9 (Landscape)`, `9:16 (Stories / Reels)`. В списке только соотношения, которые принимают все модели.
- `Quality` — `Standard` или `High` (по умолчанию); подсказка `Standard = 720p / 1K, High = 1080p / 2K depending on the model`.

Генерацию оплачивает аккаунт Higgsfield, чей ключ стоит в подключении: при нехватке кредитов генерация падает с `Higgsfield: insufficient credits`. Если имя не задано, генерация называется началом промпта — основа имени до 30 символов, затем `...` при обрезке; длинное слово может оборваться внутри.

Готовые картинки живут в генерации — в разделе `Creatives` они сами не появляются. Скачайте их кнопкой `Download` из карточки или уведомления.

## Статусы генерации — `Pending`, `Processing`, `Fine`, `Issues`, `Canceled`

**У генерации пять статусов; текст рядом со статусом (в карточке и в подсказке значка) называет стадию или причину.**

| Статус | Что значит | Что доступно |
|---|---|---|
| `Pending` | ждёт запуска или свободного места у провайдера (`Queued`, `… retrying…`) | `Cancel generation` |
| `Processing` | провайдер работает или AIO сохраняет результат (`Queued at provider…`, `Generating…`, `Generating site…`, `Generating site… 40%`, `Downloading site archive…`, `Saving images…`) | `Refresh status`, `Cancel generation` |
| `Fine` | готово, текст `Ready` | скачать; у лэндинга — `Embed preview` и `Create landing` |
| `Issues` | не получилось, причина в тексте статуса | `Retry generation` |
| `Canceled` | отменена вами (`Canceled by user`) или у провайдера | `Retry generation` |

На весь путь до результата у генерации 30 минут с момента запуска (у `AIO Site Generator` в режиме `Huge site by brief` — 60 минут): не уложилась — сама переходит в `Issues` с текстом, начинающимся на `Timed out` (секции «Генерация висит в `Pending`» и «Генерация висит в `Processing`»). Страница `Generations` у того, кто запустил генерацию, обновляется сама — по её уведомлению.

## Что показывает карточка генерации

**Клик по генерации открывает карточку `Generation`: превью, строки `What to generate`, `Provider` (текущий тип подключения), `Connection` (подключение), `Status` с текстом, `Cost`, параметры запуска и результат.** Промпт креатива и бриф большого сайта (`Who the brand is`, `Wishes for the site`) показаны отдельными полями только для чтения.

- **Креатив:** первая картинка крупно (`Open full size` — оригинал во встроенном просмотрщике); если картинок несколько, ниже сетка `Result`, каждую можно скачать; кнопка `Download` внизу скачивает все по очереди.
- **Лэндинг:** скриншот главной страницы (клик открывает `Embed preview`), блок результата с именем сайта, `Pages: N` и выданными значениями (среди них `Country`, если сайт сделан не для домашней страны языка), кнопка `Download archive` с размером архива — активна только в `Fine`.
- **Правка:** в свёрнутом блоке `Name, description & tags` — имя, описание, теги и кнопка `Save`. Тип, подключение и параметры после запуска не меняются: для других параметров запускается новая генерация. Без права правки карточка открывается только на чтение.

## Как посмотреть сгенерированный сайт — `Embed preview`

**`Embed preview` открывает сгенерированный сайт во встроенном просмотре с рамкой устройства; он есть у генерации `Landing` в статусе `Fine` — пунктом контекстного меню, кликом по скриншоту в карточке или по превью в строке.** Внизу окна — `Download archive`.

Первые секунды после `Fine` сайт публикуется для просмотра: вместо него надпись `The site is being published — this takes a few seconds.` и кнопка `Refresh`. Скриншот главной для превью снимается уже после публикации, поэтому сразу после `Fine` у генерации может не быть картинки (перечёркнутый глаз `No preview image`).

Просмотр показывает сайт как есть — без обработки, которую AIO делает с лэндами (SDK-макросы `{{aio:head}}` / `{{aio}}`, плейсхолдеры). Эта обработка происходит, когда из архива создаётся лэнд — секция «Как сделать лэнд из сгенерированного сайта — `Create landing`».

## Как сделать лэнд из сгенерированного сайта — `Create landing`

**Пункт `Create landing` в контекстном меню генерации `Landing` в статусе `Fine` открывает обычный диалог загрузки лэнда из ZIP с уже подставленным архивом и именем — дальше это стандартная загрузка ZIP-лэнда.** Имя лэнда берётся из имени сайта, если генератор его вернул, иначе из имени генерации; файл в загрузчике называется `<имя>.zip` — в нижнем регистре, пробелы и спецсимволы заменены дефисами. В пустые поля языков и стран лэнда подставляются язык сайта и его страна (страна — только если сайт сделан не для домашней страны языка); уже заполненное не перетирается. `Lander type`, шаблон и остальные поля выбираются как при любой загрузке — [how-to/landings.md](landings.md). Созданный лэнд — обычный лэнд в `Content → Landings`: с генерацией он не связан, генерация остаётся в своём списке.

Пункт виден с правом `Content: landings upload` (`content.landings.edit.upload`) — тем же, что нужно для загрузки ZIP в `Content → Landings`. Если архива у генерации нет, всплывает `This generation has no site archive yet.`; если архив не подтянулся в загрузчик — `Failed to load the file, drop it manually`: скачайте архив кнопкой `Download archive` в карточке генерации и перетащите его в окно загрузки сами.

Отдельного пункта генерации в создании лэнда (`Content → Landings → + Landing`) нет: сайт генерируется только на странице `Generations`.

## Как скачать результат генерации и сколько живут ссылки

**Архив сайта и картинки скачиваются из карточки генерации, из её уведомления в колокольчике и по ссылкам в Telegram / Slack; каждая выданная ссылка подписана и по умолчанию живёт 7 дней.** Файл качается кусками, прогресс виден прямо на кнопке.

Истёкшая ссылка даёт `The download link has expired — open the generation to get a new one`: откройте карточку генерации заново — она выдаёт свежие ссылки на те же файлы, результат хранится в AIO и вместе со ссылкой не пропадает. Любая другая ошибка — `Download failed`, тогда повтор имеет смысл. Как устроены подписанные ссылки на скачивание — [mechanics/notification-center.md](../mechanics/notification-center.md).

## Что делают `Retry generation`, `Refresh status` и `Cancel generation`

### `Retry generation` — повторить генерацию с теми же настройками

**`Retry generation` перезапускает генерацию в статусе `Issues` или `Canceled` на том же подключении и с теми же параметрами («Run the generation again with the same settings.»); как и два других пункта управления генерацией, он виден при уровне доступа `Edit` на неё.** Та же генерация возвращается в `Pending` и получает новое уведомление, а `Random` выбирается заново. Кнопка `Retry` есть и в уведомлении о сбое или отмене в колокольчике. Для других параметров повтор не подходит — запускается новая генерация. У остальных статусов пункта нет; запрос повтора в обход меню отказывает с `Only failed or canceled generations can be retried`.

### `Refresh status` — опросить провайдера прямо сейчас

**`Refresh status` запускает внеплановый опрос провайдера о текущем состоянии генерации в `Processing`; новый статус приходит уведомлением и обновляет строку («The provider will be polled; the new status will arrive as a notification.»).** Обычно пункт не нужен: AIO сам опрашивает провайдера и принимает его сигнал о готовности. У других статусов пункта нет.

### `Cancel generation` — отменить генерацию до результата

**`Cancel generation` доступен в `Pending` и `Processing`: из `Pending` генерация отменяется сразу, в `Processing` — только если провайдер ещё может отказаться от задачи.** У `Higgsfield` это возможно, пока задача стоит у него в очереди (`Queued at provider…`); после начала работы отмена отказывает с `Provider already started, can't cancel`. Сайт под заказ у `AIO Site Generator` (стадия `Generating site…`) не отменяется вовсе: отмена отказывает с `Provider already started, can't cancel`, и генерация доходит до результата или таймаута. Когда сайт уходит под заказ — секция «Какие режимы у AIO Site Generator». На стадии сохранения результата (`Downloading site archive…`, `Saving images…`) отмена отказывает с `Result is already being saved`. Завершённую генерацию отменить нельзя — `Generation is already finished`. Отменённая получает статус `Canceled` с текстом `Canceled by user`.

## Уведомление о генерации в колокольчике

**На каждый запуск генерации — `Generate` или `Retry` — приходит одно уведомление в колокольчик тому, кто нажал кнопку, и оно меняется на месте по ходу работы: стадия → готово, сбой или отмена.** Пока генерация идёт, в уведомлении крутится значок и текст стадии (например, `Starting generation…`, `Queued at provider…`, `Generating…`, `Generating site… 40%`, `No ready sites of this kind yet, retrying…`). Процент бывает только у сайта под заказ от `AIO Site Generator`: под текстом `Generating site… N%` рисуется полоска прогресса, процент обновляется при опросах генератора. У остальных провайдеров и стадий процентов нет; на странице `Generations` и в карточке тот же текст виден без полоски.

Готовый креатив показан превью картинок с кнопками `Download` и `Open` (открывает карточку генерации), готовый лэндинг — кнопкой `Download archive`; под ними строка `Download links are valid for N days.` (по умолчанию 7 дней). При сбое показана причина, при сбое и отмене — кнопка `Retry`.

Заголовки уведомления английские в любом языке интерфейса: `Generating landing: <имя>…`, `Generated landing is ready: <имя>`, `Generation of landing failed: <имя>`, `Generation of landing canceled: <имя>`; у креатива вместо `landing` — `creative`.

## Уведомление о генерации в Telegram и Slack — только готово и сбой

**В Telegram и Slack уходят только два исхода генерации — готово и сбой; стадии и отмена остаются в колокольчике.** Проксирование включается строкой `Generations` в матрице Preferences (вкладка `Channels`) или мастер-тумблером `All notifications` — [mechanics/notification-center.md](../mechanics/notification-center.md).

В сообщении — заголовок (по-английски, как в колокольчике), тенант, `Type: Landing` или `Type: Creative` и строка `Provider:` с именем подключения из `Settings → Generators`. У готовой генерации дальше ссылки `📦 Download site archive` или `🖼 Image 1`, `🖼 Image 2`… и `Links valid for N days.` (по умолчанию 7 дней), у сбоя — причина. Подписи (`Type:`, `Provider:`, ссылки, срок) приходят на языке получателя, текст причины — по-английски, как в статусе генерации.

К сообщению о готовом креативе прикладывается картинка — превью первого изображения (в Telegram — фото с текстом в подписи; если текст длиннее подписи Telegram, фото уходит отдельно, а текст следующим сообщением). Сообщение о готовом лэндинге уходит без картинки: скриншот сайта снимается уже после отправки.

## Сколько стоит генерация — колонка `Cost`

**Колонка `Cost` в таблице и строка `Cost` в карточке показывают стоимость генерации в долларах, которую AIO записывает при переходе в `Fine`; у неуспешных и отменённых генераций там ноль.**

- **`Higgsfield`** — оценка стоимости запроса по тарифу аккаунта, чей `API Key` стоит в подключении; кредиты списывает сам Higgsfield с этого аккаунта.
- **`AIO Site Generator`** — **пока бесплатен**: в форме запуска стоит пометка `Free for now — we're testing AIO Site Generator before the paid launch.` Сумма, которая стоит у его генераций в `Cost`, в счёт не выставляется.

## Кто видит генерации и подключения — владелец, `Access type`, шаринг

**Генерация и подключение из `Settings → Generators` — объекты с владельцем и уровнями доступа, как лэнды и кампании: владелец — тот, кто создал, новый объект получает `Access type` `Private`, остальным его открывают шаринг, команда или `Scopes`.** Общая модель уровней — [models/permissions-model.md](../models/permissions-model.md).

- **Генерации:** в контекстном меню — шаринг, `Change ownership` (и массово), `Add / Remove Tags` на выделении, архив и папки. Генерации коллег видны по тем же правилам, что и другие объекты: команда, шаринг, `Scopes`, роли Owner / Admin.
- **Подключения:** шаринг, `Change ownership`, архив и `Generator logs` — журнал изменений подключения в `Settings → Logs` (нужен `settings.logs.view`). Архивное подключение пропадает из формы генерации, а генерации, уже запущенные через него, доходят до конца.
- **Старые подключения:** заведённые до того, как у генераторов появились владельцы, принадлежат владельцу тенанта и открыты всем в тенанте на уровне `Use`.

Чтобы коллега генерировал через ваше подключение, достаточно уровня `Use` (шара с ролью `Use` или `Access type` `Use`): подключение появится у него в поле `Provider`, а ключ он не увидит. Уровень `View` открывает карточку подключения на чтение вместе с блоком `Settings`, включая `API Key`.

## Какие права нужны — `content.generations.*` и `settings.generators.*`

**Страницу и действия с генерациями открывают ключи `content.generations.*` в Position, подключения — ключи `settings.generators.*`; у Owner и Admin тенанта доступ полный.**

| Ключ | Подпись в матрице | Что открывает |
|---|---|---|
| `content.generations.view` | `Content: generations page` | страница, карточка на чтение |
| `content.generations.edit` | `Content: generations create & edit` | запуск, `Retry`, `Refresh status`, `Cancel`, имя / описание / теги |
| `content.generations.edit.archive`, `.edit.assign-folder` | `… generations archive`, `… assign folder` | архив, раскладка по папкам |
| `content.generations.share`, `.share.ownership` | `… generations share`, `… ownership change` | шаринг, смена владельца |
| `content.generations.folders.view`, `.folders.edit` | `… folders view`, `… folders edit` | папки генераций |
| `settings.generators.*` | `Settings: generators page` / `edit` / `archive` / `share` / `ownership change` | подключения |

`Scopes: generations` / `Editor Scopes: generations` и `Scopes: generators` / `Editor Scopes: generators` дают уровень `View` / `Own` на все генерации или подключения тенанта. `Create landing` требует ещё `Content: landings upload`.

## Генерация висит в `Pending` — почему и что делать

**Если генерация ждёт места у провайдера, она остаётся в `Pending`: AIO сам повторяет запуск, а текст статуса называет причину.** До первой попытки запуска у неё текст `Queued`.

- `No ready sites of this kind yet, retrying…` — у AIO Site Generator нет готового сайта этого вида и языка; так ждёт, например, генерация со значением `Closest ready site` в `If no exact match`. Можно отменить и запустить заново с другими `Site type` / `Language`, со значением `Generate to order if no exact match` или в режиме `Generate to order`.
- `Generator is busy, retrying…`, `Generator is unavailable, retrying…` — AIO Site Generator занят или временно недоступен.
- `Higgsfield concurrency limit reached, retrying…` — у аккаунта Higgsfield занят лимит одновременных задач; `Higgsfield model is temporarily unavailable, retrying…` и `Higgsfield is unavailable, retrying…` — модель или сервис временно недоступны.
- `Provider is unreachable, retrying…` — нет связи с провайдером.

Исчерпанный лимит сайтов AIO Site Generator в `Pending` не ждёт: генерация сразу уходит в `Issues` с текстом `… limit reached …` (секция «`Issues` из-за лимита сайтов»). Через 30 минут ожидания (у режима `Huge site by brief` — 60) генерация переходит в `Issues` с текстом `Timed out: provider had no capacity for this request` — дальше `Retry generation` или новая генерация с другими параметрами. Прервать ожидание раньше можно пунктом `Cancel generation`.

## Генерация висит в `Processing` — что проверить

**`Processing` значит, что провайдер принял задачу или AIO сохраняет результат; на весь путь есть 30 минут (у `AIO Site Generator` в режиме `Huge site by brief` — 60), после чего генерация сама уходит в `Issues` с текстом `Timed out waiting for provider`.** У креатива Higgsfield стадии идут `Queued at provider…` → `Generating…` → `Saving images…`. У готового сайта из пула и у внешнего генератора сайтов `Processing` — только скачивание архива (`Downloading site archive…`), обычно секунды.

Сайт под заказ у AIO Site Generator сначала генерируется — `Generating site…` или `Generating site… N%`, это минуты, — и только потом скачивается архив. Текст и процент обновляются при опросах генератора; `Refresh status` запускает внеплановый опрос, не дожидаясь планового. `Generator is unreachable, waiting…`, `Generator is busy, waiting…` и `Waiting for generator…` значат, что AIO ждёт генератор сам и опросит его снова. Отменить сайт под заказ нельзя — секция «`Cancel generation` — отменить генерацию до результата». Текст `Saving result failed, retrying…` значит, что AIO повторяет сохранение результата сам, до трёх попыток.

Если провайдер закончил уже после таймаута, AIO этот результат не подхватывает: генерация остаётся в `Issues`, а `Retry` запускает у провайдера новую задачу — у Higgsfield снова за кредиты; у AIO Site Generator это новый запрос к генератору, а сайт опоздавшей генерации в AIO не попадает.

## Генерация упала — статус `Issues` и что значит текст причины

### `Issues` из-за ключа, кредитов или модели провайдера

**Причина `Issues` — в тексте статуса (в карточке, в подсказке значка и в уведомлении); после исправления генерацию повторяют `Retry generation`.**

- `Check Higgsfield API key in generator settings` — ключа в подключении нет или Higgsfield его не принял: вставьте ключ из `console.higgsfield.ai → API keys` в `Settings → Generators`.
- `Higgsfield: insufficient credits` — на аккаунте Higgsfield не хватает кредитов.
- `Higgsfield model is not available for this account` — выбранная `Model` недоступна этому аккаунту.
- `There is no API Key in generator settings` — у подключения внешнего генератора сайтов пустой `API Key`. Неверный ключ у этого генератора отдельным текстом не проявляется: архив не скачивается, и генерация падает с `Cannot save generation result`.

### `Issues` из-за параметров или модерации провайдера

- `Invalid generation settings: …` — параметры запуска не приняты провайдером или проверкой AIO; после двоеточия — объяснение. У AIO Site Generator это замечание генератора вида `<поле>: <сообщение>` или проверки AIO (`unknown mode`, `language is required`, `unknown site type`); его тексты про язык и страну — в секции «`Issues` у AIO Site Generator — язык без готовых сайтов или недоступная страна в `Ready site`», про бриф — в секции «`Issues` у AIO Site Generator — бриф, одностраничный сайт, несобранный сайт под заказ».
- `Prompt is required`, `Target is required`, `Language is required` — не заполнено обязательное поле. В форме кнопка `Generate` неактивна, пока обязательное поле пустое, и генерация не создаётся.
- `Unknown Higgsfield model` — значения `Model` нет в списке моделей AIO.
- `Rejected by provider moderation (NSFW)` — Higgsfield отклонил запрос модерацией. `Retry` повторит тот же промпт — для другого промпта нужна новая генерация.
- `Higgsfield: …`, `Higgsfield error: …`, `Higgsfield returned no images`, `Request not found at Higgsfield` — сбой на стороне Higgsfield.

### `Issues` у AIO Site Generator — язык без готовых сайтов или недоступная страна в `Ready site`

**Часть причин `Issues` у AIO Site Generator связана с режимом и параметрами; `Retry` повторяет те же параметры, поэтому при ошибке языка или страны нужна новая генерация.**

- `<язык> has no ready sites — choose «Generate to order»` — в `Ready site` выбран язык с пометкой `(to order)`; значение `If no exact match` это не обходит. Запустите генерацию в `Generate to order` или `Huge site by brief` либо с языком без пометки.
- `Country <код> is not available for this language` и `Country <код> is not available for this language and site type` — в `Ready site` генератор не принимает выбранную страну для этого языка или вида сайта. Форма показывает только подходящие страны, так что текст встречается, когда список стран у генератора сменился после выбора, например при `Retry` старой генерации. В новой генерации выберите страну заново или оставьте `Country` пустым. (В `Generate to order` / `Huge site by brief` страна свободная — эта пара ошибок там не возникает, допустимость проверяет генератор при сборке.)

### `Issues` у AIO Site Generator — бриф, одностраничный сайт, несобранный сайт под заказ

**`Issues` у AIO Site Generator бывает и из-за брифа, одностраничного сайта, потерянной задачи генерации или несобранного сайта под заказ; `Retry` повторяет те же параметры, поэтому при ошибке брифа или параметров нужна новая генерация.**

- `Describe who the brand is for a huge site` — в `Huge site by brief` бриф `Who the brand is` состоит из одних пробелов.
- `One page with sections is made to order only — choose «Generate to order if no exact match»` — в `Ready site` выбрана `One page with sections (to order)` без сайта под заказ (секция «Одностраничный сайт — только под заказ»).
- `Generation not found at AIO Site Generator` — генератор не нашёл задачу этой генерации; `Retry` запускает новую.
- Фраза генератора с причиной или `The generator could not build this site — try again or choose another site type` — сайт под заказ не собрался: `Retry` или новая генерация с другим `Site type`.

### `Issues` из-за лимита сайтов — `… limit reached …` и `Site generation is blocked for this workspace`

**AIO Site Generator выдаёт сайты в пределах лимитов на рабочее пространство и на пользователя за текущий час или сутки; исчерпанный лимит сразу переводит генерацию в `Issues`, без ожидания в `Pending`.** Так же спрашивают: «лимит генераций», «закончились генерации», «не даёт сгенерировать сайт». Текст причины строится по образцу `Ready-site limit reached for your workspace this hour — try again in 40 min`:

- вместо `Ready-site limit` бывает `Generation-to-order limit` — лимит сайтов под заказ (`Generate to order` и `Ready site` со значением `Generate to order (paid)`) — или `Huge-site limit` для `Huge site by brief`;
- `for your workspace` — лимит рабочего пространства, `for you` — лимит владельца генерации;
- `this hour` — текущий час, `today` — текущие сутки;
- ожидание округляется вверх до целых минут: меньше 120 минут пишется в минутах (`40 min`), от 120 — в часах, тоже с округлением вверх (`5 h`).

Дождитесь срока из текста, затем `Retry generation` или новая генерация: повтор раньше упадёт с тем же текстом. Величины лимитов интерфейс не показывает. `Site generation is blocked for this workspace — contact support` значит, что выдача сайтов для рабочего пространства закрыта: напишите в чат — поможем.

### `Issues` из-за сохранения результата, обрыва связи или таймаута

- `Cannot save generation result` — AIO трижды не смог забрать файл результата (в том числе файл больше 300 МБ).
- `Provider did not confirm the start — check the provider before retrying` — запрос на запуск оборвался, и неизвестно, начал ли провайдер работу. Перед `Retry` проверьте в кабинете Higgsfield, не запустилась ли задача, иначе генерация может пройти дважды.
- `Timed out: provider had no capacity for this request` и `Timed out waiting for provider` — истекли 30 минут, у режима `Huge site by brief` — 60 (секции «Генерация висит в `Pending`» и «Генерация висит в `Processing`»).
- `AIO Site Generator is temporarily unavailable`, `AIO Site Generator returned an invalid response`, `Unexpected generator response (HTTP …)` — сбой на стороне AIO Site Generator: повторите позже или напишите в чат — поможем. Остальные его тексты — в секциях «`Issues` у AIO Site Generator» и «`Issues` из-за лимита сайтов».

## В форме нет провайдера, пункта `Create landing` или страницы `Generations`

**Пустое поле `Provider`, пропавший пункт меню или страница почти всегда объясняются статусом генерации, типом подключения или правами.**

- `No providers available. Add a connection in Settings → Generators.` — нет неархивного подключения нужного типа с уровнем не ниже `Use`: для `Landing` это `AIO Site Generator` или внешний генератор сайтов, для `Creative` — `Higgsfield`.
- В форме AIO Site Generator нет `Topic`, `Size`, `Quality` или брифа — поля показываются по выбранному `Mode` (секция «Какие режимы у AIO Site Generator»); `Company name`, `Logo` и `Brand colour` — в свёрнутом блоке `Your brand`, свои контакты — в блоке `Contacts`; поле `Country` — только у языков, для которых генератор знает другие страны.
- Нет `Embed preview` или `Create landing` — генерация не `Landing` или не в `Fine`; для `Create landing` ещё нужен `Content: landings upload`.
- Нет `Retry generation`, `Refresh status` или `Cancel generation` — статус не подходит (таблица в секции «Статусы генерации»), нет уровня `Edit` на генерацию или права `Content: generations create & edit`.
- Нет кнопки `+ Generation` — нет `Content: generations create & edit`; нет самой страницы — нет `Content: generations page` или тенант в MTK.
- Уведомление не пришло в Telegram или Slack — отмена туда не уходит вовсе, а готово и сбой приходят только тому, кто запускал генерацию, при включённой строке `Generations` или `All notifications` в Preferences.
