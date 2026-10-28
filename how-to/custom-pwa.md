---
id: custom-pwa
title: Custom PWA — свой лэнд из ZIP внутри готового PWA-шелла
description: "Как собрать PWA из своего лэнда: + PWA → Custom PWA, загрузка ZIP (берётся только index.html, manifest.json префиллит манифест), выбор шелла (Shell template), манифест (App name, Short name, иконка 512×512, AI-описание через Description with AI), Shell overrides (Install button id, длительности, цвет), кнопка установки install-btn / data-aio-pwa, редактор страницы Code / Visual / Split (Mobile) и кнопка QA, Replace by ZIP, конверсии через Lander Type."
doc_type: how-to
builds: [erp, mtk]
related: [how-to-pwa, mechanics-pwa, pwa-ai-texts, landings, landing-qa, landing-visual-editor]
language: ru
updated: 2026-10-28
---

# Custom PWA — свой лэнд из ZIP внутри готового PWA-шелла

**`Custom PWA` — третий способ собрать PWA в AIO: вы загружаете свой лэнд архивом ZIP, а AIO вставляет его `index.html` в готовый PWA-шелл, который берёт на себя установку, прелоадер, режим «внутри приложения», манифест и push.** Витрину рисует ваш HTML, вся PWA-механика — шелл. Два других способа — PWA Builder (витрина из готовых скинов) и PWA-макрос для лэнда с готовым серверным манифестом — описаны в [how-to/pwa.md](pwa.md); что такое PWA в AIO — [mechanics/pwa.md](../mechanics/pwa.md).

## Как создать Custom PWA — `+ PWA` → `Custom PWA`

1. Откройте раздел PWA: `Content → PWA` в ERP, вкладка `PWA` в MTK (с модулем `PWA + Push`).
2. Нажмите **`+ PWA`** и выберите **`Custom PWA`** («Upload your own landing ZIP — AIO wraps index.html into a PWA shell with install flow, manifest and push»). Пункт виден с правом `content.landings.edit.custom-pwa`.
3. В диалоге `Create Custom PWA` заполните общие поля, выберите шелл, заполните манифест и загрузите ZIP (секции ниже).
4. Сохраните. Готовая PWA появляется в списке раздела PWA и выбирается на шаге `Content` флоу как обычный лэнд.

Править созданную PWA — экшен **`Manage Custom PWA`** в её строке.

## Какие поля в диалоге Custom PWA — общие, шелл, манифест

- **Общие:** `PWA name`, `Lander type`, `Countries`, `Languages`, `Description`, `Tags`. Тип лэнда обязателен, в списке — только типы с включённым `Is PWA?`: от типа PWA берёт конверсии установки, открытия и подписки.
- **`Shell`** → **`Shell template`** — какой шелл оборачивает ваш лэнд («The shell handles install, preloader, inside-PWA mode and the SDK config. Your page is rendered inside it»), и `Language` шелла.
- **`Manifest`** — то, что видно при установке и на домашнем экране: `App name` (имя на экране установки), `Short name` (подпись под иконкой — держите короче 12 символов), `Description`, `App icon` (512×512, PNG, SVG или WebP), `Color`. Значение `Color` в манифест не попадает: фон экрана запуска и цвет панели браузера у установленного приложения AIO отдаёт белыми (`#ffffff`) — так же у PWA из `PWA Builder`. Акцент самого шелла задаёт отдельное поле `Accent color` (блок `Shell overrides`). При доступных AI-функциях под `Description` есть кнопка `Description with AI`: она запрашивает до трёх вариантов одной строкой с целевым ограничением 150 символов. Язык шелла передаётся как целевой язык генерации; в пожеланиях можно запросить другой. Выбранный вариант заменяет текст поля — [how-to/pwa-ai-texts.md](pwa-ai-texts.md).

## Shell overrides — что можно поменять в поведении шелла

Блок **`Shell overrides`** меняет параметры выбранного шелла; пустое поле оставляет дефолт шелла (он показан подсказкой в поле).

- **`Install button id`** — id кнопки установки в вашем `index.html`. Подсказка показывает значение выбранного шелла, а если оно не задано — `install-btn`.
- **`Accent color`** — акцентный цвет шелла.
- **`Pre-loading duration, ms`**, **`Loading duration, ms`** — длительность экранов прелоадера и загрузки.
- **`Installed timeout, ms`** и **`Interval before Open button, ms`** — тайминги шелла вокруг установки и появления кнопки `Open`.

Что происходит с кнопкой между `Install` и `Open`, одинаково для всех способов сборки PWA — [how-to/pwa.md](pwa.md).

## Как подготовить ZIP для Custom PWA — только `index.html`

**Из архива в PWA попадает только `index.html` вместе с его файлами (картинки, стили, скрипты уходят на CDN, как у обычного лэнда); подстраницы не создаются.** `sw.js`, `manifest.json`, `privacy.html` и другие HTML-файлы из архива страницами не становятся: service worker и манифест AIO подставляет свои, манифест собирается под каждый визит.

- **`manifest.json` из архива заполняет манифест** — `name`, `short_name`, `description`, `theme_color`, `background_color` и иконку из `icons[]`, у которой в `sizes` есть 512. Заполняются только пустые поля: то, что уже введено в форме, не перезаписывается. Цвета из архива попадают в поле `Color` формы, но не в отдаваемый манифест (там фон и панель — белые).
- **Лимиты и ошибки загрузки** — те же, что у обычного ZIP-лэнда ([how-to/landings.md](landings.md)): без `index.html` архив не обработается, размер `index.html` ограничен, статусы обработки те же.
- **SDK-макросы** `{{aio:head}}` / `{{aio}}` при загрузке расставляются по настройке типа лэнда, как у обычного лэнда; если в итоговой странице их нет (например, ваш `index.html` — фрагмент без `<head>` / `<body>`), AIO дописывает их в шелл.

**`Replace by ZIP`** в диалоге правки меняет архив; история архивов сохраняется, первый остаётся доступен. Без нового ZIP архив не перезаливается: `Save` сохраняет настройки; сохранение HTML из редактора описано ниже. `Replace by ZIP` закрывает диалог без вопроса о несохранённом и открывает окно загрузки: с новым архивом уходят текущие настройки из формы, а страницу целиком заменяет `index.html` нового архива — правки из редактора не переносятся.

## Как поправить страницу Custom PWA — редактор `Code` / `Visual` / `Split` и кнопка `QA`

**У созданной Custom PWA, чей ZIP уже разобран, в центре диалога `Manage Custom PWA` стоит редактор страницы — тот же, что в `Manage landing`, с режимами `Code`, `Visual` и `Split`; у новой PWA на этом месте зона загрузки ZIP.** Редактор правит HTML вашей страницы (`index.html` из архива) без шелла: прелоадер и экран установки в нём не видны, отдельного превью собранной PWA в диалоге нет.

- `Visual` сразу открывается в виде `Mobile`, переключатель `Mobile` / `Desktop` есть. Режима `Full preview` и поиска по страницам нет — страница одна.
- По умолчанию диалог открывается в `Visual` в ERP и в `Code` в MTK; `Visual` доступен в обеих сборках. В `Split` колонки настроек прячутся, редактор занимает всю ширину.
- Если AI-функции доступны, в шапке редактора есть кнопка `QA` ([how-to/landing-qa.md](landing-qa.md)) — её результат подставляется в редактор, а в панели визуального редактора есть `AI edit of the block` и `AI section`.

Правки страницы, в том числе из `QA`, сохраняет `Save` диалога. Для сохранения HTML дополнительно нужно право `content.landings.edit.manually`; с одним `content.landings.edit.custom-pwa` сохраняются настройки PWA, а сохранение HTML отклоняется. Как работает сам визуальный редактор — [how-to/landing-visual-editor.md](landing-visual-editor.md).

## Кнопка установки в своём HTML — `id="install-btn"` или `data-aio-pwa="install"`

**При загрузке ZIP AIO находит в вашем HTML кнопку установки по `id="install-btn"` или по атрибуту `data-aio-pwa="install"` и сам доводит её до формата, который понимает SDK:** добавляет недостающие атрибуты `data-install` / `data-open` (тексты кнопки из шелла) и `data-state`, а кнопке без `id` проставляет `id="install-btn"`. Уже проставленные вами атрибуты не меняются. Обрабатывается первая такая кнопка на странице.

Если id кнопки свой (задан в `Install button id`), атрибуты дописываются, только когда на кнопке есть и `data-aio-pwa="install"`.

На этой кнопке не ставьте `href="{{link}}"` — AIO заменяет такую ссылку на `href="#"`: клик по кнопке обрабатывает SDK. Кнопка — элемент `<a>` или `<button>`.

## Конверсии Custom PWA — на типе лэнда

**`Install`, `Open PWA` и `Push Subscribe` для Custom PWA спавнятся по настройке типа лэнда: поля `Conversion type on install` / `on open` / `on subscribe` в `Lander Type` с включённым `Is PWA?`.** В самом диалоге Custom PWA полей конверсий нет. Пустое поле типа — конверсия не спавнится. Подробно — [how-to/pwa.md](pwa.md), секция «Где задать конверсии установки, открытия и подписки PWA».
